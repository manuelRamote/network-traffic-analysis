# Lab 01 — Network traffic analysis with Wireshark

**Status:** Completed and documented · November 2026  
**Focus:** ICMP, DNS, TCP, TLS 1.3 and HTTPS  
**Tools:** Kali Linux 2026.3 · Wireshark 4.6.6 · `ping` · `dig` · `curl`

This walkthrough documents the methodology used in the completed [Network Traffic Analysis Lab](../README.md). It is also intended to help reproduce the tests safely. **Packet numbers, stream indices, timings and remote IP addresses can change between runs.**

## 1. Scope and safe setup

Capture **only traffic you own or are explicitly authorized to inspect**. Use a trusted, updated lab machine and short capture windows. Avoid browsing personal accounts while capturing.

1. Start Wireshark on the Kali lab VM.
2. Select the interface carrying the test traffic (`eth0` in the documented run).
3. Start a capture **without a capture filter** so the protocol exchanges can be correlated.
4. Run the commands below from a separate terminal, one at a time.
5. Stop the capture after the tests finish.

If packet capture fails, check interface permissions and `dumpcap` setup. Do not run the whole Wireshark GUI as root merely to bypass permissions.

## 2. Generate controlled traffic

```bash
# ICMP reachability
ping -c 4 1.1.1.1

# DNS A-record resolution
dig microsoft.com

# HTTPS request, TLS negotiation and verbose certificate check
curl -4 -I -v https://microsoft.com

# Separate HEAD request to display the returned HTTP status
curl -4 -sS -o /dev/null -w 'HTTP status: %{http_code}\n' -I https://microsoft.com
```

DNS caching, proxy configuration, VM networking and remote service behavior can affect what packets are visible. Do not assume an identical response on every run.

## 3. Analyze each protocol

### A. ICMP — request/reply correlation

```text
icmp
```

Look for Echo Requests (type 8) and Echo Replies (type 0). Match the identifiers and sequence numbers, then compare capture timestamps. In the documented run, **four requests had four matching replies**, with a mean observed round-trip time of approximately **14.63 ms**.

[View the sanitized ICMP evidence](../evidence/01-icmp-echo.png).

### B. DNS — query and response

```text
dns.qry.name == "microsoft.com"
```

Inspect the domain, query type A/IN, transaction ID, response code and answer. The original test returned `NOERROR` and an A record with a measured response time of **7.186521 ms**. The later screenshot pair is from a **separate DNS test**: transaction `0x1397`, response time **3.385220 ms**.

[DNS query](../evidence/02-dns-query.png) · [DNS response](../evidence/03-dns-response.png)

Do not confuse the **DNS record TTL** with the IPv4 packet TTL.

### C. TCP — three-way handshake

To find candidate connection initiations:

```text
tcp.flags.syn == 1 && tcp.flags.ack == 0
```

Select the connection of interest and use **Follow → TCP Stream**, or filter on its specific index (`tcp.stream == N`). Identify the SYN, SYN-ACK and ACK frames. The documented original stream completed the three-way handshake on port **443**, with approximately **13.84 ms** from SYN to SYN-ACK.

The stream index `N` is assigned per capture; **do not assume it is always zero**.

[View the sanitized TCP evidence](../evidence/04-tcp-handshake.png).

### D. TLS 1.3 — handshake messages

```text
tls.handshake.type == 1 || tls.handshake.type == 2
```

Inspect ClientHello (including SNI when available), ServerHello, cipher suite and `supported_versions`. In the follow-up capture, the negotiated version was **TLS 1.3**, with cipher suite `TLS_AES_256_GCM_SHA384` and key share `X25519MLKEM768`.

[ClientHello](../evidence/08-tls-client-hello.png) · [ServerHello](../evidence/09-tls-server-hello.png)

TLS 1.3 can display a legacy record-version value of `TLS 1.2 (0x0303)`; the negotiated version is determined by the handshake negotiation, not that compatibility field alone.

### E. TLS encrypted records

```text
tls.record.opaque_type == 23
```

This filter selects records with **outer content type 23 (Application Data)**. TLS 1.3 uses this outer type for encrypted records, which can also contain encrypted handshake messages. Do not describe the encrypted HTTP payload as readable or assume every matching record carries HTTP content.

[View sanitized encrypted TLS records](../evidence/10-tls-application-data.png).

### F. HTTPS certificate and HTTP response

The verbose `curl` session reported successful certificate/hostname verification and HTTP/2 negotiation:

[Certificate verification evidence](../evidence/06-https-certificate.png).

A subsequent independent `curl -I` request returned **HTTP 301**:

[HTTP status evidence](../evidence/07-http-301.png).

The redirected destination was **not** established in the recorded evidence. These terminal commands are **not the same TCP session** used for the Wireshark handshake timings.

## 4. Interpret and report findings

When writing up a lab:

- Include the test goal, tool versions and **commands actually executed**.
- Identify which screenshot proves each claim, citing fields or matching frames.
- Distinguish facts from inferences; avoid calling ordinary TLS traffic malicious.
- Keep the **original and follow-up captures separate** when citing packet numbers or timing measurements.
- Document uncertainty where a certificate, response header or DNS TTL was not inspected.

See the [technical report](../reports/network-traffic-report.md) for the detailed analysis, or the [portfolio case study](case-study.md) for the condensed findings.

## 5. Publication and handling

Only carefully reviewed, sanitized **screenshots** belong in the public portfolio. Do not upload raw `.pcap` / `.pcapng` files containing personal IP addresses, MAC addresses, credentials, DNS history or unrelated traffic.

Before changing repository visibility, complete the [evidence privacy and release checklist](../evidence/README.md).
