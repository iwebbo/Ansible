# Ansible Role: windows_python_chocolatey_install

Rôle Ansible to install Python X from Chocolatey

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
package_name: ''
package_version: ''
package_state: present
force_install: false
chocolatey_source: https://chocolatey.org/api/v2/
chocolatey_timeout: 1800
python_install_dir: ''
python_install_pip: true
python_default_packages: []
chocolatey_missing_message: Chocolatey is not installed, Please use Chocolatey Role
  Install'.

```

## Main Tasks

- Check installation of Chocolatey
- Exit if chocolatey isn't present
- Check package {{ package_name }} if already installed
- Check package installation (already present)
- Status of Package
- Configuration dedicated for Python
- Installation/Update of {{ package_name }} from Chocolatey
- post-installation of Python
- Status of the installation

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