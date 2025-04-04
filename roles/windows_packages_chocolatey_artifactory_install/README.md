# Ansible Role: windows_packages_chocolatey_artifactory_install

Rôle Ansible to install X Package/Version from Chocolatey/artifactory

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
- Initialize variables
- Configure Chocolatey Artifactory source
- Verify Artifactory source configuration
- Check if package is installed
- Parse package info
- Debug package check
- Show package status
- Install {{ package_name }} using direct choco command
- Verify installation status
- Debug installation result
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