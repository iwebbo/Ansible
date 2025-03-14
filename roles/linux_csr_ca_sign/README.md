# Ansible Role: linux_csr_ca_sign

Rôle Ansible to sign csr to CA Authority (used with Autoritecertification)

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
server_name: server.name
ca_dir: /etc/ssl/myCA
ca_key_file: '{{ ca_dir }}/myCA.key'
ca_cert_file: '{{ ca_dir }}/myCA.crt'
server_cert_dir: /etc/ssl/certs
server_csr_file: '{{ server_cert_dir }}/{{ server_name }}.csr'
server_cert_file: '{{ server_cert_dir }}/{{ server_name }}.crt'
cert_validity_days: 365

```

## Main Tasks

- Chec if CSR file exist
- Failed if the CSR file doesn't exist
- Sign CSR with CA local
- Showing certificate generated

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
