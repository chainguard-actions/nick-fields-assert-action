<!-- markdownlint-disable -->

# Hardening Report: nick-fields--assert-action/v3.0.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **nick-fields--assert-action/v3.0.0** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Multiple `uses:` references in the workflow are pinned to mutable tags rather than full 40-character SHA digests, making the workflow vulnerable to supply-chain attacks if those tags are moved or overwritten. Failing references: `actions/checkout@v2` (lines 11 and 91), `actions/setup-node@v1` (line 93), `cycjimmy/semantic-release-action@v2` (line 97). Each should be pinned to a full commit SHA, e.g. `actions/checkout@11bd71901bbe5b1630ceea73d27597364c9af683 # v2`.

Locations:

- `.github/workflows/ci.yml:11`
- `.github/workflows/ci.yml:91`
- `.github/workflows/ci.yml:93`
- `.github/workflows/ci.yml:97`

### missing-permissions (severity: medium)

The workflow file has no top-level `permissions:` key and neither the `tests` job nor the `cd` job defines its own `permissions:` block. Without explicit permissions, the workflow runs with the default (potentially broad) token permissions. A minimal permissions block (e.g. `contents: read`) should be added at the top level or per job.

Locations:

- `.github/workflows/ci.yml:1`

### script-injection (severity: high)

Sub-rule (b) violation: In the `Tag` step (cd job), the shell variable `${MAJOR_VERSION}` is expanded **unquoted** inside the `run:` command: `git tag -f v${MAJOR_VERSION} && git push -f origin v${MAJOR_VERSION}`. `MAJOR_VERSION` is sourced from `steps.semantic.outputs.new_release_major_version` — a workflow-controllable value. An unquoted expansion allows shell metacharacters in the value to be interpreted by the shell, enabling command injection. The fix is to double-quote the expansion: `"${MAJOR_VERSION}"`.

Locations:

- `.github/workflows/ci.yml:100`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, missing-permissions, script-injection

**Notes:**

1. Pinned all four unpinned action references to full commit SHAs: actions/checkout@v2 → @0717577d45739eb3c851188b29f50ed6c0b2194e, actions/setup-node@v1 → @f1f314fca9dfce2769ece7d933488f076716723e, cycjimmy/semantic-release-action@v2 → @5982a02995853159735cb838992248c4f0f16166. Tag comments preserved for readability. 2. Added top-level `permissions: contents: read` block to restrict default token permissions. The `cd` job overrides with `contents: write` since it needs to push tags and releases. 3. Double-quoted `${MAJOR_VERSION}` in the Tag step run command (`"v${MAJOR_VERSION}"`) to prevent shell metacharacter injection from the workflow-controlled value.

