# Ansible Role: windows_docker_install

Rôle Ansible pour installer, déplacer et configurer docker on Windows

## General Information

**Author:** A&ECoding
**License:** MIT
**Minimum Ansible Version:** 2.9

**Supported Platforms:**
- Windows
  - Versions: all

## Main Tasks

- Vérifier l'architecture du système
- Définir l'URL de téléchargement de Docker Desktop en fonction de l'architecture
- Vérifier si Docker est déjà installé
- Télécharger l'installateur Docker si non installé
- Installer Docker en mode silencieux si non installé
- Supprimer l'installateur Docker après installation
- Configurer le service Docker pour démarrer automatiquement
- Vérifier si le service Docker existe
- Démarrer Docker si le service existe
- Display Guidline after installation

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