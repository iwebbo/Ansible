# Ansible Role: disk_space_report

Scans disk space on Linux/Unix hosts using `df -Ph`, classifies each partition against configurable warning and danger thresholds, prints alerts to the Ansible console, and generates per-host HTML and text summary reports on the controller.

Zero external dependencies — no Galaxy collections required (`community.general` is **not** needed).

## Requirements

- Ansible >= 2.14
- Target hosts: Linux/Unix with `df`, `awk`, `/bin/bash`
- Reports are generated on the **controller** via `delegate_to: localhost` (no `fetch` module used)

## Role Structure

```
roles/disk_space_report/
├── defaults/main.yml          # Thresholds, exclusions, report output dir
├── vars/main.yml              # Internal variables (report filenames)
├── tasks/main.yml             # Collect → Parse → Classify → Alert → Report
├── templates/
│   └── disk_report.html.j2   # Responsive HTML report with progress bars
├── handlers/main.yml          # Empty (read-only role)
└── meta/main.yml              # Galaxy metadata
```

## How It Works

1. **Collect** — Runs `df -Ph` on the remote host via shell, outputs one JSON object per line using `awk`.
2. **Parse** — Converts each JSON line with the built-in `from_json` filter. No external collections.
3. **Filter** — Excludes virtual filesystems (`tmpfs`, `devtmpfs`, `squashfs`, `overlay`, `none`) and system mount points matching `^/(snap|sys|proc|dev|run)`.
4. **Classify** — Each partition gets a status: `ok`, `warning` (usage >= warning threshold), or `danger` (usage >= danger threshold).
5. **Alert** — Prints colored warnings and danger alerts to the Ansible console output.
6. **Report** — Generates two files per host on the controller (via `delegate_to: localhost`):
   - `disk_report_<date>.html` — Responsive HTML report with system info, summary cards, alert banners, and a sortable partition table with color-coded progress bars.
   - `disk_report_summary_<date>.txt` — Plain text summary suitable for console display and email.

## Variables

### Defaults (overridable)

| Variable | Default | Description |
|---|---|---|
| `disk_warning_threshold` | `80` | Usage percentage that triggers a **warning** status |
| `disk_danger_threshold` | `90` | Usage percentage that triggers a **danger** status |
| `disk_report_dir` | `{{ playbook_dir }}/../disk_reports` | Report output directory on the controller |
| `disk_exclude_fstype` | `[tmpfs, devtmpfs, squashfs, overlay, none]` | Filesystem types to exclude from the scan |
| `disk_exclude_mount_regex` | `^/(snap\|sys\|proc\|dev\|run)` | Mount point regex to exclude |

### Internal (vars/main.yml)

| Variable | Value | Description |
|---|---|---|
| `disk_report_file_html` | `disk_report_{{ ansible_date_time.date }}.html` | HTML report filename |
| `disk_report_file_txt` | `disk_report_summary_{{ ansible_date_time.date }}.txt` | Text summary filename |

## Usage

### Basic

```yaml
- hosts: Linux
  become: yes
  roles:
    - role: disk_space_report
```

### Custom Thresholds

```yaml
- hosts: all
  become: yes
  roles:
    - role: disk_space_report
      vars:
        disk_warning_threshold: 75
        disk_danger_threshold: 85
```

### With Extra Vars (CLI)

```bash
ansible-playbook playbook/disk_space_report.yml \
  -e "HOST=Linux" \
  -e "disk_warning_threshold=70" \
  -e "disk_danger_threshold=90" \
  -e "disk_report_dir=/opt/reports"
```

## Report Output

Reports are written to the controller filesystem:

```
disk_reports/
├── 192.168.1.48/
│   ├── disk_report_2026-03-30.html
│   └── disk_report_summary_2026-03-30.txt
├── 192.168.1.85/
│   ├── disk_report_2026-03-30.html
│   └── disk_report_summary_2026-03-30.txt
└── ...
```

### Console Output Example

```
TASK [disk_space_report : Display disk status summary] *****
ok: [192.168.1.48] => (item=/) =>
  msg: "[OK] / - 63% used (71G/117G) - 42G free"
ok: [192.168.1.48] => (item=/boot/firmware) =>
  msg: "[OK] /boot/firmware - 16% used (78M/510M) - 433M free"

TASK [disk_space_report : Alert - Partitions in DANGER state] *****
ok: [10.0.0.5] => (item=/var/log) =>
  msg: "🔴 DANGER: /var/log at 95% usage (threshold: 90%)"
```

### HTML Report

The HTML report includes:

- **Header** — Hostname, OS, kernel, report date, configured thresholds
- **Summary cards** — Total, healthy, warning, and danger partition counts
- **Alert banners** — Conditional warning/danger banners listing affected partitions
- **Partition table** — Sorted by usage (descending), with color-coded status badges and progress bars

## Jenkins Integration

A Jenkinsfile (`disk_space_report.jenkinsfile`) is provided and follows the same pipeline pattern as other roles in the collection:

- **Agent**: Kubernetes pod (`agent-deploy-ansible`)
- **Credentials**: SSH key via `ssh-key-ansible-user-secret-file`
- **Parameters**: `TARGET_SERVER`, `WARNING_THRESHOLD`, `DANGER_THRESHOLD`, `ANSIBLE_VERBOSITY`, `EXTRA_VARS`
- **Post actions**: Displays partition summaries in console, archives HTML/TXT reports as build artifacts, sends email notification on success/failure

The Jenkinsfile passes `disk_report_dir='${WORKSPACE}/disk_reports'` so reports land directly in the Jenkins workspace for `archiveArtifacts` to pick up.

A Jenkins job description HTML file (`disk_space_report_job_description.html`) is also provided for the job configuration page.

## Tags

| Tag | Scope |
|---|---|
| `disk_scan` | Collection, parsing, filtering, classification |
| `disk_display` | Console status summary |
| `disk_alert` | Warning and danger console alerts |
| `disk_report` | HTML and text report generation |

## Supported Platforms

- Debian 11 (Bullseye), 12 (Bookworm)
- Ubuntu 20.04 (Focal), 22.04 (Jammy), 24.04 (Noble)
- RHEL / Rocky / AlmaLinux 8, 9
- Any Linux/Unix with `df -Ph` and `/bin/bash`

## License

MIT

## Author

AECoding