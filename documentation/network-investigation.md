# Network Investigation

## Objective

Capture and investigate a normal SSH connection between the Kali attack workstation and the Ubuntu Linux server.

## Hosts

| Host | IP Address | Role |
|---|---|---|
| Kali | 192.168.50.10 | Security testing workstation |
| Ubuntu | 192.168.50.30 | Linux server |

## Capture

Wireshark was used on the Kali `eth0` interface to capture SSH traffic between Kali and Ubuntu.

Display filter:

    ip.addr == 192.168.50.10 && ip.addr == 192.168.50.30 && tcp.port == 22

The capture showed TCP traffic between the two hosts using SSH destination port 22.

## TCP Connection

The capture was examined for the TCP connection establishment between Kali and Ubuntu.

The TCP handshake consisted of the expected SYN, SYN/ACK, and ACK packets.

## SSH Traffic

The SSH TCP stream was examined in Wireshark.

The SSH session traffic was encrypted rather than displayed as readable application commands.

## Evidence

- `screenshots/21-wireshark-ssh-capture.png`
- `screenshots/22-wireshark-tcp-handshake.png`
- `screenshots/23-wireshark-ssh-stream.png`

The raw PCAP was retained locally and is not included in the GitHub repository.
# Day 6 — Incident Network Investigation

## Objective

Investigate SSH network activity associated with the controlled post-compromise investigation.

The network investigation was correlated with host authentication, auditd, and Splunk evidence. The purpose was to confirm the network path and encrypted nature of the SSH session rather than recover plaintext commands or credentials.

## Incident SSH Path

The relevant SSH communication used:

- Kali: `192.168.50.10`
- Ubuntu: `192.168.50.30`
- Protocol: SSH
- Destination port: `22`

Wireshark was used on the Kali `eth0` interface.

Display filter:

    ip.addr == 192.168.50.30 && tcp.port == 22

The capture was used to identify SSH traffic between the Kali workstation and Ubuntu server.

## SSH Stream Analysis

The SSH TCP stream was examined in Wireshark.

The stream showed encrypted SSH application traffic. The investigation did not treat the captured stream as evidence of readable passwords or plaintext commands.

The network evidence therefore supports:

1. TCP connectivity between Kali and Ubuntu.
2. SSH communication over TCP port 22.
3. Encrypted SSH application traffic.

## Correlation With Host Evidence

The network evidence was correlated with Ubuntu authentication and audit telemetry.

Host-side evidence provided the account and activity context, including:

- SSH authentication events for `socuser`
- SSH persistence-related filesystem activity
- Sensitive-file access recorded by auditd
- Splunk ingestion of the corresponding Linux telemetry

Wireshark provided the network-level view of the SSH connection, while `auth.log`, auditd, and Splunk provided host-level evidence.

## Evidence

Day 6 network evidence:

- `screenshots/38-wireshark-ssh-stream.png`

Earlier baseline SSH network evidence:

- `screenshots/21-wireshark-ssh-capture.png`
- `screenshots/22-wireshark-tcp-handshake.png`
- `screenshots/23-wireshark-ssh-stream.png`

The raw PCAP remains local and is excluded from the GitHub repository.

## Investigation Limitation

Because SSH encrypts application traffic, the Wireshark capture alone does not provide readable SSH commands or passwords.

Command-level and account-level investigation therefore relies on host telemetry such as `auth.log`, auditd, and Splunk rather than assuming plaintext content from the network capture.

