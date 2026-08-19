<!-- markdownlint-disable -->

# Hardening Report: nick-fields--assert-action/v3.0.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **nick-fields--assert-action/v3.0.1** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

The workflow file references external actions using mutable version tags instead of pinned full-length SHA commit hashes. This exposes the workflow to supply-chain attacks if a tag is moved or a repository is compromised. Failing references: actions/checkout@v6 (line 11), actions/checkout@v6 (line 113), actions/setup-node@v6 (line 116), cycjimmy/semantic-release-action@v6 (line 121). All should be replaced with their full 40-character commit SHA.

Locations:

- `.github/workflows/ci.yml:11`
- `.github/workflows/ci.yml:113`
- `.github/workflows/ci.yml:116`
- `.github/workflows/ci.yml:121`

### missing-permissions (severity: medium)

The workflow file has no top-level permissions: key, and neither the tests job nor the cd job defines its own permissions: block. Without explicit permissions, the workflow inherits the repository default token permissions, which may be overly broad. Minimal permissions should be declared (e.g. contents: write for the cd job that pushes tags, contents: read for the tests job).

Locations:

- `.github/workflows/ci.yml:1`

### script-injection (severity: high)

Rule (b) violation in the Tag step: the shell variable ${MAJOR_VERSION} is expanded unquoted inside the run: block. MAJOR_VERSION is sourced from steps.semantic.outputs.new_release_major_version (a steps.*.outputs.* value), which is workflow-controllable. An unquoted expansion allows shell metacharacters in the value to be interpreted by the shell, enabling command injection. Offending line: `run: git tag -f v${MAJOR_VERSION} && git push -f origin v${MAJOR_VERSION}`. Fix: quote the variable — `git tag -f "v${MAJOR_VERSION}" && git push -f origin "v${MAJOR_VERSION}"`.

Locations:

- `.github/workflows/ci.yml:126`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, missing-permissions, script-injection

**Notes:**

1. Pinned all 4 unpinned action references to full SHA: actions/checkout@v6 → d23441a48e516b6c34aea4fa41551a30e30af803, actions/setup-node@v6 → 249970729cb0ef3589644e2896645e5dc5ba9c38, cycjimmy/semantic-release-action@v6 → b12c8f6015dc215fe37bc154d4ad456dd3833c90. Original tags preserved as comments. 2. Added top-level `permissions: {}`, `permissions: contents: read` for the tests job, and `permissions: contents: write` for the cd job (which pushes tags). 3. Quoted ${MAJOR_VERSION} in the Tag step run command to prevent shell metacharacter injection: `git tag -f "v${MAJOR_VERSION}" && git push -f origin "v${MAJOR_VERSION}"`.

