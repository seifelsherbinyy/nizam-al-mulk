# Development workflow and repository access

Status as of 2026-10-29 (updated after the ruleset import). No credentials
appear in this document.

## Repository

- Remote: `https://github.com/seifelsherbinyy/nizam-al-mulk`
- Visibility: **public, temporarily**, so CodeQL results can upload to code
  scanning; the owner plans to make it private again after development.
- Default branch: `main`
- Merge policy: squash merge only; head branches are deleted on merge.
- Security: secret scanning, push protection and Dependabot security updates
  enabled.

## Protection of `main`

Ruleset `main-protection` (id 26117781) is active on the default branch
(`~DEFAULT_BRANCH`) with no bypass actors. It was imported from
`.github/rulesets/main-protection.public.json`; see `.github/rulesets/README.md`.

- Deletion blocked; non-fast-forward (force) pushes blocked; linear history.
- Pull request required: squash only, all conversations resolved, 0 approvals
  (owner and AI developer share one account).
- Required status check `Analyze (actions)` from GitHub Actions, branch up to
  date with `main`.
- CodeQL code-scanning gate (errors and warnings; high or higher security alerts).
- Copilot code review on push; draft pull requests are not reviewed.

Verified 2026-10-29: a direct push to `main` was rejected with `GH013`
(pull request required, status check expected, code scanning results expected).

Do not change rulesets or repository settings without explicit owner approval.

## Code scanning

CodeQL runs from `.github/workflows/codeql.yml` on pull requests to `main`,
pushes to `main`, weekly, and on demand. It analyzes the `actions` language
only; add `python` to its matrix (and `Analyze (python)` to the ruleset's
required checks) once Python sources exist.

The workflow detects visibility at run time. When the repository is public it
uploads results to code scanning. Otherwise it skips the upload, which a
private personal-account repository cannot accept, and keeps the SARIF as a
30-day workflow artifact.

**Before making the repository private:** first replace the active ruleset
with `.github/rulesets/main-protection.private.json`. The public ruleset's
code-scanning rule would otherwise block every pull request.

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
7. Merge according to the merge policy below; squash merge only.

## Autonomous merge policy

Approved by the owner on 2026-10-29. The AI developer may squash-merge its own
pull request without further owner action only when **all** of these hold:

1. Required checks pass and code scanning reports no blocking alerts
   (enforced by ruleset `main-protection`).
2. All review conversations are resolved (enforced by the ruleset).
3. An independent verifier (`nizam-verifier`) has accepted the exact head
   commit being merged. The author never accepts its own work; a rejection
   returns the PR for fixes, and any later push voids the acceptance.
4. No changed path is protected and the PR has no protected effect (below).
5. Before merging, the AI posts a summary comment on the PR: what changed,
   verification evidence, the accepted head commit and who accepted it.

Merge command (fails if the head moved after acceptance):

    gh pr merge <n> --squash --match-head-commit <accepted-sha>

### Protected paths

Owner's explicit approval per PR is required if `gh pr diff <n> --name-only`
lists any path matching:

- `.github/workflows/**`, `.github/actions/**`, `.github/rulesets/**`
- `.github/CODEOWNERS`, `CODEOWNERS`, `.github/dependabot.yml`
- `docs/DEVELOPMENT.md` (this policy and the access documentation)
- `.env*` (except `.env.example`), `*.pem`, `*.key`, `*.p12`, `*.pfx`,
  `id_rsa*`, `id_ecdsa*`, `id_ed25519*`, `**/secrets/**`, `**/credentials/**`
- `**/finance/**`, `**/financial/**`, `**/health/**`, `**/clinical/**`,
  `**/journal/**`, `**/journals/**`

### Protected effects

Owner's explicit approval is required for any of:

- Repository settings, rulesets, visibility, collaborators or integrations
- Secrets or credentials in any file, regardless of path
- Real financial or health records, private journals, production identifiers
- Production or VPS deployment, scheduler activation, Drive writes
- Deleting or rewriting history: force-pushes, history rewrites, or deleting
  any branch or tag by the AI developer.

If anything is protected or ambiguous, do not merge; ask the owner.

The ruleset cannot distinguish the owner from the AI developer (one account),
so this policy is enforced by the AI developer's process, not by GitHub.

## Never commit

Secrets, `.env` files, keys or certificates, production identifiers,
financial or health records, or private journals. Use synthetic fixtures.
