# Linux SOC & Splunk Lab Setup

## 1. Lab Architecture

| System        | Hostname      | IP Address         | Network Interface |
| ------------- | ------------- | ------------------ | ----------------- |
| Kali Linux    | kali          | 192.168.50.10/24   | eth0              |
| Ubuntu Server | LINUX-SRV-01  | 192.168.50.30/24   | ens33             |
| Splunk Server | splunk-SRV-01 | 192.168.50.40/24   | ens33             |

The SOC lab uses the `192.168.50.0/24` network for communication between Kali, Ubuntu, and Splunk.

Splunk has a second network interface, `ens37`, with `192.168.126.151/24`.

## 2. IP Address Validation

### Kali

* IP: `192.168.50.10/24`
* Interface: `eth0`

### Ubuntu

* IP: `192.168.50.30/24`
* Interface: `ens33`
* Hostname: `LINUX-SRV-01`

### Splunk

* Primary SOC IP: `192.168.50.40/24`
* Interface: `ens33`

## 3. Connectivity Validation

### Kali → Ubuntu

```text
ping -c 4 192.168.50.30
```

Result:

```text
4 packets transmitted, 4 received, 0% packet loss
```

Status: PASS

### Kali → Splunk

```text
ping -c 4 192.168.50.40
```

Result:

```text
4 packets transmitted, 4 received, 0% packet loss
```

Status: PASS

### Ubuntu → Splunk

```text
ping -c 4 192.168.50.40
```

Result:

```text
4 packets transmitted, 4 received, 0% packet loss
```

Status: PASS

## 4. Evidence

* `01-vmware-network.png` — VMware network configuration
* `02-kali-ip.png` — Kali IP configuration
* `03-ubuntu-ip.png` — Ubuntu IP configuration
* `04-splunk-ip.png` — Splunk IP configuration
* `05-lab-connectivity.png` — Connectivity tests
* `06-architecture.png` — Lab architecture
