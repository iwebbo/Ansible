# Ansible Role: windows_packages_chocolatey_install

Rôle Ansible to install X Package/Version from Chocolatey

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
chocolatey_missing_message: Chocolatey is not installed, Please use Chocolatey Role
  Install'.
artifactory_source_name: artifactory
artifactory_url: ''
artifactory_username: ''
artifactory_password: ''

```

## Main Tasks

- Check installation of Chocolatey
- Exit if chocolatey isn't present
- Configure Chocolatey Artifactory source
- Get installed packages information
- Debug packages
- Check if package is installed
- Parse package info
- Show package status
- Installation/Update of {{ package_name }} from Chocolatey Artifactory
- Status of the installation

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