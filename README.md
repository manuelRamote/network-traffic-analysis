# Network Traffic Analysis Lab

> **Status:** In progress · Private working repository

A hands-on Blue Team laboratory for capturing, filtering, and interpreting network traffic with Wireshark. The goal is to document a reproducible analysis of **DNS**, **ICMP**, and **TCP/TLS** communications using traffic intentionally generated in a controlled environment.

## Objectives

- Identify the active network interface and capture traffic safely.
- Understand DNS queries and responses.
- Examine ICMP echo requests and replies.
- Identify the TCP three-way handshake and inspect TLS negotiation metadata (not decrypted application content).
- Document observations with filters, screenshots, and a concise technical report.

## Lab environment

| Component | Planned setup |
| --- | --- |
| Operating system | Kali GNU/Linux 2026.3 (observed in local test) |
| Network analysis | Wireshark 4.6.6, interface `eth0` |
| Test traffic | `ping`, `nslookup` or `dig`, `curl` |
| Test destination | `microsoft.com` (DNS) and `1.1.1.1` (ICMP) |

The OS and Wireshark versions were verified from local commands. ICMP and DNS captures have been reviewed; TCP/TLS testing is pending.

## Project structure

```text
network-traffic-analysis/
├── README.md
├── docs/
│   └── lab-01-basic-traffic.md
└── reports/
    └── traffic-analysis-template.md
```

Screenshots will be added only after reviewing them for sensitive details. Raw packet captures are intentionally excluded from version control by default.

## Exercises

| Exercise | Protocol | Status |
| --- | --- | --- |
| 01 — DNS resolution | DNS | Reviewed: A-record query/response (`microsoft.com`) |
| 02 — Echo request / reply | ICMP | Reviewed: 4 request/reply pairs |
| 03 — Connection setup & encryption metadata | TCP / TLS | Not started |

Follow the instructions in [Lab 01](docs/lab-01-basic-traffic.md) and see the [ongoing traffic analysis report](reports/traffic-analysis-template.md) for verified findings.

## Wireshark display filters

```text
dns.qry.name == "microsoft.com"
icmp
tcp.flags.syn == 1 && tcp.flags.ack == 0
tls
```

## Safety and disclosure

- Capture **only** traffic you are authorized to inspect.
- Do not upload real credentials, session tokens, internal infrastructure details, or personally identifying packet data.
- Review all screenshots and exported files before sharing, including if this repository is made public later.
- If an earlier VM was used for potentially vulnerable software or is of uncertain integrity, build a **fresh VM** rather than trusting an OS upgrade to make it safe.

## Planned deliverables

- A repeatable capture methodology
- A technical report with observed protocol flows
- Reviewed screenshots (no sensitive data)
- Conclusions and troubleshooting notes

---

*This is an educational, independently documented lab. Only observations supported by reviewed evidence are reported; TCP/TLS analysis is still in progress.*
