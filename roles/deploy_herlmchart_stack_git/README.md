# Ansible Role: deploy_herlmchart_stack_git

Ansible role that automates the deployment of a full Kubernetes stack using Helm Charts. Handles the complete lifecycle: Git repository management, Helm configuration, deployment, ingress setup, and user token generation.


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
project_path: ''
repo_git: ''
values_file: ''
file_enabled: false
ingress_enabled: false
values_file_ingress: ''
ingress_controler: ''
values_file_users: ''
user_token: ''
generate_token: false

```

## Main Tasks

- Cleanup previous workspace
- Recreate workspace directory
- Show Repo GIT
- Clone repository fresh
- Debug git
- Show file *YAML
- Display YAML files in project path
- Add repo Helm Chart
- Update Helm charts
- Install & Update from Helm Chart (with custom values)
- Install & Update from Helm Chart (without custom values)
- Apply ingress if ingress_enabled
- Get ingress creation status
- Display ingress status
- Get port Exposed
- Display ingress status
- Apply User if needed
- Retrieve the secret token
- Display secret token
- Cleanup workspace after build (optional safety)

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
