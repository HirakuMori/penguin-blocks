---
name: pre-push-secret-scan
description: Scan the about-to-be-pushed diff of this PUBLIC repository for secrets, credentials, and other confidential information before any push to `main`. Trigger automatically whenever a push, commit-and-push, or PR is about to happen, and on explicit `/pre-push-secret-scan`.
---

# pre-push-secret-scan

This repository (`hakai-games`) is **public** and `main` deploys to GitHub Pages on every push. Anything pushed is permanently visible in the public git history, even if amended or force-pushed later. This skill is the last-line guard against accidentally publishing secrets, credentials, or internal information.

## Trigger

- Manual: `/pre-push-secret-scan`
- Automatic: run BEFORE any of the following actions
  - `git push`
  - `commit-commands:commit-push-pr`, or any flow that ends in a push
  - Opening a PR against this repo
  - Any "push it", "ship it", "deploy", "公開して" style request

If you are about to run a push and have not run this skill in the current turn, run it first.

## What to scan

Scan the **delta that is about to enter `origin`**, not the entire tree:

1. Commits ahead of the remote: `git diff origin/<branch>...HEAD`
2. Staged changes:                `git diff --cached`
3. Unstaged working-tree changes: `git diff`
4. Untracked files that are about to be added: `git ls-files --others --exclude-standard`

If `origin/<branch>` does not exist yet (first push of a branch), diff against the default base branch instead (`git diff main...HEAD`, or `git diff --root HEAD` for an entirely new history).

Only the new/changed lines matter. Do not flag pre-existing content that is not part of the delta.

## Patterns to flag

Run these greps against the diff hunks (added lines: lines starting with `+` but not `+++`). Use `git diff ... | grep -nE '^\+' | grep -iE '<pattern>'` or equivalent.

### 1. High-confidence secrets — block the push

- Provider key prefixes: `sk-[A-Za-z0-9]{20,}`, `sk-ant-[A-Za-z0-9_-]{20,}`, `ghp_[A-Za-z0-9]{20,}`, `gho_[A-Za-z0-9]{20,}`, `ghs_[A-Za-z0-9]{20,}`, `github_pat_[A-Za-z0-9_]{20,}`, `xox[baprs]-[A-Za-z0-9-]+`, `AIza[0-9A-Za-z_-]{35}`, `AKIA[0-9A-Z]{16}`, `ASIA[0-9A-Z]{16}`
- Private keys: `-----BEGIN (RSA |OPENSSH |EC |DSA |PGP )?PRIVATE KEY-----`
- JWT-shaped tokens: `eyJ[A-Za-z0-9_-]{10,}\.[A-Za-z0-9_-]{10,}\.[A-Za-z0-9_-]{10,}`
- `.env`-style assignments with non-empty values: `(?i)(api[_-]?key|secret|token|password|passwd|bearer|credential|access[_-]?key|client[_-]?secret)\s*[:=]\s*["']?[^\s"'<>${}]{8,}`

### 2. Likely-sensitive — warn and confirm with the user

- Email addresses (outside `LICENSE`, which already contains the author line)
- Internal hostnames / private-range IPs: `localhost`, `127\.0\.0\.1`, `192\.168\.`, `10\.\d+\.`, `172\.(1[6-9]|2[0-9]|3[01])\.`, `\.internal`, `\.corp`, `\.local[: /]`
- Confidentiality markers: `confidential`, `internal use only`, `do not (share|distribute)`, `proprietary`, `社内`, `機密`, `非公開`
- Real-name / personal-data shapes (phone numbers, postal addresses) when in player-facing or doc copy

### 3. Untracked-file hygiene

If `git ls-files --others --exclude-standard` lists files like `.env*`, `*.pem`, `*.key`, `*.p12`, `*.pfx`, `id_rsa*`, `credentials*`, `secrets*`, `*.sqlite`, `*.db`, `auth.json`, `service-account*.json`, **do not stage them**. Suggest adding them to `.gitignore` instead.

## Procedure

1. Determine the diff base:
   ```sh
   git fetch origin --quiet 2>/dev/null
   git rev-parse --abbrev-ref --symbolic-full-name @{u} 2>/dev/null
   ```
   Use the upstream if present; otherwise fall back to `origin/main`.
2. Collect the candidate diff:
   ```sh
   git diff <base>...HEAD     # committed but unpushed
   git diff --cached          # staged
   git diff                   # unstaged
   git ls-files --others --exclude-standard   # untracked
   ```
3. Apply the pattern set above to **added lines only** (`^\+` but not `^\+\+\+`).
4. Report results to the user before any push:
   - **Clean:** state "No secrets detected in <N> changed files / <M> added lines" and proceed.
   - **Hit in category 1:** STOP. Do not push. Show file path, line number, and the matching snippet (redacted to first/last 4 chars). Ask the user how to proceed.
   - **Hit in category 2:** show the hits and ask the user to confirm they are intentional before continuing.
5. After a category-1 hit, the secret is already in the local commit. Remind the user that:
   - Removing it requires `git reset` / `git commit --amend` / interactive rebase **before** pushing.
   - If it has already been pushed, the secret must be **rotated**; rewriting public history does not undo exposure.

## Reporting format

Keep the report compact:

```
pre-push-secret-scan: <clean | N findings>
  base:    <upstream or fallback>
  scanned: <X commits ahead> + <Y staged> + <Z unstaged> + <U untracked>

[if findings]
  <severity> <path>:<line>  <pattern-name>
    +  <redacted snippet>
```

## Don'ts

- Don't scan the entire repo — only the delta. Pre-existing content has already been published; flagging it here is noise.
- Don't auto-amend, auto-reset, or auto-rewrite history to "fix" a finding. Always hand control back to the user.
- Don't suppress a real category-1 hit just because the user is in a hurry. Pushed secrets must be rotated, not hidden.
- Don't treat this skill as a substitute for proper secret management — it is a tripwire, not a vault.
