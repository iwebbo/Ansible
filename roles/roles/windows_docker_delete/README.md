# Ansible Role: windows_docker_delete

Rôle Ansible to delete docker on windows

## General Information

**Author:** A&ECoding
**License:** MIT
**Minimum Ansible Version:** 2.9

**Supported Platforms:**
- Windows
  - Versions: all

## Main Tasks

- Vérifier si Docker est installé
- Arrêter le service Docker si présent
- Désinstaller Docker Desktop
- Supprimer les fichiers Docker restants
- Vérifier si le service Docker a été supprimé
- Supprimer le service Docker s'il existe encore
- Redémarrer la machine après désinstallation

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