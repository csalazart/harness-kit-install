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
> Independently of the kit version, the project also uses internal **architecture
> generation** labels (v1/v2/v3/v4) in filenames like `prompts/07_upgrade_v3_to_v4.md`.
> Those map roughly to `1.2.0` (v2, Lean), `1.3.0` (v3, graphify) and `1.4.0` (v4, dual
> graph) below, but are a separate axis — don't confuse a "v3→v4 upgrade" with
> "kit 1.3.0→1.4.0".

## [Unreleased]

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
