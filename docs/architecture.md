# Architecture

Four runtimes. The design question is not "which model is best" but "which layer is
allowed to decide anything."

## Why layers at all

A single frontier model in a chat window fails on three axes at once:

- **Persistence.** Work that outlives a session has nowhere to live.
- **Availability.** Nothing happens while you are asleep or away from the desk.
- **Cost.** Paying frontier prices for a file-count check is absurd, and doing it often
  enough that the absurdity matters is the normal case.

Each layer below exists because one of those three broke.

## The layers

### 1. Claude Code, the brain

Plans, writes code, reviews it, orchestrates the others. This is the only layer permitted
to make a decision, and everything else is arranged so that the others cannot quietly
start making them.

Runs at high reasoning effort by default, with multi-agent fan-out for wide tasks and a
deliberate drop to single-agent for surgical work. Fanning parallel agents across a
one-line patch to production is not thoroughness, it is noise with a blast radius.

### 2. Hermes, the caretaker

An always-on agent runtime, one systemd service per profile. It looks after the vault
between sessions.

What it does:
- Rewrites a health file every morning: overdue dates, cold connections, uncommitted work
- Scans logs for patterns
- Validates that files still match their documented format
- Counts and inventories things on request
- Takes async work through a kanban board

What it is structurally prevented from doing:
- Interpreting anything it finds
- Authoring or editing any page in the knowledge wiki
- Committing or pushing
- Seeing the finance project or any secrets directory, both masked out of its sandbox at
  the mount level

That last distinction is the important one. The caretaker cannot read the finance data
because the path does not exist inside its sandbox, not because its instructions say not
to. An instruction is a request that a model can misread, be argued out of, or simply
forget under context pressure. A mount is a fact about the world.

**The heuristic:** if the errand you are about to hand it contains the words "decide",
"assess", "judge", or "should", it is not an errand. Do it yourself.

### 3. DSH, the off-quota executor

A separate harness running as its own unix user, on its own disk. It executes plans it did
not write, unattended, without consuming the interactive quota.

The router rule is absolute: **Claude plans, DSH executes, the small model never plans.**
DSH is reached for when a job is long, mechanical, already fully decomposed, and does not
need anyone to exercise judgment halfway through. It can delegate back to a real Claude
subagent for a step that needs reasoning, with permissions defaulted to read-and-analyze
rather than edit-and-run.

### 4. Ollama, throwaway inference

A small local model on loopback. Offline drafts, throwaway classification, anything where
the answer is disposable. It never sees business data. Not because it would leak it, but
because the discipline of "this tier never touches real data" is worth more than any
individual convenience of breaking it.

## What this buys

| Problem | Which layer solves it |
|---|---|
| Work outliving a session | The vault plus the decisions log |
| Nothing happening overnight | Hermes, scheduled and always on |
| Long mechanical jobs burning quota | DSH |
| Cheap disposable inference | Ollama |
| Anything requiring judgment | Claude Code, and only Claude Code |

## Boundaries that are enforced, not requested

- Production changes ship as backup, patch, apply, verify scripts run by hand. Never a
  direct edit against a live system.
- A rollback path exists before anything is touched, not after something breaks.
- Secrets live in one env file outside every repo, and are never pasted into a prompt, a
  commit, a log, or an external service.
- The finance project and the multi-tenant platform are separate git repos, gitignored
  from the vault, and masked from the caretaker's sandbox. Three independent mechanisms,
  because one mechanism is a single point of failure.

## Related

- [Orchestration →](orchestration.md) how work is routed between these layers
- [Vault structure →](vault-structure.md) what the brain actually reads
- [Publishing →](publishing.md) how anything gets out
