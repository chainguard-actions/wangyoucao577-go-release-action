<!-- markdownlint-disable -->

# Hardening Report: wangyoucao577--go-release-action/v1.52

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **wangyoucao577--go-release-action/v1.52** was hardened automatically. 1 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

The action.yml uses a Docker image reference with a mutable tag instead of an immutable SHA digest. The image `docker://ghcr.io/wangyoucao577/go-release-action:v1.52` uses the tag `v1.52`, which can be silently replaced with different (potentially malicious) content. It should be pinned to a specific SHA256 digest, e.g. `docker://ghcr.io/wangyoucao577/go-release-action@sha256:<64-hex-char-digest> # v1.52`.

Locations:

- `action.yml:96`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses

**Notes:**

Replaced the mutable Docker image tag `docker://ghcr.io/wangyoucao577/go-release-action:v1.52` with the immutable SHA256 digest `docker://ghcr.io/wangyoucao577/go-release-action@sha256:b484bac808dbd9566a37c12ec3238aa46e16f2a770e27624621e9381044ba091` in action.yml line 96. The original tag `v1.52` is preserved as a comment for readability.

