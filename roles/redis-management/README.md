# Ansible Role: redis-management

Rôle Ansible pour redis management

## General Information

**Author:** A&ECoding
**License:** MIT
**Minimum Ansible Version:** 2.9

**Supported Platforms:**
- Linux

## Variables

### main

```yaml
action_redis: start
redis_service_name: redis
redis_check_status: true
redis_actions_allowed:
- start
- stop
- restart
- reload

```

## Main Tasks

- Vérifier que l'action Redis est valide
- Vérifier le statut actuel du service Redis
- Afficher le statut actuel de Redis
- Déclencher l'action Redis si conditions remplies
- Forcer l'exécution des handlers

## Handlers

```yaml
- name: execute redis action
  service:
    name: '{{ redis_service_name }}'
    state: '{{ action_redis }}'
  listen: execute redis action
- name: verify redis status after action
  shell: systemctl status {{ redis_service_name }} | grep "Active:"
  register: final_status
  failed_when: false
  changed_when: false
  listen: execute redis action
- name: display final redis status
  debug:
    msg: 'Statut final de Redis: {{ final_status.stdout }}'
  listen: execute redis action

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