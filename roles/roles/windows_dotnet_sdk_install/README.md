# Ansible Role: windows_dotnet_sdk_install

Rôle Ansible to install .NET SDK on Windows

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
sdk_version: '{{ sdk_version }}'
dotnet_sdk_versions:
  '6.0':
    latest_release: 6.0.418
    download_pr_id: 5f3e34ae-2449-4c76-b00c-33c4fee4423c
    download_hash: 20f17fcb662a32492c447fd5bd83ab661f8f785e4be85b9ba57cf61a225a33f8
  '7.0':
    latest_release: 7.0.408
    download_pr_id: 50a3ccdf-bf15-4455-a39a-1cdf6c0d1fcc
    download_hash: 4d67fbb55c0159a4b86da4f974cd96c5dfde4b69dfcb6d56ccab52abb7c95304
  '8.0':
    latest_release: 8.0.309
    download_pr_id: 7b05a559-89e3-405d-8828-069fcfda286e
    download_hash: 8b90f27ea6dd948f8b87e0fbab779b01
  '9.0':
    latest_release: 9.0.100-preview.3.24104.13
    download_pr_id: ad87fbd9-5b14-4bee-a94c-5005c4e9cf14
    download_hash: d8a36d2fe490c9d5e50f81e13c9a4ffe37a41139f9cbb48dbd44b7e4b9a4aa10

```

## Main Tasks

- Check if .NET SDK are compatabile with var file (if not compatible, add the version on var file)
- Check architecture of Windows
- Check architecture from architecture_result
- Information from var file to install .NET {{ sdk_version }} SDK
- Build link .NET SDK
- Install folder
- Create temp folder
- Download .NET SDK
- Installer .NET SDK
- Check installation of .NET SDK
- Show the version has been installed
- Clean-up installation folder

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