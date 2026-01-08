# Ansible Role: ansible_inventory_role

Generate comprehensive Ansible inventory reports with all magic variables and facts

## General Information

**Author:** A&ECoding
**License:** MIT
**Minimum Ansible Version:** 2.10

**Supported Platforms:**
- Ubuntu
  - Versions: all
- Debian
  - Versions: all
- EL
  - Versions: all
- Fedora
  - Versions: all

## Default Variables

```yaml
inventory_report_dir: inventory_reports
inventory_gather_facts: true
inventory_include_environment: true

```

## Main Tasks

- Create report directory
- Gather all facts
- Collect magic variables
- Collect connection variables
- Collect system facts
- Collect network facts
- Collect date/time variables
- Collect environment variables
- Get all host variables
- Generate HTML inventory report
- Generate JSON inventory report
- Generate summary report
- Display summary

## Templates

- `ansible_inventory.html.j2`
- `ansible_inventory.json.j2`

## Role Structure

```
templates/
    ├── ansible_inventory.html.j2
    └── ansible_inventory.json.j2
meta/
    └── main.yml
tasks/
    └── main.yml
defaults/
    └── main.yml
