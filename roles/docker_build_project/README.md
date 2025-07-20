# Ansible Role: docker_build_project

Role for manage Build from Docker Windows/linux

## General Information

**Author:** A&E Coding
**License:** MIT
**Minimum Ansible Version:** 2.9


## Variables

### main

```yaml
docker_action: status
project_path: ''
project_name: server-monitoring
force_rebuild: false
remove_volumes: false

```

## Main Tasks

- Validation - Vérifier le chemin du projet
- Info - Affichage de l'action
- Vérification - Le répertoire existe
- Vérification - Le répertoire existe (Linux)
- Échec si répertoire inexistant
- Action BUILD
- Action START
- Action STOP
- Action DOWN
- Action RESTART
- Action STATUS

## Other Tasks

### start.yml

```yaml
- name: "Start - D\xE9marrage des services (Windows)"
  win_shell: 'cd "{{ project_path }}"

    docker-compose up -d

    '
  when: ansible_os_family == "Windows"
- name: "Start - D\xE9marrage des services (Linux)"
  shell: 'cd "{{ project_path }}"

    docker-compose up -d

    '
  when: ansible_os_family != "Windows"
- name: "Start - Attendre le d\xE9marrage"
  pause:
    seconds: 10
- name: "Start - V\xE9rifier les services"
  win_shell: 'cd "{{ project_path }}"

    docker-compose ps

    '
  when: ansible_os_family == "Windows"
  register: services_status
- name: "Start - V\xE9rifier les services (Linux)"
  shell: 'cd "{{ project_path }}"

    docker-compose ps

    '
  when: ansible_os_family != "Windows"
  register: services_status
- name: "Start - Afficher l'\xE9tat"
  debug:
    var: services_status.stdout_lines

```

### build.yml

```yaml
- name: Build - Construction des images (Windows)
  win_shell: 'cd "{{ project_path }}"

    docker-compose build {{ ''--no-cache'' if force_rebuild else '''' }}

    '
  when: ansible_os_family == "Windows"
  register: build_result
- name: Build - Construction des images (Linux)
  shell: 'cd "{{ project_path }}"

    docker-compose build {{ ''--no-cache'' if force_rebuild else '''' }}

    '
  when: ansible_os_family != "Windows"
  register: build_result
- name: "Build - R\xE9sultat"
  debug:
    msg: "\u2705 Build termin\xE9 pour {{ project_name }}"

```

### restart.yml

```yaml
- name: "Restart - Arr\xEAt"
  include_tasks: stop.yml
- name: Restart - Pause
  pause:
    seconds: 5
- name: "Restart - D\xE9marrage"
  include_tasks: start.yml

```

### down.yml

```yaml
- name: Down - Suppression des services (Windows)
  win_shell: 'cd "{{ project_path }}"

    docker-compose down {{ ''-v'' if remove_volumes else '''' }}

    '
  when: ansible_os_family == "Windows"
- name: Down - Suppression des services (Linux)
  shell: 'cd "{{ project_path }}"

    docker-compose down {{ ''-v'' if remove_volumes else '''' }}

    '
  when: ansible_os_family != "Windows"
- name: Down - Confirmation
  debug:
    msg: "\U0001F5D1\uFE0F Projet {{ project_name }} supprim\xE9{{ ' (avec volumes)'\
      \ if remove_volumes else '' }}"

```

### status.yml

```yaml
- name: "Status - \xC9tat des services (Windows)"
  win_shell: 'cd "{{ project_path }}"

    echo "=== Services ==="

    docker-compose ps

    echo ""

    echo "=== Images ==="

    docker images | findstr {{ project_name }}

    '
  when: ansible_os_family == "Windows"
  register: status_output
- name: "Status - \xC9tat des services (Linux)"
  shell: 'cd "{{ project_path }}"

    echo "=== Services ==="

    docker-compose ps

    echo ""

    echo "=== Images ==="

    docker images | grep {{ project_name }}

    '
  when: ansible_os_family != "Windows"
  register: status_output
- name: Status - Affichage
  debug:
    var: status_output.stdout_lines

```

### stop.yml

```yaml
- name: "Stop - Arr\xEAt des services (Windows)"
  win_shell: 'cd "{{ project_path }}"

    docker-compose stop

    '
  when: ansible_os_family == "Windows"
- name: "Stop - Arr\xEAt des services (Linux)"
  shell: 'cd "{{ project_path }}"

    docker-compose stop

    '
  when: ansible_os_family != "Windows"
- name: Stop - Confirmation
  debug:
    msg: "\U0001F6D1 Services arr\xEAt\xE9s pour {{ project_name }}"

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
    ├── build.yml
    ├── down.yml
    ├── main.yml
    ├── restart.yml
    ├── start.yml
    ├── status.yml
    └── stop.yml
handlers/
    └── main.yml
defaults/
    └── main.yml
README.md
```