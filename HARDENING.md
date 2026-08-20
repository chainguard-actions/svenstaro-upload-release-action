<!-- markdownlint-disable -->

# Hardening Report: svenstaro--upload-release-action/2.11.3

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **svenstaro--upload-release-action/2.11.3** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Workflow files reference actions by mutable tags instead of pinned full-length commit SHAs, making them vulnerable to supply-chain attacks if the tag is moved.

- build.yml: `uses: actions/checkout@v4` (line 12)
- e2e_test.yml: `uses: actions/checkout@v4` (line 16), `uses: actions/github-script@v7` (multiple steps at lines 24, 40, 57, 74, 99, 131)
- versioning.yml: `uses: Actions-R-Us/actions-tagger@latest` (line 11) — especially dangerous as `@latest` is a moving branch reference

Locations:

- `.github/workflows/build.yml:12`
- `.github/workflows/e2e_test.yml:16`
- `.github/workflows/versioning.yml:11`

### permissions (severity: medium)

None of the workflow files define a top-level `permissions:` block, and no job within them defines job-level `permissions:` either. Without explicit permissions, workflows inherit the repository's default token permissions, which may be overly broad (e.g., write access to contents, packages, etc.).

Locations:

- `.github/workflows/build.yml:1`
- `.github/workflows/e2e_test.yml:1`
- `.github/workflows/versioning.yml:1`

### script-injection (severity: high)

Sub-rule (a): A `run:` block in e2e_test.yml directly interpolates a GitHub Actions expression inside a shell command string. The expression `${{ steps.test-step.outcome }}` is substituted into the shell script before the shell parses it, allowing an attacker who can influence the step outcome string to inject shell metacharacters. The offending line is:

  `if [ "${{ steps.test-step.outcome }}" == "failure" ]; then`

Fix: move the value into an `env:` variable and reference it as a quoted shell variable, e.g.:
```yaml
env:
  STEP_OUTCOME: ${{ steps.test-step.outcome }}
run: |
  if [ "$STEP_OUTCOME" == "failure" ]; then
```

Locations:

- `.github/workflows/e2e_test.yml:128`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, permissions, script-injection

**Notes:**

Fixed all three findings across the three workflow files:

1. **unpinned-uses**: Pinned all action references to full commit SHAs:
   - actions/checkout@v4 → @11d5960a326750d5838078e36cf38b85af677262 (build.yml, e2e_test.yml)
   - actions/github-script@v7 → @f28e40c7f34bde8b3046d885e986cb6290c5673b (6 occurrences in e2e_test.yml)
   - Actions-R-Us/actions-tagger@latest → @330ddfac760021349fef7ff62b372f2f691c20fb (versioning.yml)

2. **permissions**: Added top-level permissions blocks to all three workflows:
   - build.yml: `permissions: {}` (no special permissions needed)
   - e2e_test.yml: `permissions: contents: write` (required for creating/deleting releases)
   - versioning.yml: `permissions: contents: write` (required for updating tags)

3. **script-injection**: Fixed the 'Verify failure occurred' step in e2e_test.yml by moving `${{ steps.test-step.outcome }}` into an `env:` block as `STEP_OUTCOME` and referencing it as `"$STEP_OUTCOME"` in the shell script.

