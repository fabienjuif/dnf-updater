# Maintaining the fork

`main` here = `upstream/main` (strob3/dnf-updater) + every PR branch merged + a thin fork-only layer.

## Branches

- One branch per upstream PR, cut from `upstream/main`, in upstream's commit style (`feat:` / `fix:`),
  no version bump or CHANGELOG entry (the maintainer does those at release).
- `main` merges each PR branch, then carries:
  - glue that only makes sense once two PRs are both in (today: `saveState()` for
    `last_update_timestamp`, PR #2 + #5);
  - the fork layer: plugin id `fabienjuif/dnf-updater` in `plugin.toml`, and the ids hardcoded in
    `widget.luau` (`togglePanel`) and `service.luau` (the recheck IPC), panel title "Updater", the
    README fork note.
- PR branches use `strob3/dnf-updater` ids. When a fix lands on `main`, port it to the PR branch with
  the upstream id, and the other way round.

Open PRs: https://github.com/strob3/dnf-updater/pulls?q=author%3Afabienjuif

## Updating

```sh
git fetch upstream
# a PR changed: commit on its branch, push, then on main:
git merge <pr-branch>
# upstream moved: merge upstream/main into main (and rebase/merge PR branches if they conflict)
```

When a PR merges upstream, `main` keeps working on the next upstream merge; drop the branch.

## Gotchas

- The plugin id differs from upstream so both can coexist in Noctalia's catalog; the `strob3` id is
  already taken by the community source.
- Noctalia loads this checkout live (`~/.local/share/noctalia/plugins/dnf-updater` symlinks here) and
  hot-reloads on file change. Do branch work in `.worktrees/<name>`, not by switching branches in the
  main checkout, or the shell loads half-states with the upstream id.
- Reload after an edit: `noctalia msg plugins disable fabienjuif/dnf-updater; noctalia msg plugins enable fabienjuif/dnf-updater`.
  The "unrecognized widget type" warnings logged in between are expected.
- `noctalia plugins lint .` only cross-checks settings, not Luau syntax: check the shell log
  (`journalctl --user | grep dnf-updater`) after a reload.
- A path source must be a directory *of* plugins, so this repo (plugin at its root) is installed by
  cloning or symlinking it into `~/.local/share/noctalia/plugins/`, not as a `[[plugins.source]]`.
