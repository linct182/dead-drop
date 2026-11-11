# Dead Drop — Agent Instructions

## Purpose
Dead Drop supports one-way transfer of *approved, non-confidential* files through public GitHub. Recipients can download encrypted archives without signing in.

It is **not** a tool to circumvent corporate security controls. Before using it, the operator must confirm that publishing and downloading the relevant material through public GitHub is authorized. If authorization is unclear, stop and ask.

## Non-negotiable security rules
1. Treat every commit to the public repository, including deleted or superseded versions, as permanently public.
2. Never commit or push plaintext payloads, passwords, tokens, private keys, credentials, `.env` files, personal data, internal company information, proprietary source code, or material not approved for public distribution.
3. **Encryption is defense in depth, not permission to publish restricted data.** Only package files authorized for this transfer channel.
4. All transfer payloads must be encrypted before being placed in Git's staging area. The repository must reject raw payloads, even if a user asks to bypass checks.
5. Use 7z AES-256 encryption, including encrypted filenames (`-mhe=on`). Do not accept unencrypted ZIP as equivalent.
6. Prompt interactively for the archive password using a no-echo input mechanism. Never print it, save it in files/config/logs, include it in process arguments, include it in GitHub Actions secrets for convenience, or record it in shell history.
7. Use a unique, high-entropy password or passphrase; never hardcode a default. Avoid placing password values on CLI command lines if they could be visible to other processes. Verify the installed 7z implementation's secure password prompt behavior and document platform limitations.
8. Never automatically push. Show a manifest, flags, intended archive name, and tracked-file diff, then require explicit human approval.
9. For suspicious, unknown, or likely confidential content, fail closed; don't silently redact or rewrite original user files. Do not claim automated scanning proves data is safe.
10. Do not use this repository as a return channel or as a workaround for DLP, sign-in, proxy, or access restrictions.

## Design requirements
- Keep original source files outside the repository and untouched.
- Use a temporary directory outside the repository for inspection, packaging, and verification. Clean it up on success and failure.
- Fail if the file list contains symlinks (or resolve them only within an explicitly approved root), paths escaping the source root, device files, or unexpected nested repositories.
- Detect obvious secrets and sensitive paths and allow a human to review the results. A clean scan is not authorization.
- Package only explicitly selected source files. Do not recursively package arbitrary directories without previewing the complete manifest.
- Exclude macOS/Windows metadata, dependency folders, Git internals, caches, and build outputs by default; let user review exclusions.
- The explicit `prepare --source-package` mode may accept one approved portable `.tar.gz` of UTF-8 source plus optional text companions. Inspect every nested member, path, size, hash and content, with bounded decompression and no tar extraction. Reject links, special/sparse files, unsafe/colliding paths, unknown/nested formats, sensitive/local settings and unreviewed metadata. Preserve original package bytes; never silently sanitize it or allow opaque archives. Keep the complete nested manifest in the local receipt and repeat inspection after encrypted extraction. Source roots must still be outside repositories; public-distribution approval and final publication review remain mandatory.
- Generate a random non-sensitive archive name, e.g. `drop-2026-11-08-<random>.7z`, and put it in `drops/`.
- Verify archive integrity and password-protected extraction locally before allowing staging. Never log extracted content or passwords.
- `git add` only expected encrypted output and approved repo files, never `git add .` indiscriminately.
- Keep files under GitHub's practical size limits; if too large, reject with explanation, don't silently introduce Git LFS (which may complicate anonymous downloads).
- Use minimal dependencies and cross-platform commands where possible: macOS for preparation; browser-only Windows for retrieval.

## Recommended implementation approach
Provide a local CLI with a `prepare` command and an explicitly confirmed `publish` command. The CLI handles validation, manifest review, 7z archive generation, test extraction, and safe staging. Never claim that browser ZIP download is password-protected; the downloaded **archive** requires the password to open.

## Agent workflow
1. Inspect the existing repository and the instructions here.
2. Propose an implementation plan and list any security limitations.
3. Implement the smallest working, testable version.
4. Test positive and negative cases, including staged plaintext detection, secret-like files, symlinks, missing 7z, weak/empty password, wrong password, and failed archive verification.
5. Show the final manifest, repository status, tests, and commands to use.
6. Do not publish anything until the human explicitly approves.
