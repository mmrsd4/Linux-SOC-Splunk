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