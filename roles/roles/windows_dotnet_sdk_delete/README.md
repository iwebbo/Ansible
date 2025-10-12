# Ansible Role: windows_dotnet_sdk_delete

Rôle Ansible to Delete .NET SDK on Windows

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
sdk_version: '{{ sdk_version }}'
dotnet_sdk_versions:
  '6.0':
    versions:
      6.0.100:
        uninstall_id: exemple-uninstall-id-6.0.100
      6.0.101:
        uninstall_id: exemple-uninstall-id-6.0.101
      6.0.102:
        uninstall_id: exemple-uninstall-id-6.0.102
      6.0.200:
        uninstall_id: exemple-uninstall-id-6.0.200
      6.0.201:
        uninstall_id: exemple-uninstall-id-6.0.201
      6.0.202:
        uninstall_id: exemple-uninstall-id-6.0.202
      6.0.300:
        uninstall_id: exemple-uninstall-id-6.0.300
      6.0.301:
        uninstall_id: exemple-uninstall-id-6.0.301
      6.0.302:
        uninstall_id: exemple-uninstall-id-6.0.302
      6.0.400:
        uninstall_id: exemple-uninstall-id-6.0.400
      6.0.401:
        uninstall_id: exemple-uninstall-id-6.0.401
      6.0.402:
        uninstall_id: exemple-uninstall-id-6.0.402
      6.0.403:
        uninstall_id: exemple-uninstall-id-6.0.403
      6.0.404:
        uninstall_id: exemple-uninstall-id-6.0.404
      6.0.405:
        uninstall_id: exemple-uninstall-id-6.0.405
      6.0.406:
        uninstall_id: exemple-uninstall-id-6.0.406
      6.0.407:
        uninstall_id: exemple-uninstall-id-6.0.407
      6.0.408:
        uninstall_id: exemple-uninstall-id-6.0.408
      6.0.409:
        uninstall_id: exemple-uninstall-id-6.0.409
      6.0.410:
        uninstall_id: exemple-uninstall-id-6.0.410
      6.0.411:
        uninstall_id: exemple-uninstall-id-6.0.411
      6.0.412:
        uninstall_id: exemple-uninstall-id-6.0.412
      6.0.413:
        uninstall_id: exemple-uninstall-id-6.0.413
      6.0.414:
        uninstall_id: exemple-uninstall-id-6.0.414
      6.0.415:
        uninstall_id: exemple-uninstall-id-6.0.415
      6.0.416:
        uninstall_id: exemple-uninstall-id-6.0.416
      6.0.417:
        uninstall_id: exemple-uninstall-id-6.0.417
      6.0.418:
        uninstall_id: exemple-uninstall-id-6.0.418
  '7.0':
    versions:
      7.0.100:
        uninstall_id: exemple-uninstall-id-7.0.100
      7.0.101:
        uninstall_id: exemple-uninstall-id-7.0.101
      7.0.102:
        uninstall_id: exemple-uninstall-id-7.0.102
      7.0.200:
        uninstall_id: exemple-uninstall-id-7.0.200
      7.0.201:
        uninstall_id: exemple-uninstall-id-7.0.201
      7.0.202:
        uninstall_id: exemple-uninstall-id-7.0.202
      7.0.203:
        uninstall_id: exemple-uninstall-id-7.0.203
      7.0.300:
        uninstall_id: exemple-uninstall-id-7.0.300
      7.0.301:
        uninstall_id: exemple-uninstall-id-7.0.301
      7.0.302:
        uninstall_id: exemple-uninstall-id-7.0.302
      7.0.303:
        uninstall_id: exemple-uninstall-id-7.0.303
      7.0.304:
        uninstall_id: exemple-uninstall-id-7.0.304
      7.0.305:
        uninstall_id: exemple-uninstall-id-7.0.305
      7.0.400:
        uninstall_id: exemple-uninstall-id-7.0.400
      7.0.401:
        uninstall_id: exemple-uninstall-id-7.0.401
      7.0.402:
        uninstall_id: exemple-uninstall-id-7.0.402
      7.0.403:
        uninstall_id: exemple-uninstall-id-7.0.403
      7.0.404:
        uninstall_id: exemple-uninstall-id-7.0.404
      7.0.405:
        uninstall_id: exemple-uninstall-id-7.0.405
      7.0.406:
        uninstall_id: exemple-uninstall-id-7.0.406
      7.0.407:
        uninstall_id: exemple-uninstall-id-7.0.407
      7.0.408:
        uninstall_id: exemple-uninstall-id-7.0.408
  '8.0':
    versions:
      8.0.100:
        uninstall_id: exemple-uninstall-id-8.0.100
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
      8.0.203:
        uninstall_id: exemple-uninstall-id-8.0.203
      8.0.300:
        uninstall_id: exemple-uninstall-id-8.0.300
      8.0.301:
        uninstall_id: exemple-uninstall-id-8.0.301
      8.0.302:
        uninstall_id: exemple-uninstall-id-8.0.302
      8.0.303:
        uninstall_id: exemple-uninstall-id-8.0.303
      8.0.304:
        uninstall_id: exemple-uninstall-id-8.0.304
      8.0.305:
        uninstall_id: exemple-uninstall-id-8.0.305
      8.0.306:
        uninstall_id: exemple-uninstall-id-8.0.306
      8.0.307:
        uninstall_id: exemple-uninstall-id-8.0.307
      8.0.308:
        uninstall_id: exemple-uninstall-id-8.0.308
      8.0.309:
        uninstall_id: exemple-uninstall-id-8.0.309
  '9.0':
    versions:
      9.0.100-preview.1:
        uninstall_id: exemple-uninstall-id-9.0.100-preview.1
      9.0.100-preview.2:
        uninstall_id: exemple-uninstall-id-9.0.100-preview.2
      9.0.100-preview.3.24104.13:
        uninstall_id: exemple-uninstall-id-9.0.100-preview.3.24104.13
      9.0.100-rc.1:
        uninstall_id: exemple-uninstall-id-9.0.100-rc.1
      9.0.100-rc.2:
        uninstall_id: exemple-uninstall-id-9.0.100-rc.2
      9.0.100:
        uninstall_id: exemple-uninstall-id-9.0.100
      9.0.101:
        uninstall_id: exemple-uninstall-id-9.0.101
      9.0.102:
        uninstall_id: exemple-uninstall-id-9.0.102

```

## Main Tasks

- Check installation of .NET SDK
- Verify if any .NET SDK has been installed
- Showing .NET SDK informations installed
- Uninstall SDK .NET
- Warning no .NET SDK installed

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