# TCM Security: CTF @ DEF CON 2026
## Challenge: Weak Password in a Compromised Hash Dump

**Category:** Password Cracking / Windows Internals
**Files provided:** SAM-style hash dump (5 accounts)

---

### Objective

Identify the hash type used in the dump, then recover the plaintext password for the one account using a weak password.

---

### 1. The Dump

```
Administrator:500:aad3b435b51404eeaad3b435b51404ee:ebcdafb71e081743249850098f39b0aa:::
Guest:501:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
fmulder:1001:aad3b435b51404eeaad3b435b51404ee:f773c5db7ddebefa4b0dae7ee8c50aea:::
dskully:1002:aad3b435b51404eeaad3b435b51404ee:dfbc1fad6de1554bc02a4b4d1d82c136:::
wskinner:1003:aad3b435b51404eeaad3b435b51404ee:e6c17f50dd31dd495acdab81f25cd477:::
```

### 2. Identifying the Hash Type

The format matches the classic Windows **SAM dump** layout produced by tools like `secretsdump.py`, `pwdumpx`, or a raw `reg save` of the SAM/SYSTEM hives:

```
username:RID:LM_hash:NTLM_hash:::
```

Key identifying details:

- Every account's **LM hash field** is `aad3b435b51404eeaad3b435b51404ee` — this is the fixed constant Windows writes when LM hashing is disabled (which it is, by default, on any modern Windows build). It is *not* an actual password hash and can be ignored.
- The 32-character hex value in the 4th field is the **NTLM hash** — `MD4(UTF-16LE(password))`, with no salt. This lack of salting is exactly why NTLM hashes are crackable via wordlist/brute-force at high speed.
- The `Guest` account's NTLM hash, `31d6cfe0d16ae931b73c59d7e0c089c0`, is another well-known constant: it's the NTLM hash of an **empty string**, confirming the Guest account has no password set (expected/default, and not our weak-password target).

**Hash type: NTLM (unsalted MD4 of the UTF-16LE password)**

### 3. Cracking Approach

Since NTLM is unsalted MD4, each candidate password just needs to be UTF-16LE encoded and hashed with MD4, then compared against the dumped hash:

```python
from Crypto.Hash import MD4

def ntlm(pw):
    h = MD4.new()
    h.update(pw.encode('utf-16le'))
    return h.hexdigest()
```

(Python's built-in `hashlib` doesn't support MD4 directly on modern OpenSSL builds, since MD4 is deprecated — `pycryptodome`'s `Crypto.Hash.MD4` was used instead.)

A wordlist was built combining generic weak-password patterns (`password`, `123456`, `qwerty`, `letmein`, etc.) with **context clues from the usernames themselves** — `fmulder`, `dskully`, and `wskinner` are a clear reference to *The X-Files* (Fox Mulder, Dana Scully, Walter Skinner), so show-related terms were added: `trustno1`, `xfiles`, `scully`, `mulder`, `skinner`, `believe`, `truthisoutthere`, etc.

### 4. Results

| Account | NTLM Hash | Cracked? |
|---|---|---|
| Administrator | `ebcdafb71e081743249850098f39b0aa` | No match — strong password |
| Guest | `31d6cfe0d16ae931b73c59d7e0c089c0` | Empty password (disabled account) |
| **fmulder** | `f773c5db7ddebefa4b0dae7ee8c50aea` | **`trustno1`** ✅ |
| dskully | `dfbc1fad6de1554bc02a4b4d1d82c136` | No match — strong password |
| wskinner | `e6c17f50dd31dd495acdab81f25cd477` | No match — strong password |

`trustno1` is a well-known *X-Files* catchphrase, which made it both a fitting themed password for `fmulder` (Fox Mulder) and a predictable one — exactly the kind of pop-culture-based weak password that wordlist attacks are built to catch.

Verification:
```python
ntlm("trustno1") == "f773c5db7ddebefa4b0dae7ee8c50aea"  # True
```

### 5. Findings Summary

| Item | Value |
|---|---|
| Hash format | Windows SAM dump (`user:RID:LM:NTLM:::`) |
| Target hash type | NTLM (unsalted MD4 over UTF-16LE password) |
| Weak account | `fmulder` |
| Cracked password | `trustno1` |
| Crack method | Themed wordlist attack, informed by username/pop-culture context |

### 6. Flag & Submission

```
NOOB{trustno1}
```
