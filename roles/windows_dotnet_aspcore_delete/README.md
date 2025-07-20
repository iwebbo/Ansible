# Ansible Role: windows_dotnet_aspcore_delete

Rôle Ansible to Delete .NET Runtimes on windows

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
dotnet_version: '{{ dotnet_version }}'
runtime_version: '{{ runtime_version }}'
dotnet_runtime_versions:
  '6.0':
    runtimes:
      6.0.0:
        uninstall_id: exemple-uninstall-id-6.0.0
      6.0.1:
        uninstall_id: exemple-uninstall-id-6.0.1
      6.0.2:
        uninstall_id: exemple-uninstall-id-6.0.2
      6.0.3:
        uninstall_id: exemple-uninstall-id-6.0.3
      6.0.4:
        uninstall_id: exemple-uninstall-id-6.0.4
      6.0.5:
        uninstall_id: exemple-uninstall-id-6.0.5
      6.0.6:
        uninstall_id: exemple-uninstall-id-6.0.6
      6.0.7:
        uninstall_id: exemple-uninstall-id-6.0.7
      6.0.8:
        uninstall_id: exemple-uninstall-id-6.0.8
      6.0.9:
        uninstall_id: exemple-uninstall-id-6.0.9
      6.0.10:
        uninstall_id: exemple-uninstall-id-6.0.10
      6.0.11:
        uninstall_id: exemple-uninstall-id-6.0.11
      6.0.12:
        uninstall_id: exemple-uninstall-id-6.0.12
      6.0.13:
        uninstall_id: exemple-uninstall-id-6.0.13
      6.0.14:
        uninstall_id: exemple-uninstall-id-6.0.14
      6.0.15:
        uninstall_id: exemple-uninstall-id-6.0.15
      6.0.16:
        uninstall_id: exemple-uninstall-id-6.0.16
      6.0.17:
        uninstall_id: exemple-uninstall-id-6.0.17
      6.0.18:
        uninstall_id: exemple-uninstall-id-6.0.18
      6.0.19:
        uninstall_id: exemple-uninstall-id-6.0.19
      6.0.20:
        uninstall_id: exemple-uninstall-id-6.0.20
      6.0.21:
        uninstall_id: exemple-uninstall-id-6.0.21
      6.0.22:
        uninstall_id: exemple-uninstall-id-6.0.22
      6.0.23:
        uninstall_id: exemple-uninstall-id-6.0.23
      6.0.24:
        uninstall_id: exemple-uninstall-id-6.0.24
      6.0.25:
        uninstall_id: exemple-uninstall-id-6.0.25
      6.0.26:
        uninstall_id: exemple-uninstall-id-6.0.26
      6.0.27:
        uninstall_id: exemple-uninstall-id-6.0.27
      6.0.28:
        uninstall_id: exemple-uninstall-id-6.0.28
  '7.0':
    runtimes:
      7.0.0:
        uninstall_id: exemple-uninstall-id-7.0.0
      7.0.1:
        uninstall_id: exemple-uninstall-id-7.0.1
      7.0.2:
        uninstall_id: exemple-uninstall-id-7.0.2
      7.0.3:
        uninstall_id: exemple-uninstall-id-7.0.3
      7.0.4:
        uninstall_id: exemple-uninstall-id-7.0.4
      7.0.5:
        uninstall_id: exemple-uninstall-id-7.0.5
      7.0.6:
        uninstall_id: exemple-uninstall-id-7.0.6
      7.0.7:
        uninstall_id: exemple-uninstall-id-7.0.7
      7.0.8:
        uninstall_id: exemple-uninstall-id-7.0.8
      7.0.9:
        uninstall_id: exemple-uninstall-id-7.0.9
      7.0.10:
        uninstall_id: exemple-uninstall-id-7.0.10
      7.0.11:
        uninstall_id: exemple-uninstall-id-7.0.11
      7.0.12:
        uninstall_id: exemple-uninstall-id-7.0.12
      7.0.13:
        uninstall_id: exemple-uninstall-id-7.0.13
      7.0.14:
        uninstall_id: exemple-uninstall-id-7.0.14
      7.0.15:
        uninstall_id: exemple-uninstall-id-7.0.15
      7.0.16:
        uninstall_id: exemple-uninstall-id-7.0.16
      7.0.17:
        uninstall_id: exemple-uninstall-id-7.0.17
      7.0.18:
        uninstall_id: exemple-uninstall-id-7.0.18
  '8.0':
    runtimes:
      8.0.0:
        uninstall_id: exemple-uninstall-id-8.0.0
      8.0.1:
        uninstall_id: exemple-uninstall-id-8.0.1
      8.0.2:
        uninstall_id: exemple-uninstall-id-8.0.2
      8.0.3:
        uninstall_id: exemple-uninstall-id-8.0.3
      8.0.4:
        uninstall_id: exemple-uninstall-id-8.0.4
      8.0.5:
        uninstall_id: exemple-uninstall-id-8.0.5
      8.0.6:
        uninstall_id: exemple-uninstall-id-8.0.6
      8.0.7:
        uninstall_id: exemple-uninstall-id-8.0.7
      8.0.8:
        uninstall_id: exemple-uninstall-id-8.0.8
      8.0.9:
        uninstall_id: exemple-uninstall-id-8.0.9
      8.0.10:
        uninstall_id: exemple-uninstall-id-8.0.10
      8.0.101:
        uninstall_id: exemple-uninstall-id-8.0.101
      8.0.102:
        uninstall_id: exemple-uninstall-id-8.0.102
      8.0.103:
        uninstall_id: exemple-uninstall-id-8.0.103
      8.0.104:
        uninstall_id: exemple-uninstall-id-8.0.104
      8.0.200:
        uninstall_id: exemple-uninstall-id-8.0.200
      8.0.201:
        uninstall_id: exemple-uninstall-id-8.0.201
      8.0.202:
        uninstall_id: exemple-uninstall-id-8.0.202
  '9.0':
    runtimes:
      9.0.0:
        uninstall_id: exemple-uninstall-id-9.0.0
      9.0.1:
        uninstall_id: exemple-uninstall-id-9.0.1
      9.0.2:
        uninstall_id: exemple-uninstall-id-9.0.2
      9.0.100-preview.1:
        uninstall_id: exemple-uninstall-id-9.0.100-preview.1
      9.0.100-preview.2:
        uninstall_id: exemple-uninstall-id-9.0.100-preview.2
      9.0.100-preview.3.24104.13:
        uninstall_id: exemple-uninstall-id-9.0.100-preview.3.24104.13

```

## Main Tasks

- Vérifier les runtimes .NET installés
- Extraire les versions de runtime installées
- Définir le préfixe de version
- Vérifier si le runtime spécifié est installé
- Afficher les informations de runtime
- Désinstaller le runtime .NET spécifique
- Avertissement si runtime non installé

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