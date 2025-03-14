# Ansible Role: windows_msxml_delete

Rôle Ansible to Delete MSXML on Windows

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
msxml_version: '3.0'
msxml_remove: false
cleanup_temp_files: true

```

## Main Tasks

- Create PowerShell script for MSXML check
- Check if specified MSXML version exists
- Set MSXML facts from check result
- Display MSXML status
- Create PowerShell script for MSXML removal
- Unregister and rename MSXML DLLs
- Create PowerShell script for registry removal
- Remove registry entries if they exist
- Display removal results
- Remove temporary PowerShell scripts

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
