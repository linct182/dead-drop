# Dead Drop

**Leave it here. Pick it up elsewhere.**

Dead Drop is a minimal, security-conscious workflow for depositing *approved, non-confidential* files in a public GitHub repository from home, and downloading them at an office computer where GitHub.com public repositories can be browsed without signing in.

## Basic workflow

1. At home (macOS), select approved files **outside** the repository.
2. A local tool inspects and previews selected files, then creates an encrypted `.7z` archive with AES-256 and encrypted filenames.
3. Verify the archive locally and review what will be published.
4. With explicit confirmation, commit and push **only the encrypted archive** to `drops/`.
5. At the office (Windows), browse the public repository, download the `.7z` file, and extract with a compatible 7-Zip application using the password.

GitHub does **not** provide download-password gates for public repository files. Anyone can download the encrypted archive; the password is required to extract it. Ensure that the office environment permits the transfer and has an approved archive extractor.

## Safety boundaries

Encryption is **not** authorization to share restricted content. Never upload company code, confidential documents, credentials, sensitive personal information, or unapproved content. Git history may preserve old files even after deletion, and encrypted contents remain publicly downloadable indefinitely.

## MVP implementation

See [SPEC.md](SPEC.md) and [AGENTS.md](AGENTS.md). The CLI, tests, and packaging tooling will be implemented with Codex.
# dead-drop
