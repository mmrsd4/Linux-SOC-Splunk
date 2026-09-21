# Auditd Monitoring

## Auditd Service

Auditd is installed and running on `LINUX-SRV-01`.

* Service: `auditd.service`
* Status: Active (running)
* Enabled at boot: Yes
* Log file: `/var/log/audit/audit.log`

## Audit Rule

The following rule was configured:

```text
-w /home -p wa -k home_changes
```

This monitors write and attribute changes under `/home`.

## Test Activity

A test file was created and modified:

```text
/home/socuser/audit-test.txt
```

The audit log recorded the activity with:

```text
key="home_changes"
```

The event also recorded the file path and process information.

## Evidence

* `12-auditd-status.png`
* `13-audit-rules.png`
* `14-audit-events.png`

## Result

Auditd is running and successfully recording monitored file activity.
