# Network Traffic Analysis — Lab Report

> **Status:** In progress — ICMP capture reviewed; DNS and TCP/TLS analyses pending.

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

### 4.1 ICMP — Verified findings

**Test:** `ping -c 4 1.1.1.1` (as planned; four pairs are visible in the screenshot).

**Display filter:** `icmp`

- Packets visible: **8 ICMP frames** — 4 Echo Requests followed by 4 corresponding Echo Replies.
- Test source: private lab IPv4 address (redacted from this report); destination: `1.1.1.1`. Replies reverse the source and destination.
- All four request sequence numbers (1–4) have matching replies, with the same identifier (`0x6220`).
- Frame length shown by Wireshark: **98 bytes** for each of these packets.
- IPv4 TTL displayed in the packet summary: **64**.
- Measured round-trip times from the visible Wireshark **Time** column and matched frame pairs:
  - Frames 4 → 5: **15.62 ms**
  - Frames 6 → 7: **14.18 ms**
  - Frames 8 → 9: **14.47 ms**
  - Frames 10 → 11: **14.25 ms**
  - Mean of these four observations: **approximately 14.63 ms**
- Interpretation: the capture shows successful ICMP Echo Request/Reply exchanges and no missing replies **among the four requests visible here**. This is evidence of successful ICMP reachability for the test, not proof that other ports or protocols are reachable.
- Screenshot: reviewed in the working session; **not committed** to GitHub (it shows a private lab IP and should be redacted before publication).

**Methodology note:** the ICMP types are Echo Request = **8** and Echo Reply = **0**; the `Time` column reflects capture timestamps, so the round-trip estimate includes the capture's timing and matching uncertainty.

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

- **Confirmed observations:** Four ICMP Echo Requests to `1.1.1.1` have four corresponding replies in the reviewed capture; mean observed RTT ≈ 14.63 ms.
- **Uncertainties / limitations:** These findings are based on a screenshot, not the original packet capture or ping summary. DNS and TCP/TLS evidence has not yet been reviewed.
- **Potential next tests:** Review DNS query/response and TCP/TLS handshake evidence from the controlled lab.

## 6. Conclusions

The verified ICMP test demonstrates bidirectional Echo Request/Reply communication for all four test packets visible in the screenshot. The broader investigation remains in progress pending DNS and TCP/TLS validation.

## 7. Data handling

Before publishing, remove private IP/MAC addresses, personal browsing data, usernames, tokens, internal DNS names and any other sensitive details from evidence.
