# git-stray

**English** · [日本語](./README.ja.md) · [Français](./README.fr.md)

Find the work that exists only on your disk.

```
$ git stray ~

git-stray  scanned 165 repositories

AT RISK  ~/courses/swift/Egg
    no remote configured — 1 commit exists only on this disk (last 558d ago)
    7 uncommitted files — untouched for 558d
AT RISK  ~/hackathon/team-20-app
    12 unpushed commits — main(12) (last 336d ago)
STALE    ~/work/old-service
    stash 1922d old — stash@{0}: WIP on feature/settings

2 at risk, 1 stale.
```

## Why

Every repository status tool answers *what changed*. None of them answer the
only question that matters when a disk dies: **what is here and nowhere else?**

A branch you never pushed, a repository you never gave a remote, a stash from
five years ago — none of these are broken, so nothing ever complains about
them. `git status` is silent because, locally, everything is fine. They are
simply files that happen to exist in exactly one copy.

git-stray walks every repository under a path and reports only that: work with
no second copy anywhere.

## What it deliberately does not do

It does not tell you a project is old.

Inactivity is not a problem. You are allowed to not touch something for eight
months; that is usually a schedule, not a mistake. A tool that lists your
quiet projects back at you is a tool you will uninstall.

So uncommitted changes and stashes are reported only once they are older than
`--days` (14 by default), and nothing is ever reported merely for being
inactive. Unpushed commits and remote-less repositories are always reported,
because those genuinely exist in one copy.

## Install

```sh
curl -o /usr/local/bin/git-stray \
  https://raw.githubusercontent.com/urabexon/git-stray/main/git-stray
chmod +x /usr/local/bin/git-stray
```

Anything named `git-*` on your `PATH` becomes a git subcommand, so you can then
run it as `git stray`.

Requires bash 3.2+ (the macOS system bash is fine) and git.

## Usage

```sh
git stray                  # scan below the current directory
git stray ~                # scan your whole home directory
git stray --days 30 ~      # only nag about things older than a month
git stray --days 0 ~       # report everything, however recent
git stray --fetch ~        # refresh remote refs first (network, slow)
git stray --hidden ~       # include repositories in hidden directories
git stray --depth 6 ~      # look deeper for repositories (default 4)
```

### Exit codes

| Code | Meaning |
|------|---------|
| `0`  | nothing exists in only one copy |
| `1`  | something does |

```sh
git stray ~ || echo "push something before you close the laptop"
```

## What it reports

| Severity | Finding |
|----------|---------|
| AT RISK | Commits on a local branch that no remote ref contains |
| AT RISK | Commits on a detached HEAD that no remote ref contains |
| AT RISK | A repository with no remote at all — every commit in it is single-copy |
| STALE | Uncommitted changes to tracked files, older than `--days` |
| STALE | Stashes older than `--days` |
| STALE | Untracked, unignored files that were never added, older than `--days` |

Repositories inside hidden directories (`~/.nvm`, `~/.oh-my-zsh` and friends)
are skipped by default — those are other people's clones, not your work.

Ages for commits and stashes come from git itself and are exact.

Ages for uncommitted changes and untracked files come from file mtimes, and are
always printed with a `~`. An mtime records when a file was last *written*, not
when the work was done: `git checkout`, `git pull` and `git stash pop` all
rewrite files without changing anything you did. Work from two years ago can
therefore look a week old, and be suppressed by `--days`. Treat those numbers as
a lower bound on age, not a measurement.

When there is no file to read an mtime from at all — a deletion, a rename — the
age falls back to the last commit date. It is never silently dropped.

## Accuracy

Comparison is against the remote-tracking refs already on your disk. That is
fast and offline, but it is only as fresh as your last `fetch`. Pass `--fetch`
to refresh first, at the cost of one network round trip per repository.

Note that "unpushed" is computed against *every* remote ref, not against the
branch's configured upstream. A branch with no upstream set, or one pushed
under a different name, is correctly treated as pushed.

## Related

- [git-brink](https://github.com/urabexon/git-brink) — the sibling of this
  tool. git-brink stops things leaving that never should; git-stray finds
  things that never left and should have.
- [gita](https://github.com/nosarthur/gita), [mr](https://myrepos.branchable.com/) —
  operate on many repositories at once. Different goal: they show you
  everything, this shows you only what is at risk.
- [gitleaks](https://github.com/gitleaks/gitleaks) — scans committed content
  for secrets. Unrelated problem, frequently confused with the above.

## License

[MIT](./LICENSE)
