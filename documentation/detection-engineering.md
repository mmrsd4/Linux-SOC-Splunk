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