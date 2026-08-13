# ROT13 Decoding

**Category:** Cryptography
**Difficulty:** Easy
**Artifact:** Ciphertext string `ABBO{funyy_jr_cynl_n_tnzr}`

## Challenge Description

An encoded string was provided with a hint that it had "familiar shapes but the wrong meaning," requiring identification of the encoding scheme and decoding of the flag.

## Methodology

1. Noted the ciphertext preserved the `{...}` flag structure and character count of a normal `NOOB{...}` flag, suggesting a simple substitution cipher rather than a block cipher or encoding scheme like Base64.
2. Tested ROT13 by hand on the prefix: `ABBO` → shifting each letter 13 places forward (A↔N, B↔O) yields `NOOB`, confirming the cipher.
3. Applied ROT13 across the full string to recover the flag.

## Tools Used

- Manual substitution / mental ROT13 mapping

## Lessons Learned

- ROT13 is self-inverse (applying it twice returns the original text) and is immediately recognizable when a ciphertext preserves word/flag structure while substituting individual letters.

## Flag

```
NOOB{shall_we_play_a_game}
```
