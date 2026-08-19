<!-- markdownlint-disable -->

# Hardening Report: flox--install-flox-action/v2.5.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **flox--install-flox-action/v2.5.0** was hardened automatically. 4 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): GitHub Actions expressions are directly interpolated inside run: shell commands. In the 'Verify outputs' step of the test-new-inputs job, the values ${{ steps.install.outputs.flox-version }}, ${{ steps.install.outputs.flox-path }}, and ${{ steps.install.outputs.nix-detected }} are embedded directly in shell command strings (echo and test). These expressions flow through YAML template substitution before the shell processes them, allowing a malicious value to inject arbitrary shell commands.

Locations:

- `.github/workflows/ci.yml:89`

### unsafe-shell (severity: high)

The 'Install act' step pipes a remote script directly to a shell interpreter without first downloading and verifying it: `curl https://raw.githubusercontent.com/nektos/act/master/install.sh | sudo bash`. If the remote URL is compromised or the connection is intercepted, arbitrary code will be executed with sudo privileges on the runner.

Locations:

- `.github/workflows/ci.yml:148`

### missing-permissions (severity: medium)

The workflow file has no top-level permissions: key and none of its jobs (test-javascript, test-action, test-new-inputs, test-existing-nix, test-act, report-failure) define a job-level permissions: block. Without explicit permissions, the workflow inherits the repository's default token permissions, which may be overly broad.

Locations:

- `.github/workflows/ci.yml:1`

### missing-permissions (severity: medium)

The workflow file has no top-level permissions: key and its only job (update) has no job-level permissions: block. Without explicit permissions, the workflow inherits the repository's default token permissions, which may be overly broad.

Locations:

- `.github/workflows/update-flox-env.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, unsafe-shell, missing-permissions

**Notes:**

Fixed all 4 findings across 2 workflow files:
1. ci.yml script-injection (line 89): Moved ${{ steps.install.outputs.* }} expressions into an env: block on the 'Verify outputs' step; shell script now references $FLOX_VERSION, $FLOX_PATH, $NIX_DETECTED as plain env vars.
2. ci.yml unsafe-shell (line 148): Changed 'Install act' step from `curl ... | sudo bash` to downloading the script to /tmp/install-act.sh first, then executing `sudo bash /tmp/install-act.sh` separately.
3. ci.yml missing-permissions: Added `permissions: {}` at the top level.
4. update-flox-env.yml missing-permissions: Added `permissions: {}` at the top level. PR creation uses a custom scoped token, so GITHUB_TOKEN needs no permissions.

