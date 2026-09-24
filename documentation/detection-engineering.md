# Detection Engineering

## Objective

Develop and validate Splunk detections for SSH authentication activity generated during a controlled Linux security lab exercise.

## Test Activity

The Kali workstation (`192.168.50.10`) was used to generate SSH authentication attempts against the Ubuntu server (`192.168.50.30`) for the `socuser` account.

Hydra was used with a local password wordlist to generate the controlled authentication activity.

The Ubuntu SSH logs recorded:

- Five failed password attempts from `192.168.50.10`
- A successful password authentication for `socuser` from `192.168.50.10`

A separate normal SSH login was also performed after the controlled test.

## Brute-Force Detection

The following Splunk search was used:

    index=linux_auth "Failed password"
    | stats count as failed_attempts by src_ip user
    | where failed_attempts >= 5

The detection returned:

| Source IP | User | Failed Attempts |
|---|---|---:|
| 192.168.50.10 | socuser | 5 |

This demonstrates that repeated failed SSH authentication attempts can be identified by source IP and username.

## Successful SSH Detection

The following search was used:

    index=linux_auth "Accepted password"
    | stats count as successful_logins by src_ip user

The search returned:

| Source IP | User | Successful Logins |
|---|---|---:|
| 192.168.50.10 | socuser | 3 |

## Authentication Correlation

The following search was used to display failed and successful authentication events chronologically:

    index=linux_auth ("Failed password" OR "Accepted password") "socuser"
    | sort 0 _time
    | table _time src_ip user _raw

The results showed authentication activity involving `socuser` from `192.168.50.10`, including failed and successful SSH authentication events.

## Detection Evidence

- `screenshots/27-splunk-ssh-bruteforce-detection.png`
- `screenshots/28-splunk-successful-ssh.png`
- `screenshots/29-splunk-ssh-correlation.png`

## Attack Simulation Evidence

- `screenshots/24-hydra-ssh-attack.png`
- `screenshots/25-ubuntu-ssh-failures.png`
- `screenshots/26-ubuntu-successful-ssh.png`

## Lab Scope

The SSH authentication activity was generated only within the isolated Linux SOC lab environment.

The raw password used during the test is not documented or stored in the repository.
---

## Day 6 — Post-Compromise Detection Engineering

### 1. SSH Persistence Detection

During the controlled persistence simulation, an Ed25519 key pair was created under:

`/home/socuser/.ssh/`

and the public key was added to:

`/home/socuser/.ssh/authorized_keys`

The existing audit rule:

    -w /home -p wa -k home_changes

recorded filesystem activity associated with these files.

A Splunk search used to locate the persistence-related audit records was:

    index=linux_audit ("authorized_keys" OR "soc_lab_key")
    | sort 0 - _time

Observed records included activity involving:

- `/home/socuser/.ssh/soc_lab_key`
- `/home/socuser/.ssh/soc_lab_key.pub`
- `/home/socuser/.ssh/authorized_keys`

This provides a detection method for suspicious SSH key and `authorized_keys` filesystem activity.

Evidence:

- `screenshots/33-ssh-persistence-audit.png`
- `screenshots/33-ssh-persistence-splunk.png`
- `screenshots/33-ssh-persistence.png`

### 2. Sensitive File Access Detection

A temporary audit rule was used to monitor reads of the controlled test file:

    -w /srv/company-data/customer-data.txt -p r -k sensitive_data

The file was then accessed as `socuser`.

The following Splunk search identified records associated with the audit key:

    index=linux_audit "sensitive_data"

The observed SYSCALL record contained:

- `uid=1001`
- `comm="cat"`
- `exe="/usr/bin/cat"`
- `success=yes`
- `key="sensitive_data"`

A separate PATH record identified:

`/srv/company-data/customer-data.txt`

A filename-focused search was:

    index=linux_audit "customer-data.txt"

This demonstrated that sensitive-file access can be detected by combining audit-key and filename searches.

Evidence:

- `screenshots/34-sensitive-file-access.png`
- `screenshots/35-splunk-sensitive-file-access.png`
- `screenshots/36-splunk-sensitive-file-path.png`

### 3. Failed Authentication Threshold

The existing SSH brute-force detection was validated during the controlled Hydra exercise:

    index=linux_auth "Failed password"
    | stats count as failed_attempts by src_ip user
    | where failed_attempts >= 5

Observed result:

| Source IP | User | Failed Attempts |
|---|---|---:|
| 192.168.50.10 | socuser | 5 |

This detection identifies repeated failed SSH authentication attempts by source IP and username.

### 4. Successful Authentication After Failed Attempts

Authentication activity can be investigated chronologically using:

    index=linux_auth ("Failed password" OR "Accepted password") "socuser"
    | sort 0 _time
    | table _time src_ip user message

This allows failed and successful authentication events to be reviewed together.

The controlled exercise demonstrated both failed and successful SSH authentication activity involving `socuser` and `192.168.50.10`.

### 5. Post-Compromise Correlation

The Day 6 investigation demonstrates a broader correlation workflow:

    SSH authentication
            |
            v
    Account session
            |
            v
    Privilege discovery
            |
            v
    SSH persistence
            |
            v
    Sensitive-file access
            |
            v
    auditd telemetry
            |
            v
    Splunk investigation
            |
            v
    Network evidence

The individual detections should be correlated with timestamps, source IP addresses, usernames, filenames, audit keys, and process information rather than treating a single event as proof of compromise.

### 6. Audit Telemetry Validation

The Linux audit index was validated in Splunk with:

    index=linux_audit | stats count

The observed environment contained thousands of audit events.

Recent events were confirmed with:

    index=linux_audit
    | sort 0 - _time
    | head 20
    | table _time host sourcetype source

Observed metadata included:

- host: `LINUX-SRV-01`
- sourcetype: `linux:audit`
- source: `/var/log/audit/audit.log`

This confirms that auditd telemetry was being forwarded from Ubuntu to Splunk during the investigation.

### Detection Limitations

The current audit ingestion has several limitations:

1. Related audit records such as `SYSCALL` and `PATH` may appear as separate Splunk events.
2. The `message` field was not populated for the tested audit events, so detections should use observed parsed fields and search terms instead of assuming `message` contains the full audit record.
3. The broad `home_changes` rule also records filesystem activity generated by legitimate investigation and administrative commands.
4. The current detections demonstrate lab validation and are not production-tuned detection rules.

### Day 6 Detection Evidence

- `screenshots/30-post-compromise-session.png`
- `screenshots/31-post-compromise-enumeration.png`
- `screenshots/32-socuser-sudo-check.png`
- `screenshots/33-ssh-persistence-audit.png`
- `screenshots/33-ssh-persistence-splunk.png`
- `screenshots/33-ssh-persistence.png`
- `screenshots/34-sensitive-file-access.png`
- `screenshots/35-splunk-sensitive-file-access.png`
- `screenshots/36-splunk-sensitive-file-path.png`
- `screenshots/38-wireshark-ssh-stream.png`

