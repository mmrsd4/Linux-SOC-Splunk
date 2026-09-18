# Linux SOC & Splunk

A hands-on Linux Security Operations Center lab using Kali Linux, Ubuntu Server, Splunk Enterprise, auditd, and Wireshark.

## Lab Architecture

| System        | IP Address      | Role                             |
| ------------- | --------------- | -------------------------------- |
| Kali Linux    | 192.168.50.10   | Attacker / Network Investigation |
| Ubuntu Server | 192.168.50.30   | Monitored Linux Server           |
| Splunk Server | 192.168.50.40   | SIEM / Detection                 |
| Splunk Server | 192.168.126.151 | Secondary Network Interface      |

SOC network:

```text
192.168.50.0/24
```

## Current Status

### Day 1 — Completed

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
