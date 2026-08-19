<!-- markdownlint-disable -->

# Hardening Report: nick-fields--assert-action/v4.0.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **nick-fields--assert-action/v4.0.1** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

The workflow references three external actions using mutable version tags instead of full 40-character commit SHAs. These can be silently updated to point to malicious code: `uses: actions/checkout@v6` (lines 11 and 80), `uses: actions/setup-node@v6` (line 82), and `uses: cycjimmy/semantic-release-action@v6` (line 88). Each should be pinned to a full SHA, e.g. `actions/checkout@<40-hex-sha> # v6`.

Locations:

- `.github/workflows/ci.yml:11`
- `.github/workflows/ci.yml:80`
- `.github/workflows/ci.yml:82`
- `.github/workflows/ci.yml:88`

### missing-permissions (severity: medium)

The workflow file has no top-level `permissions:` key and neither the `tests` job nor the `cd` job defines its own `permissions:` block. Without explicit permissions, the workflow inherits the repository's default token permissions (which may be `write-all`). A minimal permissions block (e.g. `contents: write` for the cd job that pushes tags, and `contents: read` for the tests job) should be added.

Locations:

- `.github/workflows/ci.yml:1`

### script-injection (severity: high)

Rule (b) violation: The `Tag` step in the `cd` job expands `${MAJOR_VERSION}` unquoted inside the `run:` shell command (`git tag -f v${MAJOR_VERSION} && git push -f origin v${MAJOR_VERSION}`). The variable `MAJOR_VERSION` is sourced from `steps.semantic.outputs.new_release_major_version` (a `steps.*.outputs.*` value, which is workflow-controllable/untrusted). An attacker who can influence the semantic-release output could inject shell metacharacters. The fix is to double-quote the expansion: `git tag -f "v${MAJOR_VERSION}" && git push -f origin "v${MAJOR_VERSION}"`.

Locations:

- `.github/workflows/ci.yml:92`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, missing-permissions, script-injection

**Notes:**

Fixed all three findings in .github/workflows/ci.yml: (1) Pinned actions/checkout@v6 → @df4cb1c069e1874edd31b4311f1884172cec0e10 # v6 (used in both tests and cd jobs), actions/setup-node@v6 → @249970729cb0ef3589644e2896645e5dc5ba9c38 # v6, and cycjimmy/semantic-release-action@v6 → @b12c8f6015dc215fe37bc154d4ad456dd3833c90 # v6. (2) Added `permissions: contents: read` to the tests job and `permissions: contents: write` to the cd job (minimum needed to push tags). (3) Double-quoted ${MAJOR_VERSION} in the Tag step's run command to prevent shell injection from the semantic-release output.

