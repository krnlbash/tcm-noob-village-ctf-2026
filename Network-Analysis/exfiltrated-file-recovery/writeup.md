# Recovering a Transferred File from a Packet Capture

**Category:** Network Analysis
**Difficulty:** Medium
**Artifact:** `file_finder.pcap`

## Challenge Description

A second packet capture was provided, this time requiring extraction of a file transferred over HTTP and identification of the source host.

## Methodology

1. Parsed the capture with Scapy and reviewed all HTTP request/response pairs.
2. Multiple decoy files were downloaded over the course of the capture, each intended to bait a premature submission:
   - `readme.txt` — content explicitly states "This is not a challenge flag."
   - `old_incident_note.txt` — labeled `Decoy: NOOB{not_the_export}`, `Status: superseded`.
   - `browser_cache.bin` — a fake binary blob containing `NOOB{browser_cache_decoy}`.
   - `admin_backup.zip` — returned `403 Forbidden`, never actually transferred.
3. The genuine file transfer was identified by two corroborating signals absent from the decoys:
   - A `Content-Disposition: attachment; filename="incident_note.txt"` header (indicating a real file download, not just a page fetch).
   - A distinct `ETag: "fs-incident-final"` (vs. `"decoy-…"` on every earlier download).
4. Retrieved content explicitly instructed to disregard old/cached copies and included the real flag plus evidence fields.
5. Identified the source host of the transfer as `10.20.30.40`, corroborated by an earlier telemetry check-in payload (`{"host":"WS-40", ...}`) originating from the same IP.

## Tools Used

- Python 3 + Scapy

## Lessons Learned

- `Content-Disposition: attachment` and ETag values are reliable differentiators between a "real" file transfer and page/asset fetches in a pcap full of noise.
- Source host attribution should be corroborated by an independent data point (e.g. an application-layer identifier like a telemetry payload) rather than relying on IP alone.

## Flag

```
NOOB{exported_file_found}|file=incident_note.txt|host=files.first-signal.local
```
