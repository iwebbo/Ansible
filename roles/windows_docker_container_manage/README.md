# Ansible Role: windows_docker_container_manage

Rôle Ansible to manage a container on windows

## General Information

**Author:** A&ECoding
**License:** MIT
**Minimum Ansible Version:** 2.9

**Supported Platforms:**
- Windows
  - Versions: all

## Main Tasks

- Check Docker and Docker Compose
- Stop playbook if Docker/Compose not present
- Show docker commands to parse
- Parse docker run command with simpler approach
- Debug parsed services
- Create docker-compose directory
- Initialize docker-compose.yml file
- Add each service to docker-compose.yml
- Display generated docker-compose.yml
- Show generated compose file
- Check if services are already running
- Stop existing services if running
- Start services with docker-compose (cd to directory first)
- Show docker-compose up results
- Wait for services to initialize
- Check services status
- Display final services status

## Templates

- `docker-compose.yml.j2`

## Role Structure

```
vars/
    └── main.yml
templates/
    └── docker-compose.yml.j2
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