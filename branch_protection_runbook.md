# Branch Protection Runbook

This runbook provides exact `gh` CLI commands to apply branch protection to the `main` branch across all organization repositories.

Notes before running:
- You must run these commands as an org admin with `repo` and `admin:org` scopes and `gh` CLI authenticated to an account that has those privileges.
- The commands assume the `gh` CLI is installed and authenticated (`gh auth login`).
- These commands set required status checks. If a repository doesn't yet publish a status-check with the exact name, the CLI will fail — run the command after the CI has executed at least once to register the checks, or adjust the `--required-checks` value accordingly.

# Replaceable variables
org=Fintech-Blueprint
repos=(infra platform-libs contracts concierge db-manager api-gateway catalog design-system example-service .github staging-env)

# Apply protection per-repo (copy & paste block):

for repo in "${repos[@]}"; do
  echo "Applying branch protection to $repo"
  gh api --method PUT /repos/$org/$repo/branches/main/protection -F required_status_checks='{"strict":true,"contexts":["unit","contract","lint","sast","sbom-sign"]}' -F enforce_admins=true -F required_pull_request_reviews='{"dismiss_stale_reviews":true,"require_code_owner_reviews":true,"required_approving_review_count":1}' -F restrictions=null
  gh api --method PUT /repos/$org/$repo/branches/main/protection -f required_linear_history=true -f allow_force_pushes=false -f allow_deletions=false || true
done

# Explanation
- `required_status_checks.strict=true`: Ensures status checks are up-to-date before merging.
- `contexts`: status check names to require (`unit`, `contract`, `lint`, `sast`, `sbom-sign`) — adjust to match actual workflow check names if they differ.
- `enforce_admins=true`: Enforces protections for admins as well.
- `required_pull_request_reviews`: Requires one approving review and code-owner reviews.
- `required_linear_history=true`: Enforces linear history.
- `allow_force_pushes=false` & `allow_deletions=false`: Blocks force pushes and deletion of the `main` branch.

# Troubleshooting
- If you receive an HTTP 422 error about unknown `contexts`, run CI once to register check names, then re-run.
- If your organization plan restricts branch protection, you'll see an error; apply protections manually via the web UI or upgrade the plan.

# One-shot single-line command (for automation)

printf '%s\n' "${repos[@]}" | xargs -I{} -n1 -P0 gh api --method PUT /repos/$org/{}/branches/main/protection -F required_status_checks='{\"strict\":true,\"contexts\":[\"unit\",\"contract\",\"lint\",\"sast\",\"sbom-sign\"]}' -F enforce_admins=true -F required_pull_request_reviews='{\"dismiss_stale_reviews\":true,\"require_code_owner_reviews\":true,\"required_approving_review_count\":1}' -F required_linear_history=true -F allow_force_pushes=false -F allow_deletions=false || true
