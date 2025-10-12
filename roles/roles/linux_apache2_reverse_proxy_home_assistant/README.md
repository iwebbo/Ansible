# Ansible Role: linux_apache2_reverse_proxy_home_assistant

your role description

## General Information

**Author:** your name
**License:** license (GPL-2.0-or-later, MIT, etc)
**Minimum Ansible Version:** 2.1

## Variables

### main

```yaml
apache_proxy_config: /etc/apache2/conf-available/reverseproxy.app
apache_vhost_file: /etc/apache2/sites-available/reverse_proxy_http.conf
apache_vhost_link: /etc/apache2/sites-enabled/reverse_proxy_http.conf
apache_vhost_ssl_file: /etc/apache2/sites-available/reverse_proxy_https.conf
apache_vhost_ssl_link: /etc/apache2/sites-enabled/reverse_proxy_https.conf
ssl_cert_file: /etc/apache2/certificates/{{ server_name }}.crt
ssl_cert_key_file: /etc/apache2/certificates/{{ server_name }}.key
ssl_cert_CA_file: /etc/apache2/certificates/myCA.crt
proxy_pass: http://192.168.1.16:8123/
rewrite_home: ws://192.168.1.16:8123/$1
server_name: reverseproxy.local

```

## Main Tasks

- Check installation of Apache2
- Activate module proxy & SSL & Rewrite
- Deployment of configuration file (reverseproxy.app)
- Deployment Vhost 8080
- Deployment Vhost 4443 (SSL)
- Add port 8080 in apache (ports.conf) in case with needed
- Add port 4443 in apache (ports.conf)
- Activate with a2ensite (SSL)

## Handlers

```yaml
- name: Restart Apache
  systemd:
    name: apache2
    state: restarted
    enabled: true
- name: Reload Apache
  systemd:
    name: apache2
    state: reloaded
    enabled: true

```

## Templates

- `vhost.conf.j2`
- `reverseproxy.app.j2`
- `vhost_ssl.conf.j2`

## Role Structure

```
vars/
    └── main.yml
templates/
    ├── reverseproxy.app.j2
    ├── vhost.conf.j2
    └── vhost_ssl.conf.j2
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