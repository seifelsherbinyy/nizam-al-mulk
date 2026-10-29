# Branch rulesets

GitHub ruleset definitions for `main`, in the REST API body format
(`POST /repos/{owner}/{repo}/rulesets`). Validated against GitHub's official
OpenAPI description (`github/rest-api-description`).

| File | Use when | Rules |
| --- | --- | --- |
| `main-protection.public.json` | Repository is **public** | deletion, non-fast-forward, linear history, PR required (squash only, conversations resolved), `Analyze (actions)` check (up to date), CodeQL code scanning gate, Copilot review on push |
| `main-protection.private.json` | Repository is **private** | Same, without the CodeQL code-scanning gate (unavailable for private repos owned by a personal account) |

## Import

Settings > Rules > Rulesets > New ruleset > Import a ruleset > choose the
file > review > Create. Keep only one `main-protection` ruleset active.

## Before changing visibility

Switch rulesets **before** making the repository private: the public file's
`code_scanning` rule waits for results that a private personal repository can
never upload, which would block every pull request. Rulesets on private
repositories owned by a personal account require GitHub Pro; on Free the
private file may not be enforced.

## Notes

- No bypass actors: the owner and the AI developer share one account, so a
  bypass would also let the AI skip the rules. To recover, an admin disables
  the ruleset.
- 0 required approvals for the same reason; a solo account cannot approve its
  own pull request.
- If `python` is added to the CodeQL matrix, add `Analyze (python)` to
  `required_status_checks`.
- `enforcement: evaluate` is Enterprise-only; use `disabled` to stage a
  ruleset without enforcing it.
