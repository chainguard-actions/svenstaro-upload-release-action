<!-- markdownlint-disable -->

# Hardening Report: svenstaro--upload-release-action/2.11.5

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **svenstaro--upload-release-action/2.11.5** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Multiple workflow files reference GitHub Actions using mutable tags or branch names instead of pinned full 40-character commit SHAs, making them vulnerable to supply-chain attacks if the referenced action is compromised or modified.

- build.yml: `uses: actions/checkout@v6` (tag)
- e2e_test.yml: `uses: actions/checkout@v6` (tag), `uses: actions/github-script@v8` (tag, used 6 times)
- versioning.yml: `uses: Actions-R-Us/actions-tagger@latest` (branch)

Locations:

- `.github/workflows/build.yml:11`
- `.github/workflows/e2e_test.yml:16`
- `.github/workflows/versioning.yml:11`

### missing-permissions (severity: medium)

None of the workflow files define a top-level `permissions:` key, and no individual jobs define job-level `permissions:` keys. Without explicit permissions, workflows run with the default (potentially broad) token permissions, violating the principle of least privilege.

Locations:

- `.github/workflows/build.yml:1`
- `.github/workflows/e2e_test.yml:1`
- `.github/workflows/versioning.yml:1`

### script-injection (severity: high)

Sub-rule (a) violation: A `run:` block in e2e_test.yml directly interpolates a GitHub Actions expression `${{ steps.test-step.outcome }}` inside a shell command string. Although `steps.*.outputs.*` is not directly attacker-controlled in this case, any `${{ ... }}` expression inside a `run:` block is processed by the template engine before the shell sees it, bypassing shell quoting and creating a script injection risk. Offending line: `if [ "${{ steps.test-step.outcome }}" == "failure" ]; then`

Locations:

- `.github/workflows/e2e_test.yml:167`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, missing-permissions, script-injection

**Notes:**

Fixed all three findings across three workflow files:

1. **unpinned-uses**: Pinned all action references to full 40-char SHAs with tag comments preserved:
   - actions/checkout@v6 → @d23441a48e516b6c34aea4fa41551a30e30af803 (build.yml, e2e_test.yml)
   - actions/github-script@v8 → @ed597411d8f924073f98dfc5c65a23a2325f34cd (6 occurrences in e2e_test.yml)
   - Actions-R-Us/actions-tagger@latest → @330ddfac760021349fef7ff62b372f2f691c20fb (versioning.yml)

2. **missing-permissions**: Added `permissions: {}` at the top level of all three workflow files. Added job-level `permissions: contents: write` for jobs that need to create/delete releases (e2e_test.yml) and update tags (versioning.yml).

3. **script-injection**: In e2e_test.yml's 'Verify failure occurred' step, moved `${{ steps.test-step.outcome }}` out of the `run:` shell string into the step's `env:` block as `TEST_STEP_OUTCOME`, then referenced it as `$TEST_STEP_OUTCOME` in the shell script.

