# Self-hosted macOS runner

The `build` job in `.github/workflows/ci.yml` runs on a macOS arm64 runner registered to this repository. GitHub-hosted jobs can't start while the account's Actions billing is locked, but self-hosted jobs still run.

## Registration

| Field | Value |
|-------|-------|
| Labels | `self-hosted`, `macOS`, `ARM64`, `mfs` |
| Register at | [Settings → Actions → Runners → New self-hosted runner](https://github.com/donaldfilimon/mfs/settings/actions/runners/new?arch=arm64) (macOS, ARM64) |

A runner is registered to one repository. If the same Mac already serves another repository (for example the `abi` or `gama` runner), install a second runner in its own directory (for example `~/actions-runner-mfs`), run `./config.sh` with the URL and token from the page above, add the custom label `mfs`, then run `./svc.sh install && ./svc.sh start`.

Until a runner with these labels is online, same-repo `build` jobs wait in the queue.

## Host requirements

- Xcode Command Line Tools (`xcode-select --install`), or Xcode. `zig build` links the Cocoa, Metal, OpenGL and other system frameworks listed in `addMacosDependencies` in `build.zig`, and Zig finds the macOS SDK through `xcrun`.
- Outbound HTTPS to `ziglang.org`. `goto-bus-stop/setup-zig@v2` downloads the `aarch64-macos` Zig build into the runner's tool cache, so Zig does not need to be installed on the host.
- Nothing needs `sudo`.

## Security

This repository is public, so the self-hosted job runs only when the workflow is in `donaldfilimon/mfs` and the event is a `push` to `main`/`master` or a pull request from a branch in this repository. Pull requests from forks use the GitHub-hosted `build-hosted` job ("build (GitHub-hosted, fork PRs)") instead. The self-hosted checkout uses `persist-credentials: false`, and the workflow token is `contents: read`.

No workflow in this repository uses `pull_request_target`, `issue_comment` or `workflow_run`. Don't add a self-hosted job behind one of those triggers.

Where you can, run the runner under a dedicated macOS user rather than your daily account, and keep no production secrets on the host.

## Stays GitHub-hosted

- `build-hosted` runs only for fork pull requests, on `ubuntu-latest`, so untrusted code never reaches the Mac. It stays blocked until the billing lock is cleared.

## Known issue (not caused by the runner)

The workflow pins Zig 0.12.0, but `build.zig.zon` declares `minimum_zig_version = "0.16.0-dev.1484+d0ba6642b"` and `build.zig` uses the newer `root_module` build API. The `build` job will reach the runner and then fail at `zig build` until the pinned Zig version matches the code.
