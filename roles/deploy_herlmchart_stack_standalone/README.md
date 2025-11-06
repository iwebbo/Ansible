# Ansible Role: deploy_herlmchart_stack_standalone

Minimal Ansible role to manage Helm repositories and deploy or upgrade Kubernetes Helm Charts. Ensures repository synchronization and namespace creation for consistent and automated delivery.


## General Information

**Author:** A&ECoding
**License:** MIT
**Minimum Ansible Version:** 2.14

**Supported Platforms:**
- Ubuntu
  - Versions: jammy, focal
- Debian
  - Versions: bullseye, bookworm
- CentOS
  - Versions: 8, 9

## Default Variables

```yaml
helm_repo_name: ''
helm_repo_url: ''
chart_name: ''
namespace: ''
release_name: ''

```

## Main Tasks

- Add repo if missing Helm charts
- Update list repo Helm charts
- Install or Update Helm charts

## Role Structure

```
vars/
    └── main.yml
tasks/
    └── main.yml
defaults/
    └── main.yml
tests/
    ├── inventory
    └── test.yml
meta/
    └── main.yml
handlers/
    └── main.yml
README.md
