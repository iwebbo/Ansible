# Ansible Role: linux_apache2_install_configure

Rôle Ansible to configure Apache2 Reverse Proxy

## General Information

**Author:** A&ECoding
**License:** MIT
**Minimum Ansible Version:** 2.9

**Supported Platforms:**
- Linux
  - Versions: all

## Playbook Example 
```yaml
- name: Playbook deployment GIT Webserver
  hosts: host
  become: yes
  vars:
    apache_configure_firewall: false
    apache_vhosts_port: 8090
    apache_vhosts:
      - name: git
        type: website
        document_root: "/srv/git"
        server_name: git.local
        port: 8080
        custom_directives: |
                # Variables d'environnement pour Git
                SetEnv GIT_PROJECT_ROOT /srv/git
                SetEnv GIT_HTTP_EXPORT_ALL
                
                # Configuration FastCGI pour Git
                ScriptAlias /git/ /usr/lib/git-core/git-http-backend/
                
                <Directory "/usr/lib/git-core">
                    Options +ExecCGI +FollowSymLinks
                    Require all granted
                </Directory>
                
                <LocationMatch "^/git/.*$">
                    Options +ExecCGI
                    Require all granted
                    
                    # Variables CGI
                    SetEnv GIT_PROJECT_ROOT /srv/git
                    SetEnv GIT_HTTP_EXPORT_ALL
                </LocationMatch>
                
                # Accès aux repos
                <Directory /srv/git>
                    Options Indexes FollowSymLinks
                    AllowOverride None
                    Require all granted
                </Directory>
        state: enabled
  roles:
    - linux_apache2_install_configure
```

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

- `vhost-website.conf.j2`
- `vhost-proxy.conf.j2`
- `apache2.conf.j2`
- `reverseproxy.app.j2`
- `index.html.j2`

## Role Structure

```
vars/
    └── main.yml
tasks/
    ├── configure.yml
    ├── install.yml
    ├── main.yml
    └── vhosts.yml
defaults/
    └── main.yml
templates/
    ├── apache2.conf.j2
    ├── index.html.j2
    ├── reverseproxy.app.j2
    ├── vhost-proxy.conf.j2
    └── vhost-website.conf.j2
tests/
    ├── inventory
    └── test.yml
meta/
    └── main.yml
handlers/
    └── main.yml
README.md
```debian@vm-debian-dev-01:/tmp/AnsibleRoleDoc$ cd AnsibleRoleDoc/^C
debian@vm-debian-dev-01:/tmp/AnsibleRoleDoc$ cd ..
debian@vm-debian-dev-01:/tmp$ rm -rf Ansible
debian@vm-debian-dev-01:/tmp$ git clone https://github.com/iwebbo/Ansible.git
Cloning into 'Ansible'...
remote: Enumerating objects: 1259, done.
remote: Counting objects: 100% (489/489), done.
remote: Compressing objects: 100% (292/292), done.
remote: Total 1259 (delta 124), reused 417 (delta 84), pack-reused 770 (from 1)
Receiving objects: 100% (1259/1259), 489.87 KiB | 7.90 MiB/s, done.
Resolving deltas: 100% (388/388), done.
debian@vm-debian-dev-01:/tmp$ cd AnsibleRoleDoc/
debian@vm-debian-dev-01:/tmp/AnsibleRoleDoc$ python3 ansible-role-generator.py  -f markdown -o README.md ../Ansible/roles/linux_apache2_install_configure
Documentation generated in README.md
debian@vm-debian-dev-01:/tmp/AnsibleRoleDoc$ cat README.md
# Ansible Role: linux_apache2_install_configure

Rôle Ansible to configure Apache2 & Deploy vhost for Proxy or Website

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

- `vhost-website.conf.j2`
- `vhost-proxy.conf.j2`
- `apache2.conf.j2`
- `reverseproxy.app.j2`
- `index.html.j2`

## Role Structure

```
vars/
    └── main.yml
tasks/
    ├── configure.yml
    ├── install.yml
    ├── main.yml
    └── vhosts.yml
defaults/
    └── main.yml
templates/
    ├── apache2.conf.j2
    ├── index.html.j2
    ├── reverseproxy.app.j2
    ├── vhost-proxy.conf.j2
    └── vhost-website.conf.j2
tests/
    ├── inventory
    └── test.yml
meta/
    └── main.yml
handlers/
    └── main.yml
