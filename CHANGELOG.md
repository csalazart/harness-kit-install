# Changelog

All notable changes to this project are documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project uses a versioning scheme inspired by [Semantic Versioning](https://semver.org/).

> **On version numbers:** this file was written retroactively from `git log`, since the
> project didn't keep a changelog before. The first commit here was labeled `3.0`, but it
> shipped the same content the private development repo (`harness-kit`) still called `2.0`
> at the time — a historical mislabeling. It is renumbered below as `1.3.0`. `1.1.0` and
> `1.2.0` never had a public release here (they existed only in the private repo, before
> this repo's first sync) and are listed for context. `4.0` (briefly used for the
> dual-graph release) was already self-corrected to `1.4.0` before this file existed, so it
> needed no change. Going forward, bump this file on every sync from `harness-kit` instead
> of only the `Version:` line in the README.
>
> **Short labels = kit versions.** Filenames and docs also use the short labels
> **v1/v2/v3/v4** (e.g. `prompts/07_upgrade_v3_to_v4.md`). They predate the `1.x`
> numbering but are the same lines of work, so they map one-to-one:
>
> | Label | Kit version | What it is |
> |---|---|---|
> | v1 | 1.0 | Initial release (agents, async logs, `git-reviewer`) — there is no 1.1 |
> | v2 | 1.2 | Lean workspace architecture |
> | v3 | 1.3 | graphify + contextual memory |
> | v4 | 1.4 | Dual knowledge graph |
>
> So a "v3→v4 upgrade" is a "1.3→1.4 upgrade". Filenames keep the short label so that
> existing links and already-installed projects keep working.

## [Unreleased]

## [1.4.2] - 2026-10-15

Multi-agent skill linking and marker-based instruction merging.

### Added
- `templates/harness-agents.json`: agent matrix (detection paths, native skills folder,
  instruction file) for Claude Code, Gemini CLI, OpenCode, Cursor, Codex, Qwen Code,
  Kilo Code, Kimi Code and the open `.agents` standard.
- `templates/harness-merge-instructions.js.template`: merges the kit's sections into
  `AGENTS.md`, `CLAUDE.md`, `GEMINI.md` and other agents' instruction files using
  `<!-- harness-kit:ID -->` … `<!-- harness-kit:ID end -->` markers. Never overwrites
  project content: appends missing blocks, updates unedited ones, respects hand edits,
  backs up to `.harness/backups/`, and adopts pre-marker sections with `--adopt`.
- `templates/AGENT_POINTER.md.template` for agents with their own instruction file.
- **CodeGraph as an alternative code-graph engine** (`@colbymchenry/codegraph`, MCP
  `codegraph serve --mcp`, tool `codegraph_explore`, index in `.codegraph/`). The user picks
  `codebase-memory`, `codegraph` or `auto` (the AI decides with explicit criteria, see
  `01_init.md` phase B.6.b); only one engine runs at a time. New `templates/code-graph/`
  (per-engine routing/maintenance fragments, `.mcp.json` entries, `engines.json`).
- `{{include:...}}` directive in `harness-merge-instructions` and `CODE_ENGINE` variable, so the
  engine-dependent blocks of `AGENTS.md` are rendered per engine.

### Changed
- `harness-link-skills.js` now links `.harness/skills/` into every agent detected in the
  project (was Claude Code, Gemini and `.agents` only); new `--list`, `--agents`, `--all`,
  `--dry-run`, `--copy`, `--source`, `--root` options; never overwrites an existing folder.
- `AGENTS.md.template`, `CLAUDE.md.template` and `GEMINI.md.template` wrap each kit-managed
  section in `harness-kit` markers.
- `init.js` reads the agent matrix to report which agents have skills available.
- `prompts/01_init.md` (sections 1, 3, 5, 9, 10) and `prompts/07_upgrade_v3_to_v4.md` use
  the new scripts instead of copying instruction files by hand.
- `Kit-Version` stamp bumped to `1.4.2`; version banners updated in `README.md`,
  `README_ES.md` and `INSTALL.md`.
- `.harness/PROFILE.md` records `Motor de código`, how it was selected and why; `init.js`/`init.sh`,
  `index-code.*`, the `doc-code-audit` skill, `docs/KNOWLEDGE_GRAPHS.md` and prompt 07 are
  engine-aware. Projects without the field keep working as `codebase-memory`.

## [1.4.1] - 2026-10-15

Documentation and version-stamp patch release. No structural changes to the kit:
projects installed with 1.4.0 keep working and need no upgrade.

### Changed
- Documented that the short `v1`–`v4` labels are the kit versions (v2 = 1.2, v3 = 1.3,
  v4 = 1.4); the README, this changelog and prompt 07 now say so.
- `Kit-Version` stamp bumped to `1.4.1` in `templates/PROFILE.md.template`,
  `prompts/01_init.md` and `prompts/07_upgrade_v3_to_v4.md`.
- `prompts/01_init.md`: the "needs a targeted upgrade" check now compares against
  `1.4.0` (the first dual-graph release) explicitly, so a `1.4.x` patch never triggers
  an upgrade.
- Version banner updated to 1.4.1 in `README.md`, `README_ES.md` and `INSTALL.md`.

## [1.4.0] - 2026-09-08

Internally referred to as architecture generation **v4 (dual knowledge graph)**.

### Added
- **Dual knowledge graph:** two graphs with disjoint domains — `codebase-memory-mcp`
  (source code: AST + semantic call resolution) and `graphify` (knowledge layer:
  `.harness/`, `docs/`, `plan/`).
- Automatic project profiling (DOC / MIXTO / CODE) based on codebase size —
  `templates/PROFILE.md.template`.
- `graphifyignore-library/`: per-stack `.graphifyignore` fragments (nodejs, nextjs, nuxt,
  vue-react, python, php, rust, solidity-hardhat, docker, cloudflare, common
  build/linting), replacing the old monolithic `.graphifyignore.template`.
- `templates/.mcp.json.template` (declares the `codebase-memory-mcp` server).
- `templates/docs/KNOWLEDGE_GRAPHS.md.template` (routing/cost/troubleshooting guide for
  the dual-graph model).
- `doc-code-audit` skill (docs↔code cross-check, CODE/MIXTO-with-MCP profiles only).
- `index-code.ps1` / `index-code.sh` scripts to reindex the code graph.
- `prompts/07_upgrade_v3_to_v4.md`: upgrade path for existing v3 (Lean) installs,
  without reinstalling from scratch.
- `harness-okf` companion project documented in README/README_ES (persistent domain
  knowledge base under `.harness/llm-wiki/`).

### Fixed
- Initial-install fixes validated upstream before this sync (see `01_init.md` and
  `07_upgrade_v3_to_v4.md` changes).

## [1.3.0] - 2026-07-11

Initial public release of this repository.

*(Originally tagged `3.0` here — the same sync point the private `harness-kit` repo's own
README still called `2.0` at the time. See the note above.)*

### Added
- **graphify** integrated as the project's knowledge graph (`.harness/`, `docs/`,
  `plan/`), with query/update scripts and a dedicated skill.
- `harness-okf` companion project (domain knowledge base under `.harness/llm-wiki/`),
  indexed by graphify.
- Skill-linking script to install skills into the active AI agent.
- `rules-library/` updates made graphify-aware.

## [1.2.0] - 2026-05-22 (pre-dates this repository)

Developed in the private `harness-kit` repo before this repo's first public sync; listed
here for continuity.

### Added
- **Lean workspace architecture:** each task gets an isolated workspace with its
  briefing (`orden.md`) and working area (`session.md`); results archived automatically.
- Protocol prompts: phase handshake, phase handoff, pending-decision resolver, migration
  from the old structure.
- Multi-AI support: Gemini-specific addendum (`GEMINI.md.template`).
- `rules-library/` expanded: frontend, backend, testing, deploy, web2/web3 security.

## [1.0.0] - 2026-05-11 (pre-dates this repository)

Developed in the private `harness-kit` repo before this repo existed.

### Added
- Initial kit release: harness agent architecture, async logs, `git-reviewer` agent for
  semantic and safe commits.
