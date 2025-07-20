# Ansible Role: linux_package_manager

Universal Linux package manager role

## General Information

**Author:** DevOps Team
**License:** MIT
**Minimum Ansible Version:** 2.9

**Supported Platforms:**
- Ubuntu
  - Versions: 18.04, 20.04, 22.04
- Debian
  - Versions: 10, 11
- EL
  - Versions: 7, 8, 9

## Variables

### main

```yaml
package_name: ''
package_version: ''
package_state: present
apt_update_cache: true
apt_cache_valid_time: 3600
yum_update_cache: true
package_managers:
  Debian:
    command: apt
    version_separator: '='
  Ubuntu:
    command: apt
    version_separator: '='
  RedHat:
    command: yum
    version_separator: '-'
  CentOS:
    command: yum
    version_separator: '-'
  Rocky:
    command: dnf
    version_separator: '-'
  AlmaLinux:
    command: dnf
    version_separator: '-'

```

## Main Tasks

- Detect Linux distribution
- Install package on Debian/Ubuntu systems
- Install package on RedHat/CentOS systems
- Install package on RHEL/CentOS 8+ systems (dnf)
- Verify package installation
- Display installation result

## Handlers

```yaml
- name: restart service
  service:
    name: '{{ service_name }}'
    state: restarted
  when: service_name is defined

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
    └── main.yml
handlers/
    └── main.yml
defaults/
    └── main.yml
README.md
```