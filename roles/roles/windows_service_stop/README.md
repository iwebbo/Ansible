# Ansible Role: windows_service_stop

Rôle Ansible pour Stop Service Windows

## General Information

**Author:** A&ECoding
**License:** MIT
**Minimum Ansible Version:** 2.9

**Supported Platforms:**
- Windows
  - Versions: 2012R2, 2016, 2019, 2022

## Variables

### main

```yaml
windows_services: []

```

## Main Tasks

- Arrêter le service Windows
- Vérifier l'état du service
- Afficher le statut des services

## Handlers

```yaml
- name: stop services
  win_service:
    name: '{{ item }}'
    state: stopped
  loop: '{{ windows_services | default([]) }}'
  when: windows_services is defined and windows_services | length > 0

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