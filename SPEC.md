# Dead Drop — MVP Specification

## Goal
Simple approved-file transfer from home to office using encrypted artifacts in a public GitHub repository, with no login necessary at the download endpoint.

## Environments
- Upload/prepare: personal macOS computer, Git client and 7-Zip CLI available (install/check dependencies as needed).
- Download: corporate Windows computer, public github.com pages accessible, personal GitHub.com sign-in unavailable. A policy-approved 7z-compatible extractor is needed.
- Repository: GitHub.com public, named `dead-drop`.

## Scope
**In:** explicit source selection; preview and allowlist/exclusion checks; secret/path heuristic checks; opt-in inspection of one approved portable UTF-8 source `.tar.gz` plus text companions; encrypted archive; encrypted filenames; local verification including nested source hashes; manifest displayed locally; strict Git staging; manual publication approval; download/extraction instructions; automated tests.

**Out:** hosted website, GitHub Actions handling plaintext, password service, unattended publishing, automatic corporate-policy bypass, opaque/nested archives, arbitrary binaries, recursive directory selection, storage of plaintext manifests in the public repo, two-way transfer, delete-history guarantees.

## Suggested structure
```text
dead-drop/
├── AGENTS.md
├── README.md
├── SPEC.md
├── .gitignore
├── drops/                 # encrypted .7z files only
├── scripts/               # local CLI implementation
└── tests/
```

## CLI behavior (interfaces may be refined by Codex)
- `dead-drop prepare <paths...>`: inspect sources, show full manifest, block or flag risky content, package into encrypted archive in temporary storage, verify using the entered password, and copy only verified archive to `drops/`. No push.
- `dead-drop publish <archive>`: ensure only allowlisted paths are staged, show final diff and ask explicit consent before commit/push.
- `dead-drop check`: validate repository state, prerequisites, and absence of plaintext payloads.
- `dead-drop help`: concise instructions.

No password in CLI arguments, environment variables, shell history, committed config, or log files. If a compatible 7z binary cannot securely obtain the password, investigate safe tooling and document any tradeoff rather than guessing.

## Acceptance criteria
1. Approved sample file → locally prepared encrypted `.7z` archive → verified extraction with correct password.
2. Wrong password fails; archive member names cannot be listed without password.
3. No plaintext payloads or passwords appear in `git status`, public repo contents, application logs, or command arguments.
4. Secret-like files, symlinks, path traversal, missing dependency, and failed verification cause a safe failure.
5. No remote publication occurs without explicit confirmation.
6. Download and extract work on Windows without GitHub.com login, if organizational policies and required software permit.
7. Unit/integration tests are provided for the security boundary.

## Known limits
- Files committed to a public repo cannot be reliably recalled, even after deletion.
- Password protection prevents casual viewing, not anonymous download.
- Automated scanning cannot reliably prove a file is non-confidential.
- 7-Zip's handling of interactive passwords varies by platform/binary; validate it during implementation.
