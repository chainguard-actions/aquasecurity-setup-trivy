<!-- markdownlint-disable -->

# Hardening Report: aquasecurity--setup-trivy/v0.3.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `374911c4be584df24e6da57af2d990570ef0f859`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **aquasecurity--setup-trivy/v0.3.1** was hardened automatically. 2 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### github-env-injection (severity: high)

Step 'Binary dir' writes the untrusted input inputs.path (via env INPUT_PATH) to $GITHUB_OUTPUT: `echo "dir=${INPUT_PATH:-$HOME/.local/bin}/trivy-bin" >> "$GITHUB_OUTPUT"`. The script does reject values containing newlines with a case-pattern check first, so the risk is mostly mitigated. However, it does not apply the required sanitization step (`printf '%s' "$INPUT_PATH" | tr -d '\n\r'`) immediately before the write, so it fails the check as written.

Locations:

- `action.yaml:68`

### github-env-injection (severity: high)

Step 'Add Trivy binary to $GITHUB_PATH' writes BINARY_DIR (set from steps.binary-dir.outputs.dir, which comes from inputs.path) to $GITHUB_PATH without sanitization: `run: echo "${BINARY_DIR}" >> "$GITHUB_PATH"`. The required `printf '%s' "$BINARY_DIR" | tr -d '\n\r'` step is missing. $GITHUB_PATH has no heredoc form, so the newline strip is always required here.

Locations:

- `action.yaml:122`

## Iteration Notes

### Iteration 1

**Fixes applied:** github-env-injection

**Notes:**

action.yaml: in the 'Binary dir' step, inputs.path now has newlines and carriage returns stripped (printf '%s' | tr -d '\n\r') right before the dir output is written to $GITHUB_OUTPUT. The existing checks that reject '$', '~' and newlines are still in place. In the 'Add Trivy binary to $GITHUB_PATH' step, BINARY_DIR gets the same strip before it is written to $GITHUB_PATH. Behaviour is unchanged for valid paths, and the default still falls back to $HOME/.local/bin. These edits haven't been run or re-scanned yet.

### Iteration 2

**Fixes applied:** github-env-injection

**Notes:**

In action.yaml step 'Add Trivy binary to $GITHUB_PATH', BINARY_DIR is now sanitized with printf '%s' | tr -d '\n\r' immediately before being appended to $GITHUB_PATH.

