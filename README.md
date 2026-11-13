# Harness Engineering Kit
> Reusable kit to apply the Engineering Harness methodology to any software project.
> Version: 1.5.0 | Updated: 2026-11-13 | [Changelog](CHANGELOG.md)

> **Version labels:** the short labels used across this kit (`v2`, `v3`, `v4`, e.g. in
> `prompts/07_upgrade_v3_to_v4.md`) are the same numbering as the kit version: **v2 = 1.2,
> v3 = 1.3, v4 = 1.4**. Wherever you read "v4", read "1.4" — a "v3→v4 upgrade" is a
> "1.3→1.4 upgrade". Details in the [Changelog](CHANGELOG.md).

---

## What is this kit?

This kit turns a "text generator" AI into a **responsible and autonomous engineer** that can work on a project persistently, verifiably, and without going off track.

It uses a **Workspace-based Lean architecture**: each task gets an isolated workspace with its briefing (`orden.md`) and working area (`session.md`). Sub-agents work with individual context and full code access. Results are archived automatically.

### Code engine and knowledge layer

**Source code** is served by a **code engine** (AST + semantic call resolution, reindexed
for free), or by none, depending on a project **profile** computed from the codebase's
size: **DOC** (0–20 files) installs no engine and the code is read directly; **MIXTO**
(21–100) makes the engine optional; **CODE** (>100) makes it mandatory.

The **knowledge layer** (`.harness/`, `docs/`, `plan/`) has no index: it is consulted
through `.harness/context/index.md`, the OKF brain if installed (see harness-okf below)
and Grep/Read. See `docs/KNOWLEDGE_GRAPHS.md` in an installed project for the routing details.

The **code engine** is selectable: `codebase-memory-mcp`, `codegraph` (alternative), or
`auto`, where the AI picks the better fit for the project. Only one runs at a time; the choice is
recorded in `.harness/PROFILE.md`.

### Companion project: harness-okf

[harness-okf](https://github.com/csalazart/harness-okf) is a sibling, standalone tool
that adds a persistent **domain knowledge base** (`.harness/llm-wiki/`) — concepts,
verified sources, playbooks. It doesn't require this kit, but installs under the same
`.harness/` container and never touches `context/`, `rules/`, or `STATE.md`. If installed,
its knowledge base becomes part of the knowledge layer and is consulted directly (see the
model above).

---

## Requirements

- **Node.js** installed globally (`node -v` to verify)
- Works on **Mac, Linux and Windows** — Node.js is the only dependency

> For Node.js projects: this kit recommends **pnpm** over npm for security and performance. The initialization prompt will detect npm usage and ask you how to proceed.

---

## Kit Contents

```
harness-kit-install/
├── README.md                              ← This file
├── README_ES.md                           ← Spanish version
├── CHANGELOG.md                           ← Release history
├── CHANGELOG_ES.md                        ← Spanish version
├── QUICKSTART.md                          ← Quick start in 5 steps
├── QUICKSTART_ES.md                       ← Spanish version
├── INSTALL.md                             ← Step-by-step install guide (Spanish)
│
├── prompts/                               ← Ready to copy-paste prompts
│   ├── 01_init.md                         ← Harness initialization (MAIN — Claude Code)
│   ├── 02_new_agent.md                    ← Create a new specialized agent
│   ├── 03_phase_handshake.md              ← Phase start protocol
│   ├── 04_phase_handoff.md                ← Phase end protocol
│   ├── 05_decision_resolver.md            ← Unlock pending critical decisions
│   ├── 06_migrate.md                      ← Migrate a project from the old Harness structure
│   └── 07_upgrade_v3_to_v4.md             ← Upgrade an existing v3 (Lean) install to v4
│
├── questionnaire/
│   └── project_discovery.md               ← Questions the AI asks if context is missing
│
├── templates/                             ← Templates for control files
│   ├── CLAUDE.md.template                 ← Master project file (Claude Code)
│   ├── AGENTS.md.template                 ← Universal protocol (all models)
│   ├── GEMINI.md.template                 ← Gemini-specific addendum
│   ├── STATE.md.template                  ← Backlog + decisions (source of truth)
│   ├── orden.md.template                  ← Task workspace briefing
│   ├── SESSION.md.template                ← Task workspace working area
│   ├── PROFILE.md.template                ← Project profile: DOC/MIXTO/CODE (v4)
│   ├── .mcp.json.template                 ← codebase-memory-mcp server declaration (v4)
│   ├── init.js.template                   ← Environment health check (node init.js)
│   ├── init.sh.template                   ← Bash-only alternative to init.js
│   ├── harness-start.js.template          ← Creates a task workspace
│   ├── harness-finish.js.template         ← Archives and closes a workspace
│   ├── harness-link-skills.js.template    ← Links .harness/skills/ into every detected agent (.claude, .gemini, .opencode, .cursor, .qwen, .kilocode, .kimi-code, .agents)
│   ├── harness-merge-instructions.js.template ← Merges kit sections into AGENTS.md / CLAUDE.md / GEMINI.md / … using <!-- harness-kit:ID --> markers, never overwriting the project's own content
│   ├── harness-agents.json                ← Agent matrix (paths, detection, instruction files) read by both scripts
│   ├── AGENT_POINTER.md.template          ← "Read AGENTS.md first" file for agents with their own instruction file (QWEN.md…)
│   ├── code-graph/                        ← Code-graph engine fragments (codebase-memory | codegraph | none): AGENTS.md routing, maintenance, .mcp.json entry
│   ├── index-code.ps1/sh.template         ← Reindex the code graph (codebase-memory-mcp, v4)
│   ├── .claude/
│   │   └── settings.json.template         ← Claude Code project configuration
│   ├── agents/
│   │   ├── AGENT_00_ORCHESTRATOR.template.md  ← Orchestrator agent
│   │   └── AGENT_SPECIALIST.template.md       ← Generic specialist agent
│   ├── context/                           ← .harness/context/ templates (index, activeContext, progress, …)
│   ├── docs/
│   │   ├── AGENTS_REFERENCE.md.template   ← Extended protocol reference
│   │   └── KNOWLEDGE_GRAPHS.md.template   ← Code engine + knowledge routing: profiles, cost, troubleshooting (v4)
│   └── skills/
│       ├── doc-code-audit/                ← Docs↔code cross-check skill (v4, CODE/MIXTO-con-MCP only)
│       ├── load-memory.md
│       └── update-memory.md
│
├── rules-library/                         ← Technology rules library
│   ├── agent-git-reviewer.md              ← Semantic and safe commits
│   ├── contract-solidity.md
│   ├── contract-rust-anchor.md
│   ├── frontend-nextjs.md
│   ├── frontend-react.md
│   ├── frontend-vue.md
│   ├── frontend-nuxt.md
│   ├── backend-nodejs.md
│   ├── backend-python.md
│   ├── backend-php-laravel.md
│   ├── backend-php-oop.md
│   ├── backend-php-symfony.md
│   ├── testing-jest.md
│   ├── testing-hardhat.md
│   ├── deploy-docker.md
│   ├── deploy-blockchain.md
│   ├── security-web3.md
│   └── security-web2.md
```

---

## How to use this kit

### Option A — New project from scratch
1. Copy `harness-kit-install/` to the root of your new project
2. Open a chat with Claude Code
3. Paste the content of `prompts/01_init.md`
4. The AI will ask questions if context is missing (via `questionnaire/project_discovery.md`)
5. Upon completion, you will have the full harness for your project

### Option B — Existing project without harness
1. Copy only `harness-kit-install/` to the repository
2. Use `prompts/01_init.md` — the AI will analyze what already exists and complete what is missing

### Option C — Add a new agent to an already harnessed project
1. Use `prompts/02_new_agent.md` with your project context
2. The AI will generate the agent file using `templates/agents/AGENT_SPECIALIST.template.md`

---

## The 5 Principles of the Harness

1. **Single Task** — One agent, one workspace, one set of acceptance criteria. Do not advance until they pass.
2. **Handshake** — On startup: run `node init.js` + read `STATE.md` + check active workspaces. Always.
3. **Hand-off** — On completion: fill `session.md` + run `harness-finish.js` + commit. Always.
4. **Orchestrator first** — The orchestrator coordinates and creates workspaces. Specialists execute.
5. **Verification before advancement** — If there is no command to confirm it works, it doesn't count as done.

---

## Compatibility

Designed to work with:
- **Claude Code** (Anthropic CLI) — primary use
- **Gemini CLI**, **OpenCode**, **Cursor**, **Codex**, **Qwen Code**, **Kilo Code**, **Kimi Code** —
  skills are linked into each agent's native folder and instruction files are merged with
  `<!-- harness-kit:ID -->` markers (agent matrix: `templates/harness-agents.json`)
- **Any LLM** with file access — using the prompts manually
