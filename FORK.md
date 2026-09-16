# MAYFCHOI/orca — WSL patch fork

Personal fork of [stablyai/orca](https://github.com/stablyai/orca) that carries a
small set of WSL fixes on top of each upstream stable release.

## Layout

| Branch / tag | Meaning |
|---|---|
| `wsl-patches` | default branch. Upstream tag + a few patch commits on top. Never merged, always rebased. |
| `vX.Y.Z-wsl.N` | fork release: `wsl-patches` rebased on upstream `vX.Y.Z`, N-th build. Unsigned Windows installer + `latest.yml`. |
| `main` | untouched mirror of upstream `main` at fork time. Not used. |

Upstream moves fast (hundreds of commits a week, a release almost daily), so the
patch set must stay small enough to rebase mechanically. Every patch is also sent
upstream as a PR; once merged there it is dropped from `wsl-patches`.

## Patches carried

| Commit | Upstream ref | What |
|---|---|---|
| fix(cli): stop the WSL launcher from forwarding a duplicate PATH | [#9498](https://github.com/stablyai/orca/issues/9498), PR [#11341](https://github.com/stablyai/orca/pull/11341) | WSLENV `PATH/l` reaches orca.exe as a second `PATH` next to `Path`; .NET aborts on the duplicate. Cherry-pick of the open upstream PR. |
| fix(cli): hand PowerShell the legacy \\wsl$\ bridge path | [#20082](https://github.com/stablyai/orca/issues/20082) | `wslpath -w` returns `\\wsl.localhost\...`, which Windows PowerShell treats as an untrusted zone and refuses with `AuthorizationManager check failed`. The launcher rewrites it to `\\wsl$\...`. |
| build: add the fork Windows release channel | fork only | `ORCA_WIN_FORK=1` packages unsigned and publishes to `MAYFCHOI/orca` releases so the installed app updates from the fork. |
| ci: fork-wsl-release workflow | fork only | Every 6h (or on dispatch): rebase `wsl-patches` on the newest upstream `vX.Y.Z`, tag `-wsl.N`, build on `windows-2022`, publish. Rebase conflicts open an issue labelled `fork-sync`. |

## Install / update

Download `orca-windows-setup.exe` from the newest
[`vX.Y.Z-wsl.N` release](https://github.com/MAYFCHOI/orca/releases). The app then
auto-updates from this fork's releases (prerelease channel).

## Rollback

Only roll back when the problem is caused by a fork patch. Install the matching
upstream installer, `https://github.com/stablyai/orca/releases/tag/vX.Y.Z`
(`orca-windows-setup.exe`). Same version number, signed, updates from upstream
again. Settings and worktrees are untouched by either direction.

## Working on a patch

```bash
git fetch upstream --tags
git rebase --onto vX.Y.Z "$(git describe --tags --abbrev=0 --exclude '*-wsl.*')" wsl-patches
pnpm install --frozen-lockfile
pnpm exec vitest run --config config/vitest.config.ts src/main/cli/wsl-cli-scripts.test.ts
git push --force-with-lease origin wsl-patches
gh workflow run fork-wsl-release.yml
```
