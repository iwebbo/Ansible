# Ansible Role: windows_python_install

Rôle Ansible to Install & Download Python version 

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
python_version: 3.9.7
python_installer_url: https://www.python.org/ftp/python/{{ python_version }}/python-{{
  python_version }}-amd64.exe
python_install_dir: C:\Python{{ python_version | regex_replace('\.', '') }}

```

## Main Tasks

- Check if the specified Python version is already installed
- Version already installed output version
- Version Python need to be install
- Delete existing version before ton install it !
- Download Python installer
- Install Python
- Ensure Python path is in PATH environment variable

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
