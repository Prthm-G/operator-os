# Vault structure

The vault is a git repo of mostly markdown. Its job is to make a fresh agent session
useful in one read, without browsing and without being told anything twice.

## The taxonomy

```
context/          who I am, what the business is, what this quarter's priorities are
decisions/log.md  append-only record of what was decided and why
connections.md    registry of every system the OS can reach, and its wiring status
references/       frameworks, voice samples, API guides. Researched once, saved forever.
raw/              immutable source material. Never edited, renamed, or deleted.
wiki/             LLM-maintained knowledge synthesized from raw/
projects/         one directory per workstream, each with its own decisions log
archives/         superseded work. Moved here, never deleted.
publish/          the public-facing pipeline and everything authored for publication
.claude/skills/   task-specific skills
```

Two conventions do most of the work:

**`raw/` is immutable and `wiki/` is derived.** Sources are never edited. Corrections
arrive as new sources. The wiki is free to be reorganized, resynthesized, and rewritten by
an agent because it can always be rebuilt, and because nothing in it is the last copy of
anything.

**`archives/` instead of deletion.** Superseded material moves, keeping its history and
its context. Deletion destroys the record of what was tried and abandoned, which is
exactly the record that stops you retrying it in eighteen months.

## The decisions log

This is the highest-signal file in the system, and the one an agent should read first.

Format per entry:

```markdown
## YYYY-MM-DD · Short title

**Decision:** what was decided.
**Why:** the reasoning, the constraints, and what would change my mind.
**Alternatives considered:** what else was on the table, and why it lost.
**Owner:** who is accountable.
```

The **Why** and **Alternatives** fields are the point. Code records what the system does.
Git records when it changed. Only this file records why a given option was allowed to win,
and what evidence would justify revisiting it.

Practical consequences:

- An agent reads the last few entries before touching anything, so it does not redo
  shipped work or reopen a settled question.
- Corrections belong in the log, in place. When a later entry finds that an earlier one
  was wrong about a fact, it says so explicitly rather than quietly editing history.
- Project-scoped decisions live in that project's own log, with the root log carrying a
  pointer. One file per blast radius.

## The wiki layer

A small schema keeps LLM-maintained knowledge from drifting into confident fiction:

- Every evidence-bearing claim cites a source note, and where possible a section, page,
  timestamp, or row.
- Never invent a source, a locator, a quotation, a date, or a certainty level.
- **Preserve disagreement.** Conflicting claims are marked as conflicting, with an
  explanation of which evidence is newer or stronger. They are not silently reconciled
  into a single confident sentence.
- Prefer updating an existing page over creating a near-duplicate.
- Every completed operation is appended to an operations log, so the wiki's own history is
  auditable.

The rule against silent reconciliation is the one that matters most. A knowledge base that
smooths over contradictions is worse than no knowledge base, because it reads as settled
when it is not.

## The health file

Written every morning by the always-on caretaker: overdue dates, connections that have
gone cold, uncommitted work.

It is detection only. Nothing in it has been acted on, and the file says so at the top.
Whoever reads it does the deciding. Splitting detection from judgment is what keeps a
cheap always-on agent from accumulating authority nobody granted it.

## What is not in here

Live credentials, customer data, and financial records are not in the vault, not in any
repo, and not in any prompt. Secrets live in a single env file outside every git tree.
Two projects are separate repos, gitignored, and masked out of the caretaker's sandbox at
the mount level.

## Related

- [Architecture →](architecture.md)
- [Publishing →](publishing.md) how a subset of this becomes public
