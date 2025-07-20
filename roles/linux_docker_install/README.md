# Ansible Role: linux_docker_install

Rôle Ansible for Deocker installation on Linux

## General Information

**Author:** A&ECoding
**License:** MIT
**Minimum Ansible Version:** 2.9

**Supported Platforms:**
- Debian, Redhat, Suse
  - Versions: all

## Default Variables

```yaml
supported_architectures:
- amd64
- arm64
- armhf

```

## Main Tasks

- Get all information systems from Ansible
- Display all informations of Systems Arch/OS
- Installation of Docker on Debian/Ubuntu
- Installation of Docker on RedHat/CentOS/Fedora
- Installation of Docker on SUSE
- Start & Init service docker
- Add user if needed for Docker

## Other Tasks

### setup-debian.yml

```yaml
- name: Installation of dependencies
  ansible.builtin.apt:
    name:
    - ca-certificates
    - curl
    - gnupg
    - apt-transport-https
    state: present
    update_cache: true
- name: Installation of Docker via script officiel
  ansible.builtin.shell: 'curl -fsSL https://get.docker.com -o get-docker.sh

    sh get-docker.sh

    '
  args:
    creates: /usr/bin/docker
  notify: restart docker

```

### setup-redhat.yml

```yaml
- name: Installation of prerequisite
  ansible.builtin.yum:
    name:
    - yum-utils
    - device-mapper-persistent-data
    - lvm2
    state: present
- name: Installation of Docker via script officiel
  ansible.builtin.shell: 'curl -fsSL https://get.docker.com -o get-docker.sh

    sh get-docker.sh

    '
  args:
    creates: /usr/bin/docker
  notify: restart docker

```

### setup-suse.yml

```yaml
- name: Installation of prerequisites
  ansible.builtin.zypper:
    name:
    - ca-certificates
    - curl
    state: present
- name: Installation of Docker via script officiel
  ansible.builtin.shell: 'curl -fsSL https://get.docker.com -o get-docker.sh

    sh get-docker.sh

    '
  args:
    creates: /usr/bin/docker
  notify: restart docker

```

## Handlers

```yaml
- name: restart docker
  ansible.builtin.service:
    name: docker
    state: '{{ docker_restart_handler_state }}'

```

## Templates

- `daemon.json.j2`

## Role Structure

```
vars/
    └── main.yml
templates/
    └── daemon.json.j2
meta/
    └── main.yml
tests/
    ├── inventory
    └── test.yml
tasks/
    ├── main.yml
    ├── setup-debian.yml
    ├── setup-redhat.yml
    └── setup-suse.yml
handlers/
    └── main.yml
defaults/
    └── main.yml
README.md
```