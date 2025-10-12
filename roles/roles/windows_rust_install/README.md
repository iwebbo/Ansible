# Ansible Role: windows_rust_install

Install RUST on Windows

## General Information

**Author:** A&ECoding
**License:** MIT
**Minimum Ansible Version:** 2.1

**Supported Platforms:**
- Windows
  - Versions: all

## Variables

### main

```yaml
rust_install_path: E:\Tools\build\Rust
rust_version: latest
rust_components:
- rustfmt
- clippy
- rust-src
- cargo
rust_default_host_triple: ''
rust_default_toolchain: stable
rust_profile: default
rust_update_path: true
rust_temp_download_dir: '{{ ansible_env.TEMP }}\rust_installer'
rust_install_mode: quiet

```

## Main Tasks

- Create temp folder for installation
- Detect arch system
- Set Architecture
- Check if RUST is already installed
- Set fact of Architecture
- Download installation of Rust (rustup-init)
- Create folder of installation Path folder variable
- Installation of RUST with Path folder changed
- Installation of components
- If needed change value of Path
- Clean-up temp folder installation

## Handlers

```yaml
- name: Actualiser les variables d'environnement
  win_shell: "$process = Get-Process -Id $PID\nforeach ($session in [System.Diagnostics.Process]::GetCurrentProcess().ProcessorAffinity)\
    \ {\n    if ($session.ProcessId -eq $process.Id) {\n        $session.RefreshEnvironment()\n\
    \    }\n}\n"
  listen: refresh_env

```

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