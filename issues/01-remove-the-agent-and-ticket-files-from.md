# 01 — Remove the agent and ticket files from the public repo and its history (gap 3)

Type: build
Status: needs you (captured 2026-10-12)
Blocked by: —
Tags: #security

## State at filing

- stage collect_carriers: failed, 5054.2 min ago
- stage collect_messages: late, 7071.3 min ago
- stage mail_collect: failed, 26.0 min ago
- stage backups: stale, 0.0 min ago

## What we know

System review 2026-10-07 gap 3: this is a public repo (GitHub Pages, icsapo.github.io) and it carries agent/CLAUDE.md, .claude/CLAUDE.md, issues/_template.md and map.md -- internal Daybook process files with no business being public. Remove them from the tree and rewrite history to drop the blobs.

All four are in the current tree and in history: `agent/CLAUDE.md` (2 commits), `.claude/CLAUDE.md` (1 commit, it
is a symlink to `agent/CLAUDE.md`), `issues/_template.md` (4 commits), `map.md` (1 commit). None holds personal
data -- they are the stamped process files every Daybook project carries -- but they describe internal workflow
and have no reason to be in a repo GitHub Pages serves publicly.

Done by the agent (2026-10-12): a mirror backup at `~/Daybook-backups/2026-10-12-privacy-site/Site.git` (mode 700,
delete when satisfied); the blob ids for the four paths' history in
`~/Daybook-backups/2026-10-12-privacy-site/Site.ids` (7 ids), via a new sibling script `path_blobs.py` next to
`plain_blobs.py` in the same backup folder -- `plain_blobs.py` only judges git-crypt-marked paths, and this repo
marks none. One blob was dropped from the list by hand: the `.claude/CLAUDE.md` symlink's own blob (it stores only
the path string "agent/CLAUDE.md"; git never filters a symlink's content through git-crypt, the same exception
noted on Index ticket 142).

## For the user: remove the files, rewrite, force-push

Four phases. Phase 1 touches the working tree, so it is yours; phases 2-4 can go back to an agent once phase 1 is
done, or you can run all four yourself.

**Phase 1 - remove and commit** (in `~/Daybook/10-Admin/Site`):

```zsh
cd ~/Daybook/10-Admin/Site
git rm agent/CLAUDE.md .claude/CLAUDE.md issues/_template.md map.md
```

Add one line to `README.md`: the site is published from this repo and holds no project files (so a later
onboarding pass does not re-stamp them back in; check `tools/onboard.py`'s per-repo opt-out, or ask the
coordinator to add one for this repo). Then:

```zsh
git add README.md
git commit -m "Remove the agent and ticket files from the public repo (ticket 01)"
git push origin main
```

**Phase 2 - rewrite a fresh mirror:**

```zsh
B=~/Daybook-backups/2026-10-12-privacy-site
W=$B/work/Site.git; u=git@github.com:icsapo/icsapo.github.io.git
git clone -q --mirror $u $W
old=$(git -C $W rev-parse main)
git -C $W filter-repo --force --strip-blobs-with-ids $B/Site.ids
```

**Phase 3 - force-push with lease:**

```zsh
git -C $W push --force-with-lease=refs/heads/main:$old $u refs/heads/main:refs/heads/main
```

**Phase 4 - realign the local clone:**

```zsh
r=~/Daybook/10-Admin/Site
git -C $r fetch -q origin
git -C $r reset -q --hard origin/main
git -C $r reflog expire --expire=now --all; git -C $r gc -q --prune=now
```

Then tell the coordinator: the remote tree no longer has the four files, none of the 7 listed blobs is in the
remote's history, and GitHub Pages still serves `docs/` correctly. Delete
`~/Daybook-backups/2026-10-12-privacy-site/` when satisfied.

Note: GitHub's cached views (raw.githubusercontent.com, its search index, a fork or a PR that touched these
commits) can keep serving the old blob content for a while after a force-push; a support request to GitHub purges
it sooner if that matters here.

## Decision

## Progress

- 2026-10-23: the user parked it for a calmer moment (b)
