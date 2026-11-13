# Publishing

This repo is generated from a private vault that holds live business data. That is a
dangerous thing to automate, so the pipeline is built to fail closed at every step.

## Why it exists

The setup this replaced had no pipeline at all. A nightly cron job committed the entire
vault and pushed it to a repo that was public, and the script the vault documented as the
one doing the work was not the script actually running. Nothing catastrophic was in it,
no credentials, but business internals and a live site's source were public for weeks and
nobody had decided that they should be.

The interesting failure was not the exposure. It was that **every individual piece was
sensible**. A nightly backup is good. Committing your work is good. Pushing to a remote is
good. The composition was wrong, and nothing in the system was watching the composition.

So the replacement is designed around one question: what happens when someone adds a file
without thinking about this pipeline at all?

## Four properties

### 1. Allowlist, never denylist

A denylist fails open. The day you add a folder nobody thought to exclude, it ships. An
allowlist fails closed: an unlisted folder simply does not go anywhere.

Every path into a public repo is an explicit list of what to include. There is no
"everything except" anywhere in the pipeline.

### 2. Authored, not selected

Public documentation is written into a dedicated directory. It is copied wholesale into
the public repo.

This inverts the usual arrangement, and the inversion is the safety property. A file
reaches the public repo because someone wrote it there on purpose, not because a glob
matched it. There is no pattern over the private tree that could misfire, because there is
no pattern over the private tree.

The exception is per-project mirrors, which do copy from a real source tree. Those carry
an explicit include-allowlist plus the gate below.

### 3. A gate that aborts the push

Every staged tree is scanned before anything leaves the machine. Any hit fails the whole
publish. The gate does not decide what is probably fine; it stops and makes a human look.
A false positive costs a minute. A false negative costs a credential.

Four independent checks:

| Check | Catches |
|---|---|
| **Path deny** | Directories that must never exist publicly, whatever they contain |
| **Extension deny** | File types never published: documents, databases, keys, dumps |
| **Forbidden literals** | An editable list of exact strings: personal identifiers, account IDs, machine paths |
| **Secret shapes** | Credential patterns: private keys, provider tokens, bearer headers, credentials embedded in URLs |

Two details worth stealing:

**The extension check exists because content scanning has a blind spot.** `grep -I` skips
binary files entirely, so a scanner that only greps content will pass a PDF resume or a
SQLite database without ever looking inside. Those are judged by extension instead.
The blind spot is not fixable by better regexes; it needs a different kind of check.

The first version of this gate got that half right and half wrong. It denied documents and
databases but not images or archives, so a key hidden in a PNG comment chunk passed both
checks: the extension list did not cover it, and the content scanner could not read it.
Two checks, one shared blind spot, is one check. Layers only help when they fail
differently.

**A gate must fail closed, including when the gate itself breaks.** The forbidden-literal
list was loaded best-effort: if the file was missing, or the temp directory was
unwritable, the check was skipped and the scan still reported clean. A check that can
silently disappear is worse than no check, because it produces confident all-clear output.
Every such path now aborts.

**Forbidden strings are literals, not regexes.** A regex guesses at a shape and produces
false positives forever. A literal string is certain. The list lives in its own file, so
it extends without touching code, and it is the single most sensitive file in the system:
it is precisely the enumeration of what must not leak. It is in no manifest.

The gate reuses its credential-shape patterns verbatim from the commit-time scanner rather
than reimplementing them, so the two cannot drift into disagreeing about what a secret
looks like.

### 4. Opt-in per item

- A decisions-log entry publishes when its heading carries an explicit marker. Untagged
  entries never leave. Adding the marker is the entire publish action.
- A project publishes when it has a manifest declaring `public: true`. No manifest, no
  repo. A newly created project is private by default, without anyone remembering to make
  it so.

### Discovery is pruned, not filtered

A project opts in by dropping a manifest file. That filename is also a common CI workflow
filename, and vendored dependencies ship them by the dozen. Treating one as a project
manifest would root a publish inside whatever tree that dependency happens to live in,
which in this vault includes the most sensitive directory on the machine.

So discovery is locked twice, independently:

1. The nightly driver **prunes** denied trees during the walk. It does not descend into
   them at all, rather than walking them and hoping a filter catches every case.
2. The per-project publisher **refuses** a denied root itself, before reading the manifest,
   resolving the real path first so a symlink or a `..` cannot walk out of the allowed area
   and then pass a string check.

The second lock is the one that matters, because that script can be invoked by hand with
any path, and a manifest is untrusted input. A guard that only exists in the caller is a
guard that disappears the moment someone calls the callee directly.

## History is rebuilt, not appended

Per-project mirrors force-push a fresh single commit on every publish.

This is deliberate. If a file is published by mistake and then removed from the manifest,
an incremental history would keep it retrievable from the public object store forever.
Rebuilding from nothing means the mistake is genuinely gone on the next run.

The cost is that public mirrors have no commit history. For a sanitized mirror of a private
tree, that is the right trade. The showcase repo keeps a normal history, because every file
in it was authored for publication to begin with.

## Running it

```bash
publish-showcase.sh --dry-run    # build, scan, print the file list, push nothing
publish-showcase.sh              # build, scan, push if clean
publish-project.sh <name>        # one project mirror
publish-all.sh                   # everything, nightly
```

The nightly driver runs each target independently. One failed gate stops that target, not
the run, and the exit code is nonzero if anything failed, so a partial failure cannot read
as success in a log nobody opens.

The dry run prints exactly what would go public. The discipline is to actually read it.

## The rule when the gate complains

Fix the content. Do not widen the gate.

If a hit is a genuine false positive, add that specific benign case. Never a blanket
exception. A gate with a blanket exception is a gate that has been turned off by someone
who told themselves they were fixing it.
