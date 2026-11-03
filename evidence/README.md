# Evidence preparation and privacy checklist

The original Wireshark and terminal screenshots were reviewed during the lab but **have not been uploaded**. This folder will hold sanitized images when they are ready.

## Planned screenshots

| Suggested filename | Content to preserve | Redact or crop |
| --- | --- | --- |
| `icmp-echo.png` | Four request/reply pairs, sequence correlation, timings | Private IPs, interface MACs, machine-identifying labels |
| `dns-query-response.png` | Query name, A-record answer, transaction ID, NOERROR, response time | Local resolver and client addresses where identifiable; MACs |
| `tcp-handshake.png` | SYN, SYN-ACK, ACK, TCP stream index | Client private IP and MAC addresses |
| `tls-13-session.png` | ClientHello, ServerHello, TLS version, encrypted-data markers | Client private IP and any unrelated connection data |
| `https-verification.png` | Certificate verify result, negotiated TLS/HTTP, HTTP 301 | Hostname of local machine, paths, unrelated output and local IP |

## Publication checks

- [ ] Screenshots were produced from my own controlled lab.
- [ ] Personal hostnames, local IPs, MAC addresses, usernames and file paths are hidden where not required.
- [ ] No tokens, credentials, cookies, personal browsing, or unrelated network captures are shown.
- [ ] Important technical data (flags, packet numbers, timestamps, response codes) remains clearly legible.
- [ ] Redaction is visibly applied instead of replacing packet values with fabricated ones.
- [ ] Explanations clearly distinguish **observed** data from inference.
- [ ] Every image is referenced in the report or README only **after** it is added.
- [ ] Repository visibility is reviewed deliberately before making it public.

## Upload policy

Do **not** commit raw `.pcap` or `.pcapng` files from a personal or corporate network. Even when a repository is private, packet captures may contain sensitive payloads and identifying information.

Until screenshot redaction is complete, use the packet summaries and the [technical report](../reports/network-traffic-report.md) as the validated evidence.

> **Caution:** Simply blurring a small area can leave information recoverable or readable. Prefer opaque masks or cropping. Keep original evidence privately outside the public repository.
