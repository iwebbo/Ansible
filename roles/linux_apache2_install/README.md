# Ansible Role: linux_apache2_install

Rôle Ansible to install Apache2 on linux

## General Information

**Author:** A&ECoding
**License:** MIT
**Minimum Ansible Version:** 2.9

**Supported Platforms:**
- Windows
  - Versions: all

## Main Tasks

- Packages Updating
- Install Apache2
- Starting apache after installation

## Role Structure

```
defaults/
    └── main.yml
handlers/
    └── main.yml
meta/
    └── main.yml
tasks/
    └── main.yml
tests/
    ├── inventory
    └── test.yml
vars/
    └── main.yml
README.md
```
