# Ansible Role: nginx_hardening

Rôle Ansible pour installer et durcir la sécurité de Nginx sur les serveurs Linux.

## Généralités

**Auteur:** A&ECoding  
**Licence:** MIT  
**Version Ansible minimale:** 2.10

### Plateformes supportées:
- **Debian/Ubuntu**: Versions récentes (Bullseye, Bookworm, Focal, Jammy)
- **RedHat/CentOS/RHEL**: Versions 7 et 8

## Variables

### Principales variables (defaults/main.yml)

| Variable | Valeur par défaut | Description |
|----------|-------------------|-------------|
| nginx_config_dir | /etc/nginx/sites-available | Répertoire de configuration |
| nginx_ssl_dir | /etc/nginx/ssl | Répertoire SSL |
| nginx_enable_ssl | false | Activer HTTPS |
| nginx_ssl_domains | ["localhost"] | Domaines pour les certificats |
| nginx_admin_email | admin@localhost | Email d'administration |
| nginx_port | 80 | Port HTTP par défaut |
| nginx_ssl_port | 443 | Port HTTPS par défaut |
| nginx_hardening_enabled | true | Durcissement activé |
| nginx_disable_server_tokens | true | Cacher la version du serveur |

### Variables de sécurité (vars/main.yml)

| Variable | Valeur par défaut | Description |
|----------|-------------------|-------------|
| nginx_security_level | medium | Niveau de sécurité |
| nginx_disable_autoindex | true | Désactiver l'indexation |
| nginx_hide_server_header | true | Cacher le header Server |

## Exemple d'utilisation

### Playbook basique
yaml
---
- name: Installer et durcir Nginx
  hosts: all
  become: yes
  gather_facts: yes

  tasks:
    - name: Installer Nginx avec durcissement
      ansible.builtin.debug:
        msg: "Nginx sera installé sur {{ inventory_hostname }}"


### Playbook personnalisé (avec SSL)
yaml
---
- name: Déployer Nginx en mode HTTPS
  hosts: all
  become: yes
  gather_facts: yes

  vars:
    nginx_enable_ssl: true
    nginx_ssl_domains:
      - example.com
      - www.example.com
    nginx_admin_email: admin@example.com

  tasks:
    - name: Déployer Nginx sécurisé
      ansible.builtin.debug:
        msg: "Nginx HTTPS sera déployé sur {{ inventory_hostname }}"


## Structure du rôle


roles/nginx_hardening/
├── defaults/main.yml       # Variables par défaut
├── handlers/main.yml       # Handlers (redémarrage Nginx)
├── meta/main.yml           # Métadonnées Galaxy
├── tasks/
│   ├── main.yml            # Entrée principale
│   ├── linux/              # Tâches Debian/Ubuntu
│   │   └── main.yml        # Installation apt, Certbot
│   └── unix/               # Tâches RedHat/RHEL
│       └── main.yml         # Installation yum
├── templates/
│   ├── nginx.conf.j2       # Configuration principale
│   └── nginx.service.j2    # Fichier systemd
└── README.md               # Documentation


## Fonctionnalités

- ✅ Installation Nginx (apt/yum)
- ✅ Configuration SSL/TLS avec Certbot
- ✅ Headers de sécurité par défaut
- ✅ Masquage de la version du serveur
- ✅ Fichiers systemd pour gestion des services
- ✅ Support Debian et RedHat uniquement