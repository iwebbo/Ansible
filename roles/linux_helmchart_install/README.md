# Ansible Role: linux_helmchart_install

Install Helm package manager for Kubernetes

## General Information

**Author:** A&ECoding
**License:** MIT
**Minimum Ansible Version:** 2.9

**Supported Platforms:**
- Debian
  - Versions: bullseye, bookworm
- Ubuntu
  - Versions: focal, jammy, noble

## Main Tasks

- Install prerequisites
- Download and install Helm GPG key
- Add Helm repository
- Update apt cache
- Install Helm
- Verify Helm installation
- Display Helm version

## Role Structure

```
meta/
    └── main.yml
tasks/
    └── main.yml
```