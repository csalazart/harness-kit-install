# Changelog

Todos los cambios notables de este proyecto se documentan en este archivo.

El formato sigue [Keep a Changelog](https://keepachangelog.com/es-ES/1.0.0/), y este
proyecto usa un versionado inspirado en [Semantic Versioning](https://semver.org/lang/es/).

> **Sobre los números de versión:** este archivo se escribió de forma retroactiva a
> partir de `git log`, ya que el proyecto no llevaba changelog hasta ahora. El primer
> commit de este repo se etiquetó `3.0`, pero publicaba el mismo contenido que el repo
> privado de desarrollo (`harness-kit`) seguía llamando `2.0` en ese momento — un error
> histórico de etiquetado. Aquí se renumera como `1.3.0`. Las versiones `1.1.0` y `1.2.0`
> nunca tuvieron un release público en este repo (solo existieron en el repo privado,
> antes de la primera sincronización) y se listan a modo de contexto. La `4.0` (usada
> brevemente para el release del grafo dual) ya se había autocorregido a `1.4.0` antes de
> que existiera este archivo, así que no necesitó cambios. De aquí en adelante, actualiza
> este archivo en cada sincronización desde `harness-kit`, en vez de tocar solo la línea
> `Version:` del README.
>
> Independientemente de la versión del kit, el proyecto también usa etiquetas internas de
> **generación de arquitectura** (v1/v2/v3/v4) en nombres de fichero como
> `prompts/07_upgrade_v3_to_v4.md`. Corresponden aproximadamente a `1.2.0` (v2, Lean),
> `1.3.0` (v3, graphify) y `1.4.0` (v4, grafo dual) más abajo, pero son un eje distinto —
> no confundas un "upgrade v3→v4" con "kit 1.3.0→1.4.0".

## [Unreleased]

## [1.4.0] - 2026-09-08

Referido internamente como generación de arquitectura **v4 (grafo de conocimiento dual)**.

### Added
- **Grafo de conocimiento dual:** dos grafos con dominios separados — `codebase-memory-mcp`
  (código fuente: AST + resolución semántica de llamadas) y `graphify` (capa de
  conocimiento: `.harness/`, `docs/`, `plan/`).
- Perfilado automático de proyecto (DOC / MIXTO / CODE) según el tamaño del código —
  `templates/PROFILE.md.template`.
- `graphifyignore-library/`: fragmentos de `.graphifyignore` por stack (nodejs, nextjs,
  nuxt, vue-react, python, php, rust, solidity-hardhat, docker, cloudflare, comunes de
  build/linting), en sustitución del `.graphifyignore.template` monolítico.
- `templates/.mcp.json.template` (declara el servidor `codebase-memory-mcp`).
- `templates/docs/KNOWLEDGE_GRAPHS.md.template` (guía de routing/coste/troubleshooting
  del modelo dual).
- Skill `doc-code-audit` (cruce docs↔código, solo perfiles CODE/MIXTO con MCP).
- Scripts `index-code.ps1` / `index-code.sh` para reindexar el grafo de código.
- `prompts/07_upgrade_v3_to_v4.md`: ruta de actualización para instalaciones v3 (Lean)
  existentes, sin reinstalar desde cero.
- Proyecto hermano `harness-okf` documentado en README/README_ES (base de conocimiento
  de dominio persistente en `.harness/llm-wiki/`).

### Fixed
- Correcciones de la instalación inicial validadas en el repo privado antes de esta
  sincronización (ver cambios en `01_init.md` y `07_upgrade_v3_to_v4.md`).

## [1.3.0] - 2026-07-11

Primer release público de este repositorio.

*(Etiquetado originalmente `3.0` aquí — el mismo punto de sincronización que el README
del repo privado `harness-kit` seguía llamando `2.0` en ese momento. Ver la nota de
arriba.)*

### Added
- **graphify** integrado como grafo de conocimiento del proyecto (`.harness/`, `docs/`,
  `plan/`), con scripts de query/update y skill dedicada.
- Proyecto hermano `harness-okf` (base de conocimiento de dominio en
  `.harness/llm-wiki/`), indexado por graphify.
- Script de enlace de skills al agente de IA activo.
- `rules-library/` actualizada para ser consciente de graphify.

## [1.2.0] - 2026-05-22 (anterior a este repositorio)

Desarrollado en el repo privado `harness-kit` antes de la primera sincronización pública
de este repo; se lista aquí por continuidad.

### Added
- **Arquitectura Lean basada en workspaces:** cada tarea obtiene un workspace aislado
  con su briefing (`orden.md`) y área de trabajo (`session.md`); resultados archivados
  automáticamente.
- Prompts de protocolo: handshake de fase, handoff de fase, resolución de decisiones
  pendientes, migración desde la estructura antigua.
- Soporte multi-IA: addendum específico para Gemini (`GEMINI.md.template`).
- `rules-library/` ampliada: frontend, backend, testing, deploy, seguridad web2/web3.

## [1.0.0] - 2026-05-11 (anterior a este repositorio)

Desarrollado en el repo privado `harness-kit` antes de que este repo existiera.

### Added
- Versión inicial del kit: arquitectura de agentes del harness, logs asíncronos, agente
  `git-reviewer` para commits semánticos y seguros.
