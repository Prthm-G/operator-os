# Operator OS

A personal AI operating system. This is the public face of a private vault I use to run
two businesses and my own engineering work on top of Claude Code.

I am Pratham Goel. I run [Skeure Education](https://education.skeure.com), an education
consulting business in Patiala, Punjab that counsels students into online UG and PG
programs at partner universities. Alongside it I build and operate **Auretris** (a
WhatsApp bot on the Meta Cloud API) and **Kuanli** (a WhatsApp CRM), both in production
handling real lead traffic.

This repo documents how that system is put together. It is not a starter kit and there is
nothing to install. It is a working record of an architecture, published because the
design decisions are the interesting part.

## The idea

Most "AI assistant" setups are one model in one chat window. That stops scaling the moment
you have work that outlives a session, work that has to run while you sleep, and work you
cannot afford to pay frontier prices for.

So the system is layered instead. Four runtimes, each doing only what it is actually good
at, with hard rules about which one is allowed to make decisions.

| Runtime | Role | Decides anything? |
|---|---|---|
| **Claude Code** | The brain. Plans, writes, reviews, orchestrates. | Yes. Only this. |
| **Hermes** | The caretaker. Always on, watches the vault between sessions. | No. Detection and validation only. |
| **DSH** | The executor. Unattended, off-quota, runs plans it did not write. | No. Executes a plan Claude made. |
| **Ollama** | Throwaway local inference. Offline and non-sensitive work. | No. Never sees real data. |

The rule that holds it together: **Claude plans, DSH executes, the small model never
plans.** Every layer that cannot reason well enough to be trusted with a decision is
structurally prevented from making one, rather than just asked nicely.

[Full architecture →](docs/architecture.md)

## What is actually in here

| Doc | What it covers |
|---|---|
| [architecture.md](docs/architecture.md) | The four runtimes, how they are isolated, what each may and may not touch |
| [orchestration.md](docs/orchestration.md) | Choosing between solo work, parallel subagents, and scripted workflows. The pre-work ritual. |
| [vault-structure.md](docs/vault-structure.md) | The file taxonomy, the wiki schema, why the decisions log matters more than the code |
| [publishing.md](docs/publishing.md) | The private-by-default pipeline that produced this repo |
| [engineering-notes.md](docs/engineering-notes.md) | Islands over a rewrite, what a motion pass has to hold, and measuring tooling before trusting it |
| [decisions.md](decisions.md) | Selected entries from the vault's decisions log |
| [skills/](skills/) | Two Claude Code skills, genericized |

## Three things I would argue for

**The decisions log is the highest-value file in the system.** Not the code. An
append-only record of what was decided, why, what else was considered, and what would
change my mind. Six months later the code tells you what happens; only the log tells you
why it was allowed to happen that way. Every agent reads it before touching anything.

**Detection and judgment belong to different agents.** The always-on caretaker finds
things: overdue dates, cold connections, uncommitted work, format drift. It is explicitly
forbidden from interpreting any of it. It writes a health file and stops. I read that file
and decide. Cheap agents that quietly start making calls are how automated systems go
wrong slowly enough that nobody notices.

**Isolation should be structural, not instructed.** The caretaker cannot see the finance
project because that path is masked out of its sandbox at the mount level, not because its
prompt says not to look. A prompt is a request. A mount is a fact.

## The publishing pipeline

This repo is generated. The vault it comes from is private and holds live business data,
so the pipeline that produces the public side is built to fail closed:

- **Allowlist, never denylist.** A denylist fails open. The day you add a folder nobody
  thought to exclude, it ships. An allowlist fails closed: an unlisted folder simply does
  not go anywhere.
- **Authored, not selected.** Public content is written into a dedicated directory. There
  is no glob over the private tree that could misfire.
- **A gate that aborts the push.** Every staged tree is scanned for credential shapes,
  forbidden literals, denied paths, and denied file types before anything leaves the
  machine. Any hit stops the publish.
- **Opt-in per item.** A decisions entry publishes when it is explicitly marked. A project
  publishes when it has a manifest. Silence means private.

[How it works →](docs/publishing.md)

## License

MIT. See [LICENSE](LICENSE).

Contact: [main@prathamgoel.com](mailto:main@prathamgoel.com) · [prathamgoel.com](https://prathamgoel.com)
