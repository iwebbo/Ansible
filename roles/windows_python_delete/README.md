# Ansible Role: windows_python_delete

Rôle Ansible to Delete python on Windows

## General Information

**Author:** A&ECoding
**License:** MIT
**Minimum Ansible Version:** 2.9

**Supported Platforms:**
- Windows
  - Versions: all

## Variables

### main

```yaml
target_version: 3.9.7

```

## Main Tasks

- Check if Python is installed and get its path
- Uninstall Python if installed and matches the specified version

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