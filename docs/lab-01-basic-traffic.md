# Lab 01 — Basic network traffic investigation

**Status:** Planned / not yet executed

## 0. Prerequisites and safety

Use a **clean, patched VM** or a trusted host. If the existing Kali VM was previously isolated due to vulnerable services or concerns about compromise, do **not** reconnect it; install a fresh VM instead.

Capture only your own traffic or traffic you are expressly authorized to analyze. Use a short capture window. Avoid browsing personal accounts while capturing.

Record the following (do not publish your full personal IP addresses or local network topology):

```bash
cat /etc/os-release
wireshark --version
ip -br addr
ip route
```

The last two commands may contain network details: redact them from screenshots shared publicly.

## 1. Start a capture

1. Open Wireshark on the trusted system.
2. Select the active network interface that carries your lab traffic (e.g., `eth0`, `ens33`, `enp0s3`, depending on the VM).
3. Begin capturing. For the first run, use **no capture filter** so that you can compare different protocols.
4. Keep the capture to a short session and generate the test traffic below.

If the GUI cannot capture packets, first check interface permissions and `dumpcap` configuration; do not run the entire Wireshark GUI as root just to bypass permissions.

## 2. Generate known test traffic

Run the following in a separate terminal, one command at a time:

```bash
# ICMP echo
ping -c 4 1.1.1.1

# DNS lookup
dig microsoft.com

# HTTPS request (TCP + TLS, unless a proxy changes the flow)
curl -I https://example.com
```

The output and traces depend on DNS cache, proxy settings, VM networking and network connectivity. A DNS query, TLS handshake or response might not be visible in every configuration.

Stop the capture when you finish.

## 3. Inspect the capture with display filters

### A. ICMP

```text
icmp
```

Look for ICMP echo requests and echo replies. Record the packet count, source/destination **roles** and the observable request/reply relationship. No reply can be caused by filtering or reachability constraints.

### B. DNS

```text
dns
```

Use `dns.qry.name == "microsoft.com"` to focus on this lab's query. Inspect request and response packets, transaction ID, queried name, record type, response code, DNS response time, and the answer record TTL if visible in the expanded fields. Note the possibility of different DNS record types or resolver behavior.

### C. TCP connection establishment

```text
tcp.flags.syn == 1 && tcp.flags.ack == 0
```

Find the first SYN packet. Use `Follow → TCP Stream` or the `tcp.stream` filter to inspect the same connection. Where present, identify the SYN, SYN-ACK and ACK packets.

### D. TLS

```text
tls
```

Look for `Client Hello` and `Server Hello` and relevant metadata. HTTPS application content is encrypted and should **not** be described as visible in plaintext. Depending on TLS version and capture timing, parts of the handshake may also be encrypted.

## 4. Record findings

Use [the technical report](../reports/network-traffic-report.md) as a reference and record findings from your own capture. Include:

- Date (approximate is enough for a public version) and isolated lab description
- OS and Wireshark versions
- Commands used
- Display filters and packet numbers **from your own capture**
- What you actually observed vs. what you expected
- Limitations and explanations for missing traffic

## 5. Export policy

Do not commit unreviewed `.pcap` or `.pcapng` files. Packet captures can contain MAC addresses, IP addresses, DNS history, tokens, cookies and other sensitive information.

For a public-facing portfolio, favor screenshots with identifying values redacted and a written analysis; if a PCAP is essential, generate a separate synthetic or safely sanitized example first. See the [evidence checklist](../evidence/README.md).
