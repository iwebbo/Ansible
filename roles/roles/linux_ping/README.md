# Ansible Role: linux_ping

Rôle pour tester la connectivité des serveurs Linux

## General Information

**Author:** A&E Coding
**License:** MIT
**Minimum Ansible Version:** 2.9

**Supported Platforms:**
- Ubuntu
  - Versions: 18.04, 20.04, 22.04
- EL
  - Versions: 7, 8, 9

## Main Tasks

- Test de connectivité Linux avec ping
- Afficher le résultat du ping Linux
- Collecter des informations système de base
- Afficher les informations système Linux

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