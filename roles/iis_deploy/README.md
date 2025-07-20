# Ansible Role: iis_deploy

your role description

## General Information

**Author:** your name
**License:** license (GPL-2.0-or-later, MIT, etc)
**Minimum Ansible Version:** 2.1

## Variables

### main

```yaml
iis_deploy_app_name: DefaultAppName
iis_deploy_physical_path: C:\inetpub\wwwroot\{{ iis_deploy_app_name }}
iis_deploy_site_name: Default Web Site
iis_deploy_app_pool_name: '{{ iis_deploy_app_name }}_pool'
iis_deploy_app_path: /{{ iis_deploy_app_name }}
iis_deploy_port: 80
iis_deploy_binding_info: '*:{{ iis_deploy_port }}:'
artifactory_url: http://artifactory.example.com/artifactory
artifactory_repository: webapp-releases
artifactory_username: ''
artifactory_password: ''
artifactory_artifact_path: '{{ iis_deploy_app_name }}/{{ iis_deploy_version }}/{{
  iis_deploy_app_name }}-{{ iis_deploy_version }}.zip'
iis_deploy_version: latest
artifactory_api_key: ''
iis_deploy_app_environment: production
iis_deploy_dotnet_version: v4.0
iis_deploy_enable_32bit: false
iis_deploy_managed_pipeline_mode: Integrated
iis_deploy_identity_type: ApplicationPoolIdentity
iis_deploy_username: ''
iis_deploy_password: ''
iis_deploy_site_bindings:
- protocol: http
  binding_information: '{{ iis_deploy_binding_info }}'
  ssl_flags: 0
iis_deploy_idle_timeout_minutes: 20
iis_deploy_max_processes: 1
iis_deploy_web_config_transform: false
iis_deploy_clean_before_deploy: true
iis_deploy_keep_versions: 3

```

## Main Tasks

- Inclure les variables spécifiques à la plateforme
- Vérifier que le système d'exploitation est Windows
- Inclure les tâches d'installation des prérequis
- Inclure les tâches de téléchargement des artefacts
- Inclure les tâches de configuration IIS
- Inclure les tâches de déploiement d'application

## Other Tasks

### configure_iis.yml

```yaml
- name: "V\xE9rifier si le pool d'applications existe"
  win_shell: 'Import-Module WebAdministration

    Test-Path "IIS:\AppPools\{{ iis_deploy_app_pool_name }}"

    '
  register: app_pool_exists
  changed_when: false
- name: "Cr\xE9er le pool d'applications"
  win_shell: "Import-Module WebAdministration\nNew-WebAppPool -Name \"{{ iis_deploy_app_pool_name\
    \ }}\"\nSet-ItemProperty -Path \"IIS:\\AppPools\\{{ iis_deploy_app_pool_name }}\"\
    \ -Name managedRuntimeVersion -Value \"{{ iis_deploy_dotnet_version }}\"\nSet-ItemProperty\
    \ -Path \"IIS:\\AppPools\\{{ iis_deploy_app_pool_name }}\" -Name enable32BitAppOnWin64\
    \ -Value ${{ iis_deploy_enable_32bit | lower }}\nSet-ItemProperty -Path \"IIS:\\\
    AppPools\\{{ iis_deploy_app_pool_name }}\" -Name managedPipelineMode -Value \"\
    {{ iis_deploy_managed_pipeline_mode }}\"\nSet-ItemProperty -Path \"IIS:\\AppPools\\\
    {{ iis_deploy_app_pool_name }}\" -Name processModel.idleTimeout -Value \"{{ iis_deploy_idle_timeout_minutes\
    \ }}\"\nSet-ItemProperty -Path \"IIS:\\AppPools\\{{ iis_deploy_app_pool_name }}\"\
    \ -Name processModel.maxProcesses -Value \"{{ iis_deploy_max_processes }}\"\n\n\
    # Configurer l'identit\xE9 du pool\n{% raw %}\nif (\"{{ iis_deploy_identity_type\
    \ }}\" -eq \"SpecificUser\") {\n    $appPool = Get-Item \"IIS:\\AppPools\\{{ iis_deploy_app_pool_name\
    \ }}\" \n    $appPool.processModel.identityType = \"SpecificUser\"\n    $appPool.processModel.userName\
    \ = \"{{ iis_deploy_username }}\"\n    $appPool.processModel.password = \"{{ iis_deploy_password\
    \ }}\"\n    $appPool | Set-Item\n} else {\n    Set-ItemProperty -Path \"IIS:\\\
    AppPools\\{{ iis_deploy_app_pool_name }}\" -Name processModel.identityType -Value\
    \ \"{{ iis_deploy_identity_type }}\"\n}\n{% endraw %}\n"
  when: app_pool_exists.stdout | trim == "False"
- name: "V\xE9rifier si le r\xE9pertoire physique existe"
  win_stat:
    path: '{{ iis_deploy_physical_path }}'
  register: physical_path_check
- name: "Cr\xE9er le r\xE9pertoire physique si n\xE9cessaire"
  win_file:
    path: '{{ iis_deploy_physical_path }}'
    state: directory
  when: not physical_path_check.stat.exists
- name: "V\xE9rifier si l'application existe"
  win_shell: 'Import-Module WebAdministration

    Test-Path "IIS:\Sites\{{ iis_deploy_site_name }}\{{ iis_deploy_app_path }}"

    '
  register: app_exists
  changed_when: false
  failed_when: false
- name: "V\xE9rifier si le site Web existe"
  win_shell: 'Import-Module WebAdministration

    Test-Path "IIS:\Sites\{{ iis_deploy_site_name }}"

    '
  register: site_exists
  changed_when: false
- name: "Cr\xE9er le site Web si n\xE9cessaire"
  win_shell: 'Import-Module WebAdministration

    New-Website -Name "{{ iis_deploy_site_name }}" -PhysicalPath "{{ iis_deploy_physical_path
    }}" -ApplicationPool "{{ iis_deploy_app_pool_name }}" -Port {{ iis_deploy_port
    }} -HostHeader ""

    '
  when: site_exists.stdout | trim == "False"
- name: "Ajouter des liaisons suppl\xE9mentaires si n\xE9cessaire"
  win_shell: 'Import-Module WebAdministration

    New-WebBinding -Name "{{ iis_deploy_site_name }}" -Protocol "{{ item.protocol
    }}" -BindingInformation "{{ item.binding_information }}" -SslFlags {{ item.ssl_flags
    }}

    '
  loop: '{{ iis_deploy_site_bindings }}'
  when: site_exists.stdout | trim == "True"
- name: "Cr\xE9er l'application Web si elle n'existe pas"
  win_shell: 'Import-Module WebAdministration

    New-WebApplication -Name "{{ iis_deploy_app_name }}" -Site "{{ iis_deploy_site_name
    }}" -PhysicalPath "{{ iis_deploy_physical_path }}" -ApplicationPool "{{ iis_deploy_app_pool_name
    }}"

    '
  when: app_exists.stdout | trim == "False" and iis_deploy_app_path != "/"
- name: Configurer l'application Web existante
  win_shell: 'Import-Module WebAdministration

    Set-ItemProperty "IIS:\Sites\{{ iis_deploy_site_name }}\{{ iis_deploy_app_path
    }}" -Name physicalPath -Value "{{ iis_deploy_physical_path }}"

    Set-ItemProperty "IIS:\Sites\{{ iis_deploy_site_name }}\{{ iis_deploy_app_path
    }}" -Name applicationPool -Value "{{ iis_deploy_app_pool_name }}"

    '
  when: app_exists.stdout | trim == "True" and iis_deploy_app_path != "/"

```

### install_prequesites.yml

```yaml
- name: "V\xE9rifier que IIS est install\xE9"
  win_feature:
    name: Web-Server
    state: present
    include_management_tools: true
  register: iis_install
- name: "Installer les fonctionnalit\xE9s IIS requises"
  win_feature:
    name: '{{ item }}'
    state: present
  loop:
  - Web-Default-Doc
  - Web-Dir-Browsing
  - Web-Http-Errors
  - Web-Static-Content
  - Web-Http-Logging
  - Web-Stat-Compression
  - Web-Filtering
  - Web-Net-Ext45
  - Web-Asp-Net45
  - Web-ISAPI-Ext
  - Web-ISAPI-Filter
  - Web-Mgmt-Console
  register: iis_features
- name: "Red\xE9marrer le serveur si n\xE9cessaire"
  win_reboot: null
  when: iis_install.reboot_required or iis_features.reboot_required
- name: "V\xE9rifier que le service IIS est en cours d'ex\xE9cution"
  win_service:
    name: W3SVC
    state: started
    start_mode: auto

```

### deploy_application.yml

```yaml
- name: "Arr\xEAter le pool d'applications avant le d\xE9ploiement"
  win_shell: "Import-Module WebAdministration\nif((Get-WebAppPoolState -Name \"{{\
    \ iis_deploy_app_pool_name }}\").Value -eq \"Started\") {\n    Stop-WebAppPool\
    \ -Name \"{{ iis_deploy_app_pool_name }}\"\n}\n"
  ignore_errors: true
- name: "Nettoyer le r\xE9pertoire cible si n\xE9cessaire"
  win_file:
    path: '{{ iis_deploy_physical_path }}'
    state: absent
  when: iis_deploy_clean_before_deploy | bool
- name: "Recr\xE9er le r\xE9pertoire physique"
  win_file:
    path: '{{ iis_deploy_physical_path }}'
    state: directory
  when: iis_deploy_clean_before_deploy | bool
- name: "Extraire l'artefact dans le r\xE9pertoire cible"
  win_unzip:
    src: '{{ ansible_env.TEMP }}\{{ iis_deploy_app_name }}\{{ iis_deploy_app_name
      }}-{{ iis_deploy_actual_version }}.zip'
    dest: '{{ iis_deploy_physical_path }}'
    delete_archive: false
    recurse: true
- name: "Appliquer les transformations Web.config si n\xE9cessaire"
  win_shell: "Import-Module WebAdministration\n$webConfigPath = \"{{ iis_deploy_physical_path\
    \ }}\\Web.config\"\n$transformPath = \"{{ iis_deploy_physical_path }}\\Web.{{\
    \ iis_deploy_app_environment }}.config\"\n\nif(Test-Path $transformPath) {\n \
    \   # Utiliser Microsoft.Web.XmlTransform pour appliquer la transformation\n \
    \   Add-Type -Path \"C:\\Program Files (x86)\\MSBuild\\Microsoft\\VisualStudio\\\
    v14.0\\Web\\Microsoft.Web.XmlTransform.dll\"\n    $xmlDoc = New-Object Microsoft.Web.XmlTransform.XmlTransformableDocument\n\
    \    $xmlDoc.Load($webConfigPath)\n    $xmlTransform = New-Object Microsoft.Web.XmlTransform.XmlTransformation($transformPath)\n\
    \    if($xmlTransform.Apply($xmlDoc)) {\n        $xmlDoc.Save($webConfigPath)\n\
    \    }\n}\n"
  when: iis_deploy_web_config_transform | bool
  ignore_errors: true
- name: "G\xE9n\xE9rer le fichier Web.config \xE0 partir du template si n\xE9cessaire"
  win_template:
    src: web.config.j2
    dest: '{{ iis_deploy_physical_path }}\web.config'
    backup: true
  when: not iis_deploy_web_config_transform | bool
- name: "D\xE9marrer le pool d'applications"
  win_shell: 'Import-Module WebAdministration

    Start-WebAppPool -Name "{{ iis_deploy_app_pool_name }}"

    '
- name: "Valider le d\xE9ploiement"
  win_uri:
    url: http://localhost{{ iis_deploy_app_path }}
    method: GET
    status_code: 200, 302, 401
  register: deployment_validation
  ignore_errors: true
  when: iis_deploy_app_path != "/"
- name: Nettoyer les anciennes versions
  win_shell: "$versionsDirectory = \"{{ ansible_env.TEMP }}\\\\{{ iis_deploy_app_name\
    \ }}\"\n$files = Get-ChildItem -Path $versionsDirectory -Filter \"{{ iis_deploy_app_name\
    \ }}-*.zip\" | Sort-Object LastWriteTime -Descending\nif($files.Count -gt {{ iis_deploy_keep_versions\
    \ }}) {\n    $filesToDelete = $files | Select-Object -Skip {{ iis_deploy_keep_versions\
    \ }}\n    foreach($file in $filesToDelete) {\n        Remove-Item $file.FullName\
    \ -Force\n    }\n}\n"
  when: iis_deploy_keep_versions > 0
  ignore_errors: true
- name: "Afficher les r\xE9sultats du d\xE9ploiement"
  debug:
    msg: "Application {{ iis_deploy_app_name }} version {{ iis_deploy_actual_version\
      \ }} d\xE9ploy\xE9e avec succ\xE8s.\nURL: http://{{ ansible_host }}{{ iis_deploy_app_path\
      \ }}\nPool d'applications: {{ iis_deploy_app_pool_name }}\nR\xE9pertoire physique:\
      \ {{ iis_deploy_physical_path }}"

```

### download_artifact.yml

```yaml
- name: "Cr\xE9er le r\xE9pertoire temporaire pour le t\xE9l\xE9chargement"
  win_file:
    path: '{{ ansible_env.TEMP }}\{{ iis_deploy_app_name }}'
    state: directory
- name: "D\xE9terminer la derni\xE8re version de l'artefact si version='latest'"
  block:
  - name: "R\xE9cup\xE9rer la derni\xE8re version depuis Artifactory via API REST"
    win_uri:
      url: '{{ artifactory_url }}/api/search/latestVersion?g={{ iis_deploy_app_name
        }}&repos={{ artifactory_repository }}'
      method: GET
      headers:
        X-JFrog-Art-Api: '{{ artifactory_api_key | default(omit) }}'
      user: '{{ artifactory_username | default(omit) }}'
      password: '{{ artifactory_password | default(omit) }}'
      return_content: true
    register: latest_version_response
    when: iis_deploy_version == "latest"
  - name: "D\xE9finir la version r\xE9elle"
    set_fact:
      actual_version: '{{ latest_version_response.content | trim }}'
    when: iis_deploy_version == "latest"
  when: iis_deploy_version == "latest"
- name: "Utiliser la version sp\xE9cifi\xE9e si ce n'est pas 'latest'"
  set_fact:
    actual_version: '{{ iis_deploy_version }}'
  when: iis_deploy_version != "latest"
- name: Construire le chemin Artifactory
  set_fact:
    artifact_full_path: '{{ artifactory_repository }}/{{ iis_deploy_app_name }}/{{
      actual_version }}/{{ iis_deploy_app_name }}-{{ actual_version }}.zip'
- name: "T\xE9l\xE9charger l'artefact depuis Artifactory"
  win_uri:
    url: '{{ artifactory_url }}/{{ artifact_full_path }}'
    method: GET
    dest: '{{ ansible_env.TEMP }}\{{ iis_deploy_app_name }}\{{ iis_deploy_app_name
      }}-{{ actual_version }}.zip'
    headers:
      X-JFrog-Art-Api: '{{ artifactory_api_key | default(omit) }}'
    user: '{{ artifactory_username | default(omit) }}'
    password: '{{ artifactory_password | default(omit) }}'
    follow_redirects: all
  register: download_result
- name: "Afficher les informations sur le t\xE9l\xE9chargement"
  debug:
    msg: "Artefact t\xE9l\xE9charg\xE9: {{ iis_deploy_app_name }}-{{ actual_version\
      \ }}.zip, statut: {{ download_result.status_code }}"
- name: "D\xE9finir la version d\xE9ploy\xE9e dans les faits"
  set_fact:
    iis_deploy_actual_version: '{{ actual_version }}'

```

## Handlers

```yaml
- name: restart_iis
  win_shell: 'iisreset /restart

    '
  async: 120
  poll: 5
- name: restart_app_pool
  win_shell: "Import-Module WebAdministration\nif((Get-WebAppPoolState -Name \"{{\
    \ iis_deploy_app_pool_name }}\").Value -eq \"Started\") {\n    Restart-WebAppPool\
    \ -Name \"{{ iis_deploy_app_pool_name }}\"\n} else {\n    Start-WebAppPool -Name\
    \ \"{{ iis_deploy_app_pool_name }}\"\n}"

```

## Templates

- `web.config.j2`

## Role Structure

```
vars/
    └── main.yml
templates/
    └── web.config.j2
meta/
    └── main.yml
tests/
    ├── inventory
    └── test.yml
tasks/
    ├── configure_iis.yml
    ├── deploy_application.yml
    ├── download_artifact.yml
    ├── install_prequesites.yml
    └── main.yml
handlers/
    └── main.yml
defaults/
    └── main.yml
README.md
```