# Ansible Role: security_report_windows

Generate comprehensive Security Reporting for Windows Server from Ansible Role

## General Information

**Author:** A&ECoding
**License:** MIT
**Minimum Ansible Version:** 2.10

**Supported Platforms:**
- Windows
  - Versions: all

## Default Variables

```yaml
report_dir: security_reports
security_scan_antivirus: true
security_scan_firewall: true

```

## Main Tasks

- Create report directory
- Gather system information
- Check for Windows security updates
- Scan open ports
- Check user accounts and permissions
- Verify RDP configuration
- Check file permissions on critical directories
- Check Windows Firewall status
- Check Antivirus status
- Generate HTML report
- Generate JSON summary

## Other Tasks

### user_audit.yml

```yaml
- name: List local administrators
  win_shell: 'Get-LocalGroupMember -Group "Administrators" |

    Select-Object Name, ObjectClass, PrincipalSource |

    ConvertTo-Json

    '
  register: admin_users_raw
  changed_when: false
  failed_when: false
- name: Parse admin users
  set_fact:
    admin_users: '{{ admin_users_raw.stdout | from_json if admin_users_raw.stdout
      | length > 0 else [] }}'
- name: List all local users
  win_shell: 'Get-LocalUser |

    Select-Object Name, Enabled, PasswordRequired, PasswordLastSet, LastLogon, Description
    |

    ConvertTo-Json

    '
  register: local_users_raw
  changed_when: false
  failed_when: false
- name: Parse local users
  set_fact:
    local_users: '{{ local_users_raw.stdout | from_json if local_users_raw.stdout
      | length > 0 else [] }}'
- name: Check for users with passwords that never expire
  win_shell: 'Get-LocalUser |

    Where-Object { $_.PasswordNeverExpires -eq $true -and $_.Enabled -eq $true } |

    Select-Object Name, PasswordLastSet, PasswordNeverExpires |

    ConvertTo-Json

    '
  register: never_expire_users_raw
  changed_when: false
  failed_when: false
- name: Parse users with non-expiring passwords
  set_fact:
    never_expire_users: '{{ never_expire_users_raw.stdout | from_json if never_expire_users_raw.stdout
      | length > 0 else [] }}'
- name: Check for disabled accounts
  win_shell: 'Get-LocalUser |

    Where-Object { $_.Enabled -eq $false } |

    Select-Object Name, LastLogon, Description |

    ConvertTo-Json

    '
  register: disabled_users_raw
  changed_when: false
  failed_when: false
- name: Parse disabled users
  set_fact:
    disabled_users: '{{ disabled_users_raw.stdout | from_json if disabled_users_raw.stdout
      | length > 0 else [] }}'
- name: Check for inactive users (no login for 90+ days)
  win_shell: '$threshold = (Get-Date).AddDays(-90)

    Get-LocalUser |

    Where-Object { $_.LastLogon -lt $threshold -and $_.Enabled -eq $true } |

    Select-Object Name, LastLogon, PasswordLastSet |

    ConvertTo-Json

    '
  register: inactive_users_raw
  changed_when: false
  failed_when: false
- name: Parse inactive users
  set_fact:
    inactive_users: '{{ inactive_users_raw.stdout | from_json if inactive_users_raw.stdout
      | length > 0 else [] }}'

```

### antivirus_check.yml

```yaml
- name: Check Windows Defender status
  win_shell: "$defender = Get-MpComputerStatus\n@{\n  AntivirusEnabled = $defender.AntivirusEnabled\n\
    \  AntispywareEnabled = $defender.AntispywareEnabled\n  RealTimeProtectionEnabled\
    \ = $defender.RealTimeProtectionEnabled\n  OnAccessProtectionEnabled = $defender.OnAccessProtectionEnabled\n\
    \  BehaviorMonitorEnabled = $defender.BehaviorMonitorEnabled\n  IoavProtectionEnabled\
    \ = $defender.IoavProtectionEnabled\n  AntivirusSignatureLastUpdated = $defender.AntivirusSignatureLastUpdated\n\
    \  QuickScanAge = $defender.QuickScanAge\n  FullScanAge = $defender.FullScanAge\n\
    \  DefenderVersion = $defender.AMProductVersion\n} | ConvertTo-Json\n"
  register: defender_status_raw
  changed_when: false
  failed_when: false
- name: Parse Defender status
  set_fact:
    defender_status: '{{ defender_status_raw.stdout | from_json }}'
  when: defender_status_raw.rc == 0
- name: Set default Defender status if check failed
  set_fact:
    defender_status:
      AntivirusEnabled: N/A
      Status: Unable to check Windows Defender status
  when: defender_status_raw.rc != 0
- name: Check for recent threats
  win_shell: "Get-MpThreatDetection | \nSelect-Object -First 10 ThreatName, Resources,\
    \ ProcessName, InitialDetectionTime, RemediationTime |\nConvertTo-Json\n"
  register: recent_threats_raw
  changed_when: false
  failed_when: false
- name: Parse recent threats
  set_fact:
    recent_threats: '{{ recent_threats_raw.stdout | from_json if recent_threats_raw.stdout
      | length > 0 else [] }}'
- name: Check third-party antivirus
  win_shell: '$av = Get-CimInstance -Namespace root/SecurityCenter2 -ClassName AntiVirusProduct

    $av | Select-Object displayName, productState, timestamp | ConvertTo-Json

    '
  register: third_party_av_raw
  changed_when: false
  failed_when: false
- name: Parse third-party antivirus
  set_fact:
    third_party_av: '{{ third_party_av_raw.stdout | from_json if third_party_av_raw.stdout
      | length > 0 else [] }}'
- name: Check Defender exclusions
  win_shell: "$prefs = Get-MpPreference\n@{\n  ExclusionPath = $prefs.ExclusionPath\n\
    \  ExclusionExtension = $prefs.ExclusionExtension\n  ExclusionProcess = $prefs.ExclusionProcess\n\
    } | ConvertTo-Json\n"
  register: defender_exclusions_raw
  changed_when: false
  failed_when: false
- name: Parse Defender exclusions
  set_fact:
    defender_exclusions: '{{ defender_exclusions_raw.stdout | from_json }}'
  when: defender_exclusions_raw.rc == 0

```

### port_scan.yml

```yaml
- name: Get listening TCP ports
  win_shell: "Get-NetTCPConnection -State Listen | \nSelect-Object LocalAddress, LocalPort,\
    \ OwningProcess, \n@{Name='ProcessName';Expression={(Get-Process -Id $_.OwningProcess\
    \ -ErrorAction SilentlyContinue).ProcessName}} |\nSort-Object LocalPort | \nConvertTo-Json\n"
  register: listening_ports_raw
  changed_when: false
  failed_when: false
- name: Parse listening ports
  set_fact:
    listening_ports: '{{ listening_ports_raw.stdout | from_json if listening_ports_raw.stdout
      | length > 0 else [] }}'
- name: Check for common vulnerable services
  win_shell: "$vulnerablePorts = @(21, 23, 80, 135, 139, 445, 1433, 3389, 5432, 5900)\n\
    Get-NetTCPConnection -State Listen | \nWhere-Object { $vulnerablePorts -contains\
    \ $_.LocalPort } |\nSelect-Object LocalAddress, LocalPort, OwningProcess,\n@{Name='ProcessName';Expression={(Get-Process\
    \ -Id $_.OwningProcess -ErrorAction SilentlyContinue).ProcessName}} |\nConvertTo-Json\n"
  register: vulnerable_ports_raw
  changed_when: false
  failed_when: false
- name: Parse vulnerable ports
  set_fact:
    vulnerable_ports: '{{ vulnerable_ports_raw.stdout | from_json if vulnerable_ports_raw.stdout
      | length > 0 else [] }}'
- name: Get UDP listening ports
  win_shell: 'Get-NetUDPEndpoint |

    Select-Object LocalAddress, LocalPort, OwningProcess,

    @{Name=''ProcessName'';Expression={(Get-Process -Id $_.OwningProcess -ErrorAction
    SilentlyContinue).ProcessName}} |

    Sort-Object LocalPort |

    ConvertTo-Json

    '
  register: udp_ports_raw
  changed_when: false
  failed_when: false
- name: Parse UDP ports
  set_fact:
    udp_ports: '{{ udp_ports_raw.stdout | from_json if udp_ports_raw.stdout | length
      > 0 else [] }}'

```

### security_updates.yml

```yaml
- name: Check for pending Windows updates
  win_shell: "$UpdateSession = New-Object -ComObject Microsoft.Update.Session\n$UpdateSearcher\
    \ = $UpdateSession.CreateUpdateSearcher()\n$SearchResult = $UpdateSearcher.Search(\"\
    IsInstalled=0 and Type='Software'\")\n$SecurityUpdates = $SearchResult.Updates\
    \ | Where-Object { $_.MsrcSeverity -in @('Critical', 'Important') }\n@{\n  TotalUpdates\
    \ = $SearchResult.Updates.Count\n  SecurityUpdates = $SecurityUpdates.Count\n\
    \  CriticalUpdates = ($SecurityUpdates | Where-Object { $_.MsrcSeverity -eq 'Critical'\
    \ }).Count\n  ImportantUpdates = ($SecurityUpdates | Where-Object { $_.MsrcSeverity\
    \ -eq 'Important' }).Count\n} | ConvertTo-Json\n"
  register: windows_updates_raw
  changed_when: false
  failed_when: false
- name: Parse Windows updates result
  set_fact:
    windows_updates: '{{ windows_updates_raw.stdout | from_json }}'
  when: windows_updates_raw.rc == 0 and windows_updates_raw.stdout | length > 0
- name: Set default if updates check failed
  set_fact:
    windows_updates:
      TotalUpdates: N/A
      SecurityUpdates: N/A
      CriticalUpdates: N/A
      ImportantUpdates: N/A
  when: windows_updates_raw.rc != 0 or windows_updates_raw.stdout | length == 0
- name: List pending security updates
  win_shell: '$UpdateSession = New-Object -ComObject Microsoft.Update.Session

    $UpdateSearcher = $UpdateSession.CreateUpdateSearcher()

    $SearchResult = $UpdateSearcher.Search("IsInstalled=0 and Type=''Software''")

    $SecurityUpdates = $SearchResult.Updates | Where-Object { $_.MsrcSeverity -in
    @(''Critical'', ''Important'') }

    $SecurityUpdates | Select-Object Title, MsrcSeverity, Description | ConvertTo-Json

    '
  register: windows_security_updates_list
  changed_when: false
  failed_when: false

```

### rdp_audit.yml

```yaml
- name: Check RDP status
  win_shell: "$rdpEnabled = (Get-ItemProperty -Path \"HKLM:\\System\\CurrentControlSet\\\
    Control\\Terminal Server\" -Name \"fDenyTSConnections\").fDenyTSConnections\n\
    @{\n  Enabled = if ($rdpEnabled -eq 0) { \"Yes\" } else { \"No\" }\n  Status =\
    \ if ($rdpEnabled -eq 0) { \"RDP is enabled\" } else { \"RDP is disabled\" }\n\
    } | ConvertTo-Json\n"
  register: rdp_status_raw
  changed_when: false
  failed_when: false
- name: Parse RDP status
  set_fact:
    rdp_status: '{{ rdp_status_raw.stdout | from_json }}'
  when: rdp_status_raw.rc == 0
- name: Check RDP port
  win_shell: "$rdpPort = (Get-ItemProperty -Path \"HKLM:\\System\\CurrentControlSet\\\
    Control\\Terminal Server\\WinStations\\RDP-Tcp\" -Name \"PortNumber\").PortNumber\n\
    @{\n  Port = $rdpPort\n  IsDefault = if ($rdpPort -eq 3389) { \"Yes\" } else {\
    \ \"No (Custom: $rdpPort)\" }\n} | ConvertTo-Json\n"
  register: rdp_port_raw
  changed_when: false
  failed_when: false
- name: Parse RDP port
  set_fact:
    rdp_port: '{{ rdp_port_raw.stdout | from_json }}'
  when: rdp_port_raw.rc == 0
- name: Check Network Level Authentication (NLA)
  win_shell: "$nla = (Get-ItemProperty -Path \"HKLM:\\System\\CurrentControlSet\\\
    Control\\Terminal Server\\WinStations\\RDP-Tcp\" -Name \"UserAuthentication\"\
    ).UserAuthentication\n@{\n  Enabled = if ($nla -eq 1) { \"Yes\" } else { \"No\"\
    \ }\n  Status = if ($nla -eq 1) { \"NLA is enabled (Secure)\" } else { \"NLA is\
    \ disabled (Insecure)\" }\n  Recommendation = if ($nla -eq 1) { \"Good\" } else\
    \ { \"Enable NLA for better security\" }\n} | ConvertTo-Json\n"
  register: rdp_nla_raw
  changed_when: false
  failed_when: false
- name: Parse NLA status
  set_fact:
    rdp_nla: '{{ rdp_nla_raw.stdout | from_json }}'
  when: rdp_nla_raw.rc == 0
- name: Check RDP encryption level
  win_shell: "$encLevel = (Get-ItemProperty -Path \"HKLM:\\System\\CurrentControlSet\\\
    Control\\Terminal Server\\WinStations\\RDP-Tcp\" -Name \"MinEncryptionLevel\"\
    \ -ErrorAction SilentlyContinue).MinEncryptionLevel\n$encLevelText = switch ($encLevel)\
    \ {\n  1 { \"Low\" }\n  2 { \"Client Compatible\" }\n  3 { \"High\" }\n  4 { \"\
    FIPS Compliant\" }\n  default { \"Unknown\" }\n}\n@{\n  Level = $encLevel\n  Description\
    \ = $encLevelText\n  Secure = if ($encLevel -ge 3) { \"Yes\" } else { \"No - Increase\
    \ to High or FIPS\" }\n} | ConvertTo-Json\n"
  register: rdp_encryption_raw
  changed_when: false
  failed_when: false
- name: Parse RDP encryption
  set_fact:
    rdp_encryption: '{{ rdp_encryption_raw.stdout | from_json }}'
  when: rdp_encryption_raw.rc == 0

```

### firewall_check.yml

```yaml
- name: Check Windows Firewall status
  win_shell: "$profiles = @(\"Domain\", \"Private\", \"Public\")\n$results = @()\n\
    foreach ($profile in $profiles) {\n  $status = Get-NetFirewallProfile -Name $profile\n\
    \  $results += @{\n    Profile = $profile\n    Enabled = $status.Enabled\n   \
    \ DefaultInboundAction = $status.DefaultInboundAction\n    DefaultOutboundAction\
    \ = $status.DefaultOutboundAction\n    Status = if ($status.Enabled) { \"Active\"\
    \ } else { \"Disabled - Security Risk!\" }\n  }\n}\n$results | ConvertTo-Json\n"
  register: firewall_status_raw
  changed_when: false
  failed_when: false
- name: Parse firewall status
  set_fact:
    firewall_status: '{{ firewall_status_raw.stdout | from_json }}'
- name: List firewall rules (inbound)
  win_shell: 'Get-NetFirewallRule -Direction Inbound -Enabled True |

    Select-Object Name, DisplayName, Enabled, Direction, Action, Profile |

    Sort-Object DisplayName |

    Select-Object -First 50 |

    ConvertTo-Json

    '
  register: firewall_rules_inbound_raw
  changed_when: false
  failed_when: false
- name: Parse inbound firewall rules
  set_fact:
    firewall_rules_inbound: '{{ firewall_rules_inbound_raw.stdout | from_json if firewall_rules_inbound_raw.stdout
      | length > 0 else [] }}'
- name: Check for disabled firewall profiles
  win_shell: "$disabled = Get-NetFirewallProfile | Where-Object { -not $_.Enabled\
    \ }\nif ($disabled) {\n  $disabled | Select-Object Name, Enabled | ConvertTo-Json\n\
    } else {\n  @() | ConvertTo-Json\n}\n"
  register: disabled_firewall_raw
  changed_when: false
  failed_when: false
- name: Parse disabled firewall profiles
  set_fact:
    disabled_firewall: '{{ disabled_firewall_raw.stdout | from_json if disabled_firewall_raw.stdout
      | length > 0 else [] }}'

```

### file_permissions.yml

```yaml
- name: Check writable system directories
  win_shell: "$paths = @(\"C:\\Windows\\System32\", \"C:\\Windows\", \"C:\\Program\
    \ Files\")\n$results = @()\nforeach ($path in $paths) {\n  $acl = Get-Acl $path\
    \ -ErrorAction SilentlyContinue\n  if ($acl) {\n    $writableByUsers = $acl.Access\
    \ | Where-Object { \n      $_.IdentityReference -match \"Users|Everyone\" -and\
    \ \n      $_.FileSystemRights -match \"Write|FullControl|Modify\"\n    }\n   \
    \ if ($writableByUsers) {\n      $results += @{\n        Path = $path\n      \
    \  Writable = \"Yes\"\n        Issue = \"Writable by Users/Everyone - Security\
    \ Risk\"\n      }\n    }\n  }\n}\n$results | ConvertTo-Json\n"
  register: writable_directories_raw
  changed_when: false
  failed_when: false
- name: Parse writable directories
  set_fact:
    writable_directories: '{{ writable_directories_raw.stdout | from_json if writable_directories_raw.stdout
      | length > 0 else [] }}'
- name: Check startup programs
  win_shell: "$startupPaths = @(\n  \"HKLM:\\SOFTWARE\\Microsoft\\Windows\\CurrentVersion\\\
    Run\",\n  \"HKCU:\\SOFTWARE\\Microsoft\\Windows\\CurrentVersion\\Run\",\n  \"\
    HKLM:\\SOFTWARE\\Microsoft\\Windows\\CurrentVersion\\RunOnce\",\n  \"HKCU:\\SOFTWARE\\\
    Microsoft\\Windows\\CurrentVersion\\RunOnce\"\n)\n$results = @()\nforeach ($path\
    \ in $startupPaths) {\n  if (Test-Path $path) {\n    $items = Get-ItemProperty\
    \ $path -ErrorAction SilentlyContinue\n    if ($items) {\n      $items.PSObject.Properties\
    \ | Where-Object { $_.Name -notmatch \"^PS\" } | ForEach-Object {\n        $results\
    \ += @{\n          Location = $path\n          Name = $_.Name\n          Command\
    \ = $_.Value\n        }\n      }\n    }\n  }\n}\n$results | ConvertTo-Json\n"
  register: startup_programs_raw
  changed_when: false
  failed_when: false
- name: Parse startup programs
  set_fact:
    startup_programs: '{{ startup_programs_raw.stdout | from_json if startup_programs_raw.stdout
      | length > 0 else [] }}'
- name: Check scheduled tasks
  win_shell: "Get-ScheduledTask |\nWhere-Object { $_.State -eq \"Ready\" -and $_.Principal.UserId\
    \ -ne \"SYSTEM\" } |\nSelect-Object TaskName, TaskPath, State, \n@{Name='RunAsUser';Expression={$_.Principal.UserId}},\n\
    @{Name='LastRunTime';Expression={$_.LastRunTime}} |\nConvertTo-Json\n"
  register: scheduled_tasks_raw
  changed_when: false
  failed_when: false
- name: Parse scheduled tasks
  set_fact:
    scheduled_tasks: '{{ scheduled_tasks_raw.stdout | from_json if scheduled_tasks_raw.stdout
      | length > 0 else [] }}'

```

## Templates

- `security_report_windows.html.j2`
- `security_summary_windows.json.j2`

## Role Structure

```
vars/
    └── main.yml
templates/
    ├── security_report_windows.html.j2
    └── security_summary_windows.json.j2
meta/
    └── main.yml
tests/
    ├── inventory
    └── test.yml
tasks/
    ├── antivirus_check.yml
    ├── file_permissions.yml
    ├── firewall_check.yml
    ├── main.yml
    ├── port_scan.yml
    ├── rdp_audit.yml
    ├── security_updates.yml
    └── user_audit.yml
handlers/
    └── main.yml
defaults/
    └── main.yml
README.md
