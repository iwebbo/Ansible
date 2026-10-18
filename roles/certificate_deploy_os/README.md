# Ansible Role: Certificate Deploy OS

Rôle Ansible pour installer et déployer des certificats SSL/TLS sur Debian, Red Hat, AIX et Windows.

## Prérequis

- Ansible 2.14 ou plus récent
- Serveurs Linux/Unix avec OpenSSL installé
- Serveurs Windows avec PowerShell 5.1+

## Fonctionnalités

- Déploiement automatique de certificats depuis templates Jinja2
- Génération automatique des clés privées si non existantes
- Support multi-OS (Debian, Ubuntu, Red Hat, AIX, Windows)
- Validation des certificats après déploiement
- Notification des services web pour rechargement du certificat

## Structure du rôle


certificate_deploy_os/
├── tasks/main.yml          # Point d'entrée avec routage OS
├── handlers/main.yml       # Handlers de redémarrage services
├── defaults/main.yml       # Variables par défaut
├── vars/main.yml           # Variables internes
├── meta/main.yml           # Métadonnées Galaxy
├── templates/              # Templates Jinja2 pour certificats
└── README.md               # Documentation


## Variables

### Principales variables (defaults/main.yml)

| Variable | Type | Valeur par défaut | Description |
|----------|------|-------------------|-------------|
| cert_dir | string | `/etc/ssl/certs` | Répertoire des certificats |
| key_dir | string | `/etc/ssl/private` | Répertoire des clés privées |
| cert_file | string | `` | Chemin du fichier certificat (obligatoire) |
| key_file | string | `` | Chemin du fichier clé privée (obligatoire) |
| server_name | string | `localhost` | Nom du serveur |
| common_name | string | `{{ server_name }}` | CN du certificat |
| key_size | integer | `2048` | Taille de la clé RSA |
| file_permissions_cert | string | `'0644'` | Permissions du certificat |
| file_permissions_key | string | `'0600'` | Permissions de la clé |

### Variables obligatoires

Pour utiliser ce rôle, vous devez définir au moins:
- `cert_file`: Chemin complet vers le fichier certificat
- `key_file`: Chemin complet vers le fichier clé privée

## Exemple d'utilisation

yaml
---
# playbook.yml

- name: Déployer les certificats SSL/TLS
  hosts: all
  become: true
  gather_facts: yes

  tasks:
    - name: Générer la clé privée si elle n'existe pas
      ansible.builtin.command:
        cmd: openssl genrsa -out {{ key_file }} {{ key_size }}
      args:
        creates: "{{ key_file }}"

    - name: Générer le certificat auto-signé
      ansible.builtin.command:
        cmd: openssl req -new -x509 -days 365 -key {{ key_file }} \
              -out {{ cert_file }} -subj "/CN={{ common_name }}"
      args:
        creates: "{{ cert_file }}"

    - name: Déployer les certificats
      certificate_deploy_os:
        cert_file: /etc/ssl/certs/{{ server_name }}.crt
        key_file: /etc/ssl/private/{{ server_name }}.key
        key_size: 2048

    - name: Vérifier le déploiement
      ansible.builtin.command:
        cmd: openssl x509 -in {{ cert_file }} -text -noout | head -10
      register: cert_check
      changed_when: false


## Handlers

Ce rôle déclenche les handlers suivants après le déploiement de certificats:

| Service | OS | Action |
|---------|-----|--------|
| nginx | Debian/Ubuntu | Reload |
| apache2 | Debian/Ubuntu | Restart |
| systemd-nginx | Red Hat/CentOS | Reload |
| httpd (Apache) | Red Hat/CentOS | Restart |
| World Wide Web Publishing Service | Windows | Restart IIS |

## Notes importantes

1. **Windows**: Les clés privées sont générées au format PKCS#8 pour compatibilité avec PowerShell et .NET
2. **Linux**: Les certificats sont au format PEM standard
3. **AIX**: Support via les modules de gestion de fichiers standards d'AIX
4. **Idempotence**: Le rôle est idempotent - il ne modifie rien si les fichiers existent déjà