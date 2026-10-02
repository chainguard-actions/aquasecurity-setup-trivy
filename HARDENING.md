<!-- markdownlint-disable -->

# Hardening Report: aquasecurity--setup-trivy/v0.2.6

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **aquasecurity--setup-trivy/v0.2.6** was hardened automatically. 7 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): The 'Binary dir' step directly interpolates `${{ inputs.path }}` inside a `run:` shell command: `echo "dir=${{ inputs.path }}/trivy-bin" >> $GITHUB_OUTPUT`. The expression is substituted by the Actions runner before the shell ever sees it, allowing an attacker-controlled value to break out of the string and inject arbitrary shell commands.

Locations:

- `action.yaml:40`

### script-injection (severity: high)

Sub-rule (a): The 'Install Trivy' step directly interpolates `${{ steps.binary-dir.outputs.dir }}` and `${{ inputs.version }}` inside `run:` shell commands: `bash ./trivy/contrib/install.sh -b ${{ steps.binary-dir.outputs.dir }} -c setup-trivy ${{ inputs.version }}` and `cp -r ./trivy/contrib ${{ steps.binary-dir.outputs.dir }}/contrib`. Both expressions are substituted before the shell parses the command, enabling shell metacharacter injection via attacker-controlled inputs.

Locations:

- `action.yaml:81`
- `action.yaml:82`

### script-injection (severity: high)

Sub-rule (a) and (b): The 'Add Trivy binary to $GITHUB_PATH' step directly interpolates `${{ steps.binary-dir.outputs.dir }}` inside a `run:` shell command AND the value is unquoted: `echo ${{ steps.binary-dir.outputs.dir }} >> $GITHUB_PATH`. This violates both sub-rule (a) (direct expression interpolation) and sub-rule (b) (unquoted shell expansion of untrusted data).

Locations:

- `action.yaml:87`

### github-env-injection (severity: high)

The 'Binary dir' step writes the untrusted input `inputs.path` directly to `$GITHUB_OUTPUT` without sanitization: `echo "dir=${{ inputs.path }}/trivy-bin" >> $GITHUB_OUTPUT`. An attacker-controlled value containing newlines could inject additional key=value pairs into the output, poisoning subsequent steps that read `steps.binary-dir.outputs.*`. The required sanitization (`printf '%s' "$VAR" | tr -d '\n\r'`) is absent.

Locations:

- `action.yaml:40`

### github-env-injection (severity: high)

The 'Add Trivy binary to $GITHUB_PATH' step writes `steps.binary-dir.outputs.dir` (which derives from the untrusted `inputs.path`) directly to `$GITHUB_PATH` without sanitization: `echo ${{ steps.binary-dir.outputs.dir }} >> $GITHUB_PATH`. A newline-containing value could inject additional entries into PATH, enabling path-hijacking attacks. The required sanitization (`printf '%s' "$VAR" | tr -d '\n\r'`) is absent.

Locations:

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

Fixed all findings in hardened/action/action.yaml:
1. Binary dir step: Moved `${{ inputs.path }}` to `INPUT_PATH` env var; sanitized with `printf '%s' "$INPUT_PATH" | tr -d '\n\r'` before writing to `$GITHUB_OUTPUT`.
2. Install Trivy step: Moved `${{ steps.binary-dir.outputs.dir }}` to `BINARY_DIR` and `${{ inputs.version }}` to `INPUT_VERSION` env vars; both properly double-quoted in shell commands.
3. Add Trivy binary to $GITHUB_PATH step: Moved `${{ steps.binary-dir.outputs.dir }}` to `BINARY_DIR` env var; sanitized with `printf '%s' "$BINARY_DIR" | tr -d '\n\r'` before writing to `$GITHUB_PATH`.
Note: The findings reference both `action.yaml` and `action.yml` but only `action.yaml` exists; all fixes were applied there.

