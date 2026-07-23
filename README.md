<div align="center">

# Continuity

**The model starts fresh. The agent does not.**

Persistent memory, beliefs, governance, re-entry, self-model, and compaction — one continuity layer across OpenCode, Codex, Claude Code, local models, and future hosts.

<p>
  <img src="https://img.shields.io/badge/TypeScript-5.x-3178c6?logo=typescript&logoColor=white" alt="TypeScript 5">
  <img src="https://img.shields.io/badge/PostgreSQL-14%20%7C%2016-336791?logo=postgresql&logoColor=white" alt="PostgreSQL 14 and 16">
  <img src="https://img.shields.io/badge/SQLite-core%20mode-003B57?logo=sqlite&logoColor=white" alt="SQLite core mode">
  <img src="https://img.shields.io/badge/platform-Windows%20%7C%20Linux%20%7C%20macOS-4b5563" alt="Windows Linux macOS">
  <a href="LICENSE"><img src="https://img.shields.io/badge/license-MIT-22c55e" alt="MIT License"></a>
</p>

[What is this?](#what-is-this) · [Why it matters](#why-it-matters) · [How it works](#how-it-works) · [Quick start](#quick-start) · [Systems](#systems) · [Host support](#host-support) · [Contributing](#contributing)

</div>

---

## What is this?

Continuity is infrastructure that gives AI coding agents durable memory, structured beliefs, operational state, and governed recall across sessions. It is not a chat-history dump or a thin vector-store wrapper. It is a runtime that records what happened, retrieves what matters, reconstructs working state, and tells a new agent how to continue safely.

Without continuity, every session starts from zero. The agent re-explores the codebase, re-asks questions you already answered, and repeats mistakes it made yesterday. Important decisions disappear into old transcripts, project state drifts, and every fresh context window spends tokens rebuilding a partial understanding of the work.

**Continuity turns that into infrastructure.**

## Why it matters

| Without Continuity | With Continuity |
|---|---|
| Fresh-session amnesia | Structured re-entry briefing on arrival |
| Repeated mistakes across sessions | Durable lessons and failure evidence |
| Context pressure in long sessions | Compaction, budgeting, and checkpoints |
| Flat retrieval (nearest vectors) | Hybrid search with entity, type, importance, and relationship signals |
| No trust signal on recalled data | Provenance, quality scoring, and governance reports |
| Project state guesswork | Append-only operational events and current-state projection |
| One host, one model | One layer across every host and model |

## How it works

```
Session 1          Session 2          Session 3
┌──────────┐       ┌──────────┐       ┌──────────┐
│ Work     │       │ Briefed  │       │ Knows    │
│ happens  │──────▶│ on       │──────▶│ your     │
│          │       │ arrival  │       │ preferences│
│ Packets  │       │ Continues│       │ Direct,  │
│ captured │       │ where    │       │ no       │
│          │       │ left off │       │ ramp-up  │
│ Beliefs  │       │ Confidence│      │ Avoids   │
│ forming  │       │ growing  │       │ past     │
│          │       │          │       │ failures │
└──────────┘       └──────────┘       └──────────┘
      │                  │                  │
      └──────────────────┴──────────────────┘
                         │
          ┌──────────────────────────────┐
          │        Continuity Layer      │
          │                              │
          │  Memory · Beliefs · Re-entry │
          │  Governance · Self-model     │
          │  Compaction · AgentBook      │
          └──────────────────────────────┘
                         │
                    ┌────┴────┐
                    │ Storage │
                    │ PG │ SQ │
                    └─────────┘
```

The continuity layer sits between the agent host and storage. Every host — OpenCode, Codex, Claude Code, or a future integration — connects through the same runtime. Memories, beliefs, and operational state are shared. Switching hosts or models doesn't reset the agent's understanding.

## Systems

| System | What it provides |
|---|---|
| **Memory engine** | Durable memories, lessons, transcripts, hybrid search, related-memory traversal, distillation, and embedding backfill |
| **AgentBook** | Append-only project events, rolling summaries, explicit rules, current-state projection, and a turn-1 front page |
| **Re-entry** | Identity, project state, constraints, goals, checkpoints, relevant memories, advisories, and source-aware injection |
| **Context control** | Token-pressure measurement, compaction, context cache, selective fetch, checkpoints, and recovery |
| **Living State** | Experience packets, capability confidence, belief candidates, promoted knowledge, and preview/debug tools |
| **Governance** | Recall-quality scoring, provenance checks, duplicate detection, safe merge, archive candidates, and continuity reports |
| **Work continuity** | Goals, checkpoints, decision/error retrieval, work-ledger survival, causal stitching, and session handoff |
| **Self-model** | Per-capability confidence tracking, evidence counts, drift warnings, and uncertainty quantification |

### The tool surface

The full PostgreSQL runtime registers **50+ tools** across memory, governance, living state, AgentBook, continuity, checkpoints, goals, context cache, and review. The native Claude Code plugin exposes **82 tool entries** including 12 slash commands, 3 subagents, and 3 skills.

## Quick start

**Runtime requirement:** Node.js `^22.22.2`, `^24.15.0`, or `>=26.0.0`

### 1. Configure

Create a `.env` in your project. SQLite is the smallest local setup:

```dotenv
CSM_DATABASE_PROVIDER=sqlite
CSM_SQLITE_PATH=.data/continuity.db
CSM_EMBEDDING_PROVIDER=ollama
OLLAMA_HOST=http://localhost:11434
```

For the complete feature path, use PostgreSQL:

```dotenv
CSM_DATABASE_PROVIDER=postgres
CSM_DATABASE_URL=postgresql://user:pass@localhost:5432/continuity
CSM_REQUIRE_EXPLICIT_DATABASE_URL=true
CSM_EMBEDDING_PROVIDER=ollama
OLLAMA_HOST=http://localhost:11434
```

### 2. Initialize storage

```bash
npx --yes --package=opencode-cross-session-memory@1.0.0 csm-init
```

### 3. Verify

```bash
npx --yes --package=opencode-cross-session-memory@1.0.0 csm-doctor --online
```

### 4. Connect a host

<details>
<summary><b>Claude Code</b> (native plugin — full surface)</summary>

Install the native plugin for the complete experience — lifecycle hooks, MCP tools, slash commands, subagents, and skills:

```
/plugin marketplace add <repo>/.agents/plugins
/plugin install cross-session-memory
```

See [Claude Code Installation](docs/CLAUDE_INSTALLATION.md) for details.
</details>

<details>
<summary><b>Codex</b> (MCP bridge)</summary>

Add to `.codex/config.toml`:

```toml
[mcp_servers.cross_session_memory]
command = "npx"
args = ["--yes", "--package=opencode-cross-session-memory@1.0.0", "csm-mcp"]
cwd = "."
startup_timeout_sec = 30
tool_timeout_sec = 120
required = true
```
</details>

<details>
<summary><b>OpenCode</b> (plugin)</summary>

Add to `opencode.json`:

```json
{
  "$schema": "https://opencode.ai/config.json",
  "instructions": ["AGENTS.md", "AGENTBOOK_STATE.md"],
  "plugin": [["opencode-cross-session-memory@1.0.0", {}]]
}
```
</details>

## Host support

| Host | Integration | Surface |
|---|---|---|
| **Claude Code** | Native plugin (hooks + MCP + commands + agents + skills) | Full: 82 tool entries, 12 commands, 3 agents, 3 skills |
| **Codex** | Native plugin (hooks + MCP) | Full: 50+ tools, lifecycle hooks |
| **OpenCode** | Plugin with lifecycle hooks | Full: 50+ tools, automatic hooks |
| **Any MCP client** | MCP server | Tools only (no lifecycle hooks) |

All hosts share one continuity layer. Run Claude Code and Codex simultaneously on the same project — transport isolation (`csm-<host>-<hash>`) prevents collisions.

## Database modes

| Capability | PostgreSQL | SQLite |
|---|:---:|:---:|
| Core save, search, list, delete, lessons, transcripts | Yes | Yes |
| AgentBook events, state, summaries, rules | Yes | Yes |
| Vector backfill and advanced maintenance | Yes | No |
| Belief promotion and Living State analysis | Yes | No |
| Context cache, checkpoints, goals, review tools | Yes | No |

SQLite is a smaller local mode. PostgreSQL is the target when the entire continuity stack is required.

## Architecture

The runtime is intentionally layered:

- **AgentBook** answers: *"What is happening in this project right now?"*
- **Memory** answers: *"What has been learned across sessions?"*
- **Re-entry** answers: *"What does this agent need before it acts?"*
- **Governance** answers: *"Why should this information be trusted?"*
- **Context control** answers: *"What should remain active, cached, compacted, or fetched later?"*
- **Self-model** answers: *"What is the agent confident about, and where is it uncertain?"*

No system operates in isolation. Re-entry draws from memory, beliefs, governance scores, and AgentBook state. Governance watches the quality of everything memory stores. The self-model is updated by experience packets and informs belief promotion. Compaction respects all of them.

## Development

```bash
git clone https://github.com/NovasPlace/continuity.git
cd continuity
npm ci
npm run build
npm run typecheck
npm run lint:src
npm test
```

The source lint gate is locked at zero errors and a bounded warning baseline. The test suite covers 1,500+ automated tests.

## Contributing

Contributions are welcome. Please open an issue to discuss significant changes before submitting a PR.

- Each change should be behavior-preserving where possible
- All PRs must pass typecheck, build, lint, and tests
- New tools or subsystems should include tests and documentation

## Security

Memory systems can retain sensitive project context. Review [SECURITY.md](SECURITY.md) before using Continuity with private source code, credentials, customer data, or regulated information.

## License

MIT. See [LICENSE](LICENSE).

---

<div align="center">
<sub>Built by <a href="https://github.com/NovasPlace">NovasPlace</a> — because agents deserve to remember.</sub>
</div>
