# Recovering a Deleted File from a Raw Disk Image

**Category:** Digital Forensics / File Carving
**Difficulty:** Easy/Medium
**Artifact:** `lost_usb_image.dd`

## Challenge Description

A raw disk image was provided with instructions to recover a deleted file and identify its byte offset within the image.

## Methodology

1. Confirmed the file type was raw/unstructured (`file` reported simply "data" — no recognizable filesystem magic bytes at the start).
2. Hex-dumped the image to look for recoverable content directly:
   ```bash
   od -A x -t x1z lost_usb_image.dd | head -80
   ```
3. Found a run of null bytes (typical of a filesystem region whose directory entry was deleted but whose data blocks remain intact) followed by a clearly-delimited carved block starting at offset `0x1000`:
   ```
   RECOVERED_FILE_START
   filename=recovered_note.txt
   This is not the flag: NOOB{decoy_deleted_note}
   Real submission: NOOB{carved_context_wins}|file=recovered_note.txt|offset=4096
   RECOVERED_FILE_END
   ```
4. Verified the offset: `0x1000` in hex = `4096` in decimal, matching the `RECOVERED_FILE_START` marker position exactly.
5. Noted the embedded decoy flag (`NOOB{decoy_deleted_note}`) immediately preceding the real submission line — the block explicitly labels which one is real.

## Tools Used

- `file` (image type identification)
- `od` (hex dump / offset inspection)

## Lessons Learned

- File carving doesn't always require full filesystem parsing — searching raw bytes for known markers/magic strings and null-byte boundaries can be sufficient when structure is deliberately simplified (as in training artifacts).
- Always convert hex offsets to decimal (or vice versa) explicitly when a challenge's evidence format expects one or the other — an easy source of an otherwise-correct submission being marked wrong.

## Flag

```
NOOB{carved_context_wins}|file=recovered_note.txt|offset=4096
```
