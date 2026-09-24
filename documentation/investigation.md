# Linux SSH Compromise Investigation

## Objective

This investigation documents a controlled SSH compromise simulation and the post-compromise investigation performed in the Linux SOC & Splunk lab.

## Lab Systems

- Kali: `192.168.50.10`
- Ubuntu: `192.168.50.30`
- Splunk: `192.168.50.40`
- Target account: `socuser`

## 1. SSH Access

A successful SSH session was established from Kali to Ubuntu as `socuser`.

Observed:

```text
whoami: socuser
hostname: LINUX-SRV-01
UID: 1001
````

Evidence:

* `screenshots/30-post-compromise-session.png`

## 2. System Enumeration

The following commands were used during investigation:

```bash
whoami
id
hostname
uname -a
ip addr
ss -tulpn
sudo -l
```

The Ubuntu server was identified as:

```text
LINUX-SRV-01
192.168.50.30
```

SSH was listening on TCP port 22.

The `socuser` account was checked for sudo access.

Observed result:

```text
Sorry, user socuser may not run sudo on LINUX-SRV-01.
```

No sudo permission was added to `socuser`.

Evidence:

* `screenshots/31-post-compromise-enumeration.png`
* `screenshots/32-socuser-sudo-check.png`

## 3. SSH Persistence

An Ed25519 SSH key pair was created for the controlled persistence simulation.

Files created:

```text
/home/socuser/.ssh/soc_lab_key
/home/socuser/.ssh/soc_lab_key.pub
```

The public key was added to:

```text
/home/socuser/.ssh/authorized_keys
```

A subsequent SSH connection using the key successfully authenticated as `socuser`.

The existing audit rule:

```text
-w /home -p wa -k home_changes
```

recorded activity involving the SSH key files and `authorized_keys`.

Splunk also received these audit events.

Evidence:

* `screenshots/33-ssh-persistence-audit.png`
* `screenshots/33-ssh-persistence-splunk.png`
* `screenshots/33-ssh-persistence.png`

The private SSH key was not added to the repository.

## 4. Sensitive File Access

A synthetic test file was created:

```text
/srv/company-data/customer-data.txt
```

It contained:

```text
LAB CUSTOMER DATA - NOT REAL
```

The file was successfully read as `socuser`.

A temporary audit rule was added:

```text
-w /srv/company-data/customer-data.txt -p r -k sensitive_data
```

Auditd recorded the file access.

Important observed fields included:

```text
uid=1001
comm="cat"
exe="/usr/bin/cat"
success=yes
key="sensitive_data"
```

Splunk received the related audit events.

The temporary audit rule was removed after testing.

Evidence:

* `screenshots/34-sensitive-file-access.png`
* `screenshots/35-splunk-sensitive-file-access.png`
* `screenshots/36-splunk-sensitive-file-path.png`

## 5. Network Investigation

Wireshark was used to inspect SSH traffic between:

```text
192.168.50.10
192.168.50.30
```

The traffic used TCP port 22.

The SSH stream was encrypted. The capture does not show plaintext SSH passwords or commands.

Evidence:

* `screenshots/38-wireshark-ssh-stream.png`

The raw PCAP was kept outside the Git repository.

## 6. Authentication Investigation

Ubuntu `/var/log/auth.log` recorded failed and successful SSH authentication attempts for `socuser`.

The Day 5 controlled Hydra exercise generated multiple failed authentication events and a successful authentication.

Splunk was used to identify:

* failed SSH authentication
* successful SSH authentication
* source IP
* username
* chronological authentication activity

Example detection:

```spl
index=linux_auth "Failed password"
| stats count as failed_attempts by src_ip user
| where failed_attempts >= 5
```

## 7. Evidence Correlation

The investigation correlated:

* SSH authentication logs
* auditd events
* Splunk events
* SSH persistence activity
* sensitive-file access
* Wireshark network traffic

The detailed timeline is available in:

```text
incident/timeline.md
```

## 8. Investigation Limitations

Auditd `SYSCALL` and `PATH` records were received by Splunk as separate events.

Therefore, process information and file-path information may appear in different Splunk events.

Investigation commands also generated some audit activity. These records were treated as investigation noise rather than attacker activity.

SSH traffic was encrypted, so Wireshark did not provide plaintext SSH commands or passwords.

## 9. Conclusion

The controlled lab demonstrated how SSH authentication, Linux auditd, Splunk, filesystem activity, and network traffic can be correlated during a Linux security investigation.

All sensitive data used in the exercise was synthetic lab data.

