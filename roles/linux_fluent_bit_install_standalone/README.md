# Ansible Role: Fluent Bit Standalone

Installation et configuration de **Fluent Bit en mode standalone** pour collecter et envoyer des logs vers OpenSearch/Elasticsearch.

## 🎯 Objectif

Ce rôle installe Fluent Bit comme **service systemd** sur vos serveurs et configure les **inputs**, **parsers**, **filters** et **outputs** pour collecter vos logs.

```
┌─────────────────────────────────────────────┐
│  FLUENT BIT (service systemd)               │
│  Installé sur le serveur                   │
└─────────────────────────────────────────────┘
                    ↓
┌─────────────────────────────────────────────┐
│  INPUTS - Sources monitorées                │
│  ✓ Docker containers (via socket)          │
│  ✓ Kubernetes pods (fichiers logs)         │
│  ✓ Services systemd                        │
│  ✓ Apache access/error logs                │
│  ✓ Nginx access/error logs                 │
│  ✓ Fichiers logs personnalisés             │
└─────────────────────────────────────────────┘
                    ↓
┌─────────────────────────────────────────────┐
│  PARSERS + FILTERS                          │
│  ✓ Parse JSON, Apache, Nginx, etc.         │
│  ✓ Enrichissement (hostname, cluster)      │
│  ✓ Champs personnalisés                    │
└─────────────────────────────────────────────┘
                    ↓
┌─────────────────────────────────────────────┐
│  OUTPUT - Destination                       │
│  ✓ OpenSearch / Elasticsearch              │
│  ✓ Loki (support futur)                    │
└─────────────────────────────────────────────┘
```

## 📋 Prérequis

- Ansible >= 2.10
- OS supportés: Ubuntu 20.04+, Debian 11+, RHEL 8+, CentOS 8+, Fedora 37+
- Accès sudo sur les serveurs cibles
- OpenSearch/Elasticsearch accessible

## 🚀 Installation rapide

### 1. Installer le rôle

```bash
# Via Ansible Galaxy
ansible-galaxy install your_namespace.fluent_bit_standalone

# Ou clone Git
git clone https://github.com/your-org/fluent-bit-standalone.git roles/fluent-bit-standalone
```

### 2. Créer un playbook

```yaml
# playbook.yml
---
- name: Install Fluent Bit on Docker hosts
  hosts: docker_servers
  become: yes
  
  roles:
    - role: fluent-bit-standalone
      vars:
        # Inputs
        fluent_bit_inputs:
          docker: true
          systemd: true
        
        # OpenSearch
        fluent_bit_opensearch_host: "opensearch.example.com"
        fluent_bit_opensearch_port: 9200
        fluent_bit_opensearch_user: "fluent-bit"
        fluent_bit_opensearch_password: "{{ vault_password }}"
        fluent_bit_opensearch_index: "docker-logs"
```

### 3. Déployer

```bash
ansible-playbook -i inventory.ini playbook.yml --ask-vault-pass
```

## 📖 Variables principales

### Configuration globale

| Variable | Défaut | Description |
|----------|--------|-------------|
| `fluent_bit_version` | `3.0.0` | Version de Fluent Bit |
| `fluent_bit_log_level` | `info` | Niveau de log (error, warning, info, debug) |
| `fluent_bit_flush_seconds` | `5` | Intervalle de flush (secondes) |
| `fluent_bit_http_server` | `true` | Activer HTTP monitoring |
| `fluent_bit_http_port` | `2020` | Port HTTP monitoring |

### Inputs (sources de logs)

```yaml
fluent_bit_inputs:
  docker: false          # Containers Docker
  kubernetes: false      # Pods Kubernetes  
  systemd: false         # Services systemd
  apache: false          # Apache logs
  nginx: false           # Nginx logs
  custom_files: false    # Fichiers custom
```

### Output OpenSearch/Elasticsearch

| Variable | Défaut | Description |
|----------|--------|-------------|
| `fluent_bit_opensearch_host` | `opensearch.example.com` | Hostname |
| `fluent_bit_opensearch_port` | `9200` | Port |
| `fluent_bit_opensearch_user` | `admin` | User |
| `fluent_bit_opensearch_password` | `admin` | Password |
| `fluent_bit_opensearch_index` | `fluent-bit` | Index de base |
| `fluent_bit_opensearch_tls` | `true` | Activer TLS |
| `fluent_bit_opensearch_logstash_format` | `true` | Format Logstash (index-YYYY.MM.DD) |

### Filters

| Variable | Défaut | Description |
|----------|--------|-------------|
| `fluent_bit_filter_add_hostname` | `true` | Ajouter hostname |
| `fluent_bit_filter_add_cluster` | `false` | Ajouter cluster name |
| `fluent_bit_cluster_name` | `production` | Nom du cluster |

## 💡 Exemples d'utilisation

### Exemple 1: Monitoring Docker

```yaml
- hosts: docker_hosts
  roles:
    - fluent-bit-standalone
      vars:
        fluent_bit_inputs:
          docker: true
          systemd: true
        
        fluent_bit_systemd_services:
          - docker
          - sshd
        
        fluent_bit_opensearch_host: "opensearch.prod.local"
        fluent_bit_opensearch_index: "docker-logs"
        fluent_bit_opensearch_password: "{{ vault_password }}"
```

**Résultat**: Logs de tous les containers Docker → `docker-logs-2025.02.14`

### Exemple 2: Monitoring Kubernetes nodes

```yaml
- hosts: k8s_worker_nodes
  roles:
    - fluent-bit-standalone
      vars:
        fluent_bit_inputs:
          kubernetes: true
          systemd: true
        
        fluent_bit_systemd_services:
          - kubelet
          - containerd
        
        fluent_bit_filter_add_cluster: true
        fluent_bit_cluster_name: "prod-k8s-01"
        
        fluent_bit_opensearch_host: "opensearch.k8s.local"
        fluent_bit_opensearch_index: "k8s-logs"
```

**Résultat**: Logs des pods K8s avec metadata (namespace, pod, labels) → `k8s-logs-2025.02.14`

### Exemple 3: Web servers (Apache + custom apps)

```yaml
- hosts: web_servers
  roles:
    - fluent-bit-standalone
      vars:
        fluent_bit_inputs:
          apache: true
          custom_files: true
        
        fluent_bit_apache_access_log: "/var/log/apache2/access.log"
        fluent_bit_apache_error_log: "/var/log/apache2/error.log"
        
        fluent_bit_custom_files:
          - path: /var/log/myapp/app.log
            tag: myapp
            parser: json
          - path: /var/log/api/*.log
            tag: api
            parser: json
            multiline: true
            multiline_parser: multiline-java
        
        fluent_bit_opensearch_host: "opensearch.web.local"
        fluent_bit_opensearch_index: "web-logs"
```

**Résultat**: Apache logs + logs applicatifs JSON parsés → `web-logs-2025.02.14`

### Exemple 4: Tout en un (hybride)

```yaml
- hosts: hybrid_servers
  roles:
    - fluent-bit-standalone
      vars:
        fluent_bit_inputs:
          docker: true
          kubernetes: true
          systemd: true
          nginx: true
        
        fluent_bit_opensearch_host: "opensearch.central.local"
        fluent_bit_opensearch_index: "all-logs"
        
        # Enrichissement
        fluent_bit_filter_custom_records:
          environment: production
          datacenter: eu-west-1
```

## 📂 Structure des index OpenSearch

Les logs sont automatiquement indexés par jour avec le format Logstash:

```
docker-logs-2025.02.14
k8s-logs-2025.02.14
web-logs-2025.02.14
```

Chaque log contient au minimum:
- `@timestamp`: Timestamp du log
- `hostname`: Hostname du serveur
- `tag`: Source (docker.*, kube.*, systemd.*, etc.)
- Métadonnées spécifiques à la source

## 🔍 Vérification post-installation

```bash
# Status du service
systemctl status fluent-bit

# Logs en temps réel
journalctl -u fluent-bit -f

# Health check HTTP
curl http://localhost:2020/api/v1/health

# Metrics
curl http://localhost:2020/api/v1/metrics

# Vérifier dans OpenSearch
curl -u user:pass https://opensearch:9200/_cat/indices?v | grep fluent-bit
```

## 🛠️ Configuration avancée

### Fichiers custom avec multiline

```yaml
fluent_bit_custom_files:
  - path: /var/log/java-app/app.log
    tag: javaapp
    parser: json
    multiline: true
    multiline_parser: multiline-java  # Stack traces Java
  
  - path: /var/log/python-app/app.log
    tag: pythonapp
    parser: json
    multiline: true
    multiline_parser: multiline-python  # Tracebacks Python
```

### Parsers personnalisés

```yaml
fluent_bit_custom_parsers:
  - name: myapp
    format: regex
    regex: '^(?<time>[^ ]+) (?<level>[^ ]+) (?<message>.*)$'
    time_key: time
    time_format: "%Y-%m-%dT%H:%M:%S.%L"
```

### Performance tuning

```yaml
# Pour haute charge
fluent_bit_flush_seconds: 3
fluent_bit_mem_buf_limit: "20MB"
fluent_bit_retry_limit: 10

# Pour économiser ressources
fluent_bit_flush_seconds: 10
fluent_bit_mem_buf_limit: "5MB"
```

## 🔐 Sécurité

### Utiliser Ansible Vault pour les secrets

```bash
# Créer vault
ansible-vault create group_vars/all/vault.yml

# Contenu
vault_opensearch_password: "secure-password"
vault_opensearch_user: "fluent-bit"
```

Dans le playbook:
```yaml
fluent_bit_opensearch_user: "{{ vault_opensearch_user }}"
fluent_bit_opensearch_password: "{{ vault_opensearch_password }}"
```

### TLS avec certificats

```yaml
fluent_bit_opensearch_tls: true
fluent_bit_opensearch_tls_verify: true
fluent_bit_opensearch_tls_ca_file: "/etc/ssl/certs/ca.crt"
fluent_bit_opensearch_tls_crt_file: "/etc/ssl/certs/client.crt"
fluent_bit_opensearch_tls_key_file: "/etc/ssl/private/client.key"
```

## 🐛 Troubleshooting

Voir [TROUBLESHOOTING.md](TROUBLESHOOTING.md) pour:
- Problèmes de permissions (Docker, systemd)
- Connexion OpenSearch
- Parsing de logs
- Performance
- Debug avancé

## 📝 Licence

MIT

## 👥 Auteur

DevOps Team
