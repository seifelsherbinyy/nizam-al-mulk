# Development workflow and repository access

Status as of 2026-10-29. No credentials appear in this document.

## Repository

- Remote: `https://github.com/seifelsherbinyy/nizam-al-mulk` (private)
- Default branch: `main`
- Merge policy: squash merge only; head branches are deleted on merge.

## Protection of `main`

Ruleset `main` (id 26113601) is **active but targets no refs**
(`conditions.ref_name.include` is empty), so GitHub currently applies **no
rules** to `main` (`gh api repos/{owner}/{repo}/rules/branches/main` returns
`[]`). Until the owner sets the target to the default branch, treat `main` as
protected by convention only: never push to it directly, never force-push.

Rules defined in the ruleset (enforced only once it targets `main`), with no
bypass actors:

- Deletion blocked; non-fast-forward (force) pushes blocked.
- CodeQL code-scanning gate (errors and warnings; high or higher security alerts).
- Copilot code review on push; draft pull requests are not reviewed.
- Required status checks rule present with an empty check list.

Do not change rulesets or repository settings without explicit owner approval.

## Code scanning

CodeQL runs from `.github/workflows/codeql.yml` on pull requests to `main`,
pushes to `main`, weekly, and on demand. It currently analyzes the `actions`
language only; add `python` to its matrix once Python sources exist. The
code-scanning gate needs uploaded results, which requires code scanning to be
enabled for this private repository (GitHub Code Security).

## Authentication

| Context | Method | Notes |
| --- | --- | --- |
| Interactive workstation | `gh auth login --web --git-protocol https` (GitHub CLI OAuth) | Token kept in the OS keyring. Git uses Git Credential Manager over HTTPS. |
| Unattended VPS | **Not configured.** Recommended: dedicated GitHub App with short-lived installation tokens, installed on this repository only. Fallback: fine-grained token restricted to this repository with an expiry. | Owner must create it; secrets must never be pasted into chat, commits or logs. |

Minimum permissions for an automation identity (GitHub App or fine-grained token):

- Contents: read and write
- Pull requests: read and write
- Issues: read and write
- Metadata: read (mandatory)
- Workflows: write **only** when a change touches `.github/workflows/`
- Administration: none

## Branch and PR workflow

1. `git switch main && git pull --ff-only`
2. `git switch -c <type>/<short-topic>`; types: `feat`, `fix`, `docs`, `chore`, `refactor`, `test`, `ci`
3. Commit small, meaningful changes (`type(scope): Summary`).
4. Run the test suite locally before pushing.
5. `git push -u origin <branch>`; never `--force` to `main`.
6. `gh pr create --draft --base main`; mark ready for review so Copilot
   review and checks run.
7. Merge only after review and owner authorization; squash merge.

## Never commit

Secrets, `.env` files, keys or certificates, production identifiers,
financial or health records, or private journals. Use synthetic fixtures.
