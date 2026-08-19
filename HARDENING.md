<!-- markdownlint-disable -->

# Hardening Report: aquasecurity--setup-trivy/v0.2.6

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **aquasecurity--setup-trivy/v0.2.6** was hardened automatically. 4 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): Three run: blocks in action.yaml directly interpolate ${{ }} expressions into shell command strings, enabling script injection.

1. Line 43 — `run: echo "dir=${{ inputs.path }}/trivy-bin" >> $GITHUB_OUTPUT`: the user-controlled `inputs.path` is interpolated directly into the shell command.

2. Line 81 — `bash ./trivy/contrib/install.sh -b ${{ steps.binary-dir.outputs.dir }} -c setup-trivy ${{ inputs.version }}`: both `steps.binary-dir.outputs.dir` (derived from `inputs.path`) and `inputs.version` are interpolated directly as unquoted shell arguments, allowing an attacker to inject arbitrary shell metacharacters.

3. Line 88 — `run: echo ${{ steps.binary-dir.outputs.dir }} >> $GITHUB_PATH`: the step output (derived from `inputs.path`) is interpolated directly and unquoted into the shell command.

All three should route the values through env: variables and double-quote the expansions in the shell script.

Locations:

- `action.yaml:43`
- `action.yaml:81`
- `action.yaml:88`

### github-env-injection (severity: high)

Two run: blocks write values derived from untrusted inputs to GitHub special environment files without the required sanitization step (`printf '%s' ... | tr -d '\n\r'`).

1. Line 43 — `echo "dir=${{ inputs.path }}/trivy-bin" >> $GITHUB_OUTPUT`: the user-controlled `inputs.path` is written directly to $GITHUB_OUTPUT. A newline character in the input could inject additional key=value pairs into the output file.

2. Line 88 — `echo ${{ steps.binary-dir.outputs.dir }} >> $GITHUB_PATH`: the step output (which is derived from `inputs.path`) is written directly to $GITHUB_PATH without sanitization, allowing newline injection that could add arbitrary entries to the runner's PATH.

Both writes must be preceded by sanitization: `safe=$(printf '%s' "$VAR" | tr -d '\n\r')` before writing to the special file.

Locations:

- `action.yaml:43`
- `action.yaml:88`

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

Fixed all three run: blocks in action.yaml:
1. 'Binary dir' step: moved inputs.path into INPUT_PATH env var, added sanitization (printf '%s' "$INPUT_PATH" | tr -d '\n\r') before writing to $GITHUB_OUTPUT.
2. 'Install Trivy' step: moved steps.binary-dir.outputs.dir and inputs.version into BINARY_DIR and INPUT_VERSION env vars, double-quoted both in the shell script.
3. 'Add Trivy binary to $GITHUB_PATH' step: moved steps.binary-dir.outputs.dir into BINARY_DIR env var, added sanitization before writing to $GITHUB_PATH.

