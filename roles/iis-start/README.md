# Ansible Role: iis-start

your role description

## General Information

**Author:** your name
**License:** license (GPL-2.0-or-later, MIT, etc)
**Minimum Ansible Version:** 2.1

## Variables

### main

```yaml
iis_default_site_name: Default Web Site
iis_app_pools:
- DefaultAppPool

```

## Main Tasks

- Démarrer le service World Wide Web Publishing Service
- Démarrer le service IIS Admin Service
- Démarrer le site web par défaut
- Démarrer les pools d'applications
- Vérifier que IIS répond
- Afficher le statut d'IIS

## Handlers

```yaml
- name: restart iis
  win_service:
    name: W3SVC
    state: restarted
- name: restart iis admin
  win_service:
    name: IISADMIN
    state: restarted

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