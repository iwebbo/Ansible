# Ansible Role: iis-stop

Rôle pour arrêter IIS sur Windows Server

## General Information

**Author:** Votre nom
**License:** MIT
**Minimum Ansible Version:** 2.9

**Supported Platforms:**
- Windows
  - Versions: 2012R2, 2016, 2019, 2022

## Variables

### main

```yaml
iis_default_site_name: Default Web Site
iis_app_pools:
- DefaultAppPool

```

## Main Tasks

- Arrêter les pools d'applications
- Arrêter le site web par défaut
- Arrêter le service World Wide Web Publishing Service
- Arrêter le service IIS Admin Service
- Vérifier que les services IIS sont arrêtés
- Afficher le statut d'arrêt d'IIS

## Handlers

```yaml
- name: stop iis
  win_service:
    name: W3SVC
    state: stopped
- name: stop iis admin
  win_service:
    name: IISADMIN
    state: stopped

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