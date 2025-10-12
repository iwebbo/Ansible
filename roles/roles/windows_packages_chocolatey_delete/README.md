# Ansible Role: windows_packages_chocolatey_delete

your role description

## General Information

**Author:** your name
**License:** license (GPL-2.0-or-later, MIT, etc)
**Minimum Ansible Version:** 2.1

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
- Get Chocolatey packages with PowerShell
- Parse Chocolatey packages JSON
- Debug chocolatey packages
- Check if already installed
- Update installation status if package is found
- Show package status
- Uninstall package {{ package_name }} from win_chocolatey
- Uninstall package {{ package_name }} from win_command if win_chocolatey failed
- Check status package after uninstall
- Parse post-uninstall packages
- Initialize still_installed variable
- Check if package is still installed
- Uninstall view
- Clean-up if needed (ex Python,Perl,Dotnet)

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