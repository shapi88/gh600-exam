# Branch Protection Settings

This document records the required branch protection configuration for `main`.
It is auditable as documentation alongside the code it governs.
Apply these settings via **Settings → Branches → Edit rule for `main`**.

---

## Required Settings for `main`

| Setting | Required value | Why |
| --- | --- | --- |
| Require a pull request before merging | ✅ Enabled | No direct pushes; all changes reviewed |
| Required approving reviews | 1 | At least one human must approve agent PRs |
| Dismiss stale pull request approvals when new commits are pushed | ✅ Enabled | Prevents approval-bypass by pushing after review |
| Require review from Code Owners | ✅ Enabled | Sensitive paths need CODEOWNERS sign-off |
| Require status checks to pass | ✅ Enabled | guardrails-check, agent-eval (if changed) |
| Require branches to be up to date before merging | ✅ Enabled | Reduces merge conflicts from parallel agent branches |
| Do not allow bypassing the above settings | ✅ Enabled | Admins and agents must follow the same rules |
| Restrict who can push to matching branches | ✅ Enabled | Only maintainers; no direct agent push |
| Allow force pushes | ❌ Disabled | Preserves audit trail |
| Allow deletions | ❌ Disabled | Preserves history |

---

## Required Status Checks

Add the following check names to the "Require status checks to pass" list:

| Check name | Source workflow |
| --- | --- |
| `guardrails-check` | `.github/workflows/guardrails-check.yml` |
| `scope-control` | `.github/workflows/guardrails-check.yml` |
| `secret-scan` | `.github/workflows/guardrails-check.yml` |

---

## Verification Command

```bash
gh api repos/{owner}/{repo}/branches/main/protection \
  --jq '{
    enforce_admins:     .enforce_admins.enabled,
    required_reviews:   .required_pull_request_reviews.required_approving_review_count,
    dismiss_stale:      .required_pull_request_reviews.dismiss_stale_reviews,
    codeowner_review:   .required_pull_request_reviews.require_code_owner_reviews,
    required_checks:    [.required_status_checks.contexts[]],
    allow_force_push:   .allow_force_pushes.enabled,
    allow_deletions:    .allow_deletions.enabled
  }'
```

Expected output:

```json
{
  "enforce_admins": true,
  "required_reviews": 1,
  "dismiss_stale": true,
  "codeowner_review": true,
  "required_checks": ["guardrails-check", "scope-control", "secret-scan"],
  "allow_force_push": false,
  "allow_deletions": false
}
```

---

## Restore Command (if protection is accidentally removed)

```bash
gh api repos/{owner}/{repo}/branches/main/protection \
  --method PUT \
  --field required_status_checks='{"strict":true,"contexts":["guardrails-check","scope-control","secret-scan"]}' \
  --field enforce_admins=true \
  --field required_pull_request_reviews='{"required_approving_review_count":1,"dismiss_stale_reviews":true,"require_code_owner_reviews":true}' \
  --field restrictions=null
```

---

## Reference

- [GitHub Docs — About protected branches](https://docs.github.com/en/repositories/configuring-branches-and-merges-in-your-repository/managing-protected-branches/about-protected-branches)
- [GitHub Docs — Branch protection REST API](https://docs.github.com/en/rest/branches/branch-protection)
