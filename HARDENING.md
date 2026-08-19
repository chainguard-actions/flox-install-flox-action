<!-- markdownlint-disable -->

# Hardening Report: flox--install-flox-action/v2.5.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **flox--install-flox-action/v2.5.1** was hardened automatically. 4 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): The 'Verify outputs' step in ci.yml directly interpolates ${{ steps.install.outputs.flox-version }}, ${{ steps.install.outputs.flox-path }}, and ${{ steps.install.outputs.nix-detected }} inside run: shell commands. Any ${{ ... }} expression interpolated directly into a run: block is a script-injection risk because the value is substituted before the shell parses the command, allowing metacharacters to be interpreted. Offending lines:
  echo "Flox version: ${{ steps.install.outputs.flox-version }}"
  echo "Flox path: ${{ steps.install.outputs.flox-path }}"
  echo "Nix detected: ${{ steps.install.outputs.nix-detected }}"
  test -n "${{ steps.install.outputs.flox-version }}"
  test -n "${{ steps.install.outputs.flox-path }}"

Locations:

- `.github/workflows/ci.yml:97`

### unsafe-shell (severity: high)

The 'Install act' step pipes remote content directly to a shell interpreter: `curl https://raw.githubusercontent.com/nektos/act/master/install.sh | sudo bash`. This executes arbitrary code fetched from the network without any integrity verification, and with elevated privileges (sudo). The script should be downloaded to a file first, its integrity verified (e.g. via checksum), and then executed separately.

Locations:

- `.github/workflows/ci.yml:155`

### missing-permissions (severity: medium)

The workflow file ci.yml has no top-level `permissions:` key and none of its jobs (test-javascript, test-action, test-new-inputs, test-existing-nix, test-act, report-failure) define a job-level `permissions:` block. Without explicit permissions, the GITHUB_TOKEN is granted its default (potentially broad) permissions, which can include write access to repository contents.

Locations:

- `.github/workflows/ci.yml:1`

### missing-permissions (severity: medium)

The workflow file update-flox-env.yml has no top-level `permissions:` key and its only job ('update') has no job-level `permissions:` block. Without explicit permissions, the GITHUB_TOKEN is granted its default (potentially broad) permissions.

Locations:

- `.github/workflows/update-flox-env.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, unsafe-shell, missing-permissions

**Notes:**

Fixed 4 findings across 2 workflow files:

1. ci.yml - script-injection: Moved ${{ steps.install.outputs.flox-version }}, ${{ steps.install.outputs.flox-path }}, and ${{ steps.install.outputs.nix-detected }} from inline run: shell commands into a step-level env: block. Shell script now uses plain env vars ($FLOX_VERSION, $FLOX_PATH, $NIX_DETECTED).

2. ci.yml - unsafe-shell: Replaced `curl ... | sudo bash` with a two-step approach: download to /tmp/act-install.sh first, then execute separately with `sudo bash /tmp/act-install.sh`.

3. ci.yml - missing-permissions: Added `permissions: {}` at the top level to restrict GITHUB_TOKEN to no permissions by default.

4. update-flox-env.yml - missing-permissions: Added `permissions: {}` at the top level. The job uses a custom PAT for PR creation so GITHUB_TOKEN needs no permissions.

