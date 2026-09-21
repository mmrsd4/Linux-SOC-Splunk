# Ubuntu SSH Configuration

## SSH Service

Ubuntu server `LINUX-SRV-01` is running OpenSSH.

* SSH service: `ssh.service`
* Status: Active (running)
* SSH port: 22
* SSH enabled at boot: Yes

## SOC User

A dedicated user was created for the lab:

* Username: `socuser`
* UID: `1002`
* GID: `1002`

## SSH Validation

A successful SSH login was performed from Kali:

```text
Source: 192.168.50.10
Target: 192.168.50.30
User: socuser
```

The Ubuntu authentication log recorded:

```text
Accepted password for socuser from 192.168.50.10
```

A controlled failed SSH login was also generated.

The authentication log recorded:

```text
Failed password for socuser from 192.168.50.10
```

## Evidence

* `07-ssh-service.png`
* `08-auth-log.png`
* `09-ssh-success.png`
* `10-ssh-failure.png`

## Result

SSH is configured and working. Both successful and failed SSH authentication events are being recorded in `/var/log/auth.log`.
