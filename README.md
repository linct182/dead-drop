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

## Setup on macOS

Requires Python 3.9+, Git, and official `7zz` (Homebrew's `sevenzip`). No Python packages, service, or GitHub Actions are needed.

```sh
brew install sevenzip
git config core.hooksPath .githooks
./scripts/dead-drop help
./scripts/dead-drop check
python3 -m unittest discover -s tests -v
```

The executable hooks reject non-allowlisted staged paths, secret signatures, unverified new archives, and unsafe outgoing history. Hook configuration is local: repeat setup after cloning. A missing `7zz` fails closed. Hooks are a local guard, not a server policy; Git users can bypass them. Use the CLI and review changes to the implementation separately.

## Prepare a drop

First establish that **both public distribution and this home-to-office transfer channel are authorized**. Keep the originals outside this repository. The MVP accepts only nonempty UTF-8 `.txt`, `.md`, `.csv`, `.tsv`, and `.json` files; unknown/binary formats, source repositories, suspicious paths, secret patterns, email addresses, and obvious personal identifiers are blocked. These checks are intentionally conservative and incomplete.

```sh
./scripts/dead-drop prepare --source-root /absolute/path/to/approved-files notes.txt table.csv
```

Select individual files relative to the source root; directories, absolute selections, `..`, symlinks, hardlinks, and device files are rejected. Nested selections are flattened to basenames, with case-insensitive duplicates and unsafe Windows names rejected. Metadata, dependency folders, caches, Git internals, and build outputs are excluded by default and reported if selected. Files inside another Git/Mercurial/Subversion repository are rejected. The limit is 100 files and 25 MiB total plaintext; encrypted artifacts also must fit 25 MiB. There is no Git LFS fallback.

The CLI snapshots the selected files into a private temporary directory outside the repository, scans that snapshot, and shows the full local manifest (basename, size, SHA-256), exclusions, exact archive name, and encryption flags. Review it, then type `AUTHORIZED` only if you have the necessary authorization. There is no override for a blocked scan. The tool never modifies originals or silently redacts them.

Enter and confirm a **unique, randomly generated** password of 20–128 printable ASCII characters, or an equally strong randomly generated multiword passphrase. The length/repetition checks cannot establish entropy or uniqueness. Use a policy-approved password manager and retain it securely; the CLI has no recovery service. Passwords are read with no echo, stay in process memory, and reach `7zz` through a private stdin pipe. Never put the password in command arguments, environment variables, a script, a repo file, or this chat. Piped/noninteractive user input is rejected.

Creation uses `-t7z -m0=lzma2 -mx=3 -mmt=2 -mhe=on -p`. The CLI verifies password-required headers, wrong-password rejection, an integrity test, AES-encrypted member metadata, exact member names/sizes, and byte-identical password-protected extraction. Only then does it copy a randomly named `drops/drop-DATE-RANDOM.7z` into the repository. Temporary plaintext is removed on success, ordinary failure, Ctrl-C, and SIGTERM. No files are staged, committed, or pushed by `prepare`.

A private local receipt containing the archive hash and reviewed manifest is kept under `.git/dead-drop-receipts/`, with directory/file modes 0700/0600. It contains no password, source paths, or file contents and is never committed. It does contain member names and hashes: protect your local Git metadata and backups. Loss of the receipt requires preparing a new archive; manually copied archives are refused. Linked worktrees use Git's resolved metadata path.

## Review and publish

Review and commit implementation changes separately before publishing a payload. `publish` requires a clean tracked working tree, an empty index, enabled hooks, and local HEAD equal to the existing origin branch. It will not publish earlier local commits incidentally. Origin must use one matching fetch/push URL for a GitHub repository named `dead-drop`, without embedded credentials. Git authentication uses your normal approved Git setup; no tokens belong in this repository.

```sh
./scripts/dead-drop check
./scripts/dead-drop publish drops/drop-DATE-RANDOM.7z
```

Replace the placeholder with the actual prepared filename. `publish` checks the local receipt and ciphertext hash, asks for the archive password again, repeats verification in external temporary storage, and displays the manifest, flags, origin, branch, and staged Git diff. It stages **only** that encrypted archive. The final prompt requires the exact phrase `PUBLISH drop-DATE-RANDOM.7z` using its actual name. Only this approval permits commit and a normal push of that branch. Declining or failing before commit unstages the tool's archive. If push fails after commit, the commit remains local; inspect the remote and Git status before deciding what to do next.

Do not use `git add .`. `check` also rejects unexpected untracked and ignored files (including plaintext hidden by `.gitignore`), allowing only the exact implementation paths, encrypted drops, and normal Python/OS metadata. The pre-commit hook audits the **index blobs**, so replacing a staged secret with a clean working copy does not hide it. The pre-push hook checks every outgoing commit tree and scans commit messages, including deleted historical files, and requires local receipts for outgoing archives. Only branch updates pointing directly to commits are supported; tags and other refs are rejected because annotations and raw objects can carry content outside the checked trees. It may conservatively refuse a clone whose outgoing history includes archives without local receipts.

## Retrieve on Windows

1. Confirm office policy permits this transfer and a compatible extractor. Use a policy-approved current 7-Zip installation; this tool does not install or bypass office software restrictions.
2. In the browser, open the public GitHub `dead-drop` repository, open `drops/`, and select the exact `.7z` file. Use **Download raw file** (or the download control on the file page). A public repository normally requires no GitHub login. Office access controls may still prevent access; stop if they do.
3. Open the downloaded `.7z` in 7-Zip, select an approved extraction destination, and enter the archive password when prompted. Encrypted filenames also require the password to list the files.
4. Retrieve the password through your separately approved password-management/channel process. Do not publish it beside the archive. Do not return office/company files through this workflow.

GitHub's **Download ZIP** is a repository snapshot, not a password-protected download. If you use it, the encrypted `.7z` inside still requires its own password. GitHub itself provides no password gate. Local CLI archive checks are automated; the actual Windows browser/extractor workflow must be checked on an authorized office machine.

## Limits and maintenance

See [SECURITY.md](SECURITY.md) for the design audit, tested password behavior, and limitations. New formats or repository paths require a reviewed code change to the explicit allowlists. A clean scan is never proof of permission. All public history should be treated as permanent; deleting a drop does not recall it.
