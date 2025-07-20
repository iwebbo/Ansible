# Ansible Role: iis-deploy-v2

Rôle Ansible to install Apache2 on linux

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
artifactory_base_url: ''
artifactory_username: ''
artifactory_password: ''
artifactory_repository: ''
package_filename: ''
artifactory_package_url: '{{ artifactory_base_url }}/{{ artifactory_repository }}/{{
  package_filename }}'
deploy_temp_dir: C:\temp\deploy
deploy_destination_path: C:\inetpub\wwwroot
force_download: true
download_timeout: 300
delete_package_after_extract: false
cleanup_temp_files: true
create_backup: true
backup_path: C:\backup
site_name: ''
site_port: 80
site_ip: '*'
site_hostname: ''
app_pool_name: ''
app_pool_identity: ApplicationPoolIdentity
app_pool_username: ''
app_pool_password: ''
dotnet_version: v4.0
enable_32bit: false
msi_arguments: /quiet
perform_health_check: true
health_check_path: /
health_check_retries: 5
health_check_delay: 30
expected_status_code:
- 200
- 301
- 302
config_files: []
config_transformations: []
ssl_bindings: []
directory_permissions: []

```

## Main Tasks

- Créer le répertoire de téléchargement temporaire
- Télécharger le package depuis Artifactory
- Vérifier que le téléchargement a réussi
- Échec si le package n'existe pas
- Arrêter le pool d'applications avant le déploiement
- Arrêter le site web avant le déploiement
- Créer le répertoire de destination s'il n'existe pas
- Sauvegarder la version précédente (optionnel)
- Extraire le package (ZIP)
- Extraire le package (MSI) - Installation
- Copier les fichiers de configuration personnalisés
- Appliquer les transformations de configuration
- Créer le pool d'applications s'il n'existe pas
- Créer ou mettre à jour le site web IIS
- Configurer les bindings SSL si spécifiés
- Définir les permissions sur le répertoire
- Démarrer le pool d'applications
- Démarrer le site web
- Nettoyer les fichiers temporaires
- Vérifier que l'application répond
- Afficher le résultat du déploiement

## Handlers

```yaml
- name: restart iis site
  win_iis_website:
    name: '{{ site_name }}'
    state: restarted
  when: site_name is defined
- name: restart app pool
  win_iis_webapppool:
    name: '{{ app_pool_name }}'
    state: restarted
  when: app_pool_name is defined

```

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