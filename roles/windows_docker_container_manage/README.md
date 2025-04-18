# Ansible Role: windows_docker_container_manage

Rôle Ansible to manage a container on windows

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
docker_images:
- name: mintplexlabs/anythingllm
  container_name: anything-llm
  ports: 3001:3001
  volumes: G:\docker\docker-data\anything-llm:/app/data
  restart_policy: unless-stopped
  docker_run_var: docker run -p 3001:3001 --name mintplexlabs/anythingllm -ti --rm
    -v G:\docker\docker-data\anything-llm:/app/data mintplexlabs/anythingllm:latest

```

## Main Tasks

- Check if Docker has been installed
- Stop playbook if Docker is not present
- Display Docker Image will be Starting
- Check if the container/images is present
- Download if container/images are missing
- Check if container is in running state
- Vérifier si le conteneur LocalAI est en cours d'exécution
- Check if container/images are presents maybe stopped
- Debug
- Start container/image if present and stopped
- Start container/images with docker run

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
