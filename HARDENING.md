<!-- markdownlint-disable -->

# Hardening Report: aquasecurity--setup-trivy/v0.2.6

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **aquasecurity--setup-trivy/v0.2.6** was hardened automatically. 4 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Rule (a): Multiple `run:` blocks in action.yaml directly interpolate `${{ ... }}` expressions inside shell command strings, enabling script injection. (1) Line 40: `echo "dir=${{ inputs.path }}/trivy-bin" >> $GITHUB_OUTPUT` — `inputs.path` is interpolated directly into the shell command before the shell ever sees it, allowing an attacker-controlled value to inject arbitrary shell metacharacters. (2) Line 82: `bash ./trivy/contrib/install.sh -b ${{ steps.binary-dir.outputs.dir }} -c setup-trivy ${{ inputs.version }}` — both `steps.binary-dir.outputs.dir` (derived from `inputs.path`) and `inputs.version` are interpolated unquoted into the shell command. (3) Line 84: `cp -r ./trivy/contrib ${{ steps.binary-dir.outputs.dir }}/contrib` — same issue. (4) Line 87: `echo ${{ steps.binary-dir.outputs.dir }} >> $GITHUB_PATH` — interpolated unquoted. All of these should be moved to `env:` variables and then referenced as double-quoted shell variables (e.g., `"$INSTALL_PATH"`).

Locations:

- `action.yaml:40`
- `action.yaml:82`
- `action.yaml:84`
- `action.yaml:87`

### github-env-injection (severity: high)

Two `run:` blocks write values derived from untrusted inputs to GitHub special environment files without sanitization. (1) Line 40: `echo "dir=${{ inputs.path }}/trivy-bin" >> $GITHUB_OUTPUT` — the user-controlled `inputs.path` is written directly to `$GITHUB_OUTPUT`; a newline embedded in the value could inject additional key=value pairs into the output. (2) Line 87: `echo ${{ steps.binary-dir.outputs.dir }} >> $GITHUB_PATH` — the step output (derived from `inputs.path`) is written directly to `$GITHUB_PATH` without sanitization. Both writes must be preceded by `safe=$(printf '%s' "$VAR" | tr -d '\n\r')` before the `echo ... >>` write.

Locations:

- `action.yaml:40`
- `action.yaml:87`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.path }}" appears directly in run: block of step "Binary dir"; move to env: map

Locations:

- `action.yml:44`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.version }}" appears directly in run: block of step "Install Trivy"; move to env: map

Locations:

- `action.yml:85`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, github-env-injection, static-inline-injection

**Notes:**

Fixed all script-injection and github-env-injection findings in hardened/action/action.yaml:
1. Binary dir step: Moved `inputs.path` to `env: INPUT_PATH`, sanitized with `printf '%s' | tr -d '\n\r'` before writing to `$GITHUB_OUTPUT`.
2. Install Trivy step: Moved `steps.binary-dir.outputs.dir` and `inputs.version` to `env: BINARY_DIR` and `env: INPUT_VERSION`, referenced as double-quoted shell variables `"$BINARY_DIR"` and `"$INPUT_VERSION"`.
3. Add Trivy binary step: Moved `steps.binary-dir.outputs.dir` to `env: BINARY_DIR`, sanitized with `printf '%s' | tr -d '\n\r'` before writing to `$GITHUB_PATH`.
Note: The findings reference both action.yaml and action.yml but only action.yaml exists; all fixes were applied there.

