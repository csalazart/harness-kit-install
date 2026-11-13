# Harness Engineering Kit
> Kit reutilizable para aplicar la metodología de Arnés de Ingeniería a cualquier proyecto de software.
> Versión: 1.5.0 | Actualizado: 2026-11-13 | [Historial de cambios](CHANGELOG_ES.md)

> **Etiquetas de versión:** las etiquetas cortas que se usan en todo el kit (`v2`, `v3`,
> `v4`, p. ej. en `prompts/07_upgrade_v3_to_v4.md`) son la misma numeración que la
> versión del kit: **v2 = 1.2, v3 = 1.3, v4 = 1.4**. Donde leas "v4", lee "1.4" — un
> "upgrade v3→v4" es un "upgrade 1.3→1.4". Detalle en el [Historial de cambios](CHANGELOG_ES.md).

---

## ¿Qué es este kit?

Este kit convierte a una IA de "generador de texto" en un **ingeniero responsable y autónomo** que puede trabajar en un proyecto de forma persistente, verificable y sin descarrilarse.

Usa una **arquitectura Lean basada en Workspaces**: cada tarea recibe un workspace aislado con su briefing (`orden.md`) y área de trabajo (`session.md`). Los sub-agentes trabajan con contexto individual y acceso completo al código. Los resultados se archivan automáticamente.

### Motor de código y capa de conocimiento

El **código fuente** lo sirve un **motor de código** (AST + resolución semántica de
llamadas, se reindexa gratis), o ninguno, según un **perfil** de proyecto calculado por
el tamaño del código: **DOC** (0–20 ficheros) no instala motor y el código se lee
directamente; **MIXTO** (21–100) deja el motor como opcional; **CODE** (>100) lo hace obligatorio.

La **capa de conocimiento** (`.harness/`, `docs/`, `plan/`) no tiene índice: se consulta
con `.harness/context/index.md`, el cerebro OKF si está instalado (ver harness-okf más
abajo) y Grep/Read. Ver `docs/KNOWLEDGE_GRAPHS.md` en un proyecto ya instalado para el
detalle del enrutamiento.

El **motor de código** es seleccionable: `codebase-memory-mcp`, `codegraph` (alternativa) o
`auto`, donde la IA elige el que mejor encaje con el proyecto. Solo corre uno a la vez; la elección
queda registrada en `.harness/PROFILE.md`.

### Proyecto hermano: harness-okf

[harness-okf](https://github.com/csalazart/harness-okf) es una herramienta hermana e
independiente que añade una **base de conocimiento de dominio** persistente
(`.harness/llm-wiki/`) — conceptos, fuentes verificadas, playbooks. No requiere este
kit, pero se instala bajo el mismo contenedor `.harness/` y nunca toca `context/`,
`rules/` ni `STATE.md`. Si está instalado, su base de conocimiento pasa a formar parte
de la capa de conocimiento y se consulta directamente (ver el modelo arriba).

---

## Requisitos

- **Node.js** instalado globalmente (`node -v` para verificar)
- Funciona en **Mac, Linux y Windows** — Node.js es la única dependencia

> Para proyectos Node.js: este kit recomienda **pnpm** sobre npm por seguridad y rendimiento. El prompt de inicialización detectará el uso de npm y te preguntará cómo proceder.

---

## Contenido del kit

```
harness-kit-install/
├── README.md                              ← Versión en inglés
├── README_ES.md                           ← Este fichero
├── CHANGELOG.md                           ← Historial de cambios (inglés)
├── CHANGELOG_ES.md                        ← Historial de cambios (español)
├── QUICKSTART.md                          ← Arranque rápido en 5 pasos (inglés)
├── QUICKSTART_ES.md                       ← Arranque rápido en 5 pasos (español)
├── INSTALL.md                             ← Guía de instalación paso a paso
│
├── prompts/                               ← Prompts listos para copiar y pegar
│   ├── 01_init.md                         ← Inicialización del harness (PRINCIPAL — Claude Code)
│   ├── 02_new_agent.md                    ← Crear un nuevo agente especializado
│   ├── 03_phase_handshake.md              ← Protocolo de inicio de fase
│   ├── 04_phase_handoff.md                ← Protocolo de cierre de fase
│   ├── 05_decision_resolver.md            ← Desbloquear decisiones críticas pendientes
│   ├── 06_migrate.md                      ← Migrar un proyecto desde la estructura Harness antigua
│   └── 07_upgrade_v3_to_v4.md             ← Actualizar una instalación v3 (Lean) a v4
│
├── questionnaire/
│   └── project_discovery.md               ← Preguntas que la IA hace si falta contexto
│
├── templates/                             ← Plantillas para los ficheros de control
│   ├── CLAUDE.md.template                 ← Fichero maestro del proyecto (Claude Code)
│   ├── AGENTS.md.template                 ← Protocolo universal (todos los modelos)
│   ├── GEMINI.md.template                 ← Añadido específico para Gemini
│   ├── STATE.md.template                  ← Backlog + decisiones (fuente de verdad)
│   ├── orden.md.template                  ← Briefing del workspace de tarea
│   ├── SESSION.md.template                ← Área de trabajo del workspace de tarea
│   ├── PROFILE.md.template                ← Perfil del proyecto: DOC/MIXTO/CODE (v4)
│   ├── .mcp.json.template                 ← Declaración del servidor codebase-memory-mcp (v4)
│   ├── init.js.template                   ← Verificación del entorno (node init.js)
│   ├── init.sh.template                   ← Alternativa Bash pura de init.js
│   ├── harness-start.js.template          ← Crea un workspace de tarea
│   ├── harness-finish.js.template         ← Archiva y cierra un workspace
│   ├── harness-link-skills.js.template    ← Enlaza .harness/skills/ en cada agente detectado (.claude, .gemini, .opencode, .cursor, .qwen, .kilocode, .kimi-code, .agents)
│   ├── harness-merge-instructions.js.template ← Fusiona las secciones del kit en AGENTS.md / CLAUDE.md / GEMINI.md / … con marcas <!-- harness-kit:ID -->, sin pisar lo del proyecto
│   ├── harness-agents.json                ← Matriz de agentes (rutas, detección, ficheros de instrucciones) que leen ambos scripts
│   ├── AGENT_POINTER.md.template          ← Fichero "lee AGENTS.md primero" para agentes con fichero propio (QWEN.md…)
│   ├── code-graph/                        ← Fragmentos por motor de código (codebase-memory | codegraph | none): enrutamiento de AGENTS.md, mantenimiento, entrada de .mcp.json
│   ├── index-code.ps1/sh.template         ← Reindexa el grafo de código (codebase-memory-mcp, v4)
│   ├── .claude/
│   │   └── settings.json.template         ← Configuración de Claude Code para el proyecto
│   ├── agents/
│   │   ├── AGENT_00_ORCHESTRATOR.template.md  ← Agente orquestador
│   │   └── AGENT_SPECIALIST.template.md       ← Agente especialista genérico
│   ├── context/                           ← Plantillas de .harness/context/ (index, activeContext, progress…)
│   ├── docs/
│   │   ├── AGENTS_REFERENCE.md.template   ← Referencia extendida del protocolo
│   │   └── KNOWLEDGE_GRAPHS.md.template   ← Motor de código + enrutamiento del conocimiento: perfiles, coste, troubleshooting (v4)
│   └── skills/
│       ├── doc-code-audit/                ← Skill de careo doc↔código (v4, solo CODE/MIXTO-con-MCP)
│       ├── load-memory.md
│       └── update-memory.md
│
├── rules-library/                         ← Biblioteca de reglas por tecnología
│   ├── agent-git-reviewer.md              ← Commits semánticos y seguros
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

## Cómo usar este kit

### Opción A — Proyecto nuevo desde cero
1. Copia `harness-kit-install/` a la raíz de tu nuevo proyecto
2. Abre un chat con Claude Code
3. Pega el contenido de `prompts/01_init.md`
4. La IA te hará preguntas si falta contexto (via `questionnaire/project_discovery.md`)
5. Al terminar tendrás el harness completo de tu proyecto

### Opción B — Proyecto existente sin harness
1. Copia solo `harness-kit-install/` al repositorio
2. Usa `prompts/01_init.md` — la IA analizará lo que ya existe y completará lo que falta

### Opción C — Añadir un agente nuevo a un proyecto ya harnessado
1. Usa `prompts/02_new_agent.md` con el contexto de tu proyecto
2. La IA generará el fichero del agente usando `templates/agents/AGENT_SPECIALIST.template.md`

---

## Los 5 principios del Harness

1. **Tarea Única** — Un agente, un workspace, un conjunto de criterios de aceptación. No avanzar hasta que pasen.
2. **Handshake** — Al arrancar: ejecutar `node init.js` + leer `STATE.md` + revisar workspaces activos. Siempre.
3. **Hand-off** — Al terminar: completar `session.md` + ejecutar `harness-finish.js` + commit. Siempre.
4. **Orquestador primero** — El orquestador coordina y crea workspaces. Los especialistas ejecutan.
5. **Verificación antes de avance** — Si no hay comando que confirme que funciona, no cuenta como hecho.

---

## Compatibilidad

Diseñado para funcionar con:
- **Claude Code** (CLI de Anthropic) — uso principal
- **Gemini CLI**, **OpenCode**, **Cursor**, **Codex**, **Qwen Code**, **Kilo Code**, **Kimi Code** —
  los skills se enlazan en la carpeta nativa de cada agente y los ficheros de instrucciones se
  fusionan con marcas `<!-- harness-kit:ID -->` (matriz de agentes: `templates/harness-agents.json`)
- **Cualquier LLM** con acceso a ficheros — usando los prompts manualmente
