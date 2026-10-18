# Ansible Role: sonarqube_install_server

Rôle Ansible pour installer et configurer **SonarQube** en mode serveur via Docker sur Debian, Ubuntu et RedHat.

## 🎯 Objectif

Ce rôle installe SonarQube comme conteneur Docker avec configuration persistante, prêt à être utilisé pour l'analyse statique de code.

---

## 📋 Prérequis

- **Ansible** >= 2.14
- **Serveur Linux** (Debian, Ubuntu ou RedHat/CentOS)
- **Docker** déjà installé sur le serveur cible
- **Minimum 2 Go de RAM** pour un fonctionnement optimal

---

## 🏗️ Structure du rôle


sonarqube_install_server/
├── tasks/main.yml          # Routage principal par OS
├── tasks/linux/main.yml    # Tâches Debian/Ubuntu
├── tasks/unix/main.yml     # Tâches RedHat/CentOS
├── handlers/main.yml       # Handlers de services
├── defaults/main.yml       # Variables par défaut
├── meta/main.yml           # Métadonnées Galaxy
├── templates/
│   └── sonarqube-compose.yml.j2  # Config Docker Compose
└── README.md               # Documentation


---

## 🔧 Variables

### SonarQube Configuration

| Variable | Type | Valeur par défaut | Description |
|----------|------|-------------------|-------------|
| `sonarqube_image` | string | "sonarqube" | Nom de l'image Docker |
| `sonarqube_version` | string | "9.8.0" | Version de SonarQube |
| `sonarqube_port` | int | 9000 | Port HTTP externe |
| `sonarqube_host` | string | IP du serveur | Adresse accessible |
| `sonarqube_admin_user` | string | "admin" | Utilisateur admin |
| `sonarqube_admin_password` | string | "SonarQubeAdmin123!" | Mot de passe admin |
| `sonarqube_db_type` | string | "h2" | Type de base de données |

### Directories

| Variable | Type | Valeur par défaut | Description |
|----------|------|-------------------|-------------|
| `sonarqube_data_dir` | path | "/var/lib/sonarqube" | Stockage persistant |
| `sonarqube_config_dir` | path | "/etc/sonarqube" | Configuration |

### Docker

| Variable | Type | Valeur par défaut | Description |
|----------|------|-------------------|-------------|
| `sonarqube_docker_network` | string | "sonarqube-network" | Réseau Docker |
| `sonarqube_ready_timeout` | int | 300 | Timeout de démarrage (sec) |

---

## 📝 Exemple d'usage

### Playbook de base

yaml
---
- name: Installer SonarQube serveur
  hosts: sonarqube_servers
  become: yes
  gather_facts: no

  tasks:
    - name: Lancer l'installation de SonarQube
      ansible.builtin.debug:
        msg: "Installation de SonarQube en cours..."


### Playbook complet avec variables

yaml
---
- name: Installer et configurer SonarQube
  hosts: sonarqube_servers
  become: yes
  gather_facts: no

  vars:
    # Variables globales
    sonarqube_admin_password: "ChangementDeMotDePasse!"
    
  tasks:
    - name: Installer SonarQube via Ansible
      ansible.builtin.import_role:
        name: sonarqube_install_server


---

## 🚀 Démarrage rapide

bash
# Via Galaxy
ansible-playbook install-sonarqube.yml --extra-vars "sonarqube_admin_password='MonMotDePasse!123'"

# Ou via role local
ansible-playbook install-sonarqube.yml -e "sonarqube_admin_password='MonMotDePasse!123'"


---

## 📊 Vérification post-installation

bash
# Vérifier le statut du conteneur
docker ps | grep sonarqube

# Vérifier l'accès HTTP
curl http://<IP_SERVEUR>:9000/api/health

# Accéder à SonarQube
# Navigatez vers: http://<IP_SERVEUR>:9000


---

## ⚠️ Notes importantes

1. **Mot de passe admin** : Changez le mot de passe par défaut dès la première connexion
2. **Base de données H2** : Utilise une base mémoire (non persistante) - pour production, configurez PostgreSQL/MySQL
3. **Réseau Docker** : Le réseau est créé automatiquement sur chaque exécution
4. **Volume persistant** : Les données sont sauvegardées dans `/var/lib/sonarqube`

---

## 📄 Licence

MIT License - A&ECoding