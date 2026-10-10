<!-- markdownlint-disable -->

# Hardening Report: aquasecurity--setup-trivy/v0.3.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **aquasecurity--setup-trivy/v0.3.1** was hardened automatically. 1 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### github-env-injection (severity: high)

The 'Add Trivy binary to $GITHUB_PATH' step writes the value of `${BINARY_DIR}` (sourced from `steps.binary-dir.outputs.dir`, which is derived from the user-controlled `inputs.path`) to `$GITHUB_PATH` without the required sanitization step (`printf '%s' ... | tr -d '\n\r'`) immediately before the write. The write is: `echo "${BINARY_DIR}" >> "$GITHUB_PATH"`. Although the 'Binary dir' step validates the input for newlines, the check requires sanitization to be applied immediately before every write to a special environment file when the source is not a literal computed in the same run block. A safe pattern would be: `safe=$(printf '%s' "${BINARY_DIR}" | tr -d '\n\r'); echo "${safe}" >> "$GITHUB_PATH"`.

Locations:

- `action.yaml:113`

## Iteration Notes

### Iteration 1

**Fixes applied:** github-env-injection

**Notes:**

Fixed the 'Add Trivy binary to $GITHUB_PATH' step in action.yaml (line 113) by adding sanitization immediately before writing to $GITHUB_PATH. The fix uses `safe=$(printf '%s' "${BINARY_DIR}" | tr -d '\n\r')` to strip newline and carriage return characters from the BINARY_DIR value (which is derived from user-controlled inputs.path via steps.binary-dir.outputs.dir) before writing it to the special environment file.

### Iteration 2

**Fixes applied:** github-env-injection

**Notes:**

Fixed the 'Binary dir' step in action.yaml (line 63) by adding the required sanitization pipeline before writing to $GITHUB_OUTPUT. The INPUT_PATH value (with its default fallback) is now passed through `printf '%s' ... | tr -d '\n\r'` to strip newline/carriage-return characters before being written to $GITHUB_OUTPUT. The existing `case` block that rejects newlines and shell-special characters is preserved as an early validation step, and the sanitization pipeline provides the additional defense-in-depth required by the finding.

