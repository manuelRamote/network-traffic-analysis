# Network Traffic Analysis — Technical Report

> **Status:** ICMP, DNS, TCP/TLS and HTTP checks completed; screenshot sanitization pending.

## 1. Investigation summary

**Goal:** Understand and document DNS, ICMP and TCP/TLS communication patterns in a controlled lab.

**Date of execution:** November 2026 (test session)

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
dig microsoft.com
curl -4 -I -v https://microsoft.com
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

### 4.2 DNS — Verified findings

**Test:** `dig microsoft.com`

**Display filter:** `dns.qry.name == "microsoft.com"`

**Evidence:** Two packets in the user-provided Wireshark screenshots, one request and one response.

| Field | Value observed |
| --- | --- |
| Requested domain | `microsoft.com` |
| Query type and class | `A`, `IN` |
| Protocol | UDP; source port `51955`, destination port `53` (reversed on reply) |
| DNS resolver | Resolver address redacted in this publication-ready report |
| Requesting host | Private VM address (omitted from public-facing report) |
| Transaction ID | `0x28be` on both request and response |
| Response code | `NOERROR (0)` |
| Answer | One A record: `150.171.109.83` at capture time |
| DNS response time | `7.186521 ms` (Wireshark) |
| Frame sizes | 96 bytes (request); 128 bytes (response) |
| Recursive resolver | Recursion desired in query; recursion available in reply |
| Authoritative response | No (AA flag not set) |

**Interpretation:** The DNS resolver answered the A-record question successfully, and the transaction identifier pairs the two observed DNS packets. The returned A record reflects this particular response and may change between queries or networks. The screenshots do not expose the DNS record TTL; do not substitute the IPv4 packet TTL.

**Limitations:** This demonstrates DNS resolution, not successful TCP/TLS connectivity to the resolved address. The screenshots were reviewed for analysis but are **not committed** to the repository pending redaction of local network identifiers.

### 4.3 TCP / TLS — Verified TCP handshake and TLS exchange

**Test context:** HTTPS connection to `microsoft.com`, using the IPv4 address `150.171.109.83` observed in the earlier DNS response. Packet data was inspected using Wireshark display filter `tcp.stream == 0` on interface `eth0`.

**Connection:** Kali VM (private address omitted), ephemeral port `50458` → `150.171.109.83:443`.

#### A. TCP three-way handshake

| Frame | Capture time (s) | Direction | Flags | Relative sequence / acknowledgment |
| --- | ---: | --- | --- | --- |
| 3 | 0.007664583 | Client → server | SYN | Seq=0 |
| 4 | 0.021504309 | Server → client | SYN, ACK | Seq=0, Ack=1 |
| 5 | 0.021553536 | Client → server | ACK | Seq=1, Ack=1 |

- **SYN → SYN-ACK elapsed time:** approximately **13.84 ms** (computed from capture timestamps).
- The final ACK confirms that the TCP three-way handshake completed in this observed conversation.
- The original SYN advertised MSS `1460`, SACK permitted, timestamps and window scaling.

#### B. TLS 1.3 exchange

| Frame | Wireshark description | Interpretation |
| --- | --- | --- |
| 6 | TLSv1.3 Client Hello (SNI=`microsoft.com`) | Client begins TLS negotiation and provides the indicated hostname |
| 9 | TLSv1.3 Server Hello, Change Cipher Spec, Application Data | Server responds with TLS negotiation material |
| 23 | TLSv1.3 Application Data | Reassembled TLS records containing encrypted data |
| 27 | TLSv1.3 Change Cipher Spec, Application Data | Additional client-to-server TLS records |
| 29–37 | TLSv1.3 Application Data and TCP acknowledgments | Encrypted traffic exchanged in both directions |

Wireshark decodes the connection as **TLS 1.3** and displays encrypted application-data records. The screenshots alone do **not** establish the HTTP status code, validate the server certificate chain, or reveal application payload contents.

#### C. TCP connection termination

| Frame | Direction | Flags |
| --- | --- | --- |
| 39 | Client → server | FIN, ACK |
| 40 | Server → client | ACK |
| 41 | Server → client | FIN, ACK |
| 42 | Client → server | ACK |

The shown stream closes with a normal TCP FIN/ACK exchange; no RST frame is visible in this conversation.

**Conclusion:** the supplied screenshots demonstrate the complete TCP three-way handshake, TLS 1.3 handshake messages and subsequent encrypted data exchange, and orderly connection termination.

#### D. Separate HTTPS verification with curl

**Command:** `curl -4 -I -v https://microsoft.com`

The provided terminal screenshot documents a **separate HTTPS session**. Its local source port is `52394`, rather than `50458` in the previously inspected Wireshark stream, so these are not presented as the same TCP connection.

| Field | Observed value |
| --- | --- |
| Resolved destination | `150.171.109.83:443` |
| Negotiated TLS version | TLS 1.3 |
| TLS cipher suite | `TLS_AES_256_GCM_SHA384` |
| Key exchange named by curl | `X25519MLKEM768` |
| Signature algorithm | `RSASSA-PSS` |
| Certificate hostname check | `microsoft.com` matched certificate Subject Alternative Name |
| Certificate verification | `SSL certificate verify ok`; OpenSSL verify result `0` |
| HTTP protocol | HTTP/2, via ALPN |
| HTTP request | `HEAD /` with authority `microsoft.com` |
| HTTP status | **301 (Moved Permanently)** — confirmed in a subsequent `curl -I` request |

**Assessment:** The terminal output verifies that the local curl/OpenSSL client accepted the presented certificate and negotiated a TLS 1.3 connection. A HEAD request was issued over HTTP/2. The first verbose screenshot did not include response headers or a status code; a subsequent `curl` request independently confirmed HTTP status **301** (a permanent redirect). The destination URL was not shown, so no redirect target is asserted. The observed hybrid `X25519MLKEM768` key exchange combines classic and post-quantum key establishment mechanisms.

**Follow-up verification:**

```bash
curl -4 -sS -o /dev/null -w 'HTTP status: %{http_code}\n' -I https://microsoft.com
# Observed output: HTTP status: 301
```

A 301 response means the requested resource is redirected permanently. The `Location` response header was not recorded in this screenshot; the redirect target and final destination status remain unverified. The curl result is from a separate request, not the exact packets examined in the earlier `tcp.stream == 0` capture.

**Evidence handling:** screenshots reviewed in the working session. The original screenshots include private lab identifiers and have **not** been committed to GitHub.

## 5. Findings and limitations

- **Confirmed observations:** Four ICMP Echo Requests to `1.1.1.1` have four corresponding replies (mean observed RTT ≈ 14.63 ms). A DNS A-record query for `microsoft.com` returned `NOERROR` and one observed address (response time 7.186521 ms). TCP stream 0 shows the complete SYN/SYN-ACK/ACK handshake to port 443, a TLS 1.3 ClientHello and ServerHello, encrypted application data, and a FIN/ACK closure.
- **Uncertainties / limitations:** Findings are based on screenshots, not full original PCAP files. The DNS record TTL was not visible; curl reports a successful certificate/hostname validation in a separate session, and a subsequent curl request returned HTTP status `301`; the redirect destination was not captured. Screenshots still require redaction before publication.
- **Potential next tests:** Optionally inspect the HTTP `Location` header (and the redirected endpoint if desired); prepare sanitized screenshots; optionally inspect the DNS answer TTL.

## 6. Conclusions

The ICMP test demonstrates four matched Echo Request/Reply exchanges. The DNS test shows successful resolution of an A record for `microsoft.com`. The TCP stream confirms connection establishment, TLS 1.3 negotiation and encrypted traffic exchange, followed by an orderly close. A separate curl test confirmed TLS certificate/hostname validation and an HTTP/2 HEAD request; a subsequent `curl -I` check returned HTTP 301. The investigation is complete; publication-ready screenshots still require review and redaction.

## 7. Data handling

Before publishing, remove private IP/MAC addresses, personal browsing data, usernames, tokens, internal DNS names and any other sensitive details from evidence.
