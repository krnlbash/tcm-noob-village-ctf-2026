# TCM Security: CTF @ DEF CON 2026 — Noob Village
## Challenge: Ghost Process Beacon

**Category:** Memory Forensics / Process Analysis
**Flag:** `NOOB{ghost_process_beacon}`

---

### Challenge Summary

We were given a set of forensic artifacts pulled from host `WS-23` (domain `FIRSTSIGNAL`, user `mia`) — a raw memory-scan text dump, a process listing (PID/PPID table), and a netstat-style connection table. The goal was to identify a process that had been injected/masquerading as a legitimate Windows process, and the command-and-control (C2) destination it was beaconing to.

### Initial Triage

The memory dump was noisy by design — it mixed in legitimate host telemetry (Windows Update, OneDrive, Teams, Defender signature versions, browser cache headers) with two base64-encoded blobs meant to look like "diagnostic" or "cached note" data:

```
cached_note=Tk9PQnt0aGlzX2lzX29ubHlfYV9kZWNveX18cHJvY2Vzcz1ub3RlcGFkLmV4ZXxjMj11cGRhdGVzLmV4YW1wbGUuY29t
encoded_beacon_note=Tk9PQntnaG9zdF9wcm9jZXNzX2JlYWNvbn18cHJvY2Vzcz1zdmNob3N0MzIuZXhlfGMyPWJlYWNvbi5maXJzdC1zaWduYWwubG9jYWw=
```

Decoding both from base64:

```
cached_note          -> NOOB{this_is_only_a_decoy}|process=notepad.exe|c2=updates.example.com
encoded_beacon_note   -> NOOB{ghost_process_beacon}|process=svchost32.exe|c2=beacon.first-signal.local
```

Two candidate flags meant one of them was a planted red herring — the process table and netstat output were used to determine which.

### Cross-Referencing the Process Tree

Looking at the PID/PPID table:

```
PID   PPID  Name                 User    Notes
1337  620   svchost32.exe        mia     windows update
1888  1337  powershell.exe       mia     encoded beacon helper
```

Two things stand out immediately:

1. **`svchost32.exe`** — note the trailing `32`. Legitimate `svchost.exe` processes on Windows are always spawned by `services.exe` (PID 564 in this dump) and run under `SYSTEM`/`NETWORK SERVICE`/`LOCAL SERVICE`. This process instead has **PPID 620**, which is `explorer.exe` — a user's interactive desktop session — and it's running as `mia`, not `SYSTEM`. That's a classic masquerade: a malicious binary named to look like a trusted system process, dropped into the wrong process lineage.

2. **`svchost32.exe` (PID 1337) is the parent of `powershell.exe` (PID 1888)**, explicitly annotated `encoded beacon helper`. The memory dump corroborates this:

```
CommandLine=powershell.exe -NoProfile -WindowStyle Hidden -EncodedCommand <redacted>
parent_process=svchost32.exe
pid=1888
ppid=1337
network_destination=beacon.first-signal.local:443
```

A hidden-window PowerShell process launched with `-EncodedCommand`, spawned by a spoofed `svchost` clone — this is textbook fileless-malware / living-off-the-land beacon behavior.

### Confirming with Network Data

The connection table confirms the process is actively communicating with its C2:

```
Proto  Local Address       Foreign Address                    PID   State
TCP    10.10.5.23:49701    beacon.first-signal.local:443       1337  ESTABLISHED
```

PID 1337 (`svchost32.exe`) has an active TLS session to `beacon.first-signal.local`. Compare this against the decoy's claimed C2, `updates.example.com` — that domain is real traffic in the capture, but it belongs to PID 820, a **legitimate** `svchost.exe` instance parented correctly under `smss.exe`/`services.exe`. It has nothing to do with the injected process.

### Why the Decoy Fails

The `cached_note` blob pointed to `notepad.exe` / `updates.example.com`. Checking the process table, `notepad.exe` (PID 1224) is parented normally under `explorer.exe` with no suspicious child processes, no encoded command line, and no matching entry in the netstat table at all — it never talks to `updates.example.com` or anywhere else. It's a plausible-looking but evidence-free flag, included specifically to punish investigators who decode-and-submit without cross-checking against the process/network tables.

### Conclusion

The injected/masquerading process is **`svchost32.exe`** (PID 1337, incorrectly parented under `explorer.exe`, running as a standard user rather than SYSTEM), which spawned a hidden encoded PowerShell beacon helper and maintained an active outbound connection to **`beacon.first-signal.local:443`**.

### Final Submission

```
NOOB{ghost_process_beacon}|process=svchost32.exe|c2=beacon.first-signal.local
```

### Key Takeaways

- Always validate a suspicious process against its **parent-child lineage** and **execution context (user vs. SYSTEM)** — name alone (`svchost32.exe` vs `svchost.exe`) is often the only visible tell.
- Multiple "flag-shaped" strings in an artifact should be triangulated against independent evidence (process table + netstat) before submission — CTF authors plant decoys specifically to catch rushed analysis.
- Hidden-window PowerShell with `-EncodedCommand` spawned from a non-standard parent is a strong indicator of beaconing/C2 activity.
