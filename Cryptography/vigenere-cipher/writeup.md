# Vigenère Cipher with Contextual Key

**Category:** Cryptography
**Difficulty:** Medium
**Artifacts:** `context_note.txt`, `encrypted_message.txt` (hosted on S3)

## Challenge Description

Two files were provided: a short "context note" and an encrypted message. The submission format required identifying both the cipher and the key used.

## Methodology

1. Fetched `context_note.txt`:
   > "Welcome to the First Signal table. When the signal is noisy, start with FIRSTSIGNAL and verify the result."

   This was not flavor text — it directly named the key (`FIRSTSIGNAL`) to use.

2. Fetched `encrypted_message.txt`:
   ```
   SWFT{vgvzrxe_pmpk_fsbzrr}
   ```
   The preserved `{...}` structure (with `SWFT` in place of `NOOB`) strongly suggested a polyalphabetic substitution cipher rather than a monoalphabetic one (ROT13 would not turn `NOOB` into `SWFT` consistently across positions).

3. Implemented a standard Vigenère decryption (key `FIRSTSIGNAL`, cycling per alphabetic character, non-alphabetic characters passed through unshifted):

   ```python
   def vigenere_decrypt(text, key):
       result = []
       ki = 0
       for ch in text:
           if ch.isalpha():
               base = ord('A') if ch.isupper() else ord('a')
               shift = ord(key[ki % len(key)].upper()) - ord('A')
               dec = (ord(ch.upper()) - ord('A') - shift) % 26
               result.append(chr(dec + base))
               ki += 1
           else:
               result.append(ch)
       return ''.join(result)
   ```

4. Running this against the ciphertext with key `FIRSTSIGNAL` correctly unwrapped `SWFT` → `NOOB`, confirming both the cipher choice and key.

## Tools Used

- Python 3 (custom Vigenère implementation)

## Lessons Learned

- Read all provided artifacts carefully — the "context note" here wasn't scene-setting, it was the literal decryption key.
- Verifying a cipher choice by confirming the flag wrapper (`NOOB{...}`) decodes correctly is a reliable sanity check before trusting the rest of the decoded output.

## Flag

```
NOOB{context_keys_matter}|cipher=vigenere|key=FIRSTSIGNAL
```
