# Firewall Security Testing Lab

> **Reconstruction note:** The original project document was lost. The lab was therefore recreated using retained screenshots from the original work, which documented the firewall configurations and testing performed. These screenshots were used as the reference to reproduce and organize the same lab scenarios, configurations, and evidence in this repository.

A hands-on firewall security testing lab using **pfSense** and **nftables** to configure, test, and document network access-control policies in an isolated VirtualBox environment.

## Objectives

- Configure firewall rules for inbound and outbound traffic.
- Test access-control restrictions against an authorized Kali Linux test host.
- Validate firewall behavior using Nmap, curl, ping, netcat, and nftables counters.
- Demonstrate stateful filtering and connection-rate limiting.
- Document configuration, methodology, and observed results with reproducible evidence.

## Lab Environment

| Component | Role | Address |
|---|---|---|
| Kali Linux | Authorized security-testing host | `192.168.0.35` |
| pfSense WAN | Firewall WAN interface | `192.168.0.205` |
| pfSense LAN | Firewall gateway | `192.168.20.1` |
| Ubuntu 24.04 | Target / host firewall | `192.168.20.10` |

The Kali-to-Ubuntu traffic is routed through pfSense. Ubuntu also runs local nftables rules for host-based filtering.

## Test Matrix

| Section | Control tested | Primary service/protocol | Validation |
|---|---|---|---|
| pfSense Test 1 | Inbound access control | SSH / TCP 22 | Nmap |
| pfSense Test 2 | Service-specific filtering | HTTP / TCP 80 and SSH / TCP 22 | Nmap + curl |
| pfSense Test 3 | ICMP filtering | ICMP | ping |
| nftables Test 1 | Outbound filtering | HTTP / TCP 80 | curl + nftables counter |
| nftables Test 2 | Stateful/source-restricted access control | SSH / TCP 22 | Nmap + nftables counters |
| nftables Test 3 | Connection-rate limiting | FTP / TCP 21 | netcat + nftables counters |

## Evidence

### 1. Lab Environment

The initial screenshots document the Ubuntu target address, route, and SSH service.

[Lab screenshots](screenshots/01-lab-environment/)

### 2. pfSense Test 1 — SSH Access Control

A pfSense WAN rule permits TCP/22 from Kali to the Ubuntu target. The test validates the difference between the permitted and filtered SSH states.

[pfSense Test 1 evidence](screenshots/02-pfsense-test-1-ssh-access-control/)

### 3. pfSense Test 2 — HTTP Allowed / SSH Blocked

The HTTP rule permits TCP/80 to Ubuntu while SSH remains filtered. Nmap and curl provide the validation evidence.

[pfSense Test 2 evidence](screenshots/03-pfsense-test-2-http-allowed-ssh-blocked/)

### 4. pfSense Test 3 — ICMP Filtering

ICMP connectivity is tested and then blocked using a pfSense rule.

[pfSense Test 3 evidence](screenshots/04-pfsense-test-3-icmp-filtering/)

### 5. nftables Test 1 — Outbound HTTP Filtering

Ubuntu's nftables output chain contains a TCP/80 drop rule. HTTP succeeds before the rule is applied and times out after the rule is active, while HTTPS remains available.

[Rule](rules/nftables/test-1-outbound-http-filtering.nft) · [Evidence](screenshots/05-nftables-test-1-outbound-http-filtering/)

### 6. nftables Test 2 — Stateful SSH Access Control

The input policy is set to drop, with loopback, established/related traffic, ICMP from Kali, and SSH from Kali explicitly permitted. The SSH rule is then removed and Nmap shows TCP/22 as filtered.

[Rule](rules/nftables/test-2-stateful-ssh-access-control.nft) · [Evidence](screenshots/06-nftables-test-2-stateful-ssh-access-control/)

### 7. nftables Test 3 — FTP Connection Rate Limiting

Ubuntu's nftables input chain limits new FTP connections to **5 per minute with a burst of 5 packets**, followed by a drop rule for additional new connections. Ten connection attempts from Kali were used to generate measurable firewall counters.

[Rule](rules/nftables/test-3-ftp-connection-rate-limiting.nft) · [Evidence](screenshots/07-nftables-test-3-ftp-connection-rate-limiting/)

## Observed Results

The repository records the results visible in the supplied lab evidence rather than presenting expected behavior as completed results.

- pfSense SSH test: Nmap showed `22/tcp open ssh` when permitted and `22/tcp filtered ssh` after the access rule was not permitted.
- pfSense HTTP/SSH test: Nmap showed `80/tcp open http` and `22/tcp filtered ssh`; curl returned the Ubuntu Apache page.
- pfSense ICMP test: the evidence includes successful ping traffic and the subsequent ICMP block rule/test.
- nftables outbound HTTP test: HTTP returned successfully before filtering, timed out after the TCP/80 drop rule, and HTTPS remained available.
- nftables stateful SSH test: the initial ruleset allowed SSH from Kali; after the SSH rule was removed, Nmap reported `22/tcp filtered ssh`.
- nftables FTP rate-limit test: the final counter screenshot shows **5 packets accepted** by the rate-limit rule and **9 packets dropped** by the following rule.

## Repository Structure

```text
firewall-security-testing-lab/
├── README.md
├── screenshots/
│   ├── 01-lab-environment/
│   ├── 02-pfsense-test-1-ssh-access-control/
│   ├── 03-pfsense-test-2-http-allowed-ssh-blocked/
│   ├── 04-pfsense-test-3-icmp-filtering/
│   ├── 05-nftables-test-1-outbound-http-filtering/
│   ├── 06-nftables-test-2-stateful-ssh-access-control/
│   └── 07-nftables-test-3-ftp-connection-rate-limiting/
├── rules/
│   ├── pfsense/
│   └── nftables/
└── docs/
    └── testing-methodology.md
```

## Tools

- pfSense
- nftables
- Kali Linux
- Ubuntu 24.04
- Nmap
- curl
- netcat
- Apache2
- vsftpd
- VirtualBox

## Scope

All testing was performed in an isolated lab environment against systems under the lab operator's control. The techniques and results are intended for authorized security testing and firewall validation.
