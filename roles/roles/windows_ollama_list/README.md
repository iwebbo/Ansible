# Ansible Role: windows_ollama_list

Rôle simple pour lister les modèles Ollama sur serveurs Windows

## General Information

**Author:** A&E Coding
**License:** MIT
**Minimum Ansible Version:** 2.9

**Supported Platforms:**
- Windows
  - Versions: all

## Variables

### main

```yaml
ollama_executable: ollama
ollama_timeout: 30
ollama_display_output: true
ollama_save_to_file: false
ollama_output_file: C:\temp\ollama_models.txt

```

## Main Tasks

- Vérifier si Ollama est installé sur Windows
- Afficher la version d'Ollama détectée
- Lister les modèles Ollama présents
- Afficher la liste des modèles Ollama
- Sauvegarder la liste dans un fichier (optionnel)
- Extraire et formater les informations des modèles
- Résumé des modèles trouvés
- Créer un fait Ansible avec les modèles

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