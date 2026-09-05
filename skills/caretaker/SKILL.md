---
name: caretaker
description: Hand a small, mechanical vault errand to the always-on caretaker agent instead of doing it inline. Use for cheap detection, counting, log-scanning, and format validation. Do NOT use for anything needing judgment, interpretation, or a write into the knowledge wiki or project code.
---

> Published from a private vault. Machine paths and profile names are replaced with
> placeholders. The division of labour is the part worth copying.

# The caretaker bridge

There is a second agent on this machine: an always-on runtime that looks after the vault
between sessions. It is cheap, it is awake when you are not, and it is deliberately not
allowed to interpret anything.

**You are the brain. It is the caretaker.** Hand it work. Do not hand it decisions.

## Why the split exists

An always-on agent accumulates authority nobody granted it. It runs unattended, so its
output is rarely reviewed line by line, and each small judgment it gets away with becomes
the precedent for the next one. Eventually it is making calls that nobody decided it should
make, and the drift is invisible because every individual step looked reasonable.

The fix is not a better prompt. It is a job description narrow enough that judgment never
comes up: **detect, count, validate, report.** Everything it produces is a finding, never a
conclusion. The health file it writes every morning says so in its own header, so whoever
reads it knows nothing in it has been acted on.

## When to use it

Good errands. Mechanical, verifiable, no judgment:

- counting or inventorying (`how many project folders lack a README?`)
- scanning logs for a pattern
- checking that a file's format matches its documented shape
- re-running one of its own scheduled checks on demand
- confirming a claim about the filesystem before you act on it

Do **not** hand it:

- anything that decides what a claim means, or how to resolve a contradiction
- authoring or editing any page in the knowledge wiki
- project code, migrations, deploys, or production systems
- anything under a masked path (it physically cannot see those)

**The heuristic:** if you are about to write the errand and it contains the word "decide",
"assess", "judge", or "should", it is not an errand. Do it yourself.

## Three lanes

**Synchronous.** One prompt, one answer, straight to stdout. No banner, no session id. Give
it absolute paths: its sandbox mounts the vault at the same real path, so paths mean the
same thing on both sides.

**Asynchronous.** File a task on a durable board with an assignee. A dispatcher claims it
within about a minute, runs it in the vault, and records the result as a comment. The board
survives restarts, so a task filed now is still there next session.

The worker starts with **no** knowledge of your conversation. Everything it needs goes in
the task body. Writing a genuinely self-contained brief is most of the work, and the
discipline is worth it on its own: an errand you cannot describe standalone is usually an
errand that secretly needs judgment.

**Messaging.** Registered as an MCP server, exposing send, read, list, and poll tools for
reaching the operator on a chat platform mid-task.

## Reading what it found on its own

It runs several scheduled checks. Two outputs matter:

- **The health file**, rewritten every morning: overdue dates, cold connections,
  uncommitted work. Read it at session start. Detection only.
- **The escalation queue.** Anything it could not do itself lands as a task assigned to a
  profile that does not exist, so it never auto-runs. Those wait for a human or for you.
  Drain that queue when you start work.

That second mechanism is worth stealing. An agent that hits something needing judgment
should have exactly one move available: put it in a queue that only a decision-maker
drains. Not retry, not escalate to a bigger model, not do its best. Queue it and stop.

## Isolation

Sensitive paths are masked out of its sandbox at the mount level, not merely excluded by
instruction.

This distinction is the whole design. An instruction is a request a model can misread, be
argued out of, or lose under context pressure. A mount is a fact about the world. If the
consequence of a boundary being crossed is serious, the boundary belongs in the filesystem,
not in the prompt.
