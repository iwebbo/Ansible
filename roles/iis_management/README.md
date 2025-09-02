# Ansible Role: iis_management

your role description

## General Information

**Author:** your name
**License:** license (GPL-2.0-or-later, MIT, etc)
**Minimum Ansible Version:** 2.1

## Variables

### main

```yaml
action_iis: status
iis_actions_allowed:
- start
- stop
- restart
- status

```

## Main Tasks

- Check if Windows system
- Display system information
- Validate allowed action
- Check if IIS is installed
- Fail if IIS is not installed
- Start IIS
- Stop IIS
- Restart IIS
- Get IIS status
- Display IIS status

## Handlers

```yaml
- name: iis started
  debug:
    msg: IIS has been started successfully
- name: iis stopped
  debug:
    msg: IIS has been stopped successfully
- name: iis restarted
  debug:
    msg: IIS has been restarted successfully

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
    └── main.yml
tests/
    ├── inventory
    └── test.yml
vars/
    └── main.yml
README.md
```