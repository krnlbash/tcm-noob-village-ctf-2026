# robots.txt Leaking a Staging Environment

**Category:** Web Reconnaissance / OSINT
**Difficulty:** Easy/Medium
**Target:** `https://portal.ecyber.org`

## Challenge Description

Tasked with reviewing a public-facing website for accidentally exposed information.

## Methodology

1. Fetched the root page — clean, unremarkable managed-hosting marketing content. No flag, no comments, no obvious hints in the rendered page.
2. Checked standard recon paths:
   ```bash
   curl -s https://portal.ecyber.org/robots.txt
   ```
   Result:
   ```
   User-agent: *
   Disallow: /cgi-bin/
   Disallow: /portal-staging/
   ```
   `robots.txt` directly named a path the owners didn't want crawled — `Disallow` is a *request*, not an access control, so it effectively map out exactly where to look.
3. Fetched the disallowed path directly:
   ```bash
   curl -s https://portal.ecyber.org/portal-staging/
   ```
4. The staging page rendered normally, but its raw HTML source contained an HTML comment left in from a deploy:
   ```html
   <!--
     TODO before prod: strip this debug block, it dumps the staging env
     build: portal v2.1.0-rc3
     env:   staging
     DB_HOST=db-internal.ecyber.local
     DEPLOY_TOKEN=NOOB{disallow_is_not_deny}
     DEBUG=true
   -->
   ```

## Tools Used

- `curl` (raw HTTP requests, header inspection)

## Lessons Learned

- `robots.txt` is a crawler-politeness convention, not a security boundary — listing sensitive paths in it is a well-known way to accidentally advertise them to attackers who don't respect the `Disallow` directive.
- HTML comments are a classic leftover-debug-info leak vector; always pull raw page source (`curl`/View Source), not just what renders, since markdown-style content extraction tools typically strip comments.

## Flag

```
NOOB{disallow_is_not_deny}
```
