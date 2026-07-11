# Harness Engineering Kit
> Reusable kit to apply the Engineering Harness methodology to any software project.
> **Version: 3.0** | Updated: 2026-07-11 | License: [MIT](LICENSE)

📄 Versión en español: [README_ES.md](README_ES.md)

---

## What is this kit?

This kit turns a "text generator" AI into a **responsible and autonomous engineer** that can work on a project persistently, verifiably, and without going off track.

It uses a **Workspace-based Lean architecture**: each task gets an isolated workspace with its briefing (`orden.md`) and working area (`session.md`). Sub-agents work with individual context and full code access. Results are archived automatically. A **persistent memory layer** (`.harness/context/`, with its own navigation index) keeps session state and stable project knowledge across sessions and across AI tools.

### What's new in v3

- **Indexed memory layer**: `.harness/context/index.md` maps the whole memory (what to read and when — FAST vs FULL LOAD); `init.js` verifies all 7 context files.
- **Universal protocol first**: `AGENTS.md` is the single source of the working protocol; `CLAUDE.md`/`GEMINI.md` just import it (`@AGENTS.md`). The installer **merges** — it never overwrites sections owned by other tools.
- **Clean separation of concerns**: domain knowledge is no longer part of this kit — it lives in the sibling tool [harness-okf](https://github.com/csalazart/harness-okf) (see below).
- New prompts: `06_migrate.md` (migrate old harness structures to Lean v3).

---

## Requirements

- **Node.js** installed globally (`node -v` to verify)
- Works on **Mac, Linux and Windows** — Node.js is the only dependency

> For Node.js projects: this kit recommends **pnpm** over npm for security and performance. The initialization prompt will detect npm usage and ask you how to proceed.

---

## Kit Contents

```
harness-kit-install/
├── README.md / README_ES.md               ← This file / Spanish version
├── QUICKSTART.md / QUICKSTART_ES.md       ← Quick start in 5 steps
├── INSTALL.md                             ← Installation guide
├── LICENSE                                ← MIT
│
├── prompts/                               ← Ready to copy-paste prompts
│   ├── 01_init.md                         ← Harness initialization (MAIN)
│   ├── 02_new_agent.md                    ← Create a new specialized agent
│   ├── 03_phase_handshake.md              ← Session start protocol
│   ├── 04_phase_handoff.md                ← Session end protocol
│   ├── 05_decision_resolver.md            ← Unlock pending critical decisions
│   └── 06_migrate.md                      ← Migrate old harness structures to Lean v3
│
├── questionnaire/
│   └── project_discovery.md               ← Questions the AI asks if context is missing
│
├── templates/                             ← Templates for control files
│   ├── AGENTS.md.template                 ← Universal protocol (all models) — source of truth
│   ├── CLAUDE.md.template / GEMINI.md.template ← Per-tool files (import @AGENTS.md)
│   ├── STATE.md.template                  ← Backlog + decisions (source of truth)
│   ├── orden.md.template / SESSION.md.template ← Task workspace files
│   ├── init.js.template / init.sh.template    ← Environment health check
│   ├── harness-start.js.template          ← Creates a task workspace
│   ├── harness-finish.js.template         ← Archives and closes a workspace
│   ├── harness-link-skills.js.template    ← Links .harness/skills to each AI tool
│   ├── context/                           ← Persistent memory (7 files + index.md)
│   ├── skills/                            ← load-memory, update-memory (+ graphify, optional)
│   ├── .claude/settings.json.template     ← Claude Code project configuration
│   └── agents/                            ← Orchestrator + generic specialist templates
│
└── rules-library/                         ← Technology rules library
    (git-reviewer, solidity, rust-anchor, nextjs, react, vue, nuxt, nodejs,
     python, laravel, php-oop, symfony, jest, hardhat, docker, blockchain,
     security-web3, security-web2)
```

---

## How to use this kit

### Option A — New project from scratch
1. Clone or copy this repo next to (or inside) your new project
2. Open a chat with your AI agent (Claude Code, Gemini, Cursor…) at the project root
3. Paste the content of `prompts/01_init.md`
4. The AI will ask questions if context is missing (via `questionnaire/project_discovery.md`)
5. Upon completion, you will have the full harness for your project

### Option B — Existing project without harness
1. Same as Option A — the AI analyzes what already exists and completes what is missing, merging (never overwriting) existing `AGENTS.md`/`CLAUDE.md` files

### Option C — Add a new agent to an already harnessed project
1. Use `prompts/02_new_agent.md` with your project context

### Option D — Full kit: harness-kit + harness-okf (recommended)

The complete system has **two independent, compatible tools** sharing the `.harness/` ecosystem:

| Tool | Responsibility | Installs |
|---|---|---|
| **harness-kit** (this repo) | Workflow: tasks, session state, agent roles, rules | `.harness/context/`, `workspaces/`, `rules/`, `STATE.md` |
| **[harness-okf](https://github.com/csalazart/harness-okf)** | Knowledge: a self-contained OKF brain of the project's domain (sources, concepts, playbooks) | `.harness/llm-wiki/`, `okf-*` skills |

To install the full kit:
1. Install harness-kit first (Option A/B above)
2. Then follow `prompts/01_init.md` of [harness-okf](https://github.com/csalazart/harness-okf) — it detects the existing `.harness/` and adds only its own pieces (never touches yours)

> Either order works, and each tool is fully usable on its own.
> Boundary: `context/` = where we are (session) · `llm-wiki/` = what we know (domain).

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
- **Gemini CLI** — via `GEMINI.md` → `@AGENTS.md`
- **Cursor / Windsurf / opencode** — compatible via `AGENTS.md`
- **Any LLM** with file access — using the prompts manually

---

## License

[MIT](LICENSE) — free to use, modify and redistribute.
