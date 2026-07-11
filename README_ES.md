# Harness Engineering Kit
> Kit reutilizable para aplicar la metodología Engineering Harness a cualquier proyecto de software.
> **Versión: 3.0** | Actualizado: 2026-07-11 | Licencia: [MIT](LICENSE)

📄 English version: [README.md](README.md)

---

## ¿Qué es este kit?

Este kit convierte a una IA "generadora de texto" en un **ingeniero responsable y autónomo** capaz de trabajar en un proyecto de forma persistente, verificable y sin desviarse.

Usa una **arquitectura Lean basada en workspaces**: cada tarea recibe un workspace aislado con su briefing (`orden.md`) y su área de trabajo (`session.md`). Los sub-agentes trabajan con contexto individual y acceso completo al código. Los resultados se archivan automáticamente. Una **capa de memoria persistente** (`.harness/context/`, con su propio índice de navegación) mantiene el estado de sesión y el conocimiento estable del proyecto entre sesiones y entre herramientas de IA.

### Novedades de la v3

- **Memoria indexada**: `.harness/context/index.md` es el mapa de la memoria (qué leer y cuándo — FAST vs FULL LOAD); `init.js` verifica los 7 ficheros de contexto.
- **Protocolo universal primero**: `AGENTS.md` es la fuente única del protocolo de trabajo; `CLAUDE.md`/`GEMINI.md` solo lo importan (`@AGENTS.md`). El instalador **fusiona** — nunca sobrescribe secciones de otras herramientas.
- **Separación limpia de responsabilidades**: el conocimiento de dominio ya no forma parte de este kit — vive en la herramienta hermana [harness-okf](https://github.com/csalazart/harness-okf) (ver abajo).
- Prompt nuevo: `06_migrate.md` (migrar estructuras harness antiguas a Lean v3).

---

## Requisitos

- **Node.js** instalado globalmente (`node -v` para verificar)
- Funciona en **Mac, Linux y Windows** — Node.js es la única dependencia

> Para proyectos Node.js: el kit recomienda **pnpm** sobre npm por seguridad y rendimiento. El prompt de inicialización detecta el uso de npm y pregunta cómo proceder.

---

## Contenido del kit

```
harness-kit-install/
├── README.md / README_ES.md               ← Versión inglesa / este fichero
├── QUICKSTART.md / QUICKSTART_ES.md       ← Arranque rápido en 5 pasos
├── INSTALL.md                             ← Guía de instalación
├── LICENSE                                ← MIT
│
├── prompts/                               ← Prompts listos para copiar y pegar
│   ├── 01_init.md                         ← Inicialización del harness (PRINCIPAL)
│   ├── 02_new_agent.md                    ← Crear un nuevo agente especializado
│   ├── 03_phase_handshake.md              ← Protocolo de inicio de sesión
│   ├── 04_phase_handoff.md                ← Protocolo de cierre de sesión
│   ├── 05_decision_resolver.md            ← Desbloquear decisiones críticas pendientes
│   └── 06_migrate.md                      ← Migrar estructuras harness antiguas a Lean v3
│
├── questionnaire/
│   └── project_discovery.md               ← Preguntas que la IA hace si falta contexto
│
├── templates/                             ← Plantillas de los ficheros de control
│   ├── AGENTS.md.template                 ← Protocolo universal (todos los modelos) — fuente única
│   ├── CLAUDE.md.template / GEMINI.md.template ← Ficheros por herramienta (importan @AGENTS.md)
│   ├── STATE.md.template                  ← Backlog + decisiones (fuente de verdad)
│   ├── orden.md.template / SESSION.md.template ← Ficheros del workspace de tarea
│   ├── init.js.template / init.sh.template    ← Health-check del entorno
│   ├── harness-start.js.template          ← Crea un workspace de tarea
│   ├── harness-finish.js.template         ← Archiva y cierra un workspace
│   ├── harness-link-skills.js.template    ← Enlaza .harness/skills a cada herramienta IA
│   ├── context/                           ← Memoria persistente (7 ficheros + index.md)
│   ├── skills/                            ← load-memory, update-memory (+ graphify, opcional)
│   ├── .claude/settings.json.template     ← Configuración de proyecto para Claude Code
│   └── agents/                            ← Plantillas de Orquestador + especialista genérico
│
└── rules-library/                         ← Biblioteca de reglas por tecnología
    (git-reviewer, solidity, rust-anchor, nextjs, react, vue, nuxt, nodejs,
     python, laravel, php-oop, symfony, jest, hardhat, docker, blockchain,
     security-web3, security-web2)
```

---

## Cómo usar este kit

### Opción A — Proyecto nuevo desde cero
1. Clona o copia este repo junto a (o dentro de) tu proyecto nuevo
2. Abre un chat con tu agente IA (Claude Code, Gemini, Cursor…) en la raíz del proyecto
3. Pega el contenido de `prompts/01_init.md`
4. La IA preguntará lo que falte (vía `questionnaire/project_discovery.md`)
5. Al terminar tendrás el harness completo para tu proyecto

### Opción B — Proyecto existente sin harness
1. Igual que la Opción A — la IA analiza lo que ya existe y completa lo que falta, fusionando (nunca sobrescribiendo) los `AGENTS.md`/`CLAUDE.md` existentes

### Opción C — Añadir un agente nuevo a un proyecto ya "harnessado"
1. Usa `prompts/02_new_agent.md` con el contexto de tu proyecto

### Opción D — Kit completo: harness-kit + harness-okf (recomendado)

El sistema completo son **dos herramientas independientes y compatibles** que comparten el ecosistema `.harness/`:

| Herramienta | Responsabilidad | Instala |
|---|---|---|
| **harness-kit** (este repo) | Flujo de trabajo: tareas, estado de sesión, roles de agentes, reglas | `.harness/context/`, `workspaces/`, `rules/`, `STATE.md` |
| **[harness-okf](https://github.com/csalazart/harness-okf)** | Conocimiento: un cerebro OKF autocontenido del dominio del proyecto (fuentes, conceptos, playbooks) | `.harness/llm-wiki/`, skills `okf-*` |

Para instalar el kit completo:
1. Instala primero harness-kit (Opción A/B)
2. Después sigue el `prompts/01_init.md` de [harness-okf](https://github.com/csalazart/harness-okf) — detecta el `.harness/` existente y añade solo sus piezas (nunca toca las tuyas)

> Cualquier orden funciona, y cada herramienta es totalmente usable por sí sola.
> Frontera: `context/` = dónde vamos (sesión) · `llm-wiki/` = qué sabemos (dominio).

---

## Los 5 principios del Harness

1. **Tarea única** — Un agente, un workspace, unos criterios de aceptación. No avanzar hasta que pasen.
2. **Handshake** — Al arrancar: `node init.js` + leer `STATE.md` + revisar workspaces activos. Siempre.
3. **Hand-off** — Al terminar: completar `session.md` + `harness-finish.js` + commit. Siempre.
4. **Orquestador primero** — El orquestador coordina y crea workspaces. Los especialistas ejecutan.
5. **Verificación antes de avanzar** — Si no hay comando que confirme que funciona, no cuenta como hecho.

---

## Compatibilidad

Diseñado para funcionar con:
- **Claude Code** (CLI de Anthropic) — uso principal
- **Gemini CLI** — vía `GEMINI.md` → `@AGENTS.md`
- **Cursor / Windsurf / opencode** — compatible vía `AGENTS.md`
- **Cualquier LLM** con acceso a ficheros — usando los prompts manualmente

---

## Licencia

[MIT](LICENSE) — libre para usar, modificar y redistribuir.
