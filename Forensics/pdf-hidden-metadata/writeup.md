# Flag Hidden in a PDF's Object Structure

**Category:** Digital Forensics / File Format Analysis
**Difficulty:** Medium
**Artifact:** `archive_record_414.pdf`

## Challenge Description

A PDF that appeared to be a harmless access log was provided, with a hint that its internal structure contained hidden information beyond the rendered page content.

## Methodology

1. Confirmed the file was a legitimate PDF (`PDF document, version 1.5`).
2. Ran `strings` for a quick pass — surfaced nothing beyond the visible page text and PDF boilerplate.
3. Used `pikepdf` (Python) to walk the PDF's internal object graph directly rather than trusting rendered content:
   ```python
   import pikepdf
   pdf = pikepdf.open('archive_record_414.pdf')
   print(pdf.docinfo)
   print(pdf.trailer)
   ```
4. Found a **decoy** in the standard `/Info` dictionary (the metadata visible via any PDF viewer's "Document Properties"):
   ```
   /Keywords: NOOB{this_is_not_the_flag_you_are_looking_for}
   ```
5. Found the **real flag** in a non-standard custom dictionary injected directly into the PDF's Catalog (root object) — not part of the PDF spec, not rendered or surfaced by any standard viewer:
   ```
   /TCMExtra:
     /Type: /TCMChallenge
     /Flag: NOOB{conscience_of_a_hacker}
     /Note: "The Gibson never forgets. Some records only surface once
             you stop trusting the printed page."
   ```

## Tools Used

- `strings`
- `pikepdf` (Python library for PDF object-level manipulation/inspection)

## Lessons Learned

- PDFs are structured object graphs (Catalog → Pages → content streams, plus arbitrary custom dictionaries), not flat documents — sensitive data can be injected anywhere in that graph, completely independent of both the rendered page content and the standard `/Info` metadata block most people check first.
- Standard metadata fields (Author, Keywords, Subject) are the *first* place a scanner or curious viewer looks — a well-designed decoy in exactly that location can catch people who stop investigating too early.

## Flag

```
NOOB{conscience_of_a_hacker}
```
