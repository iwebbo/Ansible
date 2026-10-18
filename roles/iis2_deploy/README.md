# Ansible Role: iis2_deploy

Rôle Ansible pour installer et gérer Apache2 sur serveurs Debian/RedHat Linux.

## General Information

**Author:** A&ECoding
**License:** MIT
**Minimum Ansible Version:** 2.9

## Supported Platforms

- **Debian**: Versions 10 (Buster), 11 (Bullseye), 12 (Bookworm)
- **Ubuntu**: Versions 20.04, 22.04, 24.04
- **RHEL/CentOS/Fedora**: Versions 8, 9, 38+, 39+, 40+

## Variables

### main

yaml
# Nom du site web Apache (valeur par défaut)
iis2_apache_site_name: "Apache Default Site"

# Port d'écoute du serveur Apache
iis2_apache_port: 80

# Chemin racine du document root
iis2_apache_docroot: "/var/www/html"

# Nom de l'utilisateur Apache (Debian/Ubuntu)
iis2_apache_user: apache

# Nom du groupe Apache (Debian/Ubuntu)
iis2_apache_group: apache

# Habilitation du module SSL par défaut
iis2_enable_ssl: false


### linux/main.yml

yaml
# Modules à installer pour Debian/Ubuntu
iis2_apache_modules:
  - mpm_prefork
  - ssl
  - rewrite
  - headers
  - env1
  - env2
  - expires
  - unixd


### unix/main.yml

yaml
# Modules à installer pour RHEL/CentOS/Fedora
iis2_apache_rhel_modules:
  - httpd
  - mod_ssl
  - mod_rewrite
  - mod_headers
  - mod_env
  - mod_expires
  - mod_unixd


## Main Tasks

- Mettre à jour les listes de paquets
- Installer Apache2 et ses modules requis
- Créer l'utilisateur et le groupe Apache
- Configurer la protection des fichiers cachés
- Activer le suivi d'activité (access logging)
- Activer le suivi d'erreurs (error logging)
- Démarrer le service Apache

## Handlers

- Restart Apache2 service (Debian/Ubuntu)
- Restart httpd service (RHEL/CentOS/Fedora)

## Example Playbook Usage

yaml
---
- name: Install and configure Apache2 on Linux servers
  hosts: apache_servers
  become: yes
  gather_facts: true

  tasks:
    - name: Deploy iis2_deploy role
      ansible.builtin.import_role:
        name: iis2_deploy

    - name: Configure custom site
      vars:
        iis2_apache_site_name: "My Custom Site"
        iis2_apache_docroot: "/var/www/myapp"
      ansible.builtin.debug:
        msg: "Configuration applied successfully"


## Role Structure


iis2_deploy/
├── tasks/
│   ├── main.yml          # Entry point with OS detection
│   ├── linux/            # Debian/Ubuntu specific tasks
│   └── unix/             # RHEL/CentOS/Fedora specific tasks
├── handlers/
│   └── main.yml          # Service restart handlers
├── defaults/
│   └── main.yml           # Default variables
├── vars/
│   └── main.yml           # Internal variables (if needed)
├── meta/
│   └── main.yml           # Galaxy metadata
├── templates/
│   └── apache.conf.j2     # Apache configuration template
└── README.md              # This file


## Dependencies

- None