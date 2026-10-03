# Orchestration

How a request becomes work, and how that work gets routed.

## The ritual

Every substantial task runs through the same four steps before a single file changes.
The point is not ceremony. Each step exists because skipping it cost me something.

### 1. Sharpen the prompt

Run the request through a prompt optimizer first. Show the rewrite only if it changed
materially, otherwise proceed silently. Most of the value is in the agent noticing what
the request did not specify, before it starts guessing.

### 2. Interview, and the permissions pass

Ask enough questions that the goal, the scope, and the done-criteria are unambiguous.
Ten or more, mixing quick multiple choice with open questions. Surface tradeoffs instead
of picking silently.

In the same pass, enumerate every command the task will need that a permission classifier
tends to block: privilege escalation, container exec, package installs, pushes,
background daemons. Get them all approved in one batch, up front.

This second half is the part people skip, and it is the part that makes autonomous runs
actually work. Without it, an agent stalls in the middle of a task and hands you a command
to paste in by hand, which is exactly the interruption the automation was supposed to
remove. Approving up front is not a weaker gate. It is the same gate, moved to where it
does not break the run.

Carve-out: genuine production mutations stay separate. Deploys, migrations, campaigns,
anything touching money. Those get named individually and approved individually, every
time.

### 3. Choose the execution shape

| Shape | Use when |
|---|---|
| **Solo** | Surgical edits, production changes, anything where one wrong parallel write is unrecoverable |
| **Parallel subagents** | Independent read-heavy work. Searching several subsystems, reviewing several dimensions at once |
| **Scripted workflow** | Deterministic fan-out over a known work list. Migrations, audits, broad sweeps |
| **Off-quota executor** | Long, mechanical, fully decomposed, unattended |

Two rules that matter more than the table:

- **Delegation means Claude orchestrating Claude.** The off-quota harness is an executor,
  never a planner.
- **Match the tool to the job, not to the keyword.** A task that mentions "audit" is not
  automatically a twelve-agent fan-out. Over-firing agents on trivial edits is its own
  failure mode, and an expensive one.

### 4. Plan, approve, back up, execute

Write the plan. Wait for approval. Back up before writing. Then execute.

The backup step is non-negotiable and comes before the first write, not after the first
scare.

## Escape valves

A ritual with no escape valve gets abandoned wholesale the first time it is too heavy for
the task. So:

- **Short tasks:** do not auto-run the ritual. Ask first: "small task, full ritual, or
  just do it?"
- **`raw` prefix:** skip the ritual entirely and execute directly.

## Standing rules

These hold on every task, regardless of shape:

- **One task at a time.** Never batch. Ask before starting the next.
- **Production is gated every time.** A reminder is not approval. Previous approval for a
  similar thing is not approval for this thing.
- **Never patch production directly.** Backup, then patch, then apply, then verify, as
  scripts run by hand.
- **Read the decisions log first.** Never redo shipped work. Never reopen a settled
  decision without new information.
- **Verify live product facts.** Before recommending an API, a pricing tier, or a model,
  check that it currently exists. Do not assert it from training data. This has been wrong
  before, which is why it is a written rule and not a good intention.
- **End substantial work with the exact next step**, plus a paste-ready resume prompt.

## Model routing

The standing default is the strongest available model at high reasoning effort, with
multi-agent fan-out enabled for substantial tasks.

The deliberate exception: drop to a single agent for surgical, production, or financial
edits. Parallel agents are for covering ground. A one-line apply script against live data
is not ground to be covered, and the fan-out only adds ways to be wrong simultaneously.

Benchmark discipline: route on published first-party numbers. Figures that circulate
widely but appear on no primary source get discarded, however flattering. That rule caught
a pair of invented benchmark scores that were about to justify a model choice.

## Related

- [Architecture →](architecture.md) what the layers are
- [Vault structure →](vault-structure.md) what gets read before work starts
