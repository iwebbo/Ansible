# Ansible Role: minio_client

Ansible role to install and configure the MinIO client (mc) on multiple Linux distributions.

## Description

This role allows you to install the MinIO client from https://www.min.io/download and configure it with aliases to connect to your MinIO servers. It supports multiple Linux distributions and uses a tag system for flexible installation.

## Supported Distributions

- Debian/Ubuntu (uses apt)
- RedHat/CentOS/Rocky/AlmaLinux (uses yum/dnf)
- Arch Linux (uses pacman)

## Requirements

- Ansible >= 2.10
- Root or sudo access on target machines
- Internet connection to download the MinIO binary

## Role Variables

### Main Variables

| Variable | Default | Description |
|----------|---------|-------------|
| `minio_client_download_url` | `https://dl.min.io/client/mc/release/linux-amd64/mc` | Client download URL |
| `minio_client_install_dir` | `/usr/local/bin` | Installation directory |
| `minio_client_binary_name` | `mc` | Binary name |
| `minio_client_config_dir` | `/root/.mc` | Configuration directory |
| `minio_client_force_reinstall` | `false` | Force reinstallation |

### Token and Alias Configuration

| Variable | Default | Description |
|----------|---------|-------------|
| `minio_access_key` | `""` | MinIO access key |
| `minio_secret_key` | `""` | MinIO secret key |

### Permissions

| Variable | Default | Description |
|----------|---------|-------------|
| `minio_client_binary_mode` | `0755` | Binary permissions |
| `minio_client_config_mode` | `0600` | Config file permissions |
| `minio_client_owner` | `root` | File owner |
| `minio_client_group` | `root` | File group |


## Available Tags

| Tag | Description | Related Files |
|-----|-------------|---------------|
| `install` | MinIO client installation | `install.yml` |
| `configure` | Alias configuration | `configure.yml` |
| `minio_client` | All role tasks | `main.yml` |
| `dependencies` | System dependencies installation | `install.yml` |
| `download` | Binary download | `install.yml` |
| `verify` | Installation verification | `install.yml`, `configure.yml` |
| `aliases` | Alias configuration only | `configure.yml` |
| `test` | Server connection test | `configure.yml` |

## Playbook Examples

### Example 1: Basic Installation

```yaml
---
- name: Install MinIO Client
  hosts: all
  become: yes
  
  roles:
    - role: minio_client
```

**Execution:**
```bash
ansible-playbook -i inventory.ini playbook.yml
```

### Example 2: Installation with Simple Configuration

```yaml
---
- name: Install and Configure MinIO Client
  hosts: all
  become: yes
  
  vars:
    minio_access_key: "minioadmin"
    minio_secret_key: "minioadmin"
    
    minio_client_aliases:
      - name: "myminio"
        url: "https://minio.example.com"
        access_key: "{{ minio_access_key }}"
        secret_key: "{{ minio_secret_key }}"
        api: "S3v4"
  
  roles:
    - role: minio_client
```

**Execution:**
```bash
# Complete installation
ansible-playbook -i inventory.ini playbook.yml

# Installation only (without configuration)
ansible-playbook -i inventory.ini playbook.yml --tags install

# Configuration only (if already installed)
ansible-playbook -i inventory.ini playbook.yml --tags configure
```

### Example 3: Multi-Environment with Secrets

```yaml
---
- name: MinIO Client Installation with Secrets
  hosts: all
  become: yes
  
  vars_files:
    - vars/secrets.yml  # File encrypted with ansible-vault
  
  vars:
    minio_client_aliases:
      - name: "production"
        url: "https://minio.prod.example.com"
        access_key: "{{ vault_prod_access_key }}"
        secret_key: "{{ vault_prod_secret_key }}"
        api: "S3v4"
      
      - name: "staging"
        url: "https://minio.staging.example.com"
        access_key: "{{ vault_staging_access_key }}"
        secret_key: "{{ vault_staging_secret_key }}"
        api: "S3v4"
  
  roles:
    - role: minio_client
```

**vars/secrets.yml file (to encrypt):**
```yaml
vault_prod_access_key: "PROD_ACCESS_KEY"
vault_prod_secret_key: "PROD_SECRET_KEY"
vault_staging_access_key: "STAGING_ACCESS_KEY"
vault_staging_secret_key: "STAGING_SECRET_KEY"
```

**Execution:**
```bash
# Create secrets file
ansible-vault create vars/secrets.yml

# Install with secrets
ansible-playbook -i inventory.ini playbook.yml --ask-vault-pass
```

### Example 4: Installation with Custom Options

```yaml
---
- name: Custom MinIO Client Installation
  hosts: minio_servers
  become: yes
  
  vars:
    # Installation options
    minio_client_install_dir: "/opt/minio"
    minio_client_binary_name: "mc"
    minio_client_force_reinstall: false
    
    # Configuration
    minio_client_owner: "minio"
    minio_client_group: "minio"
    minio_client_config_dir: "/etc/minio"
    
    # Aliases
    minio_client_aliases:
      - name: "local"
        url: "http://localhost:9000"
        access_key: "{{ lookup('env', 'MINIO_ACCESS_KEY') }}"
        secret_key: "{{ lookup('env', 'MINIO_SECRET_KEY') }}"
    
    # Connection test after configuration
    minio_client_test_connection: true
  
  roles:
    - role: minio_client
```

**Execution:**
```bash
export MINIO_ACCESS_KEY="your_key"
export MINIO_SECRET_KEY="your_secret"
ansible-playbook -i inventory.ini playbook.yml
```

### Example 5: Group Installation with Different Configurations

```yaml
---
- name: MinIO Client Installation - Production Environment
  hosts: production_servers
  become: yes
  
  vars:
    minio_client_aliases:
      - name: "production"
        url: "https://minio.prod.example.com"
        access_key: "{{ prod_access_key }}"
        secret_key: "{{ prod_secret_key }}"
  
  roles:
    - role: minio_client
      tags: [production]

- name: MinIO Client Installation - Dev Environment
  hosts: dev_servers
  become: yes
  
  vars:
    minio_client_aliases:
      - name: "dev"
        url: "http://minio-dev.local:9000"
        access_key: "devuser"
        secret_key: "devpass"
  
  roles:
    - role: minio_client
      tags: [development]
```

**Execution:**
```bash
# Deploy only on production
ansible-playbook -i inventory.ini playbook.yml --tags production

# Deploy only on dev
ansible-playbook -i inventory.ini playbook.yml --tags development
```

## Using Tags

### Complete installation (default)
```bash
ansible-playbook -i inventory.ini playbook.yml
```

### Installation only
```bash
ansible-playbook -i inventory.ini playbook.yml --tags install
```

### Configuration only
```bash
ansible-playbook -i inventory.ini playbook.yml --tags configure
```

### Installation + Configuration
```bash
ansible-playbook -i inventory.ini playbook.yml --tags "install,configure"
```

### Verification only
```bash
ansible-playbook -i inventory.ini playbook.yml --tags verify
```

### Alias configuration only
```bash
ansible-playbook -i inventory.ini playbook.yml --tags aliases
```

### Dependencies installation only
```bash
ansible-playbook -i inventory.ini playbook.yml --tags dependencies
```

### Dry-run mode (simulation)
```bash
ansible-playbook -i inventory.ini playbook.yml --check
```

### Verbose mode (debugging)
```bash
ansible-playbook -i inventory.ini playbook.yml -vvv
```

## Managing Secrets with Ansible Vault

### Create a secrets file
```bash
ansible-vault create vars/secrets.yml
```

### Edit the secrets file
```bash
ansible-vault edit vars/secrets.yml
```

### View the content
```bash
ansible-vault view vars/secrets.yml
```

### Change the password
```bash
ansible-vault rekey vars/secrets.yml
```

### Use in a playbook
```bash
ansible-playbook -i inventory.ini playbook.yml --ask-vault-pass
```

## Role Structure

```
minio_client/
├── defaults/
│   └── main.yml          # Default variables
├── tasks/
│   ├── main.yml          # Entry point (includes install and configure)
│   ├── install.yml       # Multi-distribution installation
│   └── configure.yml     # Alias configuration
├── templates/
│   └── config.json.j2    # Configuration template (optional)
├── handlers/
│   └── main.yml          # Handlers
└── meta/
    └── main.yml          # Role metadata
```

## Post-Installation Verification

```bash
# Check installed version
mc --version

# List configured aliases
mc alias list

# Test connection to an alias
mc ls production/

# Get server information
mc admin info production
```

## Example Inventory

```ini
[minio_clients]
server1.example.com ansible_user=root
server2.example.com ansible_user=admin ansible_become=yes

[minio_clients:vars]
ansible_python_interpreter=/usr/bin/python3
```

## Dependencies

No external dependencies. The role only uses core Ansible modules and native package managers.

## Troubleshooting

### Client doesn't download
```bash
# Check connectivity
curl -I https://dl.min.io/client/mc/release/linux-amd64/mc

# Install in verbose mode
ansible-playbook -i inventory.ini playbook.yml --tags download -vvv
```

### Configuration error
```bash
# Reconfigure
ansible-playbook -i inventory.ini playbook.yml --tags configure

# Manually verify on the server
ssh user@server "mc alias list"
```

### Force reinstallation
```yaml
vars:
  minio_client_force_reinstall: true
```

## License

MIT

## Author

Created for installing and configuring the MinIO client on multiple Linux distributions.

## Contributing

Contributions are welcome! Feel free to open an issue or pull request.