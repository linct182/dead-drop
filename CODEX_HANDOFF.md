# Codex Handoff — Implement Dead Drop

You are implementing the initial version of `dead-drop` in this repository.

## Context
I work from a personal Mac at home and need an easier way to transfer *approved, non-confidential* files to an office Windows PC. The office can browse public github.com repositories, but I cannot log into my personal GitHub.com account there. We will use a public GitHub repository only if this workflow is allowed by corporate policy. Anybody can download from it, so payloads must be encrypted and password-required upon extraction.

## Read first
- `AGENTS.md` — mandatory agent/security rules
- `SPEC.md` — MVP requirements and acceptance criteria
- `README.md` — user-facing purpose

## Your task
1. Audit the starter spec for safety and feasibility; flag any unrealistic assumptions, especially password input to 7z without leaking to process arguments.
2. Choose a minimal, maintainable, cross-platform-friendly implementation (the packaging CLI runs primarily on macOS; downloading is browser-based on Windows). Explain your choice.
3. Implement a small CLI: prepare, check, publish, help.
4. Support AES-256 7z encryption **including archive filenames** and verify extraction locally.
5. Prevent plaintext/staged sensitive files, risky symlinks/path traversal, and unintended Git additions.
6. Provide clear interactive preview, a local-only manifest, and explicit human approval before pushing. Never automatically publish.
7. Add tests and run them. Document exact setup and usage on macOS and extraction on Windows.
8. Show final implementation summary, security limitations, tested paths, and pending decisions.

Prefer a focused MVP over a complicated application. Avoid hosted services, GitHub Actions processing plaintext, dependencies that expand the security surface, or workarounds for company controls. Do not push without me explicitly approving.

Start by inspecting the repository and presenting the implementation plan, then proceed with implementation.
