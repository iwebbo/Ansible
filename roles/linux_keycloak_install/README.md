# Ansible Role: Keycloak

Ce rôle Ansible installe et configure Keycloak en mode standalone sur des serveurs Debian.

## Prérequis

- Debian 11 (Bullseye) ou plus récent
- Minimum 2 Go de RAM
- Ansible 2.9 ou plus récent

## Structure du rôle

Le rôle est organisé selon les bonnes pratiques Ansible avec une structure modulaire et des tags :

```
keycloak/
├── defaults/         # Variables par défaut
├── handlers/         # Handlers pour redémarrer les services
├── meta/             # Métadonnées du rôle
├── tasks/           
│   ├── main.yml      # Tâches principales
│   ├── install.yml   # Installation des dépendances et Keycloak
│   ├── configure.yml # Configuration de Keycloak
│   └── service.yml   # Gestion du service systemd
├── templates/        # Templates Jinja2 pour les fichiers de config
└── vars/             # Variables spécifiques
```

## Tags disponibles

- `install`: Tâches d'installation uniquement
- `configure`: Tâches de configuration uniquement
- `service`: Gestion du service uniquement
- `keycloak`: Toutes les tâches (tag global)

## Variables du rôle

Consultez le fichier `defaults/main.yml` pour la liste complète des variables configurables.

Variables importantes:
- `keycloak_version`: Version de Keycloak à installer (défaut: "26.3.2")
- `keycloak_launch_mode`: Mode de lancement ("start-dev" pour développement, "start" pour production)
- `keycloak_db_type`: Type de base de données ("h2" pour dev, "postgresql" pour production)

## Utilisation

Exemple d'utilisation du rôle dans un playbook:

```yaml
- hosts: keycloak_servers
  become: yes
  roles:
    - role: keycloak
      keycloak_version: "26.3.2"
      keycloak_db_type: "postgresql"
      keycloak_admin_user: "admin"
      keycloak_admin_password: "{{ vault_keycloak_admin_password }}"
```

## Utilisation avec des tags

Pour exécuter seulement certaines parties du rôle:

```bash
# Installation uniquement
ansible-playbook deploy_keycloak.yml --tags "install"

# Configuration uniquement
ansible-playbook deploy_keycloak.yml --tags "configure"

# Gestion du service uniquement
ansible-playbook deploy_keycloak.yml --tags "service"
```

```yaml
---
# Playbook: deploy_keycloak.yml
# Description: Déploie Keycloak en standalone sur un serveur Debian
# Auteur: Claude
# Date: 26 octobre 2025

- name: Deploy Keycloak Standalone
  hosts: "{{ HOST | default('Keycloak') }}"
  become: yes
  gather_facts: yes
  
  # Variables du playbook
  vars:
    # Variables par défaut du playbook
    keycloak_version: "{{ keycloak_version | default('26.3.2') }}"
    deployment_environment: "{{ env | default('production') }}"
    java_version: "21"
    keycloak_admin_user: "{{ admin_user | default('admin') }}"
    keycloak_admin_password: "{{ admin_password | default(vault_keycloak_admin_password | default('changeme')) }}"
    
    # Variables de configuration du serveur
    keycloak_hostname: "{{ ansible_fqdn }}"
    keycloak_http_port: 8080
    keycloak_https_port: 8443
    
    # Variables Java
    keycloak_java_min_memory: "512m"
    keycloak_java_max_memory: "{{ java_memory | default('1g') }}"
    
    # Configurations base de données
    keycloak_db_type: "{{ 'postgresql' if use_postgresql | default(false) | bool else 'h2' }}"
    keycloak_db_host: "{{ postgresql_host | default('localhost') }}"
    keycloak_db_port: "{{ postgresql_port | default(5432) }}"
    keycloak_db_name: "{{ postgresql_dbname | default('keycloak') }}"
    keycloak_db_username: "{{ postgresql_user | default('keycloak') }}"
    keycloak_db_password: "{{ postgresql_password | default(vault_postgresql_password | default('changeme')) }}"
    
    # Configuration de lancement
    keycloak_launch_mode: "{{ 'start-dev' if env == 'dev' else 'start' }}"
    keycloak_launch_options: "{{ launch_options | default([]) }}"
    
    # Configuration des logs
    keycloak_log_level: "{{ log_level | default('INFO') }}"
    
    # Proxy et Firewall
    keycloak_behind_proxy: "{{ behind_proxy | default(false) }}"
    keycloak_configure_firewall: "{{ configure_firewall | default(true) }}"
    
    # Cache et Clustering
    keycloak_cache_mode: "{{ 'local' if env == 'dev' else 'distributed' if cluster_mode | default(false) | bool else 'local' }}"
  
  # Vérifications préalables
  pre_tasks:
    - name: Validate system requirements
      assert:
        that:
          - ansible_distribution == 'Debian' or ansible_distribution == 'Ubuntu'
          - ansible_memtotal_mb >= 2048  # Minimum 2GB de RAM recommandé
        fail_msg: "Debian/Ubuntu avec au moins 2GB de RAM requis pour Keycloak"
      tags:
        - always
        - validate
    
    - name: Check network connectivity
      wait_for:
        host: "{{ inventory_hostname }}"
        port: 22
        timeout: 5
      delegate_to: localhost
      tags:
        - always
        - validate
        
    - name: Check PostgreSQL connectivity if enabled
      wait_for:
        host: "{{ keycloak_db_host }}"
        port: "{{ keycloak_db_port }}"
        timeout: 5
      delegate_to: localhost
      when: keycloak_db_type == 'postgresql'
      tags:
        - always
        - validate
        - database
        
    - name: Display deployment information
      debug:
        msg: 
          - "Déploiement de Keycloak {{ keycloak_version }} en mode {{ deployment_environment }}"
          - "Base de données: {{ keycloak_db_type }}"
          - "Mode de lancement: {{ keycloak_launch_mode }}"
      tags:
        - always
        - info
  
  # Rôles à exécuter
  roles:
    - role: keycloak
      tags:
        - keycloak
  
  # Tâches post-installation
  post_tasks:
    - name: Verify Keycloak is running
      uri:
        url: "http://localhost:{{ keycloak_http_port }}/health"
        return_content: yes
        status_code: 200
        timeout: 30
      register: keycloak_health
      until: keycloak_health is succeeded
      retries: 5
      delay: 10
      failed_when: false
      changed_when: false
      tags:
        - always
        - verify
    
    - name: Display deployment status
      debug:
        msg: >
          {% if keycloak_health is succeeded %}
          ✅ Keycloak {{ keycloak_version }} a été déployé avec succès et est accessible à l'URL http://{{ keycloak_hostname }}:{{ keycloak_http_port }} 
          {% else %}
          ⚠️ Keycloak {{ keycloak_version }} a été déployé mais n'est pas accessible via l'endpoint health check. Vérifiez les logs.
          {% endif %}
      tags:
        - always
        - verify
```

## Notes de sécurité

- Pour la production, assurez-vous de définir des mots de passe sécurisés et de les stocker dans Ansible Vault
- Configurez HTTPS en production en définissant les certificats appropriés
- Utilisez PostgreSQL au lieu de H2 en production

## Licence

MIT