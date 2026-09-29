# Shared Cua Fork Handoff

## Current Task

None active. Fork creation and the shared Windows installation are complete.
This is local tooling configuration, not a production application deployment.

## Next Up

- Open a fresh Codex session in any local project and verify a requested GUI
  workflow. MCP initialization and read-only desktop discovery passed; GUI input
  and fresh-session end-to-end acceptance have not been tested.
- `C:/Users/User/workspace/EMPIRE.md` has no Cua entry. Proposed addition under
  Session tooling (not applied):
  `- Cua: littleseouldream/cua; workspace/cua; Driver 0.30.4 globally configured for local Codex projects (verified 2026-09-30); next: fresh-session GUI acceptance.`
- Preserve the inherited Apple Silicon test instructions below. That upstream
  workstream was not executed or changed by this Windows setup session.

## History

### 2026-09-30 — Fork and shared Windows driver setup

- Fork: <https://github.com/littleseouldream/cua>, created from `trycua/cua`.
  Shared checkout: `C:/Users/User/workspace/cua` on `DESKTOP-JIHTKTS`.
- Branch: `main`, tracking `origin/main`. `origin` is the personal fork;
  `upstream` is <https://github.com/trycua/cua.git>.
- Source baseline: `0f29c142d7fe3e05ea0ce276cee11b3a9725ba01`.
  No application source changed; this handoff is the only repository edit.
- Installed published component `cua-driver-rs-v0.30.4` from its Windows x86_64
  release archive. This version was selected from the canonical installer's
  baked version, not the repository-wide latest-release badge. The archive's
  SHA-256 matched GitHub's release asset digest:
  `71ca8dfb3b98edd1e897ec59715269c40610879e03aeff6ae0e2ce709c6900c9`.
- Installed release directory:
  `C:/Users/User/.cua-driver/packages/releases/0.30.4-x86_64-pc-windows-msvc`.
  `C:/Users/User/.cua-driver/packages/current` is a junction to that directory.
  `C:/Users/User/AppData/Local/Programs/Cua/cua-driver/bin` is a junction to
  `packages/current` and was added to the user PATH.
- All seven shipped files match the extracted, checksum-verified archive:
  `cua-driver.exe`, `cua-cursor-theme.exe`, `cua-driver-uia.exe`,
  `cua_driver_abi.h`, `cua_driver_node_runtime.node`, `cua_driver_sdk.dll`, and `LICENSE`.
  Binaries are published upstream artifacts, not a build of this fork's HEAD.
- User-wide `C:/Users/User/.codex/config.toml` has `[mcp_servers.cua-driver]`,
  with command `C:\Users\User\AppData\Local\Programs\Cua\cua-driver\bin\cua-driver.exe`
  and arguments `["mcp"]`. Verified enabled from another project directory.
  No project-specific MCP configuration was added.
- Driver skill installed at `C:/Users/User/.cua-driver/skills/cua-driver` and
  linked into `C:/Users/User/.agents/skills/cua-driver` and
  `C:/Users/User/.claude/skills/cua-driver`. Claude Code has the skill; its MCP
  configuration was not changed.
- Telemetry is disabled persistently. Driver authorization uses its default
  standard mode. No existing-browser-profile grant or autostart task was added.
  On Windows, bare `cua-driver mcp` owns its runtime; one-shot CLI GUI calls
  require a separately running driver service.
- Verification: `doctor --json` passed, including attached interactive desktop,
  UI Automation and visible-window discovery. A real stdio MCP client negotiated
  protocol `2025-06-18`, discovered 59 tools, and called `list_apps` successfully.
  Test connection was closed afterward. No input actions or screenshots were
  needed to validate installation.
- Configuration backup: `%TEMP%/cua-shared-setup-20260930/codex-config.before-cua.toml`.
  Here `%TEMP%` is `C:/Users/User/AppData/Local/Temp`.
  This local backup exists and is intentionally excluded from Git. To unregister
  only this integration, use `codex.cmd mcp remove cua-driver`; preserve other
  configuration changes when restoring any backup. The verified archive and
  extracted release also remain under `%TEMP%/cua-shared-setup-20260930/`.
- Drift audit: no live server, website, database or cloud sandbox changed.
  Installed binaries and both junction targets were checked; configuration and
  skill links are intentionally user-local. No unresolved installation failures
  or live-only hotfixes remain. Fresh-session GUI acceptance is the remaining
  validation item, not a known failure.

---

# Apple Silicon Local Linux Docker E2E Handoff

## Objective

Validate `trycua/cua#3257` on a real Apple Silicon Mac using Docker Desktop. The run must prove that the ARM64 image starts through `cua-sandbox`, computer-server is reachable through Docker's published host port, desktop screenshots and clipboard operations work, and the ephemeral container is removed.

Do not modify code, create commits, push branches, or change the pull request. Run the test and report evidence only.

## Prerequisites

- Apple Silicon Mac (`uname -m` returns `arm64`)
- Docker Desktop running
- `uv` installed
- Local clone of `https://github.com/trycua/cua`
- Branch `codex/cua-sandbox-arm64-linux`
- Candidate Docker image tag or digest, if testing before `docker-latest` is published

## Checkout

```bash
git fetch origin codex/cua-sandbox-arm64-linux
git switch codex/cua-sandbox-arm64-linux
git pull --ff-only origin codex/cua-sandbox-arm64-linux
git status --short
```

Stop if the worktree is not clean.

## Record Environment

```bash
uname -a
uname -m
docker version
uv --version
```

## Candidate Image Run

Use the immutable Docker image produced by `trycua/cloud#7099`. A local Docker tag is also accepted.

```bash
IMAGE='<candidate-image-tag-or-digest>'

docker pull "$IMAGE" || docker image inspect "$IMAGE"

docker image inspect "$IMAGE" \
  --format 'repo_digests={{json .RepoDigests}} architecture={{.Architecture}} os={{.Os}}'

./libs/python/cua-sandbox/scripts/live_local_linux_docker.py \
  --image ""
```

Expected terminal result:

```text
PASS: local Linux Docker sandbox is healthy
```

## Published Default Run

Run this only after `public.ecr.aws/k5j5w0x5/cua-ubuntu-24.04:docker-latest` exists. This verifies the image resolution shipped by `cua#3257` rather than an explicit override.

```bash
./libs/python/cua-sandbox/scripts/live_local_linux_docker.py
```

## Success Criteria

The script must verify all of the following:

- host architecture is Apple Silicon ARM64;
- the Linux container reports `aarch64`;
- computer-server accepts shell commands through the published Docker port;
- screenshot is a valid PNG larger than 10 KB;
- screen dimensions are positive;
- clipboard write/read returns the exact marker;
- the ephemeral Docker container is removed after exit.

Artifacts are written to `/tmp/cua-linux-arm64-live`:

- `summary.json`
- `screenshot.png`
- `docker-inspect.json` on failure
- `docker.log` on failure

## Failure Report

Return:

1. Full command output.
2. `/tmp/cua-linux-arm64-live/summary.json`.
3. `/tmp/cua-linux-arm64-live/docker.log`, if present.
4. `/tmp/cua-linux-arm64-live/docker-inspect.json`, if present.
5. Output from:

```bash
docker ps -a --filter label=cua.sandbox=true
docker images --digests | grep -E 'cua-ubuntu|cua-xfce|xfce-cua' || true
git status --short
git rev-parse HEAD
```

Do not mark the validation successful if the test only passes under `--platform linux/amd64`; native ARM64 execution is required.
