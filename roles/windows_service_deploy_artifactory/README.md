# Ansible Role: windows_service_deploy_artifactory

Rôle pour déployer des services Windows depuis Artifactory

## General Information

**Author:** A&ECoding
**License:** MIT
**Minimum Ansible Version:** 2.9

**Supported Platforms:**
- Windows
  - Versions: 2012R2, 2016, 2019, 2022

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
force_download: true
download_timeout: 300
delete_package_after_extract: false
cleanup_temp_files: true
create_backup: true
backup_path: C:\backup
recreate_service: false
service_name: ''
service_executable: ''
service_display_name: ''
service_description: "Service d\xE9ploy\xE9 via Ansible"
service_install_path: ''
service_start_mode: auto
service_username: ''
service_password: ''
start_service_after_deploy: true
service_startup_timeout: 60
configure_failure_actions: true
failure_reset_period: 86400
failure_actions: restart/60000/restart/60000/restart/60000
msi_arguments: /quiet
exe_arguments: /S
service_health_check_port: ''
service_health_check_host: localhost
health_check_timeout: 30
config_files: []
config_transformations: []
service_dependencies: []
directory_permissions: []

```

## Main Tasks

- Créer le répertoire de téléchargement temporaire
- Télécharger le package du service depuis Artifactory
- Vérifier que le téléchargement a réussi
- Échec si le package n'existe pas
- Vérifier si le service existe déjà
- Arrêter le service existant avant le déploiement
- Créer le répertoire de destination s'il n'existe pas
- Sauvegarder la version précédente (optionnel)
- Extraire le package (ZIP)
- Installer le package (MSI)
- Installer le package (EXE)
- Copier les fichiers de configuration personnalisés
- Appliquer les transformations de configuration
- Supprimer le service existant s'il faut le recréer
- Créer le service Windows
- Configurer les paramètres avancés du service
- Configurer l'action en cas d'échec du service
- Définir les permissions sur le répertoire du service
- Démarrer le service
- Attendre que le service soit complètement démarré
- Vérifier l'état du service
- Nettoyer les fichiers temporaires
- Vérifier que le service fonctionne (test de port si spécifié)
- Afficher le résultat du déploiement

## Handlers

```yaml
- name: restart service
  win_service:
    name: '{{ service_name }}'
    state: restarted
  when: service_name is defined
- name: stop service
  win_service:
    name: '{{ service_name }}'
    state: stopped
  when: service_name is defined
- name: start service
  win_service:
    name: '{{ service_name }}'
    state: started
  when: service_name is defined

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