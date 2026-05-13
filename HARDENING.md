# Hardening Report: aquasecurity--setup-trivy/v0.2.6

> This file was generated automatically by the hardening agent.

**Policy SHA:** `ff50f15e4b79bfbf764dafdfd2579175a6ea9771`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **aquasecurity--setup-trivy/v0.2.6** was hardened automatically. 4 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Three run: blocks in action.yaml directly interpolate attacker-controlled expressions without first assigning them to environment variables:
1. 'Binary dir' step: `run: echo "dir=${{ inputs.path }}/trivy-bin" >> $GITHUB_OUTPUT` — inputs.path is interpolated directly in the shell command.
2. 'Install Trivy' step: `bash ./trivy/contrib/install.sh -b ${{ steps.binary-dir.outputs.dir }} -c setup-trivy ${{ inputs.version }}` — both inputs.version and the step output (derived from inputs.path) are interpolated directly.
3. 'Add Trivy binary to $GITHUB_PATH' step: `run: echo ${{ steps.binary-dir.outputs.dir }} >> $GITHUB_PATH` — step output derived from inputs.path is interpolated directly.
An attacker can supply a malicious value for inputs.path or inputs.version to inject arbitrary shell commands.

Locations:

- `action.yaml:43`
- `action.yaml:75`
- `action.yaml:81`

### github-env-injection (severity: high)

Two run: blocks write attacker-controlled values derived from inputs.* to special GitHub environment files without the required sanitization step (printf '%s' ... | tr -d '\n\r'):
1. 'Binary dir' step (line 43): `echo "dir=${{ inputs.path }}/trivy-bin" >> $GITHUB_OUTPUT` — inputs.path is written directly to $GITHUB_OUTPUT. A newline in inputs.path can inject arbitrary key=value pairs into the output context.
2. 'Add Trivy binary to $GITHUB_PATH' step (line 81): `echo ${{ steps.binary-dir.outputs.dir }} >> $GITHUB_PATH` — the step output (set from inputs.path in step 1) is written directly to $GITHUB_PATH. A newline in inputs.path can inject arbitrary paths.

Locations:

- `action.yaml:43`
- `action.yaml:81`

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
1. 'Binary dir' step: moved inputs.path to INPUT_PATH env var; sanitized with `printf '%s' "$INPUT_PATH" | tr -d '\n\r'` before writing to $GITHUB_OUTPUT.
2. 'Install Trivy' step: moved steps.binary-dir.outputs.dir to BINARY_DIR and inputs.version to INPUT_VERSION env vars; referenced as plain shell variables in the script.
3. 'Add Trivy binary to $GITHUB_PATH' step: moved steps.binary-dir.outputs.dir to BINARY_DIR env var; sanitized with `printf '%s' "$BINARY_DIR" | tr -d '\n\r'` before writing to $GITHUB_PATH.
No ${{ }} expressions remain in any run: block.

