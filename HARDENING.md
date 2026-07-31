<!-- markdownlint-disable -->

# Hardening Report: flox--install-flox-action/v2.6.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **flox--install-flox-action/v2.6.0** was hardened automatically. 5 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): GitHub Actions expressions are interpolated directly inside run: shell command strings. In the 'Verify outputs' step of the test-new-inputs job, five step output values are embedded directly: `echo "Flox version: ${{ steps.install.outputs.flox-version }}"`, `echo "Flox path: ${{ steps.install.outputs.flox-path }}"`, `echo "Nix detected: ${{ steps.install.outputs.nix-detected }}"`, `test -n "${{ steps.install.outputs.flox-version }}"`, and `test -n "${{ steps.install.outputs.flox-path }}"`  — all without routing through env: variables. These values flow through YAML template substitution before the shell sees them, enabling command injection if the action outputs contain shell metacharacters.

Locations:

- `.github/workflows/ci.yml:132`

### script-injection (severity: high)

Sub-rule (a): GitHub Actions expression interpolated directly inside a run: shell command string. In the 'Verify the second run skipped installation' step of the test-repeat-run job: `if [[ "${{ steps.second.outputs.flox-preinstalled }}" != "true" ]]; then` — the step output is embedded directly into the shell script without routing through an env: variable, enabling command injection if the output contains shell metacharacters.

Locations:

- `.github/workflows/ci.yml:213`

### unsafe-shell (severity: high)

The 'Install act' step in the test-act job pipes remote content directly to a shell interpreter: `curl https://raw.githubusercontent.com/nektos/act/master/install.sh | sudo bash`. This executes arbitrary remote code without first downloading and inspecting the script, and also fetches from a mutable branch reference (master) rather than a pinned commit.

Locations:

- `.github/workflows/ci.yml:235`

### missing-permissions (severity: medium)

The workflow file has no top-level `permissions:` key and none of its seven jobs (test-javascript, test-action, test-new-inputs, test-existing-nix, test-repeat-run, test-act, report-failure) define a job-level `permissions:` block. This means the workflow runs with the default (potentially write) token permissions, violating the principle of least privilege.

Locations:

- `.github/workflows/ci.yml:1`

### missing-permissions (severity: medium)

The workflow file has no top-level `permissions:` key and its single job (`update`) has no job-level `permissions:` block. This means the workflow runs with the default (potentially write) token permissions, violating the principle of least privilege.

Locations:

- `.github/workflows/update-flox-env.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, unsafe-shell, missing-permissions

**Notes:**

Fixed all 5 findings across 2 workflow files:

1. ci.yml - script-injection (line 132): Moved steps.install.outputs.flox-version, flox-path, and nix-detected expressions into env: block as FLOX_VERSION, FLOX_PATH, NIX_DETECTED variables; updated run: to use plain shell variable references.

2. ci.yml - script-injection (line 213): Moved steps.second.outputs.flox-preinstalled expression into env: block as FLOX_PREINSTALLED variable; updated run: to use plain shell variable reference.

3. ci.yml - unsafe-shell (line 235): Replaced `curl ... | sudo bash` with a two-step download-then-execute pattern. Script is now fetched to /tmp/install-act.sh from a pinned commit SHA (4f411281417e88660bea1c1a1749aa71ae0bd60f) instead of the mutable master branch, then executed separately.

4. ci.yml - missing-permissions: Added top-level `permissions: contents: read` block.

5. update-flox-env.yml - missing-permissions: Added top-level `permissions: contents: read` and job-level `permissions: contents: write, pull-requests: write` (minimum needed for git checkout with fetch-depth and peter-evans/create-pull-request action).

