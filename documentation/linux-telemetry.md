# Linux Telemetry

## Authentication Logs

Ubuntu records authentication activity in:

```text
/var/log/auth.log
```

The log contains SSH authentication events including:

* Successful login
* Failed login
* Session creation
* Session termination

## System Logs

Ubuntu system activity is recorded in:

```text
/var/log/syslog
```

System activity from services and processes can be observed in this log.

## Validated SSH Events

Successful SSH authentication:

```text
Accepted password for socuser from 192.168.50.10
```

Failed SSH authentication:

```text
Failed password for socuser from 192.168.50.10
```

## Evidence

* `08-auth-log.png`
* `11-syslog.png`

## Result

Ubuntu is generating authentication and system telemetry required for the later Splunk log-collection and detection stages.
