# Ansible Role: linux_kubernetes_install

Install and configure Kubernetes cluster with containerd runtime

## General Information

**Author:** AE&Coding
**License:** MIT
**Minimum Ansible Version:** 2.9

**Supported Platforms:**
- Debian
  - Versions: bullseye, bookworm
- Ubuntu
  - Versions: focal, jammy, noble

## Default Variables

```yaml
k8s_version: '1.35'
k8s_pod_network_cidr: 10.244.0.0/16
k8s_cni_url: https://raw.githubusercontent.com/flannel-io/flannel/master/Documentation/kube-flannel.yml

```

## Main Tasks

- Include prerequisites tasks
- Include containerd tasks
- Include kubernetes tools tasks

## Other Tasks

### prerequisites.yml

```yaml
- name: Disable swap
  command: swapoff -a
  changed_when: false
- name: Remove swap from fstab
  replace:
    path: /etc/fstab
    regexp: ^([^#].*\s+swap\s+.*)$
    replace: '#\1'
- name: Create k8s modules config
  template:
    src: k8s-modules.conf.j2
    dest: /etc/modules-load.d/k8s.conf
    mode: '0644'
- name: Load br_netfilter module
  modprobe:
    name: br_netfilter
    state: present
- name: Create k8s sysctl config
  template:
    src: k8s-sysctl.conf.j2
    dest: /etc/sysctl.d/k8s.conf
    mode: '0644'
- name: Apply sysctl settings
  command: sysctl --system
  changed_when: false

```

### join-worker.yml

```yaml
- name: Check if node is already joined
  stat:
    path: /etc/kubernetes/kubelet.conf
  register: kubelet_conf
- name: Join worker to cluster
  command: '{{ hostvars[groups[''k8s_master''][0]][''k8s_join_command''] }}'
  when: not kubelet_conf.stat.exists
- name: Wait for node to be ready
  command: kubectl get nodes {{ inventory_hostname }}
  register: node_status
  until: '''Ready'' in node_status.stdout'
  retries: 10
  delay: 10
  delegate_to: '{{ groups[''k8s_master''][0] }}'
  environment:
    KUBECONFIG: /etc/kubernetes/admin.conf
  when: not kubelet_conf.stat.exists
  changed_when: false

```

### kubernetes-tools.yml

```yaml
- name: Add Kubernetes GPG key
  apt_key:
    url: https://pkgs.k8s.io/core:/stable:/v{{ k8s_version }}/deb/Release.key
    keyring: /etc/apt/keyrings/kubernetes-apt-keyring.gpg
    state: present
- name: Add Kubernetes repository
  apt_repository:
    repo: deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v{{
      k8s_version }}/deb/ /
    filename: kubernetes
    state: present
- name: Install Kubernetes tools
  apt:
    name:
    - kubelet
    - kubeadm
    - kubectl
    state: present
    update_cache: true
- name: Hold Kubernetes packages
  dpkg_selections:
    name: '{{ item }}'
    selection: hold
  loop:
  - kubelet
  - kubeadm
  - kubectl
- name: Enable and start kubelet
  systemd:
    name: kubelet
    enabled: true
    state: started

```

### containerd.yml

```yaml
- name: Install prerequisites for containerd
  apt:
    name:
    - ca-certificates
    - curl
    - gnupg
    - lsb-release
    state: present
    update_cache: true
- name: Create keyrings directory
  file:
    path: /etc/apt/keyrings
    state: directory
    mode: '0755'
- name: Add Docker GPG key
  apt_key:
    url: https://download.docker.com/linux/debian/gpg
    keyring: /etc/apt/keyrings/docker.gpg
    state: present
- name: Add Docker repository
  apt_repository:
    repo: deb [arch={{ ansible_architecture }} signed-by=/etc/apt/keyrings/docker.gpg]
      https://download.docker.com/linux/debian {{ ansible_distribution_release }}
      stable
    filename: docker
    state: present
- name: Install containerd
  apt:
    name: containerd
    state: present
    update_cache: true
- name: Create containerd config directory
  file:
    path: /etc/containerd
    state: directory
    mode: '0755'
- name: Generate default containerd config
  shell: containerd config default
  register: containerd_config
  changed_when: false
- name: Write containerd config
  copy:
    content: '{{ containerd_config.stdout }}'
    dest: /etc/containerd/config.toml
    mode: '0644'
- name: Enable SystemdCgroup in containerd
  replace:
    path: /etc/containerd/config.toml
    regexp: SystemdCgroup = false
    replace: SystemdCgroup = true
- name: Restart containerd
  systemd:
    name: containerd
    state: restarted
    enabled: true

```

### init-master.yml

```yaml
- name: Check if cluster is already initialized
  stat:
    path: /etc/kubernetes/admin.conf
  register: k8s_admin_conf
- name: Initialize Kubernetes cluster
  command: 'kubeadm init --apiserver-advertise-address={{ hostvars[groups[''k8s_master''][0]][''ansible_default_ipv4''][''address'']
    }} --pod-network-cidr={{ k8s_pod_network_cidr }}

    '
  when: not k8s_admin_conf.stat.exists
  register: kubeadm_init
- name: Create .kube directory for root
  file:
    path: /root/.kube
    state: directory
    mode: '0755'
- name: Create .kube directory for user
  file:
    path: '{{ ansible_env.HOME }}/.kube'
    state: directory
    mode: '0755'
- name: Copy admin.conf to root kube config
  copy:
    src: /etc/kubernetes/admin.conf
    dest: /root/.kube/config
    remote_src: true
    mode: '0600'
- name: Copy admin.conf to user kube config
  copy:
    src: /etc/kubernetes/admin.conf
    dest: '{{ ansible_env.HOME }}/.kube/config'
    remote_src: true
    owner: '{{ ansible_user_id }}'
    group: '{{ ansible_user_gid }}'
    mode: '0600'
- name: Wait for all control plane pods to be ready
  command: kubectl get pods -n kube-system
  register: kube_pods
  until: kube_pods.rc == 0
  retries: 10
  delay: 10
  changed_when: false
  environment:
    KUBECONFIG: /etc/kubernetes/admin.conf
- name: Install Flannel CNI
  command: kubectl apply -f {{ k8s_cni_url }}
  environment:
    KUBECONFIG: /etc/kubernetes/admin.conf
  when: not k8s_admin_conf.stat.exists
- name: Wait for Flannel to be ready
  command: kubectl wait --for=condition=ready pod -l app=flannel -n kube-flannel --timeout=300s
  environment:
    KUBECONFIG: /etc/kubernetes/admin.conf
  when: not k8s_admin_conf.stat.exists
  ignore_errors: true
- name: Generate join command
  command: kubeadm token create --print-join-command
  register: k8s_join_command_output
  changed_when: false
- name: Set join command as fact
  set_fact:
    k8s_join_command: '{{ k8s_join_command_output.stdout }}'

```

## Handlers

```yaml
- name: Restart containerd
  systemd:
    name: containerd
    state: restarted

```

## Templates

- `k8s-modules.conf.j2`
- `k8s-sysctl.conf.j2`

## Role Structure

```
templates/
    ├── k8s-modules.conf.j2
    └── k8s-sysctl.conf.j2
meta/
    └── main.yml
tasks/
    ├── containerd.yml
    ├── init-master.yml
    ├── join-worker.yml
    ├── kubernetes-tools.yml
    ├── main.yml
    └── prerequisites.yml
handlers/
    └── main.yml
defaults/
    └── main.yml
```d