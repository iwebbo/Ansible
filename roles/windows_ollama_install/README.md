# Ansible Role: windows_ollama_install

Rôle Ansible pour installer, déplacer et configurer Ollama sous Windows

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
ollama_target_path: G:\LLM\Ollama

```

## Main Tasks

- Télécharger et installer Ollama en mode silencieux
- Détecter le chemin d’installation d’Ollama

## Other Tasks

### detect_ollama_path.yml

```yaml
- name: Trouver le chemin d'installation d'Ollama
  ansible.windows.win_shell: '$paths = @("C:\Program Files\Ollama", "C:\Program Files
    (x86)\Ollama")

    foreach ($path in $paths) { if (Test-Path $path) { Write-Output $path; break }
    }

    '
  register: ollama_path
  changed_when: false

```

### download_install.yml

```yaml
- name: "T\xE9l\xE9charger Ollama depuis le site officiel"
  ansible.windows.win_get_url:
    url: https://ollama.com/download/OllamaSetup.exe
    dest: C:\Temp\OllamaInstaller.exe
- name: Installer Ollama en mode silencieux
  ansible.windows.win_shell: C:\Temp\OllamaInstaller.exe /DIR={{ ollama_target_path
    }}
  register: ollama_install
  changed_when: '''installation'' in ollama_install.stdout'

```

### move_yollama.yml

```yaml
- name: "D\xE9placer Ollama vers {{ ollama_target_path }}"
  ansible.windows.win_shell: 'Move-Item -Path "{{ ollama_path.stdout }}" -Destination
    "{{ ollama_target_path }}" -Force

    '
  register: move_result
  changed_when: '''moved'' in move_result.stdout'
- name: "V\xE9rifier le d\xE9placement"
  ansible.windows.win_shell: if (Test-Path '{{ ollama_target_path }}') { Write-Output
    'moved' }
  register: move_check
  changed_when: false
- name: "Arr\xEAter le playbook si le d\xE9placement a \xE9chou\xE9"
  fail:
    msg: "Le d\xE9placement d'Ollama a \xE9chou\xE9."
  when: move_check.stdout != "moved"

```

### update_path.yml

```yaml
- name: Ajouter Ollama au PATH
  ansible.windows.win_shell: "$path = [System.Environment]::GetEnvironmentVariable(\"\
    Path\", [System.EnvironmentVariableTarget]::Machine)\nif (-not $path.Contains(\"\
    {{ ollama_target_path }}\")) {\n    [System.Environment]::SetEnvironmentVariable(\"\
    Path\", $path + \";{{ ollama_target_path }}\", [System.EnvironmentVariableTarget]::Machine)\n\
    }\n"
  register: path_update
  changed_when: '''updated'' in path_update.stdout'
- name: "V\xE9rifier si le PATH contient le nouveau chemin"
  ansible.windows.win_shell: 'if ($env:Path -match "{{ ollama_target_path }}") { Write-Output
    "updated" }

    '
  register: path_check
  changed_when: false
- name: "Arr\xEAter le playbook si le PATH n\u2019a pas \xE9t\xE9 mis \xE0 jour"
  fail:
    msg: "La mise \xE0 jour du PATH a \xE9chou\xE9."
  when: path_check.stdout != "updated"

```

## Role Structure

```
defaults/
    └── main.yml
handlers/
    └── main.yml
meta/
    └── main.yml
tasks/
    ├── detect_ollama_path.yml
    ├── download_install.yml
    ├── main.yml
    ├── move_yollama.yml
    └── update_path.yml
tests/
    ├── inventory
    └── test.yml
vars/
    └── main.yml
README.md
```
