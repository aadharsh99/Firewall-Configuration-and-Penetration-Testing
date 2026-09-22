# Testing Methodology

The lab uses three VirtualBox VMs:

- **pfSense** — firewall/router
- **Kali Linux** — authorized test source
- **Ubuntu 24.04** — target system

## Network topology

```text
Kali Linux
192.168.0.35
    |
    | Bridged
    |
pfSense WAN
192.168.0.205
    |
pfSense LAN
192.168.20.1
    |
    | Internal Network: firewall-lan
    |
Ubuntu 24.04
192.168.20.10
```

The tests use pfSense for perimeter/routing controls and nftables on Ubuntu for host-based filtering.
