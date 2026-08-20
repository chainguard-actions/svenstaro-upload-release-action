<!-- markdownlint-disable -->

# Hardening Report: svenstaro--upload-release-action/2.11.2

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **svenstaro--upload-release-action/2.11.2** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Multiple workflow files reference actions using mutable tags or branch names instead of pinned full-length SHA commit hashes, making them vulnerable to supply-chain attacks if the referenced tag is moved or the action is compromised.

- .github/workflows/build.yml: `uses: actions/checkout@v4`
- .github/workflows/e2e_test.yml: `uses: actions/checkout@v4`, `uses: actions/github-script@v7` (multiple steps)
- .github/workflows/versioning.yml: `uses: Actions-R-Us/actions-tagger@latest`

Locations:

- `.github/workflows/build.yml:11`
- `.github/workflows/e2e_test.yml:16`
- `.github/workflows/e2e_test.yml:23`
- `.github/workflows/versioning.yml:9`

### missing-permissions (severity: medium)

None of the three workflow files define a top-level `permissions:` key, and no individual jobs define job-level `permissions:` blocks. Without explicit permissions, workflows run with the default (often overly broad) token permissions, violating the principle of least privilege.

Affected files: build.yml, e2e_test.yml, versioning.yml.

Locations:

- `.github/workflows/build.yml:1`
- `.github/workflows/e2e_test.yml:1`
- `.github/workflows/versioning.yml:1`

### script-injection (severity: high)

Sub-rule (a): A `run:` block in e2e_test.yml directly interpolates a `${{ ... }}` expression inside a shell command string. The expression `${{ steps.test-step.outcome }}` is substituted into the shell script before the shell parses it, which can allow injection of shell metacharacters if the value is attacker-influenced.

Offending line:
  `if [ "${{ steps.test-step.outcome }}" == "failure" ]; then`

The value should be passed via an `env:` variable and referenced as `"$ENV_VAR"` in the shell script instead.

Locations:

- `.github/workflows/e2e_test.yml:148`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, missing-permissions, script-injection

**Notes:**

Fixed all three findings across the three workflow files:

1. **unpinned-uses**: Pinned all action references to full commit SHAs:
   - actions/checkout@v4 → @11d5960a326750d5838078e36cf38b85af677262 (build.yml, e2e_test.yml)
   - actions/github-script@v7 → @f28e40c7f34bde8b3046d885e986cb6290c5673b (6 occurrences in e2e_test.yml)
   - Actions-R-Us/actions-tagger@latest → @330ddfac760021349fef7ff62b372f2f691c20fb (versioning.yml)

2. **missing-permissions**: Added top-level `permissions: {}` to all three workflow files. Added job-level `permissions: contents: write` to e2e_test.yml (needs to create/delete releases) and versioning.yml (needs to manage tags).

3. **script-injection**: Fixed the 'Verify failure occurred' step in e2e_test.yml by moving `${{ steps.test-step.outcome }}` into an `env:` block as `TEST_STEP_OUTCOME` and referencing it as `"$TEST_STEP_OUTCOME"` in the shell script.

