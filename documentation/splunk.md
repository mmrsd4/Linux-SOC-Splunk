# Splunk Configuration

## Splunk Server

- Hostname: splunk-SRV-01
- IP address: 192.168.50.40
- Splunk Enterprise: 10.4.3

## Receiving Port

Splunk was configured to receive forwarded events on TCP port 9997.

The receiver was verified as listening on:

    0.0.0.0:9997

## Indexes

Three Linux-specific indexes were created:

- `linux_auth`
- `linux_syslog`
- `linux_audit`

## Validation

The Splunk Search & Reporting interface was used to verify that events were received from the Ubuntu server.

The searches returned Linux authentication, syslog, and auditd events.

Evidence:

- `evidence/15-splunk-receiver.png`
- `evidence/16-uf-status.png`
- `evidence/17-splunk-auth-events.png`
- `evidence/18-splunk-syslog-events.png`
- `evidence/19-splunk-audit-events.png`
- `evidence/20-log-pipeline-validation.png`