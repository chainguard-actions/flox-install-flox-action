<!-- markdownlint-disable -->

# Hardening Report: flox--install-flox-action/v2.5.2

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **flox--install-flox-action/v2.5.2** was hardened automatically. 4 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): The 'Verify outputs' run: block in ci.yml directly interpolates ${{ steps.install.outputs.flox-version }}, ${{ steps.install.outputs.flox-path }}, and ${{ steps.install.outputs.nix-detected }} inside shell commands. Any ${{ ... }} expression interpolated directly in a run: block is a script-injection risk because the value is substituted by the YAML template engine before the shell ever sees it, allowing shell metacharacters to be injected. Offending lines:
  echo "Flox version: ${{ steps.install.outputs.flox-version }}"
  echo "Flox path: ${{ steps.install.outputs.flox-path }}"
  echo "Nix detected: ${{ steps.install.outputs.nix-detected }}"
  test -n "${{ steps.install.outputs.flox-version }}"
  test -n "${{ steps.install.outputs.flox-path }}"

Locations:

- `.github/workflows/ci.yml:132`

### unsafe-shell (severity: high)

The 'Install act' step pipes the output of curl directly to sudo bash without first saving the script to a file for inspection. This allows a compromised or malicious remote server to execute arbitrary code with elevated privileges on the runner. Offending pattern: `curl https://raw.githubusercontent.com/nektos/act/master/install.sh | sudo bash`

Locations:

- `.github/workflows/ci.yml:195`

### missing-permissions (severity: medium)

ci.yml has no top-level permissions: block and none of its six jobs (test-javascript, test-action, test-new-inputs, test-existing-nix, test-act, report-failure) define a job-level permissions: block. Without explicit permissions, the workflow inherits the repository's default token permissions, which may be overly broad (write-all in some configurations). Minimal explicit permissions should be declared.

Locations:

- `.github/workflows/ci.yml:1`

### missing-permissions (severity: medium)

update-flox-env.yml has no top-level permissions: block and its single job ('update') has no job-level permissions: block. Without explicit permissions, the workflow inherits the repository's default token permissions, which may be overly broad. Minimal explicit permissions (e.g. contents: write for the PR creation) should be declared.

Locations:

- `.github/workflows/update-flox-env.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, unsafe-shell, missing-permissions

**Notes:**

Fixed all four findings in ci.yml and update-flox-env.yml:
1. script-injection: Moved ${{ steps.install.outputs.flox-version }}, ${{ steps.install.outputs.flox-path }}, and ${{ steps.install.outputs.nix-detected }} from the 'Verify outputs' run: block into an env: block (FLOX_VERSION, FLOX_PATH, NIX_DETECTED), referencing them as plain shell variables.
2. unsafe-shell: Replaced 'curl ... | sudo bash' with a two-step approach: download to /tmp/install-act.sh first, then execute with 'sudo bash /tmp/install-act.sh'.
3. missing-permissions (ci.yml): Added top-level 'permissions: contents: read' block.
4. missing-permissions (update-flox-env.yml): Added top-level 'permissions: contents: read' block (PR creation uses a custom external token, not the default GITHUB_TOKEN).

