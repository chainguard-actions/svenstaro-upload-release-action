<!-- markdownlint-disable -->

# Hardening Report: svenstaro--upload-release-action/2.11.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **svenstaro--upload-release-action/2.11.1** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Multiple workflow files reference actions by mutable tag or branch refs instead of pinned 40-character commit SHAs, making them vulnerable to supply-chain attacks if the referenced tag is moved or compromised.

- build.yml: `actions/checkout@v4` (tag ref)
- e2e_test.yml: `actions/checkout@v4` (tag ref), `actions/github-script@v7` (tag ref, used 6 times)
- versioning.yml: `Actions-R-Us/actions-tagger@latest` (branch/latest ref — especially dangerous as `latest` is a mutable floating ref)

Locations:

- `.github/workflows/build.yml:12`
- `.github/workflows/e2e_test.yml:16`
- `.github/workflows/e2e_test.yml:30`
- `.github/workflows/e2e_test.yml:62`
- `.github/workflows/e2e_test.yml:81`
- `.github/workflows/e2e_test.yml:107`
- `.github/workflows/e2e_test.yml:130`
- `.github/workflows/e2e_test.yml:172`
- `.github/workflows/versioning.yml:11`

### missing-permissions (severity: medium)

None of the three workflow files declare a top-level `permissions:` key, and no individual job within them declares job-level permissions either. Without explicit permissions, workflows inherit the default repository token permissions (which may be broad), violating the principle of least privilege.

Locations:

- `.github/workflows/build.yml:1`
- `.github/workflows/e2e_test.yml:1`
- `.github/workflows/versioning.yml:1`

### script-injection (severity: high)

Sub-rule (a): A `${{ }}` expression is interpolated directly inside a `run:` shell command string in the 'Verify failure occurred' step of e2e_test.yml. The expression `${{ steps.test-step.outcome }}` is substituted into the shell script before execution. Although `steps.*.outputs.*` values are typically controlled by the action itself, any `${{ ... }}` inside a `run:` block is a script-injection risk because the value flows through YAML template substitution before the shell ever sees it. Offending line: `if [ "${{ steps.test-step.outcome }}" == "failure" ]; then`

Locations:

- `.github/workflows/e2e_test.yml:166`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, missing-permissions, script-injection

**Notes:**

Fixed all three findings across the three workflow files:

1. **unpinned-uses**: Pinned all action references to full 40-char commit SHAs:
   - actions/checkout@v4 → @11d5960a326750d5838078e36cf38b85af677262 (build.yml, e2e_test.yml)
   - actions/github-script@v7 → @f28e40c7f34bde8b3046d885e986cb6290c5673b (6 occurrences in e2e_test.yml)
   - Actions-R-Us/actions-tagger@latest → @330ddfac760021349fef7ff62b372f2f691c20fb (versioning.yml)

2. **missing-permissions**: Added `permissions: {}` at the top level of all three workflow files. Job-level `permissions: contents: write` added to e2e_test.yml's test job (needs to create/delete releases) and versioning.yml's actions-tagger job (needs to update tags).

3. **script-injection**: Fixed the 'Verify failure occurred' step in e2e_test.yml by moving `${{ steps.test-step.outcome }}` into the step's `env:` block as `TEST_STEP_OUTCOME`, then referencing it as `$TEST_STEP_OUTCOME` in the shell script.

