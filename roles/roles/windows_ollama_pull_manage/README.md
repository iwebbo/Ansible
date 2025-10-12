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
- qwen2.5-coder:32b
- qwen2.5-coder:14b

```

## Main Tasks

- Check if ollama is installed
- Stop playbook if ollama not present
- Pull Ollama models

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