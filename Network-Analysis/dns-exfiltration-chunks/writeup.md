# TCM Security: CTF @ DEF CON 2026 — Noob Village
## Challenge: Beacon in the Noise (DNS Exfiltration)

**Category:** Network Packet Analysis
**Flag:** `NOOB{dns_chunks_tell_stories}`

---

### Challenge Summary

We were given `beacon_in_the_noise.pcap` and asked to identify the domain used for the malicious DNS activity and the number of data chunks that made up the exfiltrated payload.

### Tooling

Parsed the capture with Scapy:

```python
from scapy.all import *
pkts = rdpcap('beacon_in_the_noise.pcap')
for p in pkts:
    print(p.summary())
```

57 packets total, mostly DNS queries/answers plus a handful of unrelated TCP/HTTP sessions.

### Filtering the Noise

The capture was intentionally cluttered with legitimate-looking DNS traffic to make the real exfiltration channel harder to spot:

- Genuine external lookups: `www.example.com`, `login.microsoftonline.com`, `ocsp.digicert.com`, `api.github.com`, `docs.python.org`, etc.
- **Decoy internal-looking domains** under the same attacker-controlled zone: `cdn.first-signal.local`, `telemetry.first-signal.local`, `backup01.cdn-sync.local`, `status-check.telemetry.local`, `asset1-4.cdn.first-signal.local`
- Two throwaway "decoy" labeled queries that base64-decode to junk placeholders:
  ```
  1-QUJDREVGRw.decoy.first-signal.local  -> "ABCDEFG"
  2-SU5WQUxJRA.decoy.first-signal.local  -> "INVALID"
  ```
- A run of 10 sequential `telemetryN.example.org` queries (0–9) — looks like beacon/heartbeat traffic but carries no encoded payload, another distraction.

### Finding the Real Exfil Channel

Buried among the decoys were four DNS queries under a distinctly different subdomain, `exfil.first-signal.local`, each with a numbered, base64-encoded label:

```
1-Tk9PQntkbnNfY2h1bmtzX3Rl.exfil.first-signal.local
2-bGxfc3Rvcmllc318ZG9tYWlu.exfil.first-signal.local
3-PWV4ZmlsLmZpcnN0LXNpZ25h.exfil.first-signal.local
4-bC5sb2NhbHxjaHVua3M9NA.exfil.first-signal.local
```

The `N-` prefix on each label is a sequence number, indicating this is chunked/staged exfiltration over DNS — a common technique to smuggle data out through DNS queries by encoding it into subdomain labels (which routinely bypass egress filtering since DNS is rarely blocked outbound).

### Reassembly

Stripped the sequence prefixes, concatenated the four base64 fragments in numeric order (1 → 4), padded, and decoded:

```python
import base64
parts = [
    'Tk9PQntkbnNfY2h1bmtzX3Rl',
    'bGxfc3Rvcmllc318ZG9tYWlu',
    'PWV4ZmlsLmZpcnN0LXNpZ25h',
    'bC5sb2NhbHxjaHVua3M9NA'
]
full = ''.join(parts)
full += '=' * (-len(full) % 4)
print(base64.b64decode(full))
```

Output:

```
NOOB{dns_chunks_tell_stories}|domain=exfil.first-signal.local|chunks=4
```

The decoded payload contains the flag and its own evidence fields directly — confirming both the exfiltration domain and the chunk count in one shot.

### Conclusion

The malicious DNS exfiltration used the domain **`exfil.first-signal.local`**, split across **4** sequential, base64-encoded DNS query chunks.

### Final Submission

```
NOOB{dns_chunks_tell_stories}|domain=exfil.first-signal.local|chunks=4
```

### Key Takeaways

- DNS-based exfiltration hides payload data inside subdomain labels, encoded (commonly base64/base32) and split into chunks small enough to fit within DNS label length limits (63 bytes per label).
- A sequence-number prefix (`1-`, `2-`, `3-`...) on chunked labels is the reassembly key — order matters, and mis-ordering produces garbage on decode.
- CTF authors will litter a capture with plausible-but-empty decoys (`.local`/`.example.*` domains, base64 strings that decode to junk) specifically to slow down naive "decode everything and hope" approaches — the giveaway is usually a distinct, self-consistent subdomain/naming pattern shared only by the real exfil packets.
