---
name: onboard
description: Use at the start of a session in the vault to load current operating state before doing anything else. Triggers on "onboard", "catch me up", "where are we", "what's the state", "brief me", or the first substantive request of a fresh session. Reads the vault and the live project directories, then produces a one-screen operating brief. Read-only, it writes nothing.
---

> Published from a private vault. Machine paths, project names, and business specifics are
> replaced with placeholders. The structure is the part worth copying.

## What this skill does

Loads the operator's real, current state into the session and hands back one screen: where
things stand, what needs a decision, what has gone stale, and the single next action.

It is **not** an interview and **not** a scaffolder. Everything it reports is derived at
runtime from files and git. Nothing about the current quarter, deadlines, or project status
is hardcoded here, and that is exactly what keeps it from going stale between runs.

**The design rule:** a skill that hardcodes state is a skill that lies to you in six weeks.
Hardcode only what is genuinely fixed (who the operator is, how they want to be spoken to).
Derive everything else, every run.

## Who this is for

Fixed facts, safe to assume:

- **The operator.** Direct, concise, bullets, no em dashes, no status padding.
- **The businesses**, named, with their domain and customer type.
- **Standing commitments** that do not change month to month.

Everything else, goals, deadlines, what is shipped, what is blocked, comes from the read
order below. Do not assert it from memory.

## Read order

Cheapest first. Stop early if the request is narrow.

**1. The operating layer.** `CLAUDE.md`, then `context/about-me.md`,
`context/about-business.md`, `context/priorities.md`.

**2. `decisions/log.md`, the last 3 entries only.** The highest-signal file in the vault.
Entries are dated and each ends with an **Owner** line, often naming an open follow-up, a
deferred build, or a decision due on a date. Those follow-ups are the main input to the
"needs a decision" section. Do not read the whole log.

**3. `connections.md`.** The registry of reachable systems. Note which rows still say
`not yet connected`, and each row's `last checked` date.

**4. Live project state.** For each mapped directory, run `git status --short` and
`git log --oneline -3`. They are separate repos with their own branches, so check each
rather than assuming the vault's branch applies.

The map names each path, what it is, and critically **whether it is production**. It also
names what is gone or archived, so a session does not go looking for a directory that was
deleted three weeks ago.

**5. Conditional reads.** Only when the request touches them: a job-search handoff file,
the wiki index and schema before any wiki operation, the voice reference before drafting
anything in the operator's voice.

## Staleness check

Flag these. Do not fix them and do not offer to fix them inline. They go in the brief.

- **Passed dates.** Compare every date in `context/priorities.md` and the last 3 decision
  entries against today. A priority whose deadline has passed is the single most useful
  thing to surface.
- **Cold connections.** Any registry row unchecked for more than 30 days, or still
  `not yet connected`.
- **Uncommitted work.** Any project directory with a dirty tree or untracked directories.
  Finished work sitting undeployed is a recurring pattern.
- **Deferred builds.** A decision entry that says a build was deferred to a fresh session,
  with no later entry picking it up.

## The brief

One screen, this shape, in this order. Skip a section if it is genuinely empty. Never pad
it.

```
Where things stand
- [3 lines max. Current quarter goal plus the two most active projects, real state.]

Needs a decision from you
- [Open follow-ups from the last 3 decision entries, each with why it is waiting.]

Stale / at risk
- [Passed deadlines, cold connections, uncommitted work.]

Next
[One action. One sentence of why, tied to a stated priority.]
```

Register: short sentences, bullets, no em dashes, no preamble, no "I've analyzed your vault
and here's what I found."

## Rules

- **Read-only.** No writes, no scaffolding, no backups. If something needs writing, say so
  and wait to be asked.
- **Never invent state.** If a mapped path is missing, say `<path> not found` in the brief.
  A missing project directory is information, not something to skip silently.
- **Cite the source** for anything non-obvious. "Per the 2026-08-13 decision entry", not
  "I believe".
- **Never quote financial figures or any secret.**
- **Log decisions.** When a call gets made during the session, offer to append it to the
  relevant decisions log, the vault root for business and config decisions, the project's
  own log for project-scoped ones.
- **Default Shift.** When a new manual task comes up, ask to what extent AI could be
  leveraged on it before assuming the old way.
- **Scope the read.** If the request is clearly narrow, read steps 1 and 2 plus only that
  project, then say so. The full map is for a cold start.

## Verification

Cold-test after any edit to this skill:

1. Run it. It should name the current goal and its deadline, surface at least one open
   follow-up, flag every dirty or untracked project directory, and flag any priority whose
   date has passed.
2. `git status --short` is identical before and after. Any diff is a bug, since this skill
   writes nothing.
3. Run it in a directory with no vault. Expected: it reports what is missing and stops. It
   must not fall back to interviewing or scaffolding.
4. Temporarily rename one mapped project directory and re-run. Expected: "not found" in the
   brief, not a silent skip.

That last test is the one that catches the real bug. A skill that silently skips a missing
path will happily report all-clear on a vault that has half fallen over.
