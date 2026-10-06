# Fork

This repo is `mrsombre/mattpocock-skills`, a fork of [`mattpocock/skills`](https://github.com/mattpocock/skills). Local changes live as commits on the `trail` branch, on top of `upstream/main`. `trail` is the default branch on `origin` and the only branch there.

Remotes:

- `origin`: `https://github.com/mrsombre/mattpocock-skills.git`, the only push target.
- `upstream`: `https://github.com/mattpocock/skills.git`, fetch only. Its push URL is `DISABLED`.

`gh` defaults to the fork (`gh repo set-default mrsombre/mattpocock-skills`), so `gh pr create` targets `origin`. Never open a PR or push against `mattpocock/skills`.

Set up a fresh clone:

```sh
git remote add upstream https://github.com/mattpocock/skills.git
git remote set-url --push upstream DISABLED
gh repo set-default mrsombre/mattpocock-skills
```

## Sync with upstream

```sh
git fetch upstream
git log --oneline upstream/main..trail   # fork commits to replay
git rebase upstream/main
```

On a conflict, resolve each file, then `git add <file>` and `git rebase --continue`. Keep upstream's intent and re-apply the fork change on top. To stop and restore the pre-sync state, run `git rebase --abort`.

The fork keeps upstream's layout: `CLAUDE.md` holds the repo rules and `AGENTS.md` is a symlink to it. The fork adds only a short `## Fork` section at the top of `CLAUDE.md` that points here, so an upstream edit to `CLAUDE.md` replays like any other file. Keep every other fork rule in this file. Never turn `AGENTS.md` into a regular file.

After the sync, check the local install (below) if upstream added, removed, renamed, or moved a skill between buckets.

## Local install

Installed skills run straight from this clone. A hand-picked subset of the skills is linked, not every skill in the repo. Each installed skill `<name>` has three absolute symlinks to `skills/<bucket>/<name>` in this clone:

- `~/.agents/skills/<name>`: Codex and other Agent Skills harnesses
- `~/.claude/skills/<name>`: personal Claude Code
- `~/.claude-work/skills/<name>`: work Claude Code

The `~/.claude*` links point at the repo directly, never through `~/.agents/skills`. Each installed skill also has an entry in `~/.agents/.skill-lock.json` with `sourceType: "local"`, `source` set to this clone, and a `pluginName` that sets its group in `pnpx skills ls -g`. The lock file is the record of which skills are installed from this clone and in which group; read it instead of guessing. The entry carries no `skillFolderHash`, so `pnpx skills update` skips it and never replaces a link with a downloaded copy.

Some skills are maintained in `~/dev/agents/mrsombre-skills` under their upstream name. The folders in `~/dev/agents/mrsombre-skills/skills/` list them; a skill that exists in both repos is owned there. Its folder here stays at upstream text, its links and lock entry point at that clone, and the fork makes no commits to it. Leave such skills alone after a sync.

A sync reaches the installed skills with no reinstall. The links break only when a skill's path changes:

- Renamed or moved skill: repoint its three links and update `skillPath` in its lock entry.
- Removed skill: remove its three links and its lock entry.
- New skill: nothing is installed until you add the three links and a lock entry.

Do not run `scripts/link-skills.sh` in this fork. It links every skill outside `deprecated/` and `misc/`, which includes skills left out on purpose, it skips `~/.claude-work/skills`, and it deletes any real directory in its way. This rule overrides the `scripts/link-skills.sh` paragraph in `CLAUDE.md`.

## Fork commits

One commit per skill change, with the reason in the message. Keep the docs page and `ask-matt` in sync as `CLAUDE.md` asks, but add no changeset and do not bump the plugin version: releases are upstream's.

## Push to origin

A rebase rewrites fork commits, so push with a lease:

```sh
git push --force-with-lease origin trail
```

`--force-with-lease` refuses the push if `origin/trail` has commits the local clone has not fetched. In that case, fetch `origin`, rebase onto `origin/trail`, and push again.
