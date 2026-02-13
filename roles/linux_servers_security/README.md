# Ansible Role: Security Hardening

Ansible role to automatically secure Linux servers using Shorewall, fail2ban, and hardened SSH.

## Fonctionnalités

- **Shorewall**: Firewall with customizable rules.
- **fail2ban**: Protection against SSH brute-force and other services.
- **SSH**: Secure configuration (keys only, no root login, modern algorithms).

## Structure

```
roles/security_hardening/
├── tasks/
│   ├── main.yml                   
│   ├── install.yml                 
│   ├── configure_shorewall.yml     
│   ├── configure_fail2ban.yml      
│   └── configure_ssh.yml          
├── templates/
│   ├── shorewall/
│   │   ├── zones.j2
│   │   ├── interfaces.j2
│   │   ├── policy.j2
│   │   └── rules.j2
│   ├── fail2ban/
│   │   └── jail.local.j2
│   └── ssh/
│       └── sshd_config.j2
├── handlers/
│   └── main.yml
├── defaults/
│   └── main.yml                    # Variables par défaut
└── vars/
    └── main.yml
```

## Available Tags

- `install`: Package installation only
- `configuration`: Service configuration only
- `security`: Installation + Configuration (full)
- `shorewall`: Shorewall tasks only
- `fail2ban`: fail2ban tasks only
- `ssh`: SSH tasks only

## Usage

### Basic Playbook

```yaml
---
- name: Security Hardening Linux Server
  hosts: "{{ HOST }}"
  gather_facts: yes
  become: yes
  
  roles:
    - role: security_hardening
```

### Installation only

```bash
ansible-playbook playbook/security_hardening.yml --tags install
```

### Configuration only
```bash
ansible-playbook playbook/security_hardening.yml --tags configuration
```

### Shorewall configuration only

```bash
ansible-playbook playbook/security_hardening.yml --tags shorewall
```

### Full Installation + Configuration

```bash
ansible-playbook playbook/security_hardening.yml --tags security
```

## Main Variables

### Shorewall

```yaml
shorewall_interface: "eth0"
shorewall_ssh_port: 22
shorewall_http_enabled: false
shorewall_allow_ping: true
shorewall_custom_rules:
  - action: ACCEPT
    source: net
    dest: $FW
    proto: tcp
    port: 8080
```

### fail2ban

```yaml
fail2ban_bantime: "1h"
fail2ban_maxretry: 5
fail2ban_destemail: "admin@example.com"
fail2ban_sshd_enabled: true
fail2ban_sshd_maxretry: 3

fail2ban_custom_jails:
  - name: nginx-auth
    enabled: true
    port: http,https
    filter: nginx-auth
    logpath: /var/log/nginx/error.log
    maxretry: 5
```

### SSH

```yaml
ssh_port: 22
ssh_permit_root_login: "prohibit-password"  # ou "no"
ssh_password_authentication: "no"
ssh_allow_users:
  - admin
  - deploy
ssh_deny_users:
  - root

ssh_authorized_keys:
  - user: admin
    key: "ssh-rsa AAAAB3... admin@workstation"
```