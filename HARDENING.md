<!-- markdownlint-disable -->

# Hardening Report: thollander--actions-comment-pull-request/v3.0.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **thollander--actions-comment-pull-request/v3.0.1** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Multiple workflow steps use mutable tag-based action references instead of immutable full SHA commit hashes, making them vulnerable to supply-chain attacks if the tag is moved. Failing references in build.yaml: `actions/checkout@v3`, `actions/setup-node@v3`, `fregante/setup-git-user@v1`. Failing reference in ci.yaml: `actions/checkout@v3`.

Locations:

- `.github/workflows/build.yaml:8`
- `.github/workflows/build.yaml:11`
- `.github/workflows/build.yaml:22`
- `.github/workflows/ci.yaml:10`

### missing-permissions (severity: medium)

The workflow file build.yaml has no top-level `permissions:` key and its only job (`compile`) also has no job-level `permissions:` key. Without explicit permissions, the job inherits the default repository token permissions, which may be overly broad (write access to contents, etc.).

Locations:

- `.github/workflows/build.yaml:1`

### script-injection (severity: high)

Sub-rule (a): The 'Check outputs' step in ci.yaml directly interpolates GitHub Actions expressions inside a `run:` shell command string. The expressions `${{ steps.nrt-message.outputs.id }}`, `${{ steps.nrt-message.outputs.body }}`, and `${{ steps.nrt-message.outputs.html-url }}` are `steps.*.outputs.*` values — a workflow-controllable context — and are substituted into the shell command before the shell ever parses it, enabling script injection if the output values contain shell metacharacters.

Locations:

- `.github/workflows/ci.yaml:57`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, missing-permissions, script-injection

**Notes:**

Fixed all three findings:
1. **unpinned-uses**: Pinned all four action references to full commit SHAs with tag comments: `actions/checkout@v3` → `@a37ce9120846195fa4ece8f58b268e6043cb2f26 # v3` (in both build.yaml and ci.yaml), `actions/setup-node@v3` → `@3235b876344d2a9aa001b8d1453c930bba69e610 # v3`, `fregante/setup-git-user@v1` → `@2e28d51939d2a84005a917d2f844090637f435f8 # v1`.
2. **missing-permissions**: Added `permissions: {}` at the top level of build.yaml and `permissions: { contents: write }` at the job level (the compile job needs contents:write to push compiled artifacts back to main).
3. **script-injection**: Moved `${{ steps.nrt-message.outputs.id }}`, `${{ steps.nrt-message.outputs.body }}`, and `${{ steps.nrt-message.outputs.html-url }}` out of the `run:` shell string into an `env:` block as `NRT_ID`, `NRT_BODY`, and `NRT_HTML_URL`, then referenced them as plain environment variables in the shell script.

