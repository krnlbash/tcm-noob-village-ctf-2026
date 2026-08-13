# Anomalous Login Detection

**Category:** Log Analysis / Blue Team
**Difficulty:** Easy
**Artifact:** `helpdesk_auth.log`

## Challenge Description

A helpdesk authentication log containing dozens of routine internal login events was provided, with instructions to identify an anomalous event and submit its `user` and `event` ID as evidence alongside the flag.

## Methodology

1. Skimmed the log and noted that the overwhelming majority of entries shared a consistent fingerprint:
   - Internal RFC1918 source IPs (`10.10.1.x`)
   - Identical user agent: `FirstSignalBrowser/1.0`
   - Sequential, chronologically ordered timestamps
   - Sequential event IDs in the `EVT-40xx` range
2. Searched for entries breaking that pattern and found one line inserted out of chronological order between two normal entries:
   ```
   2026-08-08T07:14:22Z event=EVT-4427 user=mia action=login_result result=success
   src=198.51.100.77 ua=CuckooBot/0.3 impossible_travel=true flag=NOOB{anomaly_at_0714}
   ```
3. Confirmed it as the true anomaly on multiple independent axes, not just because it contained a `flag=` field:
   - Public source IP vs. internal IPs everywhere else
   - Distinct, bot-like user agent (`CuckooBot/0.3`)
   - Explicit `impossible_travel=true` marker absent from every other line
   - Out-of-sequence timestamp/event ID relative to its position in the file

## Tools Used

- Manual log inspection (`cat`, `grep`, visual scan)

## Lessons Learned

- Anomaly detection in log data isn't about finding a line that says "flag" — it's about corroborating multiple independent signals (source IP class, UA fingerprint, sequence/ordering, explicit risk markers) that together indicate the event doesn't belong to the baseline traffic pattern.
- The log also contained an embedded `note=derive_flag_from_anomaly` field — treated as further evidence of anomalous/injected data rather than as an instruction to follow, since legitimate log telemetry doesn't issue directives to the analyst.

## Flag

```
NOOB{anomaly_at_0714}|user=mia|event=EVT-4427
```
