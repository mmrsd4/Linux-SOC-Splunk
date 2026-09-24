# Incident Timeline

## Incident Scope

This timeline documents the controlled SSH compromise simulation and post-compromise investigation performed in the Linux SOC & Splunk lab.

Source systems:

- Kali: `192.168.50.10`
- Ubuntu: `192.168.50.30`
- Splunk: `192.168.50.40`
- Target account: `socuser`

All timestamps below are taken from observed lab evidence. No timestamps are inferred.

## Timeline

| Date/Time | Source | Event | Evidence |
|---|---|---|---|
| 2026-09-23 14:14:28 | Ubuntu `/var/log/auth.log` | Multiple SSH authentication failures for `socuser` from `192.168.50.10` | `auth.log` |
| 2026-09-23 14:14:29 | Ubuntu `/var/log/auth.log` | Successful password authentication for `socuser` from `192.168.50.10` | `auth.log` |
| 2026-09-23 14:14:31 | Ubuntu `/var/log/auth.log` | Five failed SSH password attempts for `socuser` from `192.168.50.10` | `auth.log` |
| 2026-09-23 14:18:11 | Ubuntu `/var/log/auth.log` | Later successful SSH authentication for `socuser` from `192.168.50.10` | `auth.log` |
| 2026-09-24 13:00:37 | Ubuntu `/var/log/auth.log` | Successful SSH authentication for `socuser` from `192.168.50.10` | `auth.log` |
| 2026-09-24 13:29:32 | Ubuntu `/var/log/auth.log` | Successful SSH authentication for `socuser` from `192.168.50.10` | `auth.log` |
| 2026-09-24 15:28:22 | Ubuntu auditd | Creation of `/home/socuser/.ssh/soc_lab_key` and `soc_lab_key.pub` was recorded by the `home_changes` audit rule | `audit.log` / Splunk |
| 2026-09-24 15:29:24 | Ubuntu `/var/log/auth.log` | Successful SSH authentication for `socuser` from `192.168.50.30` | `auth.log` |
| 2026-09-24 15:29:24 | Ubuntu auditd | Activity involving `/home/socuser/.ssh/authorized_keys` was recorded | `audit.log` / Splunk |
| 2026-09-24 15:56:47 | Ubuntu auditd | Temporary `sensitive_data` audit rule was added | `audit.log` / Splunk |
| 2026-09-24 15:57:21 | Ubuntu auditd | `socuser` successfully accessed `/srv/company-data/customer-data.txt` using `/usr/bin/cat` | `audit.log` / Splunk |
| 2026-09-24 15:57:21 | Ubuntu auditd | Corresponding PATH record identified `/srv/company-data/customer-data.txt` | `audit.log` / Splunk |
| 2026-09-24 16:10:12 | Ubuntu auditd | Temporary `sensitive_data` audit rule was removed | `audit.log` / Splunk |

## Key Investigation Findings

### SSH Authentication

The controlled Hydra exercise from Kali (`192.168.50.10`) generated multiple failed SSH authentication events for `socuser`, followed by a successful authentication.

Splunk detection identified the source IP as `192.168.50.10`, the target account as `socuser`, and five failed password attempts in the tested sequence.

### SSH Persistence

An Ed25519 SSH key pair was created under `/home/socuser/.ssh/`.

The public key was added to:

`/home/socuser/.ssh/authorized_keys`

The private key was kept outside the Git repository.

The existing `home_changes` audit rule recorded filesystem activity associated with the SSH key files and `authorized_keys`. Splunk also received audit records for this activity.

### Privilege Discovery

The `socuser` account was tested for sudo access.

Observed result:

`Sorry, user socuser may not run sudo on LINUX-SRV-01.`

No artificial sudo permission was added for `socuser`.

The separate `linux-srv-01` account has administrative sudo privileges and was used for controlled lab administration.

### Sensitive Data Access

A controlled test file was created:

`/srv/company-data/customer-data.txt`

The file contained only:

`LAB CUSTOMER DATA - NOT REAL`

The file was successfully read as `socuser`.

A temporary audit rule:

`-w /srv/company-data/customer-data.txt -p r -k sensitive_data`

was then added.

Auditd recorded the resulting access with:

- `uid=1001`
- `comm="cat"`
- `exe="/usr/bin/cat"`
- `success=yes`
- `key="sensitive_data"`

Splunk received the corresponding SYSCALL and PATH records as separate events.

The temporary audit rule was subsequently removed.

### Audit and SIEM Limitations

The current audit ingestion sends related audit records such as `SYSCALL` and `PATH` as separate Splunk events.

For the sensitive-file access, the SYSCALL event contained the executing user and process information, while the separate PATH event contained the filename.

Therefore, the current Splunk configuration does not automatically present the complete audit event as a single combined record.

Investigation commands executed by the administrative account also generated `home_changes` audit activity. These records were treated as investigation noise and were not interpreted as attacker actions.

### Network Evidence

Wireshark was used to inspect SSH traffic between the Kali and Ubuntu systems.

The capture demonstrates TCP/22 SSH communication and the encrypted SSH stream. The capture does not establish that passwords or shell commands were visible in plaintext.

## Evidence Files

### SSH Authentication and Detection

- `screenshots/24-hydra-ssh-attack.png`
- `screenshots/25-ubuntu-ssh-failures.png`
- `screenshots/26-ubuntu-successful-ssh.png`
- `screenshots/27-splunk-ssh-bruteforce-detection.png`
- `screenshots/28-splunk-successful-ssh.png`
- `screenshots/29-splunk-ssh-correlation.png`

### Post-Compromise Investigation

- `screenshots/30-post-compromise-session.png`
- `screenshots/31-post-compromise-enumeration.png`
- `screenshots/32-socuser-sudo-check.png`

### SSH Persistence

- `screenshots/33-ssh-persistence-audit.png`
- `screenshots/33-ssh-persistence-splunk.png`
- `screenshots/33-ssh-persistence.png`

### Sensitive Data Access

- `screenshots/34-sensitive-file-access.png`
- `screenshots/35-splunk-sensitive-file-access.png`
- `screenshots/36-splunk-sensitive-file-path.png`

### Network Investigation

- `screenshots/38-wireshark-ssh-stream.png`

Raw PCAP files and private SSH keys are intentionally excluded from the Git repository.