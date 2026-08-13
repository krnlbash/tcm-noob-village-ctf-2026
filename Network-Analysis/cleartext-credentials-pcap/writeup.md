# Cleartext Credentials in a Packet Capture

**Category:** Network Analysis
**Difficulty:** Easy/Medium
**Artifact:** `packets_in_the_wire.pcap`

## Challenge Description

A packet capture was provided with instructions to identify the protocol carrying cleartext credentials and the specific packet number.

## Methodology

1. Parsed the pcap with Scapy and enumerated all 46 packets to get a traffic overview (DNS queries, HTTP/TCP sessions, a fake TLS handshake).
2. Extracted the raw payload of every TCP packet containing a `Raw` layer to inspect HTTP traffic in full.
3. Identified **two decoys** designed to catch a hasty submission:
   - Packet 13: `Authorization: Basic bW9uaXRvcjpzdGF0dXM=` (`monitor:status`) — response is explicitly tagged `X-Training-Decoy: NOOB{monitoring_is_not_admin}`.
   - Packet 29: `Authorization: Basic Z3Vlc3Q6Z3Vlc3Q=` (`guest:guest`) — returns `403 Forbidden`, i.e. a failed/low-value credential.
4. Found the real credential exposure in **Packet 37**:
   ```
   GET /admin HTTP/1.1
   Host: portal.first-signal.local
   Authorization: Basic aW50ZXJuOmZpcnN0c2lnbmFs
   ```
   - Base64-decodes to `intern:firstsignal` — a working credential pair.
   - The corresponding response (Packet 38) is `200 OK` with `admin dashboard` content, confirming successful authentication.
5. Flag was included directly in the request headers (`X-Training-Flag`), confirming the answer.

## Why This Is "Cleartext"

HTTP Basic Authentication sends credentials as Base64 — a reversible encoding, not encryption. Combined with plain HTTP (no TLS), the credentials are fully recoverable by anyone capturing the traffic.

## Tools Used

- Python 3 + Scapy (`rdpcap`, `Raw` layer payload extraction)

## Lessons Learned

- Always check for multiple candidate hits before submitting — CTF pcaps commonly plant decoys with plausible-looking but incorrect credentials.
- A `200 OK` + successful authenticated response is a much stronger signal than merely finding *any* Basic Auth header.

## Flag

```
NOOB{cleartext_creds_are_loud}|protocol=http|packet=37
```
