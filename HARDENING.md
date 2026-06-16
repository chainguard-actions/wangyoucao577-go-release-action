<!-- markdownlint-disable -->

# Hardening Report: wangyoucao577--go-release-action/v1.55

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **wangyoucao577--go-release-action/v1.55** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

The action.yml uses a Docker image referenced by a mutable tag (v1.55) instead of an immutable SHA digest. This means the image could be silently replaced with a malicious version. The failing reference is: image: 'docker://ghcr.io/wangyoucao577/go-release-action:v1.55'. It should be pinned to a SHA digest, e.g. image: 'docker://ghcr.io/wangyoucao577/go-release-action@sha256:<64-hex-digest>'.

Locations:

- `action.yml:86`

### suspicious-run-content (severity: high)

release.sh passes user-controlled action inputs directly to eval, matching the eval-dynamic pattern (eval ${VAR}). This allows any caller of the action to execute arbitrary shell commands on the runner: (1) eval ${INPUT_PRE_COMMAND} — the pre_command input is eval'd without sanitization; (2) eval ${INPUT_BUILD_COMMAND} — the build_command input is eval'd when it starts with 'make'; (3) eval ${INPUT_EXECUTABLE_COMPRESSION} ${BUILD_ARTIFACTS_FOLDER}/... — the executable_compression input is eval'd; (4) eval ${INPUT_POST_COMMAND} — the post_command input is eval'd without sanitization. An attacker who controls these inputs can inject arbitrary shell commands.

Locations:

- `release.sh:54`
- `release.sh:130`
- `release.sh:144`
- `release.sh:188`

### github-env-injection (severity: high)

release.sh writes RELEASE_ASSET_DIR to $GITHUB_OUTPUT without newline sanitization: echo "release_asset_dir=${RELEASE_ASSET_DIR}" >> "${GITHUB_OUTPUT}". RELEASE_ASSET_DIR is derived from INPUT_PROJECT_PATH (a user-controlled action input) via RELEASE_ASSET_DIR=${INPUT_PROJECT_PATH}/${BUILD_ARTIFACTS_FOLDER}. A value containing newlines could inject additional key=value pairs into GITHUB_OUTPUT, poisoning the output context for downstream steps. The value should be sanitized with printf '%s' "$RELEASE_ASSET_DIR" | tr -d '\n\r' before writing.

Locations:

- `release.sh:183`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, suspicious-run-content, github-env-injection

**Notes:**

Fixed three security findings: (1) Pinned the Docker image in action.yml from mutable tag v1.55 to immutable SHA256 digest sha256:f78cf0e631a9e24271f7dc50c4d16f05b00e221d259ff46f0615761d09daa97f with the tag preserved as a comment. (2) Replaced all four eval usages in release.sh with bash -c to prevent arbitrary shell command injection via user-controlled inputs (INPUT_PRE_COMMAND, INPUT_BUILD_COMMAND, INPUT_EXECUTABLE_COMPRESSION, INPUT_POST_COMMAND). For the executable compression case, printf '%q' is used to safely quote the binary path argument. (3) Sanitized RELEASE_ASSET_DIR with printf '%s' | tr -d '\n\r' before writing to $GITHUB_OUTPUT to prevent newline injection attacks.

