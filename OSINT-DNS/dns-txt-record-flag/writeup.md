# Flag Hidden in a DNS TXT Record

**Category:** OSINT / DNS Reconnaissance
**Difficulty:** Easy
**Target:** `knock.knock.ecyber.org`

## Challenge Description

A hint indicated that DNS records can store more than service mappings — including verification data and administrative notes — and asked for investigation of a specific hostname's DNS records.

## Methodology

1. Queried the TXT record for the target hostname:
   ```bash
   dig TXT knock.knock.ecyber.org
   ```
2. Response contained a single TXT record:
   ```
   knock.knock.ecyber.org. 5  IN  TXT  "Tk9PQntub19wdW5jaGxpbmVfaGVyZX0="
   ```
3. Recognized the value as Base64 (character set, `=` padding) and decoded it directly:
   ```bash
   echo "Tk9PQntub19wdW5jaGxpbmVfaGVyZX0=" | base64 -d
   ```

## Tools Used

- `dig` (DNS TXT record query)
- `base64` (decoding)

## Lessons Learned

- DNS TXT records are a legitimate (if unconventional) covert-channel-adjacent storage location — commonly used for domain verification (SPF/DKIM/DMARC) in production but equally usable for embedding arbitrary data in a CTF context.
- Sandboxed/training DNS zones may only resolve through a specific internal resolver — general-purpose DNS-over-HTTPS APIs from outside that network won't see the record, so queries must be run from a host with the correct resolver configured.

## Flag

```
NOOB{no_punchline_here}
```
