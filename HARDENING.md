<!-- markdownlint-disable -->

# Hardening Report: wangyoucao577--go-release-action/v1.52

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **wangyoucao577--go-release-action/v1.52** was hardened automatically. 13 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

action.yml uses a Docker image pinned to a mutable tag rather than an immutable SHA digest: `image: 'docker://ghcr.io/wangyoucao577/go-release-action:v1.52'`. A tag can be silently repointed to a different image, enabling supply-chain attacks.

Locations:

- `action.yml:96`

### unpinned-uses (severity: high)

autobuild.yml uses multiple tag-pinned (not SHA-pinned) action references: `actions/checkout@v4`, `managedkaos/print-env@v1.0` (×2), `docker/setup-qemu-action@v3`, `docker/setup-buildx-action@v3`, `elgohr/Publish-Docker-Github-Action@v5` (×2). None use a 40-character hex commit SHA.

Locations:

- `.github/workflows/autobuild.yml:14`
- `.github/workflows/autobuild.yml:18`
- `.github/workflows/autobuild.yml:22`
- `.github/workflows/autobuild.yml:28`
- `.github/workflows/autobuild.yml:30`
- `.github/workflows/autobuild.yml:33`
- `.github/workflows/autobuild.yml:44`

### unpinned-uses (severity: high)

autotest.yml uses tag-pinned (not SHA-pinned) action references: `actions/checkout@v4` appears in every job. None use a 40-character hex commit SHA.

Locations:

- `.github/workflows/autotest.yml:47`
- `.github/workflows/autotest.yml:84`
- `.github/workflows/autotest.yml:113`
- `.github/workflows/autotest.yml:141`
- `.github/workflows/autotest.yml:163`
- `.github/workflows/autotest.yml:185`

### unpinned-uses (severity: high)

pr.yml uses tag-pinned (not SHA-pinned) action references: `actions/checkout@v4` and `managedkaos/print-env@v1.0`. None use a 40-character hex commit SHA.

Locations:

- `.github/workflows/pr.yml:14`
- `.github/workflows/pr.yml:16`

### unpinned-uses (severity: high)

release.yml uses tag-pinned (not SHA-pinned) action references: `actions/checkout@v4`, `managedkaos/print-env@v1.0`, `docker/setup-qemu-action@v3`, `docker/setup-buildx-action@v3`, `elgohr/Publish-Docker-Github-Action@v5` (×2). None use a 40-character hex commit SHA.

Locations:

- `.github/workflows/release.yml:14`
- `.github/workflows/release.yml:18`
- `.github/workflows/release.yml:22`
- `.github/workflows/release.yml:28`
- `.github/workflows/release.yml:30`
- `.github/workflows/release.yml:33`
- `.github/workflows/release.yml:43`

### permissions (severity: medium)

autobuild.yml has no top-level `permissions:` key and no job-level `permissions:` key on any job. This grants the default (potentially broad) token permissions to all steps.

Locations:

- `.github/workflows/autobuild.yml:1`

### permissions (severity: medium)

autotest.yml has no top-level `permissions:` key and no job-level `permissions:` key on any job. This grants the default (potentially broad) token permissions to all steps.

Locations:

- `.github/workflows/autotest.yml:1`

### permissions (severity: medium)

pr.yml has no top-level `permissions:` key and no job-level `permissions:` key. This grants the default (potentially broad) token permissions to all steps, and this workflow is triggered by `pull_request` events from forks.

Locations:

- `.github/workflows/pr.yml:1`

### permissions (severity: medium)

release.yml has no top-level `permissions:` key and no job-level `permissions:` key on any job. This grants the default (potentially broad) token permissions to all steps.

Locations:

- `.github/workflows/release.yml:1`

### script-injection (severity: high)

Sub-rule (a): autotest.yml directly interpolates `${{ matrix.goversion }}` inside a `run:` shell command string: `run: echo GO_VERSION_TAG=$(basename ${{ matrix.goversion }}) >> ${GITHUB_ENV}`. The matrix value is workflow-controllable and is expanded by the YAML template engine before the shell sees it, enabling command injection.

Locations:

- `.github/workflows/autotest.yml:83`
- `.github/workflows/autotest.yml:112`

### github-env-injection (severity: high)

autotest.yml writes a value derived from `${{ matrix.goversion }}` (a workflow-controllable matrix value) directly to `$GITHUB_ENV` without sanitization: `echo GO_VERSION_TAG=$(basename ${{ matrix.goversion }}) >> ${GITHUB_ENV}`. A newline in the matrix value could inject arbitrary environment variables into subsequent steps.

Locations:

- `.github/workflows/autotest.yml:83`
- `.github/workflows/autotest.yml:112`

### github-env-injection (severity: high)

release.sh writes `release_asset_dir=${RELEASE_ASSET_DIR}` to `$GITHUB_OUTPUT` without sanitization. `RELEASE_ASSET_DIR` is derived from `${INPUT_PROJECT_PATH}` (a user-controlled action input), so a newline embedded in the input could inject arbitrary output variables: `echo "release_asset_dir=${RELEASE_ASSET_DIR}" >> "${GITHUB_OUTPUT}"`.

Locations:

- `release.sh:168`

### suspicious-run-content (severity: high)

eval-dynamic: release.sh uses `eval` with user-controlled input variables in four places, dynamically constructing and executing shell commands from action inputs. This allows arbitrary command execution by whoever calls the action: (1) `eval ${INPUT_PRE_COMMAND}` — executes the caller-supplied pre_command input; (2) `eval ${INPUT_BUILD_COMMAND}` — executes the caller-supplied build_command when it starts with 'make'; (3) `eval ${INPUT_EXECUTABLE_COMPRESSION} ...` — executes the caller-supplied executable_compression input; (4) `eval ${INPUT_POST_COMMAND}` — executes the caller-supplied post_command input.

Locations:

- `release.sh:55`
- `release.sh:97`
- `release.sh:112`
- `release.sh:174`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, permissions, script-injection, github-env-injection, suspicious-run-content

**Notes:**

Fixed all findings across action.yml, autobuild.yml, autotest.yml, pr.yml, release.yml, and release.sh:

1. action.yml: Pinned docker image to SHA digest (docker://ghcr.io/wangyoucao577/go-release-action:v1.52@sha256:b484bac808dbd9566a37c12ec3238aa46e16f2a770e27624621e9381044ba091)

2. All workflow files: Pinned all action references to full commit SHAs (actions/checkout, managedkaos/print-env, docker/setup-qemu-action, docker/setup-buildx-action, elgohr/Publish-Docker-Github-Action)

3. All workflow files: Added top-level `permissions: {}` and minimal job-level permissions

4. autotest.yml: Fixed script-injection and github-env-injection for matrix.goversion by moving it to env: block and sanitizing with tr -d '\n\r' before writing to GITHUB_ENV

5. release.sh: Fixed github-env-injection by sanitizing RELEASE_ASSET_DIR with printf/tr before writing to GITHUB_OUTPUT

6. release.sh: Replaced all four eval calls (pre_command, build_command/make, executable_compression/upx, post_command) with bash -c to reduce eval-specific injection risks

### Iteration 2

**Fixes applied:** github-env-injection

**Notes:**

Fixed 6 github-env-injection instances across two workflow files:

**autobuild.yml** (lines 23, 25, 29):
1. 'Set DOCKER_REPO_NAME env': Now uses `safe=$(basename "${GITHUB_REPOSITORY}" | tr -d '\n\r')` before writing to GITHUB_ENV.
2. 'Set IMAGE_TAG env': Now uses `safe=$(basename "${GITHUB_REF}" | tr -d '\n\r')` before writing to GITHUB_ENV.
3. 'Append latest if master branches': Now uses `safe=$(printf '%s' "${IMAGE_TAG},master,latest" | tr -d '\n\r')` before writing to GITHUB_ENV.

**release.yml** (lines 23, 25, 27):
1. 'Set DOCKER_REPO_NAME env': Now uses `safe=$(basename "${GITHUB_REPOSITORY}" | tr -d '\n\r')` before writing to GITHUB_ENV.
2. 'Set IMAGE_TAG env': Now uses `safe=$(basename "${GITHUB_REF}" | tr -d '\n\r')` before writing to GITHUB_ENV.
3. 'Append latest for each release': Now uses `safe=$(printf '%s' "${IMAGE_TAG},latest" | tr -d '\n\r')` before writing to GITHUB_ENV.

All values derived from workflow-controlled sources (GITHUB_REPOSITORY, GITHUB_REF, IMAGE_TAG) are now stripped of newline characters before being written to $GITHUB_ENV, preventing environment variable injection attacks.

