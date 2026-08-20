<!-- markdownlint-disable -->

# Hardening Report: svenstaro--upload-release-action/2.11.4

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **svenstaro--upload-release-action/2.11.4** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Multiple workflow files reference GitHub Actions using mutable tags or branch names instead of full 40-character commit SHAs, making them vulnerable to supply-chain attacks if the referenced tag is moved or the action is compromised.

- build.yml: `uses: actions/checkout@v4` (tag, not SHA)
- e2e_test.yml: `uses: actions/checkout@v4` (tag, not SHA); `uses: actions/github-script@v7` (tag, not SHA) — appears 6 times
- versioning.yml: `uses: Actions-R-Us/actions-tagger@latest` (branch/tag, not SHA)

Locations:

- `.github/workflows/build.yml:11`
- `.github/workflows/e2e_test.yml:14`
- `.github/workflows/versioning.yml:10`

### missing-permissions (severity: medium)

None of the three workflow files define a top-level `permissions:` key, and no job within them defines job-level `permissions:` either. This means workflows run with the default (often broad) token permissions, violating the principle of least privilege.

- build.yml: no permissions block at top-level or job-level
- e2e_test.yml: no permissions block at top-level or job-level (the comment acknowledges content permissions are needed but they are never explicitly scoped)
- versioning.yml: no permissions block at top-level or job-level

Locations:

- `.github/workflows/build.yml:1`
- `.github/workflows/e2e_test.yml:1`
- `.github/workflows/versioning.yml:1`

### script-injection (severity: high)

Sub-rule (a): A `run:` block in e2e_test.yml directly interpolates a `${{ steps.test-step.outcome }}` expression into a shell command string. Although `steps.*.outputs.*` values are not directly attacker-controlled here, any `${{ ... }}` expression inside a `run:` block is a script-injection risk because the value is substituted by the YAML template engine before the shell ever sees it, bypassing shell quoting. The offending line is:

  `if [ "${{ steps.test-step.outcome }}" == "failure" ]; then`

The safe pattern is to pass the value via an `env:` variable and reference it as `"$OUTCOME"` in the shell.

Locations:

- `.github/workflows/e2e_test.yml:163`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, missing-permissions, script-injection

**Notes:**

Fixed all three findings across three workflow files:

1. **unpinned-uses**: Pinned all action references to full commit SHAs:
   - `actions/checkout@v4` → `@11d5960a326750d5838078e36cf38b85af677262 # v4` (build.yml, e2e_test.yml)
   - `actions/github-script@v7` → `@f28e40c7f34bde8b3046d885e986cb6290c5673b # v7` (6 occurrences in e2e_test.yml)
   - `Actions-R-Us/actions-tagger@latest` → `@330ddfac760021349fef7ff62b372f2f691c20fb # latest` (versioning.yml)

2. **missing-permissions**: Added top-level `permissions:` blocks:
   - `build.yml`: `permissions: {}` (no token access needed for npm build)
   - `e2e_test.yml`: `permissions: contents: write` (required to create/delete releases)
   - `versioning.yml`: `permissions: contents: write` (required to update/create tags)

3. **script-injection**: Fixed the 'Verify failure occurred' step in e2e_test.yml by moving `${{ steps.test-step.outcome }}` into an `env:` block as `OUTCOME`, then referencing it as `$OUTCOME` in the shell script, preventing template injection.

