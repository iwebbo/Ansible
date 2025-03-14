# Ansible Role: windows_dotnet_aspcore_install

Rôle Ansible to install .NET Runtimes on windows

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
dotnet_version: '{{ dotnet_version }}'
runtime_version: '{{ runtime_version }}'
aspnetcore_versions:
  '6.0':
    runtimes:
      6.0.27:
        download_pr_id: exemple-pr-id-6.0.27
        download_hash: exemple-hash-6.0.27
      6.0.26:
        download_pr_id: exemple-pr-id-6.0.26
        download_hash: exemple-hash-6.0.26
  '7.0':
    runtimes:
      7.0.17:
        download_pr_id: exemple-pr-id-7.0.17
        download_hash: exemple-hash-7.0.17
      7.0.16:
        download_pr_id: exemple-pr-id-7.0.16
        download_hash: exemple-hash-7.0.16
  '8.0':
    runtimes:
      8.0.2:
        download_pr_id: exemple-pr-id-8.0.2
        download_hash: exemple-hash-8.0.2
      8.0.1:
        download_pr_id: cede7e69-dbd4-4908-9bfb-12fa4660e2b9
        download_hash: d9ed17179d0275abee5afd29d5460b48
      8.0.3:
        download_pr_id: exemple-pr-id-8.0.3
        download_hash: exemple-hash-8.0.3
      8.0.4:
        download_pr_id: exemple-pr-id-8.0.4
        download_hash: exemple-hash-8.0.4
      8.0.5:
        download_pr_id: exemple-pr-id-8.0.5
        download_hash: exemple-hash-8.0.5
      8.0.6:
        download_pr_id: exemple-pr-id-8.0.6
        download_hash: exemple-hash-8.0.6
      8.0.7:
        download_pr_id: exemple-pr-id-8.0.7
        download_hash: exemple-hash-8.0.7
      8.0.8:
        download_pr_id: exemple-pr-id-8.0.8
        download_hash: exemple-hash-8.0.8
      8.0.9:
        download_pr_id: exemple-pr-id-8.0.9
        download_hash: exemple-hash-8.0.9
      8.0.10:
        download_pr_id: exemple-pr-id-8.0.10
        download_hash: exemple-hash-8.0.10
      8.0.11:
        download_pr_id: exemple-pr-id-8.0.11
        download_hash: exemple-hash-8.0.11
      8.0.12:
        download_pr_id: exemple-pr-id-8.0.12
        download_hash: exemple-hash-8.0.12
      8.0.13:
        download_pr_id: exemple-pr-id-8.0.13
        download_hash: exemple-hash-8.0.13
      8.0.14:
        download_pr_id: exemple-pr-id-8.0.14
        download_hash: exemple-hash-8.0.14
      8.0.15:
        download_pr_id: exemple-pr-id-8.0.15
        download_hash: exemple-hash-8.0.15
      8.0.16:
        download_pr_id: exemple-pr-id-8.0.16
        download_hash: exemple-hash-8.0.16
      8.0.17:
        download_pr_id: exemple-pr-id-8.0.17
        download_hash: exemple-hash-8.0.17
      8.0.18:
        download_pr_id: exemple-pr-id-8.0.18
        download_hash: exemple-hash-8.0.18
      8.0.19:
        download_pr_id: exemple-pr-id-8.0.19
        download_hash: exemple-hash-8.0.19
      8.0.20:
        download_pr_id: exemple-pr-id-8.0.20
        download_hash: exemple-hash-8.0.20
  '9.0':
    runtimes:
      9.0.2:
        download_pr_id: 50496354-3b3f-4080-867c-1e8f98c5c94a
        download_hash: d03d3b4da0e7c5de8d1f42da9a30c6d0
      9.0.1:
        download_pr_id: exemple-pr-id-9.0.1
        download_hash: exemple-hash-9.0.1

```

## Main Tasks

- Check Architecture
- Define Architecture for Download
- Check version of Runtime
- Information of download
- Build link ASP.NET Core Runtime
- Create temp folder
- Download ASP.NET Core Runtime
- Install ASP.NET Core Runtime
- Check runtime ASP.NET Core
- Informations of installations
- Clean-up folder installation

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
