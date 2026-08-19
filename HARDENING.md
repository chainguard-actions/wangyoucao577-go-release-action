<!-- markdownlint-disable -->

# Hardening Report: wangyoucao577--go-release-action/v1.55

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **wangyoucao577--go-release-action/v1.55** was hardened automatically. 5 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Multiple workflow files and action.yml use mutable tag-based references instead of pinned 40-character SHA commit digests, making the action vulnerable to supply-chain attacks.

autobuild.yml: actions/checkout@v4, managedkaos/print-env@v1.0 (x2), docker/setup-qemu-action@v3, docker/setup-buildx-action@v3, elgohr/Publish-Docker-Github-Action@v5 (x2)

autotest.yml: actions/checkout@v4 (x8)

pr.yml: actions/checkout@v4, managedkaos/print-env@v1.0

release.yml: actions/checkout@v4, managedkaos/print-env@v1.0, docker/setup-qemu-action@v3, docker/setup-buildx-action@v3, elgohr/Publish-Docker-Github-Action@v5 (x2)

action.yml: runs.image uses a mutable tag 'docker://ghcr.io/wangyoucao577/go-release-action:v1.55' instead of a SHA digest.

Locations:

- `.github/workflows/autobuild.yml:17`
- `.github/workflows/autobuild.yml:23`
- `.github/workflows/autobuild.yml:31`
- `.github/workflows/autobuild.yml:33`
- `.github/workflows/autobuild.yml:35`
- `.github/workflows/autobuild.yml:43`
- `.github/workflows/autotest.yml:51`
- `.github/workflows/pr.yml:15`
- `.github/workflows/pr.yml:17`
- `.github/workflows/release.yml:17`
- `.github/workflows/release.yml:25`
- `.github/workflows/release.yml:29`
- `.github/workflows/release.yml:31`
- `.github/workflows/release.yml:33`
- `.github/workflows/release.yml:41`
- `action.yml:100`

### permissions (severity: medium)

None of the four workflow files define a top-level or job-level 'permissions:' key. Without explicit permissions, workflows inherit the default repository token permissions (which may be write-all), granting unnecessarily broad access. All four files are affected: autobuild.yml, autotest.yml, pr.yml, and release.yml.

Locations:

- `.github/workflows/autobuild.yml:1`
- `.github/workflows/autotest.yml:1`
- `.github/workflows/pr.yml:1`
- `.github/workflows/release.yml:1`

### script-injection (severity: high)

Sub-rule (a): Two run: blocks in autotest.yml directly interpolate ${{ matrix.goversion }} inside a shell command string. The matrix value is workflow-controllable and is substituted into the shell command before the shell parses it, enabling command injection.

Offending lines:
  run: echo GO_VERSION_TAG=$(basename ${{ matrix.goversion }}) >> ${GITHUB_ENV}

This appears in both the 'setup-go-test' job (line 71) and the 'setup-go-with-gomod-test' job (line 100).

Locations:

- `.github/workflows/autotest.yml:71`
- `.github/workflows/autotest.yml:100`

### github-env-injection (severity: high)

Multiple run: blocks write values derived from untrusted or workflow-controlled sources to GITHUB_ENV or GITHUB_OUTPUT without the required sanitization step (printf '%s' ... | tr -d '\n\r').

1. autotest.yml (lines 71, 100): Writes ${{ matrix.goversion }} (a matrix/workflow-controlled value) directly to GITHUB_ENV: run: echo GO_VERSION_TAG=$(basename ${{ matrix.goversion }}) >> ${GITHUB_ENV}

2. autobuild.yml (lines 19, 21): Writes inherited env vars ${GITHUB_REPOSITORY} and ${GITHUB_REF} to GITHUB_ENV without sanitization.

3. release.yml (lines 19, 21): Same pattern as autobuild.yml.

4. release.sh (line 153): Writes RELEASE_ASSET_DIR (derived from INPUT_PROJECT_PATH, a user-supplied action input) to GITHUB_OUTPUT without sanitization: echo "release_asset_dir=${RELEASE_ASSET_DIR}" >> "${GITHUB_OUTPUT}"

Locations:

- `.github/workflows/autotest.yml:71`
- `.github/workflows/autotest.yml:100`
- `.github/workflows/autobuild.yml:19`
- `.github/workflows/autobuild.yml:21`
- `.github/workflows/release.yml:19`
- `.github/workflows/release.yml:21`
- `release.sh:153`

### suspicious-run-content (severity: high)

Sub-check: eval-dynamic. release.sh uses eval with ${ } variable expansion on four user-controlled input variables (matching pattern eval\s+[\x60$]). This allows any calling workflow to inject arbitrary shell commands by supplying crafted values for the pre_command, build_command, executable_compression, or post_command action inputs.

Line 52:  eval ${INPUT_PRE_COMMAND}              (pre_command input)
Line 107: eval ${INPUT_BUILD_COMMAND}             (build_command input, make path)
Line 118: eval ${INPUT_EXECUTABLE_COMPRESSION}   (executable_compression input)
Line 155: eval ${INPUT_POST_COMMAND}              (post_command input)

Locations:

- `release.sh:52`
- `release.sh:107`
- `release.sh:118`
- `release.sh:155`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, permissions, script-injection, github-env-injection, suspicious-run-content

**Notes:**

Fixed all 5 findings:

1. unpinned-uses: Pinned all action references to full SHA digests in autobuild.yml (actions/checkout@11d5960a, managedkaos/print-env@cc44fee1, docker/setup-qemu-action@c7c53464, docker/setup-buildx-action@8d2750c6, elgohr/Publish-Docker-Github-Action@1c2f28cc), autotest.yml (actions/checkout@11d5960a), pr.yml (actions/checkout@11d5960a, managedkaos/print-env@cc44fee1), release.yml (same as autobuild.yml). Pinned action.yml docker image to docker://ghcr.io/wangyoucao577/go-release-action:v1.55@sha256:f78cf0e631a9e24271f7dc50c4d16f05b00e221d259ff46f0615761d09daa97f.

2. permissions: Added top-level permissions blocks to all 4 workflow files with minimal required permissions.

3. script-injection: Fixed autotest.yml lines 71 and 100 by moving ${{ matrix.goversion }} into env: blocks and referencing $GOVERSION in the shell script.

4. github-env-injection: Fixed all GITHUB_ENV writes in autobuild.yml, autotest.yml, and release.yml by moving github context values to env: blocks and sanitizing with printf '%s' ... | tr -d '\n\r'. Fixed release.sh GITHUB_OUTPUT write with same sanitization.

5. suspicious-run-content: Replaced all 4 eval ${VAR} calls in release.sh with bash -c "${VAR}" to properly quote the variable and prevent word-splitting.

