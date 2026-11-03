# Network Traffic Analysis — Lab Report

> **Status:** Template. This report contains no validated findings yet.

## 1. Investigation summary

**Goal:** Understand and document DNS, ICMP and TCP/TLS communication patterns in a controlled lab.

**Date of execution:** _To be completed_

**Scope and authorization:** _Own test traffic only_

## 2. Environment

| Field | Value |
| --- | --- |
| Operating system/version | Kali GNU/Linux 2026.3 |
| Virtualization platform | _Pending_ |
| Wireshark version | 4.6.6 |
| Network mode (NAT/host-only/etc.) | _Pending_ |
| Interface used | `eth0` (active; addressing intentionally omitted) |

## 3. Methodology

**Traffic generation commands:**

```bash
ping -c 4 1.1.1.1
nslookup example.com
curl -I https://example.com
```

**Wireshark display filters:**

```text
icmp
dns
tcp.flags.syn == 1 && tcp.flags.ack == 0
tls
```

## 4. Observations

### 4.1 ICMP

- Packets observed: _Pending_
- Request/reply correlation: _Pending_
- Interpretation: _Pending_
- Screenshot: _Not added_

### 4.2 DNS

- Query type(s): _Pending_
- Resolver behavior and response: _Pending_
- Interpretation: _Pending_
- Screenshot: _Not added_

### 4.3 TCP / TLS

- TCP handshake: _Pending_
- TLS metadata observed: _Pending_
- Interpretation: _Pending_
- Screenshot: _Not added_

## 5. Findings and limitations

- **Confirmed observations:** _Pending_
- **Uncertainties / limitations:** _Pending_
- **Potential next tests:** _Pending_

## 6. Conclusions

_To be written after running and verifying the lab. Do not report expected behavior as if it were an observed result._

## 7. Data handling

Before publishing, remove private IP/MAC addresses, personal browsing data, usernames, tokens, internal DNS names and any other sensitive details from evidence.
