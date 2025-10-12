# Ansible Role: security_report

your role description

## General Information

**Author:** your name
**License:** license (GPL-2.0-or-later, MIT, etc)
**Minimum Ansible Version:** 2.1

## Variables

### main

```yaml
report_dir: security_reports
security_scan_rootkits: true
critical_services:
- ssh
- cron
- rsyslog
max_security_updates: 10
max_root_users: 1
critical_ports:
- 21
- 23
- 135
- 139
- 445
- 1433
- 3389
- 5900
security_categories:
  critical: Critical - Immediate action required
  high: High - Action required within 24h
  medium: Medium - Action required within 7 days
  low: Low - Monitor and plan remediation
  info: Information - For awareness

```

## Main Tasks

- Create report directory
- Gather system information
- Check for security updates
- Scan open ports
- Check user accounts and permissions
- Verify SSH configuration
- Check file permissions on critical directories
- Scan for rootkits (if rkhunter available)
- Generate HTML report
- Generate JSON summary

## Other Tasks

### user_audit.yml

```yaml
- name: List users with UID 0 (root privileges)
  shell: 'awk -F: ''$3 == 0 {print $1}'' /etc/passwd

    '
  register: root_users
  changed_when: false
- name: Check for users without passwords
  shell: 'awk -F: ''$2 == "" {print $1}'' /etc/shadow

    '
  register: no_password_users
  changed_when: false
  failed_when: false
- name: Check for inactive users (no login for 90+ days)
  shell: "lastlog | awk '$2 !~ /Never/ && $4 != \"\" {\n  cmd = \"date -d \\\"\"$4\"\
    \ \"$5\" \"$6\" \"$7\"\\\" +%s 2>/dev/null\"\n  cmd | getline last_login\n  close(cmd)\n\
    \  if ((systime() - last_login) > 7776000) print $1\n}'\n"
  register: inactive_users
  changed_when: false
  failed_when: false

```

### ssh_audit.yml

```yaml
- name: Check SSH configuration
  shell: 'sshd -T 2>/dev/null | grep -E ''^(permitrootlogin|passwordauthentication|protocol|port)''

    '
  register: ssh_config
  changed_when: false
  failed_when: false
- name: Check for SSH keys with weak permissions
  find:
    paths: /home
    patterns: '*id_*'
    recurse: true
    file_type: file
  register: ssh_keys
- name: Verify SSH key permissions
  stat:
    path: '{{ item.path }}'
  register: ssh_key_perms
  loop: '{{ ssh_keys.files }}'
  when: ssh_keys.files is defined

```

### port_scan.yml

```yaml
- name: Get listening ports
  shell: 'netstat -tuln | awk ''NR>2 {print $1,$4}'' | sort | uniq

    '
  register: listening_ports
  changed_when: false
  failed_when: false
- name: Check for common vulnerable services
  shell: 'netstat -tuln | grep -E '':(21|23|53|80|135|139|445|1433|3389|5432|5900)''
    || echo "None found"

    '
  register: vulnerable_ports
  changed_when: false

```

### security_updates.yml

```yaml
- name: Check for security updates (Debian/Ubuntu)
  shell: 'apt list --upgradable 2>/dev/null | grep -i security | wc -l

    '
  register: debian_security_updates
  when: ansible_os_family == "Debian"
  changed_when: false
  failed_when: false
- name: List available security updates (Debian/Ubuntu)
  shell: 'apt list --upgradable 2>/dev/null | grep -i security

    '
  register: debian_security_list
  when: ansible_os_family == "Debian"
  changed_when: false
  failed_when: false
- name: Check for security updates (RedHat/CentOS)
  shell: 'yum --security check-update | grep -c "needed for security" || echo "0"

    '
  register: redhat_security_updates
  when: ansible_os_family == "RedHat"
  changed_when: false
  failed_when: false
- name: List available security updates (RedHat/CentOS)
  shell: 'yum --security check-update

    '
  register: redhat_security_list
  when: ansible_os_family == "RedHat"
  changed_when: false
  failed_when: false

```

### rootkit_scan.yml

```yaml
- name: Check if rkhunter is installed
  command: which rkhunter
  register: rkhunter_check
  changed_when: false
  failed_when: false
- name: Run rkhunter scan
  shell: 'rkhunter --check --skip-keypress --report-warnings-only

    '
  register: rkhunter_results
  when: rkhunter_check.rc == 0
  changed_when: false
  failed_when: false

```

### file_permissions.yml

```yaml
- name: Check world-writable files
  shell: 'find /etc /usr /var -type f -perm -002 2>/dev/null | head -20

    '
  register: world_writable_files
  changed_when: false
  failed_when: false
- name: Check SUID/SGID files
  shell: 'find /usr /bin /sbin -type f \( -perm -4000 -o -perm -2000 \) 2>/dev/null

    '
  register: suid_sgid_files
  changed_when: false
  failed_when: false

```

## Templates

- `security_report.html.j2`
- `security_summary.json.j2`

## Role Structure

```
vars/
    └── main.yml
templates/
    ├── security_report.html.j2
    └── security_summary.json.j2
meta/
    └── main.yml
tests/
    ├── inventory
    └── test.yml
tasks/
    ├── file_permissions.yml
    ├── main.yml
    ├── port_scan.yml
    ├── rootkit_scan.yml
    ├── security_updates.yml
    ├── ssh_audit.yml
    └── user_audit.yml
handlers/
    └── main.yml
defaults/
    └── main.yml
README.md
```