# AI agents

The lab is administered with AI agents, inside the limits described in the
[security model](security-model.md). The principle: one agent acts, another
one checks, and I decide.

## Roles

| Who | Role | In practice |
| --- | --- | --- |
| **Claude Code** | Orchestrates | Plans, writes code, delegates, reviews, and explains the result to me in plain words |
| **Codex** | Executes | Gets bounded, verifiable tasks: reading, searching, inventories, scoped changes, targeted tests |
| **Observer** | Checks | Reads the real state of every machine and reports what works, what broke and what changed |
| **Me** | Decides | Visual acceptance, production releases and firewall changes need my confirmation |

Every task follows the same loop: need → plan → code → validation →
acceptance → memory.

## Delegation

Claude Code is the more expensive agent to run, so work that can be checked
afterwards goes to Codex: reading a codebase, listing things, small bounded
changes. Claude writes the brief and reviews the result. When Codex runs, a
single background wait reports when it is done, instead of polling.

## Observer: see everything, touch nothing

When an agent says "it's fixed", that is not taken at face value: it asks
Observer, which looks at the machines.

- **Everywhere, read-only.** Observer reads services, updates, disks,
  network, firewall state and the hypervisor's API. On each server, its SSH
  key can only run one predefined collector that reads and changes nothing.
- **A narrow front door.** Agents on the dev machine reach Observer through
  one account whose key is pinned to a dispatcher. The dispatcher accepts a
  strict JSON request and checks the target, the operation and the
  arguments against allowlists. The caller cannot choose a destination,
  command, path or credential.
- **Summary first.** A snapshot gives each machine a status (`ok`,
  `degraded`, `unreachable`, `unknown`) with short alerts. From there, one
  targeted operation, then bounded logs if needed. A delta call shows what
  changed since the last one: a service down, a rule added, an update waiting.
- **Honest answers.** Results carry their source, method and timestamp, and
  answers label what is a live fact, what comes from documentation, what is
  inferred and what is unknown. Live evidence wins over documentation.
- **Isolated.** Observer has its own network zone. It holds no write access
  of its own: the rare changes that go through it require a manifest signed
  with a key that never leaves my Mac.

## Shared memory

Agents start every session from a versioned Obsidian vault: project pages,
machine pages, decisions, runbooks, and a log. After real work (a release, an
infrastructure change), the agent writes a short report and updates the pages
it touched. The next session, with the same agent or another one, picks up
from there instead of starting from zero.

Before acting on a machine, an agent reads that machine's page. When the vault
and the live system disagree, the live system wins and the vault gets fixed.

The structure, scripts and hooks are public:
[second-brain](https://github.com/Antoine-DOT/second-brain).
