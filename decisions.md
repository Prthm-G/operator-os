# Decisions

Selected entries from the vault's append-only decisions log. Each records what
was decided and, more usefully, why. Including the options that lost.

This is a filtered view. Entries are private by default and appear here only
when explicitly marked for publication.

---

## 2026-08-24 · AIOS split into a private vault and a public showcase; the fork is deleted

**Decision:** Stopped being a fork. `Prthm-G/AIOS` was a **public** fork of
`nateherkai/AIS-OS`; it is now a private, standalone repo, and a separate public repo
`Prthm-G/operator-os` carries the portfolio-grade write-up. Docs across the vault were
rewritten to describe the system that actually exists rather than the starter kit it began
as. A publish pipeline now stands between the two, and it is default-private.

**What was actually wrong.** Three findings, in order of how much they mattered:

1. **A nightly cron pushed the whole vault to a public repo.** The crontab ran
   `daily-commit-and-push.sh … git@github.com:Prthm-G/AIOS.git main` at 23:30, committing
   and pushing everything: a directory of personal documents, the live marketing site's
   source, the growth workspace, and the vault's own context and decision files.
2. **The vault documented a different script than the one running.** `.scripts/auto-commit-daily.sh`
   said "Never pushes to any remote. Local commit only." True of that file, false of the
   generalized script that replaced it on 2026-07-31 and that the crontab actually calls.
   The repo described one behaviour while the machine did another, for several weeks.
3. **A public fork cannot be made private.** GitHub returns
   `422 "Public forks can't be made private"`. Deletion was the only real remedy, which is
   why the repo was destroyed and recreated rather than flipped.

No credentials were exposed. A full pattern and keyword scan over every tracked file came
back clean; the `EAA…`-shaped hits were LinkedIn tracking parameters in scraped job
clippings, and the personal documents carry a professional email and a city, no phone and
no street address. The exposure was business internals and site source, not secrets.
Nothing to rotate.

**Why the failure is interesting.** Every individual piece was sensible. A nightly backup
is good. Committing your work is good. Pushing to a remote is good. The *composition* was
wrong and nothing was watching the composition. So the fix is not "be more careful", it is
a mechanism that fails closed when someone adds a file without thinking about publishing at
all.

**The pipeline that replaced it.** Default private, opt-in, allowlist-only, gated:

- Public docs are **authored** in `publish/showcase/`, never selected out of the vault.
  There is no glob over the private tree that could misfire, because there is no glob over
  the private tree.
- A decisions entry publishes only if its heading carries `<!-- public -->`. This entry has
  it; that is the entire publish action.
- A project publishes only with a `publish.yml` declaring `public: true`. New projects are
  private without anyone remembering to make them so.
- `.scripts/lib/scan-publish.sh` gates every staged tree and aborts the push on any hit.
  Four checks: denied paths, denied extensions, forbidden literals, credential shapes.
  Validated at build time: 14/14 catches on a known-bad tree, 0 false positives on a
  known-good one.
- Project mirrors force-push a fresh single commit, so a file published by mistake and then
  removed from the manifest does not survive in the public object store.

Two details worth reusing elsewhere. The **extension check** exists because `grep -I` skips
binaries entirely, so a content scanner will pass a `resume.pdf` without looking inside; that
blind spot needs a different kind of check, not a better regex. And **forbidden strings are
literals, not regexes**, so the list has zero false positives and extends without touching
code.

**Caught during build, worth recording.** Three defects found by testing the pipeline
rather than by reading it:

- The showcase commit initially inherited the vault's git identity, which is a private
  address on the forbidden list. Git would have stamped it into every public commit's
  metadata, where no content scan would ever look, because commit metadata is not file
  content. Public repos now pin the public contact address explicitly. Worth generalizing:
  a scanner that only reads files cannot see what git writes about the author.
- `publish.yml` is also a common GitHub Actions workflow filename, and vendored dependencies
  ship them. Two live ones sit inside a `node_modules` tree under one of the masked project
  directories. A single `-not -path` glob was the only thing keeping the nightly publisher
  out of that tree. Discovery now prunes denied trees during the walk, and
  `publish-project.sh` independently refuses a denied root before reading the manifest,
  resolving the real path first so a symlink or a `..` cannot walk out of `projects/`.
  Verified refused: every masked project path, vendored manifests, and a traversal to the
  keys directory.
- The decisions generator stripped the opt-in marker from an entry's whole body, which would
  have mangled any entry that documents the marker mechanism. It now strips from the heading
  only.

**Alternatives considered:** Flip the existing repo to private (impossible, see above).
Rename and empty the fork instead of deleting (rejected: old commits stay reachable by SHA
and the fork banner survives). Keep one repo and rely on `.gitignore` (rejected: that is a
denylist, and a denylist fails open). Rewrite history with filter-repo to strip private
paths from all 38 commits (rejected as more moving parts than a fresh private repo needed).

**A leak the gate did not stop, and the fix.** The first published version of this entry
named the second business and one of its masked project directories in prose. The gate passed
it, because those trees were **path**-denied but not **content**-denied: a path rule stops a
directory from being copied and does nothing about a sentence describing it. Three changes:

- Those names are now literals in `publish/forbidden.txt`. Anything that must not be
  *mentioned* belongs there, not only in `PATH_DENY`.
- The showcase now **rebuilds history** on every publish instead of appending. The original
  reasoning, that every file here was authored for publication anyway, was wrong on day one:
  with an incremental history, untagging an entry does not remove the blob. Opt-in publishing
  is only real if opting back out actually un-publishes.
- Force-pushing was not enough. GitHub kept the old commits reachable by SHA. The showcase
  repo was deleted and recreated to remove them, verified by fetching the old SHAs and
  getting Not Found. Worth remembering: **force-push is not deletion.**

**Backup / rollback:** a verified `git bundle` of the complete pre-split history, plus an
rsync of the worktree and the pre-change crontab, held offline.

**Owner:** Pratham. Nate Herk's starter-kit material moved to `archives/aisos-starter-kit/`
(kept per the archive-don't-delete convention, published nowhere); LICENSE reissued MIT ©
Pratham Goel. Open follow-up: decide which older decision entries are worth tagging
`<!-- public -->`, and whether any project gets a `publish.yml` first.
