# Evidence index and release checklist

**Status:** Nine sanitized PNG screenshots have been uploaded to the **private** repository and are now linked from the [main README](../README.md) and [technical report](../reports/network-traffic-report.md).

All source traffic was generated for the controlled lab. The screenshots display selected packet/terminal fields; they are not raw PCAP exports. Filenames 08–10 replace the earlier single combined TLS image 05.

## Uploaded screenshots

| File | Exercise | Details |
| --- | --- | --- |
| [`01-icmp-echo.png`](01-icmp-echo.png) | ICMP | Four matched Echo Request/Reply pairs |
| [`02-dns-query.png`](02-dns-query.png) | DNS | Follow-up request, transaction ID `0x1397` |
| [`03-dns-response.png`](03-dns-response.png) | DNS | NOERROR, A record, response time **3.385220 ms** |
| [`04-tcp-handshake.png`](04-tcp-handshake.png) | TCP | SYN / SYN-ACK / ACK from the original stream |
| [`06-https-certificate.png`](06-https-certificate.png) | HTTPS | curl TLS/certificate verification |
| [`07-http-301.png`](07-http-301.png) | HTTP | Separate HEAD request returning 301 |
| [`08-tls-client-hello.png`](08-tls-client-hello.png) | TLS 1.3 | ClientHello / SNI |
| [`09-tls-server-hello.png`](09-tls-server-hello.png) | TLS 1.3 | ServerHello / cipher / key share |
| [`10-tls-application-data.png`](10-tls-application-data.png) | TLS 1.3 | Opaque encrypted records (type 23) |

## Final review before public release

- [ ] Open **each committed PNG in GitHub** and verify that private IP/MAC addresses, usernames, paths, unrelated traffic and hidden identifiers are not readable.
- [ ] Confirm that the underlying PNGs contain no unwanted metadata or embedded sensitive material.
- [ ] Ensure timestamps, packet numbers, TLS parameters and DNS response timings match the corresponding report sections.
- [ ] Distinguish follow-up DNS/TLS sessions from earlier packet streams; do not merge their timings or packet numbers.
- [ ] Verify that the README and report display correctly in GitHub's rendered view.
- [ ] Ensure no raw `.pcap`/`.pcapng` files, session tokens, passwords or personal browsing data were uploaded.
- [ ] Only then consider switching repository visibility from private to public.

**Privacy:** Redaction or cropping should use opaque removal of identifying values. A private repository is not a substitute for the checks above.
