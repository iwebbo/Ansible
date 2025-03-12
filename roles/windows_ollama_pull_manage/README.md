# Ansible Role: windows_ollama_pull_manage

Rôle Ansible to Download Pull models on Ollama windows

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
ollama_models:
- deepseek-r1:7b

```

## Main Tasks

- Check if ollama is installed
- Stop playbook if ollama not present
- Pull Ollama models

## Role Structure

```
defaults/
    └── main.yml
handlers/
    └── main.yml
meta/
    └── main.yml
tasks/
    └── main.yml
tests/
    ├── inventory
    └── test.yml
vars/
    └── main.yml
README.md
```
