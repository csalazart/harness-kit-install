# Quickstart — Harness en 5 pasos

## Para un proyecto nuevo

### Paso 1 — Requisitos y Copia del kit
Asegúrate de tener **Node.js** instalado (`node -v` para verificar).

```bash
cp -r harness-kit-install/ /tu/nuevo/proyecto/harness-kit
```

### Paso 2 — Abre Claude Code en tu proyecto
```bash
cd /tu/nuevo/proyecto
claude
```

### Paso 3 — Pega el prompt de inicialización
Copia el contenido de `harness-kit-install/prompts/01_init.md` y envíalo al chat.

### Paso 4 — Responde las preguntas
La IA te preguntará sobre tu proyecto (nombre, tipo, stack, módulos, estado).
Las preguntas están documentadas en `harness-kit-install/questionnaire/project_discovery.md` como referencia.
Si ya tienes documentación (`README.md`, `package.json`, etc.), díselo — la analizará directamente.

### Paso 5 — El harness queda listo
Al terminar tendrás en la raíz de tu proyecto:
```
CLAUDE.md                          ← contexto para Claude Code
AGENTS.md                          ← protocolo universal (todos los modelos)
init.js                            ← verificación del entorno (node init.js)
harness-kit-install/                       ← kit completo (no borrar, los prompts lo referencian)
scripts/
├── harness-start.js               ← crea un workspace de tarea
└── harness-finish.js              ← archiva y cierra un workspace
.harness/
├── STATE.md                       ← backlog + decisiones (fuente de verdad)
├── PROFILE.md                     ← perfil del proyecto: DOC/MIXTO/CODE, decide qué grafos se instalan
├── rules/
│   ├── orchestrator.md            ← reglas del agente orquestador
│   ├── git-reviewer.md            ← agente revisor de commits
│   └── {modulo}.md                ← uno por área del proyecto (backend, frontend, etc.)
├── workspaces/                    ← oficinas de trabajo temporales por tarea
└── logs/
    └── SUMMARY.md                 ← historial cronológico inverso (auto-generado)
graphify-out/                      ← grafo de conocimiento (docs, .harness/, plan/) — versionado parcial
.mcp.json                          ← servidor codebase-memory-mcp, solo si el perfil lo requiere
```

> **Grafo dual de conocimiento (v4):** `graphify` (conocimiento) se instala
> siempre; `codebase-memory-mcp` (código) solo si el perfil del proyecto es
> CODE, o MIXTO y decides activarlo. Ver `docs/KNOWLEDGE_GRAPHS.md` una vez
> instalado para la tabla de enrutamiento completa entre ambos.

---

## Para lanzar una tarea con un sub-agente

```
1. Abre el chat (CLAUDE.md ya cargado)
2. Di: "Quiero trabajar en [tarea]"
3. La IA (Orquestador) ejecuta: node scripts/harness-start.js --task nombre_tarea
   → Crea .harness/workspaces/nombre_tarea/orden.md + session.md automáticamente
4. La IA (Orquestador) completa los criterios de aceptación en el orden.md ya creado
5. Abre una sesión nueva para el especialista y di:
   "Lee .harness/rules/{area}.md y ejecuta la tarea en .harness/workspaces/nombre_tarea/orden.md"
6. Al terminar el agente especialista, el orquestador cierra:
   node scripts/harness-finish.js --task nombre_tarea
   → Archiva la sesión, actualiza STATE.md y añade entrada a logs/SUMMARY.md
   ⚠️  Después del archivo, elimina físicamente .harness/workspaces/{tarea}/
      Asegúrate de que todo esté commiteado antes de ejecutar este paso.
```

---

## Para añadir un agente nuevo

```
1. Abre el chat con el contexto del proyecto (CLAUDE.md ya cargado)
2. Pega prompts/02_new_agent.md
3. Indica: nombre del agente, área, dependencias
4. La IA crea el fichero en .harness/rules/
```

---

## Tiempo estimado de setup

| Proyecto | Tiempo de setup |
|----------|----------------|
| Sin documentación previa | 45–90 minutos |
| Con docs existentes | 20–40 minutos |
| Solo añadir agente nuevo | 10–15 minutos |
