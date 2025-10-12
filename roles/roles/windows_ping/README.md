# Ansible Role: windows_ping

Rôle pour tester la connectivité des serveurs Windows

## General Information

**Author:** A&E Coding
**License:** MIT
**Minimum Ansible Version:** 2.9

**Supported Platforms:**
- Windows
  - Versions: 2016, 2019, 2022

## Main Tasks

- Test de connectivité Windows avec win_ping
- Afficher le résultat du ping Windows
- Collecter des informations système Windows
- Afficher les informations système Windows

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