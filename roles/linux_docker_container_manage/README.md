# Ansible Role: linux_docker_container_manage

Rôle Ansible to manage container image on linux

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
- name: eclipse-mosquitto
  container_name: eclipse-mosquitto
  ports: 1883:1883
  volumes: $PWD/mosquitto/config:/mosquitto/config
  config: null
  restart_policy: unless-stopped
  docker_run_var: docker run -d -it -p 1883:1883 eclipse-mosquitto:latest mosquitto
    -c /mosquitto-no-auth.conf
- name: m4dm4rtig4n/myelectricaldata:latest
  container_name: myelectricaldata:latest
  ports: 5000:5000
  restart_policy: unless-stopped
  docker_run_var: docker run -d -it --restart=unless-stopped -v $(pwd)/myelectricaldata/data:/data
    -p 5000:5000 -e TZ='Europe/Paris' m4dm4rtig4n/myelectricaldata:latest

```

## Main Tasks

- Vérifier si Docker est installé
- Stop playbook si Docker n'est pas installé
- Afficher les images Docker à traiter
- Vérifier si l'image Docker est présente
- Télécharger l'image si elle est absente
- Vérifier si le conteneur est en cours d'exécution
- Vérifier si le conteneur existe (arrêté ou en cours)
- Afficher le statut des conteneurs existants
- Démarrer le conteneur s'il est arrêté
- Lancer le conteneur si absent

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