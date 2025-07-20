# Ansible Role: linux_autoritecertification_install

Rôle Ansible to install CA Certificate

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
ca_dir: /etc/ssl/myCA
ca_key_file: '{{ ca_dir }}/myCA.key'
ca_cert_file: '{{ ca_dir }}/myCA.crt'
ca_validity_days: 3650
ca_country: FR
ca_state: Paris
ca_locality: Paris
ca_organization: A&ECoding
ca_organizational_unit: IT
ca_common_name: A&ECoding Internal CA

```

## Main Tasks

- Create folder CA
- Generate a private key CA
- Generate certificat of CA

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