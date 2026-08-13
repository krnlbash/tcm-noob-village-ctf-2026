# Extracting a Hidden Flag from a Compiled Binary

**Category:** Reverse Engineering
**Difficulty:** Easy
**Artifact:** `system_check` (stripped x86-64 ELF binary)

## Challenge Description

A compiled program that requests authentication and denies access when run was provided, with a hint that useful information was not revealed during normal execution.

## Methodology

1. Identified the file type: `ELF 64-bit LSB pie executable, x86-64, dynamically linked, stripped`.
2. Rather than dynamically executing the (access-denying) binary, went straight to static analysis with `strings`:
   ```bash
   strings -n 6 system_check
   ```
3. The flag was present in plaintext in the binary's read-only data section, never printed by the program's actual runtime logic (which only outputs boot-sequence flavor text and an "ACCESS DENIED" message).

## Tools Used

- `file` (binary identification)
- `strings` (static string extraction)

## Lessons Learned

- Not all "hidden" data in a binary requires disassembly or debugging — a stripped binary can still leak sensitive strings directly in `.rodata` if the developer embedded them without care (e.g. leftover debug flags, hardcoded secrets, or in this case, a training flag).
- Always try the cheapest static analysis technique (`strings`) before reaching for a disassembler/debugger.

## Flag

```
NOOB{hack_the_planet}
```
