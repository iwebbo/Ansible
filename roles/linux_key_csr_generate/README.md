# Ansible Role: linux_key_csr_generate

Rôle Ansible to generate a key & csr file

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
server_name: reverseproxy.local
csr_dir: /etc/ssl/certs
csr_key_dir: /etc/ssl/private
server_key_file: '{{ csr_key_dir }}/{{ server_name }}.key'
server_csr_file: '{{ csr_dir }}/{{ server_name }}.csr'
csr_country: FR
csr_state: Paris
csr_locality: Paris
csr_organization: A&ECoding
csr_organizational_unit: IT
csr_common_name: '{{ server_name }}'

```

## Main Tasks

- Create folder for private key
- Create folder for CSR
- Generate private key
- Generate  CSR & Subject Alternative Name (SAN)

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