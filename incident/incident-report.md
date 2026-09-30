# SOC Incident Report - Simulated SSH Compromise

## 1. Executive Summary

This report documents a controlled Linux SSH compromise simulation performed in a VMware-based SOC lab.

The exercise simulated an SSH authentication attack against an Ubuntu Linux server, followed by post-compromise investigation, SSH persistence, controlled sensitive-data access, SIEM correlation, incident containment, eradication, recovery, and validation.

The investigation used:

- Kali Linux
- Ubuntu Linux
- Splunk
- Linux auditd
- OpenSSH
- Wireshark
- VMware virtual networking

The primary systems were:

| System | IP Address | Role |
|---|---|---|
| Kali | `192.168.50.10` | Attack simulation / analyst workstation |
| Ubuntu | `192.168.50.30` | Linux target / monitored endpoint |
| Splunk | `192.168.50.40` | SIEM |

The simulated target account was `socuser`.

---

## 2. Incident Scope

The exercise was performed within the controlled VMware lab environment:

`192.168.50.0/24`

The scenario focused on:

1. SSH authentication activity
2. Failed authentication detection
3. Successful authentication detection
4. Post-compromise enumeration
5. SSH key persistence
6. Sensitive-file access
7. Authentication and endpoint telemetry correlation
8. Incident containment
9. Persistence eradication
10. Recovery and validation

The exercise used controlled lab data and did not involve real customer information.

---

## 3. Attack Simulation

A controlled SSH authentication attack was performed from Kali:

`192.168.50.10`

against Ubuntu:

`192.168.50.30`

for the account:

`socuser`

Hydra was used with a local lab password list.

The observed exercise generated multiple failed SSH authentication attempts followed by a successful authentication.

Ubuntu `/var/log/auth.log` recorded the authentication activity, while Splunk received the corresponding Linux authentication telemetry through the Universal Forwarder.

---

## 4. Initial Detection

The SSH authentication telemetry was forwarded to Splunk using the `linux_auth` index.

A brute-force detection was developed using failed SSH authentication events:

```spl
index=linux_auth "Failed password"
| stats count as failed_attempts by src_ip user
| where failed_attempts >= 5
```

The observed lab activity identified:

* Source IP: `192.168.50.10`
* Account: `socuser`
* Failed attempts: 5

A successful-login detection was also developed using:

```spl
index=linux_auth "Accepted password"
| stats count by src_ip user
```

The authentication activity was then correlated chronologically to examine the relationship between failed and successful SSH authentication.

---

## 5. Post-Compromise Investigation

After establishing the simulated compromised session, post-compromise investigation was performed on the Ubuntu system.

The investigation included:

* Account identification
* Group membership
* Hostname
* Kernel and operating-system information
* Network configuration
* Listening services
* SSH configuration
* Sudo privileges

The `socuser` account was verified as a non-sudo user.

The administrative `linux-srv-01` account was separately identified as having unrestricted sudo access.

No artificial sudo permission was added to `socuser` during the exercise.

---

## 6. SSH Persistence

The exercise demonstrated SSH public-key persistence using an Ed25519 key pair.

The persistence artifacts were created under:

`/home/socuser/.ssh/`

The lab-generated files were:

* `soc_lab_key`
* `soc_lab_key.pub`

The public key was placed in:

`/home/socuser/.ssh/authorized_keys`

A key-based SSH connection was successfully established using the generated lab key.

The private key was kept outside the Git repository.

---

## 7. Persistence Detection

Ubuntu auditd monitored filesystem changes under `/home` using:

```text
-w /home -p wa -k home_changes
```

Audit telemetry recorded activity involving:

* `/home/socuser/.ssh/soc_lab_key`
* `/home/socuser/.ssh/soc_lab_key.pub`
* `/home/socuser/.ssh/authorized_keys`

The corresponding audit telemetry was forwarded to Splunk using the `linux_audit` index.

The investigation demonstrated that auditd records can contain separate SYSCALL and PATH records. Splunk visibility therefore required examination of the relevant audit fields rather than relying on a single normalized event.

---

## 8. Sensitive-Data Access Investigation

A controlled fake data file was created:

`/srv/company-data/customer-data.txt`

The file contained:

`LAB CUSTOMER DATA - NOT REAL`

A temporary audit rule was added specifically for the controlled read-access test:

```text
-w /srv/company-data/customer-data.txt -p r -k sensitive_data
```

The file was accessed using:

```text
/usr/bin/cat
```

Auditd recorded the access, including a SYSCALL event associated with the `sensitive_data` key and a PATH record identifying the file.

The telemetry was successfully ingested into Splunk.

The temporary audit rule was subsequently removed.

---

## 9. Network Investigation

Wireshark was used to investigate SSH network traffic between Kali and Ubuntu.

The investigation focused on:

* TCP/22 communication
* TCP connection establishment
* SSH traffic
* Follow TCP Stream

The capture demonstrated that the SSH session was encrypted.

No claim was made that plaintext SSH commands or passwords could be recovered from the captured stream.

Raw PCAP files were intentionally kept outside the Git repository.

---

## 10. Incident Timeline

The detailed chronological evidence is maintained separately in:

`incident/timeline.md`

The timeline uses observed timestamps from:

* Ubuntu `/var/log/auth.log`
* Ubuntu auditd
* Splunk
* Network investigation evidence

No timestamps are inferred when documenting the incident timeline.

---

## 11. Incident Response

The response followed a controlled incident-response sequence:

```text
Detect
  |
Investigate
  |
Contain
  |
Eradicate
  |
Recover
  |
Validate


### Containment

The lab-generated public-key persistence entry was removed from:

`/home/socuser/.ssh/authorized_keys`

A temporary backup was created during the remediation procedure.

### Eradication

The lab-generated persistence artifacts were removed:

* `/home/socuser/.ssh/soc_lab_key`
* `/home/socuser/.ssh/soc_lab_key.pub`
* `/home/socuser/.ssh/authorized_keys.day7-backup`

### Validation

The final validation confirmed:

* `authorized_keys` was 0 bytes
* `soc_lab_key` was removed
* `soc_lab_key.pub` was removed
* `authorized_keys.day7-backup` was removed
* SSH remained active
* the temporary `sensitive_data` audit rule was absent
* the persistent `home_changes` audit rule remained active

### Recovery

A subsequent password-based SSH connection from Kali to Ubuntu successfully authenticated as `socuser`.

The session verified:

```text
whoami = socuser
hostname = LINUX-SRV-01
```

---

## 12. SIEM Architecture

The lab used Splunk as the central SIEM.

Linux telemetry was collected through the Splunk Universal Forwarder.

The primary indexes were:

| Index          | Telemetry                 |
| -------------- | ------------------------- |
| `linux_auth`   | SSH/authentication events |
| `linux_syslog` | Linux syslog              |
| `linux_audit`  | auditd events             |

The Splunk receiver operated on TCP:

`9997`

The Ubuntu Universal Forwarder monitored:

```text
/var/log/auth.log
/var/log/syslog
/var/log/audit/audit.log
```

---

## 13. Detection and Investigation Coverage

The completed exercise demonstrated SOC coverage across:

| Area                        | Evidence               |
| --------------------------- | ---------------------- |
| SSH authentication          | `auth.log`, Splunk     |
| Brute-force detection       | Splunk                 |
| Successful authentication   | Splunk                 |
| Authentication correlation  | Splunk                 |
| Post-compromise enumeration | Ubuntu                 |
| SSH persistence             | auditd, Splunk         |
| Sensitive-file access       | auditd, Splunk         |
| Network investigation       | Wireshark              |
| Incident timeline           | `incident/timeline.md` |
| Containment                 | Ubuntu                 |
| Eradication                 | Ubuntu                 |
| Recovery                    | SSH validation         |
| Final validation            | Ubuntu                 |

---

## 14. Evidence Handling

The project repository contains screenshots and documentation supporting the investigation.

Sensitive artifacts were intentionally excluded from Git:

* Private SSH keys
* Raw PCAP files
* Temporary lab passwords

The exercise used controlled laboratory data rather than real customer information.

---

## 15. Limitations

This was a controlled SOC laboratory exercise rather than a production incident.

Important limitations include:

* The SSH attack was intentionally simulated.
* The sensitive data file contained fake laboratory data.
* Auditd PATH and SYSCALL records may appear as separate Splunk events.
* The existing `/home` audit rule monitors write/attribute activity and does not automatically provide read-access monitoring for arbitrary files.
* A temporary audit rule was therefore used for the controlled sensitive-file read exercise.
* Network captures demonstrated encrypted SSH traffic but did not provide plaintext session contents.
* Detection thresholds used in the lab are demonstration values and would require tuning for a production environment.

---

## 16. Conclusion

This project demonstrated an end-to-end Linux SOC workflow using SSH attack simulation, Linux authentication logs, auditd, Splunk, Wireshark, incident investigation, and incident response.

The exercise progressed from simulated authentication compromise through detection, investigation, persistence analysis, sensitive-data monitoring, containment, eradication, recovery, and final validation.
