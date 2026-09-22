# Linux SOC & Splunk

A hands-on Linux Security Operations Center lab using Kali Linux, Ubuntu Server, Splunk Enterprise, auditd, and Wireshark.

## Lab Architecture

| System        | IP Address      | Role                             |
| ------------- | --------------- | -------------------------------- |
| Kali Linux    | 192.168.50.10   | Attacker / Network Investigation |
| Ubuntu Server | 192.168.50.30   | Monitored Linux Server           |
| Splunk Server | 192.168.50.40   | SIEM / Detection                 |

SOC network:

```text
192.168.50.0/24
```

## Current Status

# Day 1 — Initialize Linux SOC lab architecture

### Completed

* VMware lab network configured
* Kali configured with `192.168.50.10`
* Ubuntu configured with `192.168.50.30`
* Splunk configured with `192.168.50.40`
* Ubuntu hostname verified as `LINUX-SRV-01`
* Kali → Ubuntu connectivity verified
* Kali → Splunk connectivity verified
* Ubuntu → Splunk connectivity verified

## Planned Telemetry

Ubuntu will provide:

* SSH authentication logs
* System logs
* auditd security events

Splunk will collect and analyze these logs.

Wireshark will provide network-level evidence during the attack simulation.

## Planned Attack Scenario

The lab will simulate:

1. Network reconnaissance
2. SSH brute-force attempts
3. Successful SSH login
4. Linux enumeration
5. Privileged activity
6. SSH persistence
7. Sensitive-file access

## Planned Detection

The project will build detections for:

* SSH brute force
* Successful login after failures
* Suspicious SSH activity
* Privileged activity
* SSH persistence
* Sensitive-file access
* Post-compromise activity

## Repository Structure

```text
Linux-SOC-Splunk/
├── README.md
├── documentation/
├── evidence/
├── screenshots/
├── reports/
├── scripts/
└── .gitignore
```

## Evidence

Screenshots will document the actual configuration, commands, logs, Splunk searches, network traffic, detections, and incident-response activities.

No raw PCAP files will be committed to the public repository.

## Project Goal

Build and document a complete Linux SOC investigation workflow:

```text
Attack
  ↓
Linux Logs
  ↓
auditd
  ↓
Splunk
  ↓
Detection
  ↓
Investigation
  ↓
Incident Response
  ↓
Recovery
```

# Day 2 — Ubuntu SSH & Linux Telemetry

Day 2 completed the Ubuntu SSH configuration and Linux telemetry setup.

### Completed

* OpenSSH server installed and running
* SSH enabled at boot
* `socuser` created
* Successful SSH login tested from Kali
* Failed SSH login tested from Kali
* `/var/log/auth.log` verified
* `/var/log/syslog` verified

### Validated Flow

```text
Kali 192.168.50.10
        |
        | SSH
        v
Ubuntu 192.168.50.30
        |
        +--> /var/log/auth.log
        |
        +--> /var/log/syslog
```

The authentication logs successfully recorded both successful and failed SSH activity.

## Day 2 Evidence

* `07-ssh-service.png`
* `08-auth-log.png`
* `09-ssh-success.png`
* `10-ssh-failure.png`
* `11-syslog.png`


# Day 3 — Auditd Monitoring

Day 3 completed host-level security auditing on Ubuntu.

### Completed

* Auditd installed
* Auditd enabled and running
* Audit rules verified
* `/home` write and attribute changes monitored
* Controlled file activity generated
* Audit event successfully verified

### Audit Rule

```text
-w /home -p wa -k home_changes
```

### Telemetry Flow

```text
Ubuntu
  ↓
auditd
  ↓
/var/log/audit/audit.log
  ↓
Splunk
```

## Day 3 Evidence

* `12-auditd-status.png`
* `13-audit-rules.png`
* `14-audit-events.png`

## Day 4 — Splunk Log Collection

Configured centralized Linux log collection from Ubuntu `LINUX-SRV-01` to Splunk Enterprise.

### Collected Logs

- `/var/log/auth.log` → `linux_auth`
- `/var/log/syslog` → `linux_syslog`
- `/var/log/audit/audit.log` → `linux_audit`

### Forwarding

The Splunk Universal Forwarder forwards events to:

`192.168.50.40:9997`

The forwarding connection was verified as active.

### Validation

Splunk successfully received and indexed authentication, syslog, and auditd events from the Ubuntu server.

Final combined validation returned **472 events**.

### Evidence

- `15-splunk-receiver.png`
- `16-uf-status.png`
- `17-splunk-auth-events.png`
- `18-splunk-syslog-events.png`
- `19-splunk-audit-events.png`
- `20-log-pipeline-validation.png`