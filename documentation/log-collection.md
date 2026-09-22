# Linux Log Collection

## Log Pipeline

The Linux telemetry pipeline is:

    Ubuntu Linux Server
          |
          | Universal Forwarder
          |
          v
    TCP 9997
          |
          v
    Splunk Enterprise
          |
          v
    Linux-specific indexes

## Collected Logs

The Universal Forwarder on `LINUX-SRV-01` monitors:

| Log | Splunk Index | Sourcetype |
|---|---|---|
| `/var/log/auth.log` | `linux_auth` | `linux_secure` |
| `/var/log/syslog` | `linux_syslog` | `syslog` |
| `/var/log/audit/audit.log` | `linux_audit` | `linux:audit` |

## Forwarding Validation

The Universal Forwarder reported:

    Active forwards:
        192.168.50.40:9997

    Configured but inactive forwards:
        None

The forwarder was restarted after configuring the log monitors and was confirmed to be running.

## Splunk Validation

The following searches were used:

    index=linux_auth "sshd"

    index=linux_syslog

    index=linux_audit "home_changes"

The searches returned events from the Ubuntu server.

A combined validation search was also performed:

    index=linux_auth OR index=linux_syslog OR index=linux_audit
    | stats count by index

The final validation returned 472 events in the combined search.

## Evidence

- `evidence/16-uf-status.png`
- `evidence/17-splunk-auth-events.png`
- `evidence/18-splunk-syslog-events.png`
- `evidence/19-splunk-audit-events.png`
- `evidence/20-log-pipeline-validation.png`