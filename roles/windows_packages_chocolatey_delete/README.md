# Ansible Role: windows_packages_chocolatey_delete

Rôle Ansible to manage package from chocolatey

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
force_uninstall: false
remove_dependencies: true
remove_all_versions: false
chocolatey_timeout: 1800
chocolatey_missing_message: Chocolatey is not installed, Please use Chocolatey Role
  Install.

```

## Main Tasks

- Check installation of Chocolatey
- Exit if chocolatey isn't present
- Debug
- Check if already installed
- Show package status
- Uninstall package {{ package_name }} from win_chocolatey
- Uninstall package {{ package_name }} from win_command if win_chocolatey failed
- Check status package after uninstall
- Check/confirm uninstall
- Uninstall view
- Clean-up if needed (ex Python,Perl,Dotnet)

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