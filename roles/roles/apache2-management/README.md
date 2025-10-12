# Ansible Role: apache2-management

Rôle pour gérer le service Apache HTTP Server (start, stop, restart, reload, graceful)

## General Information

**Author:** A&E Coding
**License:** MIT
**Minimum Ansible Version:** 2.9

**Supported Platforms:**
- Ubuntu
  - Versions: 18.04, 20.04, 22.04
- CentOS
  - Versions: 7, 8
- RedHat
  - Versions: 7, 8, 9
- Rocky
  - Versions: 8, 9
- AlmaLinux
  - Versions: 8, 9

## Variables

### main

```yaml
action_apache: start
apache_check_status: true
apache_check_config: true
apache_wait_timeout: 30
apache_validate_config_before_action: true
apache_actions_allowed:
- start
- stop
- restart
- reload
- graceful
apache_service_names:
  RedHat: httpd
  CentOS: httpd
  Rocky: httpd
  AlmaLinux: httpd
  Fedora: httpd
  Ubuntu: apache2
  Debian: apache2
apache_config_test:
  RedHat: httpd -t
  CentOS: httpd -t
  Rocky: httpd -t
  AlmaLinux: httpd -t
  Fedora: httpd -t
  Ubuntu: apache2ctl configtest
  Debian: apache2ctl configtest

```

## Main Tasks

- Détecter la distribution du système
- Afficher les informations détectées
- Vérifier que l'action Apache est valide
- Vérifier si Apache est installé
- Échouer si Apache n'est pas installé
- Tester la configuration Apache avant action
- Afficher le résultat du test de configuration
- Échouer si la configuration Apache est invalide
- Vérifier le statut actuel du service Apache
- Afficher le statut actuel d'Apache
- Vérifier les ports d'écoute Apache (si actif)
- Afficher les ports d'écoute
- Déclencher l'action Apache si conditions remplies
- Aucune action nécessaire
- Forcer l'exécution des handlers

## Handlers

```yaml
- name: execute apache action
  block:
  - name: "Ex\xE9cuter l'action Apache - {{ action_apache }}"
    service:
      name: '{{ apache_service_name }}'
      state: '{{ ''started'' if action_apache == ''start'' else ''stopped'' if action_apache
        == ''stop'' else ''restarted'' }}'
    when: action_apache in ['start', 'stop', 'restart']
  - name: "Ex\xE9cuter le reload Apache"
    service:
      name: '{{ apache_service_name }}'
      state: reloaded
    when: action_apache == 'reload'
  - name: "Ex\xE9cuter le graceful restart Apache"
    shell: '{{ apache_test_command.split()[0] }}ctl graceful'
    when: action_apache == 'graceful'
  - name: "Attendre que le service soit compl\xE8tement d\xE9marr\xE9/arr\xEAt\xE9"
    wait_for:
      port: 80
      host: '{{ ansible_default_ipv4.address | default(''127.0.0.1'') }}'
      timeout: '{{ apache_wait_timeout }}'
    when: action_apache in ['start', 'restart', 'graceful']
    ignore_errors: true
  listen: execute apache action
- name: verify apache status after action
  shell: systemctl status {{ apache_service_name }} | grep 'Active:'
  register: final_status
  failed_when: false
  changed_when: false
  listen: execute apache action
- name: display final apache status
  debug:
    msg: 'Statut final d''Apache: {{ final_status.stdout }}'
  listen: execute apache action
- name: verify apache ports after start actions
  shell: netstat -tlnp | grep ':80\|:443' | grep {{ apache_service_name }}
  register: final_ports
  failed_when: false
  changed_when: false
  when: action_apache in ['start', 'restart', 'graceful']
  listen: execute apache action
- name: display apache listening ports
  debug:
    msg: "Ports d'\xE9coute Apache: {{ final_ports.stdout_lines | default(['Aucun\
      \ port d\xE9tect\xE9']) }}"
  when:
  - final_ports is defined
  - action_apache in ['start', 'restart', 'graceful']
  listen: execute apache action

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