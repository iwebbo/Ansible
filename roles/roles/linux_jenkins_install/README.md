# Ansible Role: linux_jenkins_install

your role description

## General Information

**Author:** your name
**License:** license (GPL-2.0-or-later, MIT, etc)
**Minimum Ansible Version:** 2.1

## Variables

### main

```yaml
jenkins_java_package: default-jdk
jenkins_admin_user: ''
jenkins_http_port: 8080
jenkins_https_enabled: false
jenkins_https_port: 8443
jenkins_use_lts: true
jenkins_home: /var/lib/jenkins

```

## Main Tasks

- Collecter les informations du système
- Afficher les informations du système
- Installer Java sur Debian/Ubuntu
- Installer Java sur RedHat/CentOS/Fedora
- Installer Java sur SUSE
- Installation de Jenkins sur Debian/Ubuntu
- Installation de Jenkins sur RedHat/CentOS/Fedora
- Installation de Jenkins sur SUSE
- Configurer le port HTTP de Jenkins
- Configurer le port HTTP de Jenkins (RedHat)
- Démarrer et activer le service Jenkins
- Attendre que Jenkins soit disponible
- Récupérer le mot de passe initial d'administration
- Afficher le mot de passe initial d'administration

## Other Tasks

### debian.yml

```yaml
- name: "Installer les pr\xE9requis"
  become: true
  ansible.builtin.apt:
    name:
    - curl
    - gnupg
    - ca-certificates
    state: present
    update_cache: true
- name: "Cr\xE9er le r\xE9pertoire pour les cl\xE9s"
  ansible.builtin.file:
    path: /usr/share/keyrings
    state: directory
    mode: '0755'
- name: "Ajouter la cl\xE9 GPG de Jenkins"
  ansible.builtin.shell: "curl -fsSL https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key\
    \ | sudo tee \\\n  /usr/share/keyrings/jenkins-keyring.asc > /dev/null\n"
  args:
    creates: /usr/share/keyrings/jenkins-keyring.asc
- name: Ajouter le repository Jenkins
  ansible.builtin.shell: "echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc]\
    \ \\\n  https://pkg.jenkins.io/debian-stable binary/ | sudo tee \\\n  /etc/apt/sources.list.d/jenkins.list\
    \ > /dev/null\n"
  args:
    creates: /etc/apt/sources.list.d/jenkins.list
- name: "Mettre \xE0 jour les paquets et installer Jenkins"
  become: true
  apt:
    name: jenkins
    state: present
    update_cache: true
  notify: restart jenkins

```

### suse.yml

```yaml
- name: Ajouter le repository Jenkins
  ansible.builtin.zypper_repository:
    name: jenkins
    repo: https://pkg.jenkins.io/opensuse-stable/
    auto_import_keys: true
    state: present
- name: Installer Jenkins
  ansible.builtin.zypper:
    name: jenkins
    state: present
  notify: restart jenkins
- name: "Configurer le pare-feu (si firewalld est utilis\xE9)"
  ansible.builtin.firewalld:
    port: '{{ jenkins_http_port }}/tcp'
    permanent: true
    state: enabled
  when: ansible_service_mgr == 'systemd'
  ignore_errors: true
- name: Recharger le pare-feu
  ansible.builtin.service:
    name: firewalld
    state: reloaded
  when: ansible_service_mgr == 'systemd'
  ignore_errors: true

```

### redhat.yml

```yaml
- name: Ajouter le repository Jenkins
  ansible.builtin.get_url:
    url: https://pkg.jenkins.io/redhat-stable/jenkins.repo
    dest: /etc/yum.repos.d/jenkins.repo
    mode: '0644'
- name: "Importer la cl\xE9 GPG de Jenkins"
  ansible.builtin.rpm_key:
    key: https://pkg.jenkins.io/redhat-stable/jenkins.io-2023.key
    state: present
- name: Installer Jenkins
  ansible.builtin.yum:
    name: jenkins
    state: present
  notify: restart jenkins
- name: "Configurer le pare-feu (si firewalld est utilis\xE9)"
  ansible.builtin.firewalld:
    port: '{{ jenkins_http_port }}/tcp'
    permanent: true
    state: enabled
  when: ansible_service_mgr == 'systemd'
  ignore_errors: true
- name: Recharger le pare-feu
  ansible.builtin.service:
    name: firewalld
    state: reloaded
  when: ansible_service_mgr == 'systemd'
  ignore_errors: true

```

## Handlers

```yaml
- name: restart jenkins
  ansible.builtin.service:
    name: jenkins
    state: restarted

```

## Role Structure

```
vars/
    └── main.yml
meta/
    └── main.yml
tests/
    ├── inventory
    └── test.yml
tasks/
    ├── debian.yml
    ├── main.yml
    ├── redhat.yml
    └── suse.yml
handlers/
    └── main.yml
defaults/
    └── main.yml
README.md
```