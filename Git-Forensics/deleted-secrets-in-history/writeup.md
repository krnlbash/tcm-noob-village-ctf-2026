# Recovering a Deleted Secret from Git History

**Category:** Git Forensics / OSINT
**Difficulty:** Medium
**Target:** `https://github.com/pbarry25/early-risers`

## Challenge Description

A public repository looked clean in its current state, but its commit history was said to contain a removed secret.

## Methodology

1. Reviewed the current file tree — a `secrets.json` file was present at the root, an immediate red flag given the name doesn't fit the repo's actual purpose (a curated jobs list).
2. Fetched it directly and decoded its Base64 content:
   ```bash
   echo "<base64 content of secrets.json>" | base64 -d
   # -> "these are not the secrets you're looking for"
   ```
   Confirmed as a deliberate decoy.
3. Cloned the full repository (not shallow, to preserve history):
   ```bash
   git clone https://github.com/pbarry25/early-risers.git
   cd early-risers
   ```
4. Searched all commit diffs across all branches for the flag pattern directly:
   ```bash
   git log --all -p | grep -i "NOOB{"
   ```
   No hits — meaning the flag was never present in plaintext in any diff shown by default text search (it turned out to be base64-encoded).
5. Searched for any files deleted at any point in history:
   ```bash
   git log --all --diff-filter=D --summary | grep delete
   ```
   Found:
   ```
   delete mode 100644 secret.json
   ```
   Note the filename mismatch: `secret.json` (singular) vs. the decoy `secrets.json` (plural) still present in the tree.
6. Identified the exact commit that deleted it:
   ```bash
   git log --all --diff-filter=D --oneline -- secret.json
   # f9c1fc2 Cleaning up old stuff
   ```
7. Recovered the file's last content by viewing it at the **parent** of the deletion commit (i.e., the last commit where the file still existed):
   ```bash
   git show f9c1fc2^:secret.json
   # -> Tk9PQntjb25zb2xlX2Nvd2JveX0=
   ```
8. Decoded the recovered Base64 content to reveal the real flag.

## Tools Used

- `git` (`log`, `show`, `--diff-filter=D`)
- `base64`

## Lessons Learned

- **Deleting a file from a Git repository does not remove it from history.** Every prior version of every blob remains fully recoverable via `git show <commit>^:<path>` (or `<commit>~1:<path>`) as long as the commit isn't garbage-collected/rewritten (e.g. via `git filter-repo` + a forced push + GC), which is exactly why secret-scanning tools and BFG Repo-Cleaner exist.
- Filename similarity (`secret.json` vs. `secrets.json`) is a deliberate misdirection technique — always check for near-duplicate filenames when a decoy seems too easy.
- `git log --all --diff-filter=D --summary` is the fastest way to enumerate every file ever deleted across every branch/ref in a repository, regardless of which branch currently "hides" that history.

## Flag

```
NOOB{console_cowboy}
```
