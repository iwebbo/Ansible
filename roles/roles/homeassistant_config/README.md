# Ansible Role: homeassistant_config

Rôle pour déployer et gérer les configurations Home Assistant OS via templates YAML

## General Information

**Author:** A&E Coding
**License:** MIT
**Minimum Ansible Version:** 2.9

**Supported Platforms:**
- Generic
  - Versions: all
- Debian
  - Versions: 10, 11, 12
- Ubuntu
  - Versions: 20.04, 22.04, 24.04
- Alpine
  - Versions: all

## Variables

### main

```yaml
ha_config_path: /homeassistant
ha_host: '{{ inventory_hostname }}'
ha_port: 8123
ha_backup_enabled: true
ha_cleanup_old_backups: true
ha_wait_after_restart: true
ha_verify_health: true
ha_restart_delay: 30
ha_restart_timeout: 300
ha_yaml_files:
- configuration.yaml
- automations.yaml
- scripts.yaml
- scenes.yaml
- switchs.yaml

```

## Main Tasks

- Afficher les paramètres de déploiement
- Vérifier que le répertoire de configuration existe
- Échouer si le répertoire de configuration n'existe pas
- Sauvegarder les fichiers existants
- Nettoyer les anciens backups (garder seulement les 2 plus récents)
- Copier les fichiers YAML de configuration
- Forcer le redémarrage de Home Assistant si fichiers modifiés
- Attendre que Home Assistant redémarre
- Vérifier que Home Assistant répond
- Afficher le statut final

## Handlers

```yaml
- name: restart_homeassistant
  uri:
    url: http://{{ ha_host }}:{{ ha_port }}/api/services/homeassistant/restart
    method: POST
    headers:
      Authorization: Bearer {{ ha_long_lived_token }}
      Content-Type: application/json
    status_code: 200
  when: ha_long_lived_token is defined and ha_long_lived_token != ""
  ignore_errors: true
- name: restart_homeassistant_fallback
  uri:
    url: http://{{ ha_host }}:{{ ha_port }}/api/services/homeassistant/restart
    method: POST
    headers:
      Content-Type: application/json
    status_code:
    - 200
    - 401
  when: ha_long_lived_token is not defined or ha_long_lived_token == ""
  ignore_errors: true

```

## Templates

- `scenes.yaml.j2`
- `configuration.yaml.j2`
- `automations.yaml.j2`
- `switchs.yaml.j2`
- `scripts.yaml.j2`
- `secrets.yaml.j2`

## Role Structure

```
vars/
    └── main.yml
templates/
    ├── automations.yaml.j2
    ├── configuration.yaml.j2
    ├── scenes.yaml.j2
    ├── scripts.yaml.j2
    ├── secrets.yaml.j2
    └── switchs.yaml.j2
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