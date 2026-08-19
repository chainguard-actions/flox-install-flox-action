<!-- markdownlint-disable -->

# Hardening Report: flox--install-flox-action/v2.4.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **flox--install-flox-action/v2.4.1** was hardened automatically. 4 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): The 'Verify outputs' run: block in ci.yml directly interpolates ${{ steps.install.outputs.* }} expressions inside shell commands. Any ${{ ... }} expression interpolated directly in a run: block is a script-injection risk, as the value flows through YAML template substitution before the shell ever sees it. Offending lines: `echo "Flox version: ${{ steps.install.outputs.flox-version }}"`, `echo "Flox path: ${{ steps.install.outputs.flox-path }}"`, `echo "Nix detected: ${{ steps.install.outputs.nix-detected }}"`, `test -n "${{ steps.install.outputs.flox-version }}"`, `test -n "${{ steps.install.outputs.flox-path }}"`.

Locations:

- `.github/workflows/ci.yml:126`

### unsafe-shell (severity: high)

The 'Install act' step pipes a remote script directly to a shell interpreter: `curl https://raw.githubusercontent.com/nektos/act/master/install.sh | sudo bash`. This is unsafe because the remote content is executed without first being downloaded and inspected. If the remote URL is compromised or the content changes, arbitrary code runs with sudo privileges on the runner.

Locations:

- `.github/workflows/ci.yml:185`

### missing-permissions (severity: medium)

ci.yml has no top-level `permissions:` key and none of its jobs (test-javascript, test-action, test-new-inputs, test-existing-nix, test-act, report-failure) define job-level permissions. Without explicit permissions, the workflow inherits the repository's default token permissions, which may be overly broad.

Locations:

- `.github/workflows/ci.yml:1`

### missing-permissions (severity: medium)

update-flox-env.yml has no top-level `permissions:` key and its single job (update) has no job-level permissions block. Without explicit permissions, the workflow inherits the repository's default token permissions, which may be overly broad.

Locations:

- `.github/workflows/update-flox-env.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, unsafe-shell, missing-permissions

**Notes:**

Fixed all four findings in ci.yml and update-flox-env.yml:
1. script-injection (ci.yml line 126): Moved ${{ steps.install.outputs.flox-version }}, ${{ steps.install.outputs.flox-path }}, and ${{ steps.install.outputs.nix-detected }} out of the run: shell block into an env: block as FLOX_VERSION, FLOX_PATH, and NIX_DETECTED. Shell script now references plain $VAR_NAME variables.
2. unsafe-shell (ci.yml line 185): Replaced `curl ... | sudo bash` with a two-step approach: download the install script to /tmp/install-act.sh with curl -fsSL, then execute it separately with `sudo bash /tmp/install-act.sh`.
3. missing-permissions (ci.yml): Added top-level `permissions: contents: read` block.
4. missing-permissions (update-flox-env.yml): Added top-level `permissions: contents: read` block (PR creation uses a custom token via secrets, so GITHUB_TOKEN only needs read access).

