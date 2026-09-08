# Quickstart — Harness in 5 steps

## For a new project

### Step 1 — Requirements and Copy the kit
Make sure you have **Node.js** installed (`node -v` to verify).

```bash
cp -r harness-kit-install/ /your/new/project/harness-kit
```

### Step 2 — Open Claude Code in your project
```bash
cd /your/new/project
claude
```

### Step 3 — Paste the initialization prompt
Copy the content of `harness-kit-install/prompts/01_init.md` and send it to the chat.

### Step 4 — Answer the questions
The AI will ask you about your project (name, type, stack, modules, status).
The questions are documented in `harness-kit-install/questionnaire/project_discovery.md` as a reference.
If you already have documentation (`README.md`, `package.json`, etc.), tell it — it will analyze it directly.

### Step 5 — The harness is ready
Upon completion you will have in the root of your project:
```
CLAUDE.md                          ← context for Claude Code
AGENTS.md                          ← universal protocol (all models)
init.js                            ← environment health check (node init.js)
harness-kit-install/                       ← full kit (do not delete, prompts reference it)
scripts/
├── harness-start.js               ← creates a task workspace
└── harness-finish.js              ← archives and closes a workspace
.harness/
├── STATE.md                       ← backlog + decisions (source of truth)
├── PROFILE.md                     ← project profile: DOC/MIXTO/CODE, drives which graphs get installed
├── rules/
│   ├── orchestrator.md            ← orchestrator agent rules
│   ├── git-reviewer.md            ← commit reviewer agent
│   └── {module}.md                ← one per project area (backend, frontend, etc.)
├── workspaces/                    ← temporary task workspaces
└── logs/
    └── SUMMARY.md                 ← reverse-chronological history (auto-generated)
graphify-out/                      ← knowledge graph (docs, .harness/, plan/) — partially versioned
.mcp.json                          ← codebase-memory-mcp server, only if the profile requires it
```

> **Dual knowledge graph (v4):** `graphify` (knowledge) always gets installed;
> `codebase-memory-mcp` (code) only if the project's profile is CODE, or MIXTO
> and you opt in. See `docs/KNOWLEDGE_GRAPHS.md` once installed for the full
> routing table between the two.

---

## To launch a task with a sub-agent

```
1. Open the chat (CLAUDE.md already loaded)
2. Say: "I want to work on [task]"
3. The AI (Orchestrator) runs: node scripts/harness-start.js --task task_name
   → Automatically creates .harness/workspaces/task_name/orden.md + session.md
4. The AI (Orchestrator) completes the acceptance criteria in the already-created orden.md
5. Open a new session for the specialist and say:
   "Read .harness/rules/{area}.md and execute the task in .harness/workspaces/task_name/orden.md"
6. When the specialist agent finishes, the orchestrator closes:
   node scripts/harness-finish.js --task task_name
   → Archives the session, updates STATE.md and appends entry to logs/SUMMARY.md
   ⚠️  After archiving, physically removes .harness/workspaces/{task}/
      Make sure everything is committed before running this step.
```

---

## To add a new agent

```
1. Open the chat with the project context (CLAUDE.md already loaded)
2. Paste prompts/02_new_agent.md
3. Indicate: agent name, area, dependencies
4. The AI creates the file in .harness/rules/
```

---

## Estimated setup time

| Project | Setup time |
|----------|----------------|
| Without previous documentation | 45–90 minutes |
| With existing docs | 20–40 minutes |
| Only adding a new agent | 10–15 minutes |
