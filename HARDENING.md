<!-- markdownlint-disable -->

# Hardening Report: flox--install-flox-action/v2.4.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **flox--install-flox-action/v2.4.0** was hardened automatically. 4 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Multiple workflow files reference actions by mutable tags or branch names instead of full 40-character commit SHAs, making them vulnerable to supply-chain attacks.

.github/workflows/ci.yml:
- actions/checkout@v6 (lines 30, 57, 90, 117, 141)
- DeterminateSystems/nix-installer-action@main (line ~122)
- cachix/install-nix-action@v31 (line ~126)
- rtCamp/action-slack-notify@v2 (line ~163)

.github/workflows/update-dist.yml:
- actions/checkout@v6 (line ~33)
- stefanzweifel/git-auto-commit-action@v7 (line ~47)

.github/workflows/update-flox-env.yml:
- actions/checkout@v6 (line ~14)
- flox/install-flox-action@v2.3.0 (line ~18)
- peter-evans/create-pull-request@v8 (line ~22)

.github/workflows/auto-label.yml:
- actions-ecosystem/action-add-labels@v1 (line ~14)

Locations:

- `.github/workflows/ci.yml:30`
- `.github/workflows/ci.yml:122`
- `.github/workflows/ci.yml:126`
- `.github/workflows/ci.yml:163`
- `.github/workflows/update-dist.yml:33`
- `.github/workflows/update-dist.yml:47`
- `.github/workflows/update-flox-env.yml:14`
- `.github/workflows/update-flox-env.yml:18`
- `.github/workflows/update-flox-env.yml:22`
- `.github/workflows/auto-label.yml:14`

### script-injection (severity: high)

Sub-rule (a): The 'Verify outputs' step in ci.yml directly interpolates ${{ steps.install.outputs.flox-version }}, ${{ steps.install.outputs.flox-path }}, and ${{ steps.install.outputs.nix-detected }} inside a run: shell script. Although these come from a local action's outputs rather than direct user input, steps.*.outputs.* is a workflow-controllable context that flows through YAML template substitution before the shell sees it, making it a script-injection risk. Any newline or shell metacharacter in the output value would be interpreted by the shell.

Offending lines:
  echo "Flox version: ${{ steps.install.outputs.flox-version }}"
  echo "Flox path: ${{ steps.install.outputs.flox-path }}"
  echo "Nix detected: ${{ steps.install.outputs.nix-detected }}"
  test -n "${{ steps.install.outputs.flox-version }}"
  test -n "${{ steps.install.outputs.flox-path }}"

Locations:

- `.github/workflows/ci.yml:101`

### unsafe-shell (severity: high)

The 'Install act' step in ci.yml pipes a remote script directly to 'sudo bash' without first downloading and inspecting it. This pattern executes arbitrary remote code with elevated privileges and is vulnerable to man-in-the-middle attacks or compromise of the remote host (raw.githubusercontent.com).

Offending command:
  curl https://raw.githubusercontent.com/nektos/act/master/install.sh | sudo bash

Locations:

- `.github/workflows/ci.yml:145`

### missing-permissions (severity: medium)

Two workflow files have no top-level 'permissions:' block and no job-level 'permissions:' blocks on any of their jobs. Without explicit permissions, GitHub Actions grants the default token permissions (which may include write access to contents, packages, etc. depending on repository settings), violating the principle of least privilege.

- ci.yml: six jobs (test-javascript, test-action, test-new-inputs, test-existing-nix, test-act, report-failure) — none have permissions defined.
- update-flox-env.yml: one job (update) — no permissions defined.

Locations:

- `.github/workflows/ci.yml:1`
- `.github/workflows/update-flox-env.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection, unsafe-shell, missing-permissions

**Notes:**

Fixed all four findings across four workflow files:

1. unpinned-uses: Pinned all 8 action references to full 40-char SHAs with tag comments preserved: actions/checkout@v6→d23441a4, DeterminateSystems/nix-installer-action@main→33c9ab3e, cachix/install-nix-action@v31→630ae543, rtCamp/action-slack-notify@v2→33ca3be6, stefanzweifel/git-auto-commit-action@v7→4a55954c, flox/install-flox-action@v2.3.0→9428713e, peter-evans/create-pull-request@v8→5f6978fa, actions-ecosystem/action-add-labels@v1→18f1af5e.

2. script-injection: Moved all ${{ steps.install.outputs.* }} expressions in the 'Verify outputs' step into the step's env: block (FLOX_VERSION, FLOX_PATH, NIX_DETECTED) and referenced them as plain shell variables.

3. unsafe-shell: Changed 'curl ... | sudo bash' to download the script to /tmp/install-act.sh first, then execute it separately with 'sudo bash /tmp/install-act.sh'.

4. missing-permissions: Added top-level 'permissions: {}' to ci.yml and update-flox-env.yml. Added job-level 'permissions: contents: read' to all ci.yml jobs (test-javascript, test-action, test-new-inputs, test-existing-nix, test-act) and 'permissions: {}' to report-failure. Added 'permissions: contents: write; pull-requests: write' to the update job in update-flox-env.yml.

