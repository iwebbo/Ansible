# Ansible Role: windows_iis_install_configure

Rôle Ansible pour installer et configurer IIS sur Windows

## General Information

**Author:** A&ECoding
**License:** MIT
**Minimum Ansible Version:** 2.9

**Supported Platforms:**
- Windows
  - Versions: all

## Default Variables

```yaml
iis_configure_firewall: false
iis_remove_default_site: true
iis_features:
- Web-Server
- Web-WebServer
- Web-Common-Http
- Web-Default-Doc
- Web-Dir-Browsing
- Web-Http-Errors
- Web-Static-Content
- Web-Health
- Web-Http-Logging
- Web-Log-Libraries
- Web-Request-Monitor
- Web-Http-Tracing
- Web-Performance
- Web-Stat-Compression
- Web-Security
- Web-Filtering
- Web-App-Dev
- Web-Net-Ext45
- Web-Asp-Net45
- Web-ISAPI-Ext
- Web-ISAPI-Filter
- Web-Mgmt-Tools
- Web-Mgmt-Console
- Web-Mgmt-Service
- Web-Scripting-Tools
- NET-Framework-45-Features
- NET-Framework-45-Core
- NET-Framework-45-ASPNET
iis_listen_port: 80
iis_listen_port_ssl: 443
iis_log_dir: C:\inetpub\logs\LogFiles
iis_sites_defaults:
  protocol: http
  port: 80
  physical_path: C:\inetpub\wwwroot
  log_format: W3C
  log_period: Daily
  state: present
  default_doc:
  - index.html
  - Default.htm
  - Default.asp
  - index.php
  - default.aspx
  auth:
    anonymous: true
    basic: false
    windows: false
iis_sites: []

```

## Variables

### main

```yaml
artifactory_base_url: ''
artifactory_username: ''
artifactory_password: ''
artifactory_repository: ''
package_filename: ''
artifactory_package_url: '{{ artifactory_base_url }}/{{ artifactory_repository }}/{{
  package_filename }}'
deploy_temp_dir: C:\temp\deploy
deploy_destination_path: C:\inetpub\wwwroot
force_download: true
download_timeout: 300
delete_package_after_extract: false
cleanup_temp_files: true
create_backup: true
backup_path: C:\backup
site_name: ''
site_port: 80
site_ip: '*'
site_hostname: ''
app_pool_name: ''
app_pool_identity: ApplicationPoolIdentity
app_pool_username: ''
app_pool_password: ''
dotnet_version: v4.0
enable_32bit: false
msi_arguments: /quiet
perform_health_check: true
health_check_path: /
health_check_retries: 5
health_check_delay: 30
expected_status_code:
- 200
- 301
- 302
config_files: []
config_transformations: []
ssl_bindings: []
directory_permissions: []

```

## Main Tasks

- Include installation tasks
- Include configuration tasks
- Include sites management tasks

## Other Tasks

### install.yml

```yaml
- name: Install IIS features
  win_feature:
    name: '{{ iis_features }}'
    state: present
    include_management_tools: true
  register: iis_install
  tags: install
- name: Reboot if needed after feature installation
  win_reboot:
    test_command: exit 0
  when: iis_install.reboot_required
  tags: install
- name: Ensure IIS service is started and configured for automatic startup
  win_service:
    name: W3SVC
    state: started
    start_mode: auto
  tags: install
- name: Configure Windows Firewall for HTTP (if enabled)
  win_firewall_rule:
    name: IIS - HTTP (TCP-In)
    localport: 80
    action: allow
    direction: in
    protocol: tcp
    state: present
    enabled: true
  when: iis_configure_firewall | default(false)
  tags: install
- name: Configure Windows Firewall for HTTPS (if enabled)
  win_firewall_rule:
    name: IIS - HTTPS (TCP-In)
    localport: 443
    action: allow
    direction: in
    protocol: tcp
    state: present
    enabled: true
  when: iis_configure_firewall | default(false)
  tags: install
- name: Create IIS log directories
  win_file:
    path: '{{ iis_log_dir }}'
    state: directory
  tags: install

```

### configure.yml

```yaml
- name: Configure IIS global settings
  win_shell: "Import-Module WebAdministration\n\n# Configure default documents\n$DefaultDocs\
    \ = @('index.html', 'Default.htm', 'Default.asp', 'index.php', 'default.aspx')\n\
    foreach ($doc in $DefaultDocs) {\n  if (-not (Get-WebConfiguration -Filter \"\
    //defaultDocument/files/add[@value='$doc']\" -PSPath \"IIS:\\Sites\\\")) {\n \
    \   Add-WebConfiguration -Filter \"//defaultDocument/files\" -PSPath \"IIS:\\\
    Sites\\\" -Value @{value=\"$doc\"}\n  }\n}\n\n# Configure global logging settings\n\
    Set-WebConfigurationProperty -pspath 'MACHINE/WEBROOT/APPHOST' -filter 'system.applicationHost/sites/siteDefaults/logFile'\
    \ -name 'directory' -value '{{ iis_log_dir | regex_replace('\\\\\\\\', '\\\\\\\
    \\\\\\\\\\') }}'\nSet-WebConfigurationProperty -pspath 'MACHINE/WEBROOT/APPHOST'\
    \ -filter 'system.applicationHost/sites/siteDefaults/logFile' -name 'period' -value\
    \ 'Daily'\nSet-WebConfigurationProperty -pspath 'MACHINE/WEBROOT/APPHOST' -filter\
    \ 'system.applicationHost/sites/siteDefaults/logFile' -name 'logFormat' -value\
    \ 'W3C'\n\n# Configure request filtering - security settings\nSet-WebConfigurationProperty\
    \ -pspath 'MACHINE/WEBROOT/APPHOST'  -filter \"system.webServer/security/requestFiltering/requestLimits\"\
    \ -name \"maxAllowedContentLength\" -value 30000000\nSet-WebConfigurationProperty\
    \ -pspath 'MACHINE/WEBROOT/APPHOST'  -filter \"system.webServer/security/requestFiltering/requestLimits\"\
    \ -name \"maxUrl\" -value 4096\nSet-WebConfigurationProperty -pspath 'MACHINE/WEBROOT/APPHOST'\
    \  -filter \"system.webServer/security/requestFiltering/requestLimits\" -name\
    \ \"maxQueryString\" -value 2048\n\n# Hide Server version information\nSet-WebConfigurationProperty\
    \ -pspath 'MACHINE/WEBROOT/APPHOST'  -filter \"system.webServer/security/requestFiltering\"\
    \ -name \"removeServerHeader\" -value \"True\"\n\n# Configure response headers\n\
    Set-WebConfigurationProperty -pspath 'MACHINE/WEBROOT/APPHOST' -filter \"system.webServer/httpProtocol/customHeaders\"\
    \ -name \".\" -value @{name=\"X-Content-Type-Options\"; value=\"nosniff\"}\nSet-WebConfigurationProperty\
    \ -pspath 'MACHINE/WEBROOT/APPHOST' -filter \"system.webServer/httpProtocol/customHeaders\"\
    \ -name \".\" -value @{name=\"X-XSS-Protection\"; value=\"1; mode=block\"}\n"
  notify: apply iis config
  tags: configure
- name: Remove default website (if configured)
  win_iis_website:
    name: Default Web Site
    state: absent
  when: iis_remove_default_site | default(true)
  notify: apply iis config
  tags: configure
- name: Deploy IIS configuration PowerShell script
  win_template:
    src: iis_config.ps1.j2
    dest: C:\Temp\iis_config.ps1
  tags: configure
- name: Execute IIS configuration script
  win_shell: 'powershell.exe -ExecutionPolicy Bypass -File "C:\\Temp\\iis_config.ps1"

    '
  register: iis_config_result
  changed_when: iis_config_result.stdout is not search('No changes needed')
  notify: apply iis config
  tags: configure

```

### sites.yml

```yaml
- name: Create site directories
  win_file:
    path: '{{ item.physical_path | default(''C:\inetpub\wwwroot\'' + item.name) }}'
    state: directory
  loop: '{{ iis_sites }}'
  when:
  - iis_sites is defined
  - item.state | default('present') == 'present'
  tags: sites
- name: Create log directories for sites
  win_file:
    path: '{{ item.log_directory | default(iis_log_dir + ''\'' + item.name) }}'
    state: directory
  loop: '{{ iis_sites }}'
  when:
  - iis_sites is defined
  - item.state | default('present') == 'present'
  tags: sites
- name: Configure IIS application pools for sites
  win_iis_webapppool:
    name: '{{ item.app_pool | default(item.name + ''_pool'') }}'
    state: present
    attributes:
      managedRuntimeVersion: '{{ item.managed_runtime | default(''v4.0'') }}'
      managedPipelineMode: '{{ item.pipeline_mode | default(''Integrated'') }}'
      processModel.identityType: '{{ item.identity_type | default(''ApplicationPoolIdentity'')
        }}'
      processModel.idleTimeout: '{{ item.idle_timeout | default(''00:20:00'') }}'
      recycling.periodicRestart.time: '{{ item.recycle_time | default(''1.05:00:00'')
        }}'
  loop: '{{ iis_sites }}'
  when:
  - iis_sites is defined
  - item.state | default('present') == 'present'
  tags: sites
- name: Configure IIS websites
  win_iis_website:
    name: '{{ item.name }}'
    state: '{{ item.state | default(''present'') }}'
    physical_path: '{{ item.physical_path | default(''C:\inetpub\wwwroot\'' + item.name)
      }}'
    application_pool: '{{ item.app_pool | default(item.name + ''_pool'') }}'
    port: '{{ item.port | default(iis_sites_defaults.port) }}'
    host_header: '{{ item.host_header | default('''') }}'
    ip: '{{ item.ip | default(''*'') }}'
    ssl: '{{ item.ssl.enabled | default(false) if item.ssl is defined else false }}'
  loop: '{{ iis_sites }}'
  when: iis_sites is defined
  register: iis_sites_result
  tags: sites
- name: Create simple index.html for sites
  win_template:
    src: index.html.j2
    dest: '{{ item.physical_path | default(''C:\inetpub\wwwroot\'' + item.name) }}\index.html'
  loop: '{{ iis_sites }}'
  when:
  - iis_sites is defined
  - item.state | default('present') == 'present'
  - item.create_index | default(true)
  tags: sites
- name: Configure SSL for websites
  win_shell: "Import-Module WebAdministration\n\n$CertHash = \"{{ item.ssl.cert_hash\
    \ }}\"\n$StoreName = \"{{ item.ssl.store_name | default('MY') }}\"\n$SiteName\
    \ = \"{{ item.name }}\"\n\n# Add SSL binding to the site\n$Binding = Get-WebBinding\
    \ -Name $SiteName -Protocol \"https\"\nif (-not $Binding) {\n    New-WebBinding\
    \ -Name $SiteName -IP \"*\" -Port 443 -Protocol \"https\" -HostHeader \"{{ item.host_header\
    \ | default('') }}\"\n}\n\n# Assign certificate\n$Binding = Get-WebBinding -Name\
    \ $SiteName -Protocol \"https\"\n$Binding.AddSslCertificate($CertHash, $StoreName)\n"
  loop: '{{ iis_sites }}'
  when:
  - iis_sites is defined
  - item.state | default('present') == 'present'
  - item.ssl is defined
  - item.ssl.enabled | default(false)
  - item.ssl.cert_hash is defined
  tags: sites
- name: Configure authentication settings for each site
  win_shell: 'Import-Module WebAdministration


    $SiteName = "{{ item.name }}"


    # Configure Anonymous Authentication

    Set-WebConfigurationProperty -Filter "/system.webServer/security/authentication/anonymousAuthentication"
    -PSPath "IIS:\Sites\$SiteName" -Name "enabled" -Value {{ "true" if item.auth.anonymous
    | default(iis_sites_defaults.auth.anonymous) else "false" }}


    # Configure Basic Authentication

    Set-WebConfigurationProperty -Filter "/system.webServer/security/authentication/basicAuthentication"
    -PSPath "IIS:\Sites\$SiteName" -Name "enabled" -Value {{ "true" if item.auth.basic
    | default(iis_sites_defaults.auth.basic) else "false" }}


    # Configure Windows Authentication

    Set-WebConfigurationProperty -Filter "/system.webServer/security/authentication/windowsAuthentication"
    -PSPath "IIS:\Sites\$SiteName" -Name "enabled" -Value {{ "true" if item.auth.windows
    | default(iis_sites_defaults.auth.windows) else "false" }}

    '
  loop: '{{ iis_sites }}'
  when:
  - iis_sites is defined
  - item.state | default('present') == 'present'
  - item.auth is defined
  tags: sites
- name: Configure site limits and advanced settings
  win_shell: "Import-Module WebAdministration\n\n$SiteName = \"{{ item.name }}\"\n\
    \n# Connection Limits\n{% if item.limits is defined and item.limits.max_connections\
    \ is defined %}\nSet-ItemProperty \"IIS:\\Sites\\$SiteName\" -Name \"limits.maxConnections\"\
    \ -Value {{ item.limits.max_connections }}\n{% endif %}\n\n{% if item.limits is\
    \ defined and item.limits.connection_timeout is defined %}\nSet-ItemProperty \"\
    IIS:\\Sites\\$SiteName\" -Name \"limits.connectionTimeout\" -Value \"{{ item.limits.connection_timeout\
    \ }}:00:00\"\n{% endif %}\n\n{% if item.limits is defined and item.limits.max_bandwidth\
    \ is defined %}\nSet-ItemProperty \"IIS:\\Sites\\$SiteName\" -Name \"limits.maxBandwidth\"\
    \ -Value {{ item.limits.max_bandwidth }}\n{% endif %}\n\n# Default Documents for\
    \ this site\n{% if item.default_doc is defined %}\n$DefaultDocs = @({% for doc\
    \ in item.default_doc %}'{{ doc }}'{% if not loop.last %}, {% endif %}{% endfor\
    \ %})\n\n# Clear existing defaults first\nClear-WebConfiguration -Filter \"//defaultDocument/files/*\"\
    \ -PSPath \"IIS:\\Sites\\$SiteName\"\n\n# Add new defaults\nforeach ($doc in $DefaultDocs)\
    \ {\n  Add-WebConfiguration -Filter \"//defaultDocument/files\" -PSPath \"IIS:\\\
    Sites\\$SiteName\" -Value @{value=\"$doc\"}\n}\n{% endif %}\n\n# Logging settings\n\
    Set-ItemProperty \"IIS:\\Sites\\$SiteName\" -Name \"logFile.directory\" -Value\
    \ \"{{ item.log_directory | default(iis_log_dir + '\\\\\\\\' + item.name) | regex_replace('\\\
    \\\\\\', '\\\\\\\\\\\\\\\\') }}\"\nSet-ItemProperty \"IIS:\\Sites\\$SiteName\"\
    \ -Name \"logFile.period\" -Value \"{{ item.log_period | default(iis_sites_defaults.log_period)\
    \ }}\"\nSet-ItemProperty \"IIS:\\Sites\\$SiteName\" -Name \"logFile.logFormat\"\
    \ -Value \"{{ item.log_format | default(iis_sites_defaults.log_format) }}\"\n\n\
    # Custom HTTP headers\n{% if item.custom_headers is defined %}\n{% for header_name,\
    \ header_value in item.custom_headers.items() %}\n$HeaderName = \"{{ header_name\
    \ }}\"\n$HeaderValue = \"{{ header_value }}\"\n\n# Check if header exists\n$ExistingHeader\
    \ = Get-WebConfiguration -Filter \"//httpProtocol/customHeaders/add[@name='$HeaderName']\"\
    \ -PSPath \"IIS:\\Sites\\$SiteName\"\nif ($ExistingHeader) {\n  # Update existing\n\
    \  Set-WebConfigurationProperty -Filter \"//httpProtocol/customHeaders/add[@name='$HeaderName']\"\
    \ -PSPath \"IIS:\\Sites\\$SiteName\" -Name \"value\" -Value \"$HeaderValue\"\n\
    } else {\n  # Add new\n  Add-WebConfiguration -Filter \"//httpProtocol/customHeaders\"\
    \ -PSPath \"IIS:\\Sites\\$SiteName\" -Value @{name=\"$HeaderName\";value=\"$HeaderValue\"\
    }\n}\n{% endfor %}\n{% endif %}\n"
  loop: '{{ iis_sites }}'
  when:
  - iis_sites is defined
  - item.state | default('present') == 'present'
  register: site_config_result
  changed_when: site_config_result.stdout_lines | length > 0
  tags: sites
- name: Deploy custom site configuration PowerShell script
  win_template:
    src: site-config.ps1.j2
    dest: C:\Temp\site-config-{{ item.name }}.ps1
  loop: '{{ iis_sites }}'
  when:
  - iis_sites is defined
  - item.state | default('present') == 'present'
  tags: sites
- name: Execute site configuration scripts
  win_shell: 'powershell.exe -ExecutionPolicy Bypass -File "C:\\Temp\\site-config-{{
    item.name }}.ps1"

    '
  loop: '{{ iis_sites }}'
  when:
  - iis_sites is defined
  - item.state | default('present') == 'present'
  register: site_custom_config
  changed_when: site_custom_config.stdout is not search('No changes needed')
  notify: apply iis config
  tags: sites

```

## Handlers

```yaml
- name: restart iis
  win_service:
    name: W3SVC
    state: restarted
- name: start iis
  win_service:
    name: W3SVC
    state: started
- name: stop iis
  win_service:
    name: W3SVC
    state: stopped
- name: apply iis config
  win_shell: IISRESET

```

## Templates

- `iis_site.ps1.j2`
- `iis_config.ps1.j2`
- `index.html.j2`

## Playbook Example 
```yaml
---
# playbook-iis.yml
- name: Deploy IIS with Custom Websites
  hosts: windows_servers
  gather_facts: yes
  vars:
    iis_configure_firewall: true
    iis_sites:
      # Suppression du site par défaut
      - name: "Default Web Site"
        state: absent
        
      # Site web simple
      - name: monsite-web
        protocol: http
        port: 80
        host_header: www.monentreprise.com
        physical_path: C:\inetpub\wwwroot\monsite-web
        app_pool: monsite_pool
        state: present
        auth:
          anonymous: true
          basic: false
          windows: false
        default_doc:
          - index.html
          - default.htm
        headers:
          X-Frame-Options: "SAMEORIGIN"
          X-Content-Type-Options: "nosniff"
        
      # Site web sécurisé (HTTPS)
      - name: secure-site
        protocol: https
        port: 443
        host_header: secure.monentreprise.com
        physical_path: C:\inetpub\wwwroot\secure-site
        app_pool: secure_pool
        state: present
        ssl:
          enabled: true
          cert_hash: "XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX"
          store_name: "MY"
        auth:
          anonymous: false
          windows: true
        web_config_settings:
          system.web/compilation:
            debug: "false"
          system.web/customErrors:
            mode: "RemoteOnly"
        mime_types:
          - extension: ".woff"
            type: "font/woff"
          - extension: ".woff2"
            type: "font/woff2"
        url_rewrite_rules:
          - name: "HTTP to HTTPS"
            pattern: "(.*)$"
            action_type: "Redirect"
            action_value: "https://{HTTP_HOST}/{R:1}"
            stop_processing: true
            conditions:
              - input: "{HTTPS}"
                pattern: "^OFF$"
              - input: "{HTTP_HOST}"
                pattern: "^secure.monentreprise.com$"

  roles:
    - windows_iis_install_configure

# ansible-playbook -i inventory playbook-iis.yml                   # Installation complète
# ansible-playbook -i inventory playbook-iis.yml --tags "install"   # Installation seulement
# ansible-playbook -i inventory playbook-iis.yml --tags "configure" # Configuration seulement
# ansible-playbook -i inventory playbook-iis.yml --tags "sites"     # Gestion sites seulement

```

## Role Structure

```
vars/
    └── main.yml
templates/
    ├── iis_config.ps1.j2
    ├── iis_site.ps1.j2
    └── index.html.j2
meta/
    └── main.yml
tests/
    ├── inventory
    └── test.yml
tasks/
    ├── configure.yml
    ├── install.yml
    ├── main.yml
    └── sites.yml
handlers/
    └── main.yml
defaults/
    └── main.yml
README.md
```