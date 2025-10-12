# Ansible Role: linux_apache2_install_configure

Rôle Ansible to configure Apache2 Reverse Proxy

## General Information

**Author:** A&ECoding
**License:** MIT
**Minimum Ansible Version:** 2.9

**Supported Platforms:**
- Windows
  - Versions: all

## Default Variables

```yaml
apache_configure_firewall: false
apache_remove_default_site: true
apache_packages_debian:
- apache2
- apache2-utils
apache_packages_redhat:
- httpd
- httpd-tools
apache_service_name: '{{ ''apache2'' if ansible_os_family == ''Debian'' else ''httpd''
  }}'
apache_user: '{{ ''www-data'' if ansible_os_family == ''Debian'' else ''apache'' }}'
apache_group: '{{ ''www-data'' if ansible_os_family == ''Debian'' else ''apache''
  }}'
apache_config_file: '{{ ''/etc/apache2/apache2.conf'' if ansible_os_family == ''Debian''
  else ''/etc/httpd/conf/httpd.conf'' }}'
apache_vhost_dir: '{{ ''/etc/apache2/sites-available'' if ansible_os_family == ''Debian''
  else ''/etc/httpd/conf.d'' }}'
apache_log_dir: '{{ ''/var/log/apache2'' if ansible_os_family == ''Debian'' else ''/var/log/httpd''
  }}'
apache_default_site_path: /etc/apache2/sites-enabled/000-default.conf
apache_listen_port: 80
apache_listen_port_ssl: 443
apache_vhosts_port: []
apache_modules:
- rewrite
- ssl
- headers
- deflate
- proxy
- proxy_http
- proxy_balancer
- lbmethod_byrequests
apache_vhosts_defaults:
  type: website
  port: 80
  document_root: /var/www/html
  error_log_level: warn
  access_log_format: combined
  create_index: true
  state: enabled
apache_vhosts: []

```

## Main Tasks

- Include installation tasks
- Include configuration tasks
- Include vhosts management tasks

## Other Tasks

### install.yml

```yaml
- name: Update package cache (Debian/Ubuntu)
  apt:
    update_cache: true
    cache_valid_time: 3600
  when: ansible_os_family == "Debian"
  tags: install
- name: Update package cache (RedHat/CentOS)
  yum:
    update_cache: true
  when: ansible_os_family == "RedHat"
  tags: install
- name: Install Apache2 (Debian/Ubuntu)
  apt:
    name: '{{ apache_packages_debian }}'
    state: present
  when: ansible_os_family == "Debian"
  notify: restart apache
  tags: install
- name: Install Apache2 (RedHat/CentOS)
  yum:
    name: '{{ apache_packages_redhat }}'
    state: present
  when: ansible_os_family == "RedHat"
  notify: restart apache
  tags: install
- name: Enable Apache2 service
  systemd:
    name: '{{ apache_service_name }}'
    enabled: true
    state: started
  tags: install
- name: Create Apache2 directories
  file:
    path: '{{ item }}'
    state: directory
    owner: '{{ apache_user }}'
    group: '{{ apache_group }}'
    mode: '0755'
  loop:
  - '{{ apache_vhost_dir }}'
  - '{{ apache_log_dir }}'
  - /var/www/html
  tags: install

```

### configure.yml

```yaml
- name: Deploy Apache2 main configuration
  template:
    src: apache2.conf.j2
    dest: '{{ apache_config_file }}'
    owner: root
    group: root
    mode: '0644'
    backup: true
  notify: restart apache
  tags: configure
- name: Enable required Apache modules
  apache2_module:
    name: '{{ item }}'
    state: present
  loop: '{{ apache_modules }}'
  notify: restart apache
  when: ansible_os_family == "Debian"
  tags: configure
- name: Configure firewall for HTTP (Debian/Ubuntu)
  ufw:
    rule: allow
    port: '{{ item }}'
    proto: tcp
  loop:
  - '80'
  - '443'
  when:
  - ansible_os_family == "Debian"
  - apache_configure_firewall | default(false)
  tags: configure
- name: Configure firewall for HTTP (RedHat/CentOS)
  firewalld:
    service: '{{ item }}'
    permanent: true
    state: enabled
    immediate: true
  loop:
  - http
  - https
  when:
  - ansible_os_family == "RedHat"
  - apache_configure_firewall | default(false)
  tags: configure
- name: Remove default site (Debian/Ubuntu)
  file:
    path: '{{ apache_default_site_path }}'
    state: absent
  when:
  - ansible_os_family == "Debian"
  - apache_remove_default_site | default(true)
  notify: reload apache
  tags: configure

```

### vhosts.yml

```yaml
- name: Create document root directories for website vhosts
  file:
    path: '{{ item.document_root | default(''/var/www/'' + item.name) }}'
    state: directory
    owner: '{{ apache_user }}'
    group: '{{ apache_group }}'
    mode: '0755'
  loop: '{{ apache_vhosts }}'
  when:
  - apache_vhosts is defined
  - item.type | default('website') == 'website'
  tags: vhosts
- name: Create log directories for all vhosts
  file:
    path: '{{ apache_log_dir }}/{{ item.name }}'
    state: directory
    owner: '{{ apache_user }}'
    group: '{{ apache_group }}'
    mode: '0755'
  loop: '{{ apache_vhosts }}'
  when: apache_vhosts is defined
  tags: vhosts
- name: Deploy website virtual host configurations
  template:
    src: vhost-website.conf.j2
    dest: '{{ apache_vhost_dir }}/{{ item.name }}.conf'
    owner: root
    group: root
    mode: '0644'
    backup: true
  loop: '{{ apache_vhosts }}'
  when:
  - apache_vhosts is defined
  - item.type | default('website') == 'website'
  notify: reload apache
  tags: vhosts
- name: Deploy reverse proxy virtual host configurations
  template:
    src: vhost-proxy.conf.j2
    dest: '{{ apache_vhost_dir }}/{{ item.name }}.conf'
    owner: root
    group: root
    mode: '0644'
    backup: true
  loop: '{{ apache_vhosts }}'
  when:
  - apache_vhosts is defined
  - item.type | default('website') == 'proxy'
  notify: reload apache
  tags: vhosts
- name: Enable virtual hosts (Debian/Ubuntu)
  command: a2ensite {{ item.name }}
  loop: '{{ apache_vhosts }}'
  when:
  - apache_vhosts is defined
  - ansible_os_family == "Debian"
  - item.state | default('enabled') == 'enabled'
  notify: reload apache
  register: a2ensite_result
  changed_when: '''Enabling site'' in a2ensite_result.stdout'
  tags: vhosts
- name: Disable virtual hosts (Debian/Ubuntu)
  command: a2dissite {{ item.name }}
  loop: '{{ apache_vhosts }}'
  when:
  - apache_vhosts is defined
  - ansible_os_family == "Debian"
  - item.state | default('enabled') == 'disabled'
  notify: reload apache
  register: a2dissite_result
  changed_when: '''Disabling site'' in a2dissite_result.stdout'
  tags: vhosts
- name: Create simple index.html for website vhosts
  template:
    src: index.html.j2
    dest: '{{ item.document_root | default(''/var/www/'' + item.name) }}/index.html'
    owner: '{{ apache_user }}'
    group: '{{ apache_group }}'
    mode: '0644'
  loop: '{{ apache_vhosts }}'
  when:
  - apache_vhosts is defined
  - item.type | default('website') == 'website'
  - item.create_index | default(true)
  tags: vhosts
- name: Remove disabled vhost configurations
  file:
    path: '{{ apache_vhost_dir }}/{{ item.name }}.conf'
    state: absent
  loop: '{{ apache_vhosts }}'
  when:
  - apache_vhosts is defined
  - item.state | default('enabled') == 'absent'
  notify: reload apache
  tags: vhosts

```

## Handlers

```yaml
- name: restart apache
  systemd:
    name: '{{ apache_service_name }}'
    state: restarted
- name: reload apache
  systemd:
    name: '{{ apache_service_name }}'
    state: reloaded
- name: start apache
  systemd:
    name: '{{ apache_service_name }}'
    state: started
- name: stop apache
  systemd:
    name: '{{ apache_service_name }}'
    state: stopped

```

## Templates

- `reverseproxy.app.j2`
- `vhost-website.conf.j2`
- `apache2.conf.j2`
- `vhost-proxy.conf.j2`
- `index.html.j2`

## Playbook Exemple
```yaml
---
# playbook-apache.yml
- name: Deploy Apache2 with Virtual Hosts (Website + Reverse Proxy)
  hosts: web_servers
  become: yes
  vars:
    apache_configure_firewall: true
    apache_vhosts:
      # VHost de type "website" - Site web classique
      - name: monsite-web
        type: website
        server_name: www.monentreprise.com
        server_alias: monentreprise.com
        document_root: /var/www/monsite-web
        port: 80
        ssl:
          enabled: true
          certificate: /etc/ssl/certs/monentreprise.crt
          certificate_key: /etc/ssl/private/monentreprise.key
        redirect_to_ssl: true
        custom_directives: |
          # Configuration spécifique pour le site web
          DirectoryIndex index.html index.php
          <LocationMatch "^/admin">
              Require ip 192.168.1.0/24
          </LocationMatch>
        state: enabled
        
      # VHost de type "proxy" - API Backend simple
      - name: api-service
        type: proxy
        server_name: api.monentreprise.com
        port: 80
        proxy:
          backend_url: http://127.0.0.1:8080
          preserve_host: true
          timeout: 60
          retry: 3
          location: /
          additional_headers:
            X-Custom-Header: "API-Gateway"
            X-Service-Name: "{{ ansible_hostname }}"
        state: enabled
        
      # VHost de type "proxy" - Load Balancer avec plusieurs backends
      - name: app-cluster
        type: proxy
        server_name: app.monentreprise.com
        port: 443
        ssl:
          enabled: true
          certificate: /etc/ssl/certs/app-cluster.crt
          certificate_key: /etc/ssl/private/app-cluster.key
        proxy:
          backends:
            - url: http://10.0.1.10:3000
              weight: 3
              status: +H
            - url: http://10.0.1.11:3000
              weight: 2
            - url: http://10.0.1.12:3000
              weight: 1
          lb_method: byrequests
          preserve_host: true
          timeout: 45
          health_check:
            uri: /health
            method: GET
            interval: 30
          admin_ips:
            - 192.168.1.100
            - 10.0.1.0/24
          error_handling:
            override: On
            custom_pages:
              502: /errors/502.html
              503: /errors/503.html
        state: enabled
        
      # VHost de type "proxy" - Backend HTTPS avec certificat
      - name: secure-api
        type: proxy
        server_name: secure.monentreprise.com
        port: 443
        ssl:
          enabled: true
          certificate: /etc/ssl/certs/secure.crt
          certificate_key: /etc/ssl/private/secure.key
        proxy:
          backend_url: https://internal-api.domain.local:8443
          backend_ssl:
            enabled: true
            verify: false  # Pour certificats auto-signés
          preserve_host: false
          timeout: 30
          additional_headers:
            X-Forwarded-For: "%h"
            X-Real-IP: "%h"
        custom_directives: |
          # Sécurité renforcée pour API sensible
          Header always set Strict-Transport-Security "max-age=31536000; includeSubDomains"
          Header always set X-Frame-Options DENY
        state: enabled
        
  roles:
    - apache2

# Exemples d'exécution avec tags simplifiés :
# ansible-playbook -i inventory playbook-apache.yml                # Installation complète
# ansible-playbook -i inventory playbook-apache.yml --tags "install"   # Installation seulement
# ansible-playbook -i inventory playbook-apache.yml --tags "configure" # Configuration seulement
# ansible-playbook -i inventory playbook-apache.yml --tags "vhosts"    # Gestion vhosts seulement

```

## Role Structure

```
vars/
    └── main.yml
templates/
    ├── apache2.conf.j2
    ├── index.html.j2
    ├── reverseproxy.app.j2
    ├── vhost-proxy.conf.j2
    └── vhost-website.conf.j2
meta/
    └── main.yml
tests/
    ├── inventory
    └── test.yml
tasks/
    ├── configure.yml
    ├── install.yml
    ├── main.yml
    └── vhosts.yml
handlers/
    └── main.yml
defaults/
    └── main.yml
README.md
```