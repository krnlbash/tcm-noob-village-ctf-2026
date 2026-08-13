# Recovering a Removed Beta Code via the Wayback Machine

**Category:** OSINT / Web Archaeology
**Difficulty:** Medium
**Target:** `https://beta.ecyber.org`

## Challenge Description

An early-access beta code was briefly published on a beta website and later removed. Tasked with using an archived snapshot to recover it.

## Methodology

1. Checked the live site — beta access was closed, with a generic "thanks for signing up" placeholder page. No code present.
2. Queried the Wayback Machine's Availability API for the closest snapshot:
   ```bash
   curl -s "https://archive.org/wayback/available?url=beta.ecyber.org" | python3 -m json.tool
   ```
   This returned a snapshot from **Aug 7, 2026** — but fetching it showed the same "closed" placeholder, meaning it was captured *after* the code was already pulled.
3. Queried the CDX API for the exact URL — came back empty, suggesting no history for that literal URL string.
4. Broadened the query using `matchType=domain` to catch the domain more flexibly:
   ```bash
   curl -s "https://web.archive.org/cdx/search/cdx?url=ecyber.org&matchType=domain&output=json&fl=timestamp,original,statuscode&collapse=urlkey"
   ```
   This surfaced an **earlier** snapshot from **July 18, 2026** that the exact-match query had missed.
5. Fetched the raw (un-rewritten) HTML of that earlier snapshot using the `id_` modifier:
   ```bash
   curl -s "https://web.archive.org/web/20260718024534id_/https://beta.ecyber.org/"
   ```
6. This snapshot captured the page while beta access was still live, containing:
   ```html
   <p>Beta code: <span class="code">NOOB{the_interwebz_never_forget}</span></p>
   ```

## Tools Used

- `curl`
- Wayback Machine Availability API (`archive.org/wayback/available`)
- Wayback Machine CDX Server API (`web.archive.org/cdx/search/cdx`)

## Lessons Learned

- The Wayback Availability API only returns the *closest* single snapshot by default — it can silently point you at a post-change capture rather than the one you actually need. Always cross-check with the full CDX listing.
- CDX's default exact-URL match can miss captures under slightly different normalized forms; `matchType=domain` (or `prefix`) casts a wider net and can surface earlier/different snapshots.
- Appending `id_` to a Wayback URL returns the raw, unmodified page content instead of the JS-instrumented replay wrapper — cleaner for scripting/parsing.

## Flag

```
NOOB{the_interwebz_never_forget}
```
