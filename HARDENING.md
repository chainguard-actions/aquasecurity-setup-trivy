<!-- markdownlint-disable -->

# Hardening Report: aquasecurity--setup-trivy/v0.2.6

> This file was generated automatically by the hardening agent.

**Policy SHA:** `374911c4be584df24e6da57af2d990570ef0f859`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **aquasecurity--setup-trivy/v0.2.6** was hardened automatically. 8 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### github-env-injection (severity: high)

Step 'Binary dir' writes the untrusted input inputs.path straight to $GITHUB_OUTPUT with no newline stripping: `echo "dir=${{ inputs.path }}/trivy-bin" >> $GITHUB_OUTPUT`. A value containing a newline could add extra output entries.

Locations:

- `action.yaml:43`

### github-env-injection (severity: high)

Step 'Add Trivy binary to $GITHUB_PATH' writes steps.binary-dir.outputs.dir, which comes from inputs.path, to $GITHUB_PATH with no `printf '%s' ... | tr -d '\n\r'` step: `echo ${{ steps.binary-dir.outputs.dir }} >> $GITHUB_PATH`. Every line of the value becomes a PATH entry.

Locations:

- `action.yaml:92`

### script-injection (severity: high)

Sub-rule (a): step 'Binary dir' puts an expression directly inside the run: script: `echo "dir=${{ inputs.path }}/trivy-bin" >> $GITHUB_OUTPUT`.

Locations:

- `action.yaml:43`

### script-injection (severity: high)

Sub-rule (a): step 'Install Trivy' puts expressions directly inside the run: script: `bash ./trivy/contrib/install.sh -b ${{ steps.binary-dir.outputs.dir }} -c setup-trivy ${{ inputs.version }}` and `cp -r ./trivy/contrib ${{ steps.binary-dir.outputs.dir }}/contrib`.

Locations:

- `action.yaml:85`
- `action.yaml:86`

### script-injection (severity: high)

Sub-rule (a): step 'Add Trivy binary to $GITHUB_PATH' puts an expression directly inside the run: script: `echo ${{ steps.binary-dir.outputs.dir }} >> $GITHUB_PATH`.

Locations:

- `action.yaml:92`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.path }}" appears directly in run: block of step "Binary dir"; move to env: map

Locations:

- `action.yml:44`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.version }}" appears directly in run: block of step "Install Trivy"; move to env: map

Locations:

- `action.yml:85`

### prior-fix (severity: high)

Carried forward from this version's previous harden, which fixed it; fix it again unless the code no longer has the problem. The previous harden changed action.yaml without recording a finding for it, and this analysis reported nothing there. Re-apply its security changes unless the code no longer needs them. The previous patch:
```diff
diff --git a/action.yaml b/action.yaml
index 83fbeca..1dab8ca 100644
--- a/action.yaml
+++ b/action.yaml
@@ -1,7 +1,6 @@
 name: 'Aqua Security Trivy installer'
 description: 'Install Trivy binary from release page'
 author: 'Aqua Security'
-
 inputs:
   version:
     description: 'Trivy version to install'
@@ -17,11 +16,8 @@ inputs:
     default: 'false'
   token:
     description: >
-      Access token used to check out the Trivy repository.
-      The token is required when using GitHub Enterprise Server (GHES).
-      https://github.com/actions/create-github-app-token can be used to obtain such a token.
-      The token should be limited to read access only for public repositories.
-      See more details in https://github.com/aquasecurity/setup-trivy/issues/10
+      Access token used to check out the Trivy repository. The token is required when using GitHub Enterprise Server (GHES). https://github.com/actions/create-github-app-token can be used to obtain such a token. The token should be limited to read access only for public repositories. See more details in https://github.com/aquasecurity/setup-trivy/issues/10
+
     required: false
     ## ${{ github.token }} is default value for actions/checkout
     ## cf. https://github.com/actions/checkout/blob/eef61447b9ff4aafe5dcd4e0bbf5d482be7e7871/action.yml#L24
@@ -33,15 +29,34 @@ inputs:
     ## but it can optionally be set to '${{ github.server_url }}' on GHES if the repository has been mirrored (e.g., via actions/actions-sync).
     ## Ref. https://github.com/aquasecurity/setup-trivy/issues/10 and https://github.com/aquasecurity/setup-trivy/pull/16
     default: 'https://github.com'
-
 runs:
   using: 'composite'
   steps:
     - name: Binary dir
       id: binary-dir
       shell: bash
-      run: echo "dir=${{ inputs.path }}/trivy-bin" >> $GITHUB_OUTPUT
-
+      env:
+        INPUT_PATH: ${{ inputs.path }}
+      run: |
+        safe=$(printf '%s' "$INPUT_PATH" | tr -d '\n\r')
+        echo "dir=${safe}/trivy-bin" >> "$GITHUB_OUTPUT"
     ## Don't cache `latest` version
     - name: Check the version for caching
       if: ${{ inputs.cache == 'true' && inputs.version == 'latest' }}
@@ -49,21 +64,19 @@ runs:
       run: |
         echo "'setup-trivy' doesn't currently support caching the 'latest' version"
         echo "read https://github.com/aquasecurity/setup-trivy?tab=readme-ov-file#caching for more details"
-
     - name: Restore Trivy binary from cache
       if: ${{ inputs.cache == 'true' && inputs.version != 'latest' }}
       id: cache
       uses: actions/cache/restore@9255dc7a253b0ccc959486e2bca901246202afeb # v5.0.1
       with:
         path: ${{ steps.binary-dir.outputs.dir }}
         key: trivy-binary-${{ inputs.version }}-${{ runner.os }}-${{ runner.arch }}
-
     ## Security: Pin to a specific commit hash (not tag) to prevent supply chain attacks.
     ## The install script is stable and can install different Trivy versions.
     ## Update this ref as needed after reviewing changes to contrib/install.sh
     - name: Checkout install script
       if: steps.cache.outputs.cache-hit != 'true'
       uses: actions/checkout@8e8c483db84b4bee98b60c0593521ed34d9990e8 # v6.0.1
       with:
         repository: aquasecurity/trivy
         sparse-checkout: |
@@ -73,31 +86,35 @@ runs:
         ref: 75c4dc0f45c5d7ffd05ae26df1e0c666787bdf2a # main (with -c flag support)
         github-server-url: ${{ inputs.github-server-url }}
         token: ${{ inputs.token }}
-
-      ## Install Trivy using install script,
-      ## Copy the `contrib` directory to the directory with the binary
-      ## Remove the `trivy` directory produced by the checkout step, as it may cause errors in linters/checks in the calling code.
+        ## Install Trivy using install script,
+        ## Copy the `contrib` directory to the directory with the binary
+        ## Remove the `trivy` directory produced by the checkout step, as it may cause errors in linters/checks in the calling code.
     - name: Install Trivy
       if: steps.cache.outputs.cache-hit != 'true'
       shell: bash
+    
... (truncated)
```

Locations:

- `action.yaml`

## Iteration Notes

### Iteration 1

**Fixes applied:** github-env-injection, script-injection, static-inline-injection, prior-fix

**Notes:**

Fixed all findings in action.yaml. I haven't run the action to test it.

- **'Binary dir' step:** `inputs.path` now comes in through the env variable `INPUT_PATH`. Newlines are stripped before the value is written to `$GITHUB_OUTPUT`, and the file name is quoted. The default for this input is the literal text '$HOME/.local/bin', and upstream relied on the shell to expand it. An env variable doesn't get that expansion, so the script now replaces `$HOME`, `${HOME}` and a leading `~` itself. Without that, the path would contain the literal text '$HOME'.
- **'Install Trivy' step:** the binary dir and version now come in through `BINARY_DIR` and `TRIVY_VERSION` and are double-quoted. The version uses `${VAR:+"$VAR"}`, so an empty version is left out of the arguments, as it was upstream.
- **'Add Trivy binary to $GITHUB_PATH' step:** the dir comes in through env, has newlines stripped, and is written to `"$GITHUB_PATH"`.

The previous harden's changes (moving these values into env and sanitizing them) are re-applied by the edits above. Its other changes only reformatted the file, so I left those alone.

### Iteration 2

**Fixes applied:** script-injection, github-env-injection

**Notes:**

In action.yaml, the step 'Add Trivy binary to $GITHUB_PATH' now gets steps.binary-dir.outputs.dir through env (BINARY_DIR). It strips newlines with printf | tr -d '\n\r' right before writing the quoted value to "$GITHUB_PATH".

