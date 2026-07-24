# Prompt 01 — Inicialización del Harness
> Copia y pega este prompt completo al inicio de una sesión con Claude Code.
> La IA analizará el proyecto y construirá toda la infraestructura de control.

---

```
ASUNTO: Inicializar Harness Engineering para este proyecto

Actúa como un Harness Engineer Senior. Tu misión es analizar este proyecto
y construir la infraestructura de control que permita ejecución autónoma y persistente.

## FASE A — Descubrimiento (haz esto PRIMERO)

Antes de crear nada, necesito que entiendas el proyecto completamente.
Sigue este orden:

1. Busca y lee estos ficheros si existen (en orden de prioridad):
   - CLAUDE.md o README.md → contexto general del proyecto
   - Cualquier fichero en docs/ → documentación existente
   - package.json / pnpm-lock.yaml / package-lock.json → stack y gestor de paquetes
   - Estructura de carpetas con `ls` o `tree`

2. **Revisión de Gestor de Paquetes (MUY IMPORTANTE):**
   - Si el proyecto no tiene gestor de paquetes inicializado, asume `pnpm` por defecto.
   - Si detectas `package-lock.json` o uso de `npm`, haz una pausa para advertir al usuario: "He detectado el uso de npm. Por razones de seguridad, rapidez y eficiencia de almacenamiento, este harness recomienda fuertemente migrar a pnpm."
   - Pregunta al usuario qué desea hacer:
     a) Migrar a pnpm ahora mismo antes de continuar.
     b) Continuar con la inicialización y crear una tarea prioritaria para que un agente realice la migración después.
     c) Mantener npm (en cuyo caso, se adaptará el arnés para usar npm).
   - Espera su decisión antes de generar los ficheros.

3. Si NO encuentras suficiente información para responder estas preguntas,
   PREGÚNTAMELAS antes de continuar (una a una, no todas juntas):

   a) ¿Cuál es el nombre y propósito del proyecto?
   b) ¿Qué tipo de proyecto es? (web app, smart contract, API, mobile, CLI, otro)
   c) ¿Qué stack tecnológico usa? (lenguajes, frameworks, bases de datos)
   d) ¿Tiene fases o áreas de trabajo diferenciadas? (frontend, backend, contratos, etc.)
   e) ¿Hay agentes o roles especializados ya definidos?
   f) ¿Cuál es el estado actual? (idea, en desarrollo, en producción)
   g) ¿Qué es lo más urgente o bloqueante ahora mismo?

3. NO ASUMAS nada que no puedas leer en los ficheros. Pregunta si no sabes.

## FASE B — Análisis (después del descubrimiento)

Una vez que tengas suficiente contexto, produce:

1. Un resumen del proyecto en 3-5 líneas
2. Las áreas de trabajo identificadas (ej: frontend, backend, testing, deploy)
3. El grafo de dependencias entre áreas (qué desbloquea qué)
4. Las decisiones críticas pendientes que bloquean el avance
5. Las brechas que tiene el proyecto para poder usar Harness

Muéstrame este análisis y espera mi confirmación antes de continuar.

## FASE C — Construcción (solo después de mi OK)

Crea los siguientes ficheros de control adaptados a ESTE proyecto específico:

### 1. CLAUDE.md (si no existe o necesita actualización)
Usa la plantilla de harness-kit-install/templates/CLAUDE.md.template.
Adapta cada sección al proyecto real.

### 2. .harness/STATE.md
Inicia el `STATE.md` copiando la plantilla `harness-kit-install/templates/STATE.md.template`.
Rellena la sección "Backlog de Tareas" con las tareas identificadas en tu análisis de la Fase B.

### 3. Scripts de Mantenimiento y Automatización
Crea la carpeta `scripts/` en la raíz.
Copia `harness-kit-install/templates/harness-start.js.template` a `scripts/harness-start.js`
Copia `harness-kit-install/templates/harness-finish.js.template` a `scripts/harness-finish.js`

*   **Scripts de Soporte de Graphify:**
    Copia `harness-kit-install/templates/update-graph.ps1.template` a `scripts/update-graph.ps1`
    Copia `harness-kit-install/templates/update-graph.sh.template` a `scripts/update-graph.sh`
    Copia `harness-kit-install/templates/query-graph.ps1.template` a `scripts/query-graph.ps1`
    Copia `harness-kit-install/templates/query-graph.sh.template` a `scripts/query-graph.sh`

### 3.5. Instalación de Graphify (OBLIGATORIO)

Este harness requiere que el proyecto quede indexado semánticamente con **graphify**
para que los agentes puedan localizar código de forma precisa y barata en tokens
(ver Paso 0.5 de `AGENTS.md`). No es opcional: `node init.js` (Sección 7) fallará
con `[HARNESS FAIL]` si no está instalado.

**Requisito previo: Python** — graphify es una librería Python (`graphifyy`).
Para instrucciones detalladas por sistema operativo consulta `docs/GRAPHIFY_GUIDE.md`
(ya deberías haberla copiado en la Sección 5; si aún no, cópiala ahora).

Verifica que tienes Python instalado:

```bash
python --version
```

Debe mostrar algo como `Python 3.10+`. Si no lo tienes:
- Descárgalo desde [python.org](https://www.python.org/downloads/) (versión LTS)
- O usa `winget install Python.Python.3.12` (Windows)
- O `brew install python` (Mac)

a) Copia el skill:
   ```
   harness-kit-install/templates/skills/graphify/ → .claude/skills/graphify/
   harness-kit-install/templates/skills/graphify/ → .agents/skills/graphify/
   ```

b) Crea `.graphifyignore` en la raíz del proyecto copiando
   `harness-kit-install/templates/.graphifyignore.template` tal cual. Ya excluye
   dependencias, binarios y metadatos — y deliberadamente NO excluye `docs/` ni
   `plan/`, para que la documentación de arquitectura y planes quede indexada.

c) Ejecuta `/graphify` ahora mismo para generar `graphify-out/graph.json` y
   `graphify-out/GRAPH_REPORT.md`. Esto instala automáticamente el paquete
   `graphifyy` la primera vez.

d) Verifica que `graphify-out/.graphify_python` y `graphify-out/graph.json`
   existen antes de continuar a la Sección 7 (`node init.js`).

e) Añade `graphify-out/` al `.gitignore` del proyecto (créalo si no existe).
   **Nunca comitear esta carpeta**: `.graphify_python` contiene una ruta local
   de Python específica de esta máquina, y `graph.json`/`graph.html` se
   regeneran automáticamente en cada `harness-finish.js`/`.cjs` — comitearlos
   ensuciaría el historial con diffs enormes de un artefacto derivado.

> Nota: graphify es Python puro — no usa npm ni pnpm. La instalación del
> paquete `graphifyy` la maneja el propio skill automáticamente.

### 4. Entorno de Agentes (.harness/)
Crea las siguientes carpetas para la arquitectura Task-Driven:
- `.harness/workspaces/` (Aquí se crearán las oficinas de trabajo temporal)
- `.harness/logs/` (Aquí se guardarán las sesiones históricas)
- `.harness/rules/` (Reglas por módulo del proyecto)

### 5. Documentación y Reglas Técnicas
Copia el fichero maestro universal `harness-kit-install/templates/AGENTS.md.template` a `AGENTS.md`.
Crea la carpeta `docs/` en la raíz (si no existe) y copia:
- `harness-kit-install/templates/docs/AGENTS_REFERENCE.md.template` a `docs/AGENTS_REFERENCE.md`
- `harness-kit-install/templates/docs/GRAPHIFY_GUIDE.md.template` a `docs/GRAPHIFY_GUIDE.md`

> ⚠️ **Si ya existe un AGENTS.md, docs/AGENTS_REFERENCE.md o docs/GRAPHIFY_GUIDE.md en el proyecto: FUSIONA, nunca sobrescribas.**
> Conserva al final del AGENTS.md/docs/AGENTS_REFERENCE.md generado cualquier sección ajena al harness-kit
> (por ejemplo "## Cerebro OKF" de harness-okf, u otras de terceros). Mismo criterio
> con CLAUDE.md: conserva el import `@AGENTS.md` y las secciones ajenas existentes.

### 6. Ficheros de rol de agentes (.harness/rules/)
Para cada área identificada, crea el fichero de reglas en `.harness/rules/`:
- `.harness/rules/orchestrator.md` — siempre (usa `AGENT_00_ORCHESTRATOR.template.md`)
- `.harness/rules/{modulo}.md` — uno por área (usa `AGENT_SPECIALIST.template.md`)
- `.harness/rules/git-reviewer.md` — siempre (copia `rules-library/agent-git-reviewer.md`)
Copia también las reglas técnicas relevantes del stack desde `rules-library/` a `.harness/rules/`.

### 7. init.js
Adapta `harness-kit-install/templates/init.js.template` al stack detectado y crea `init.js` en la raíz.
Verifica que al ejecutar `node init.js` termina con `[HARNESS OK]` antes de continuar.

> ⚠️ `node init.js` comprueba que graphify está instalado (Sección 3.5) — si no lo
> instalaste todavía, hazlo ahora o `init.js` devolverá `[HARNESS FAIL]`.

> **Nota sobre verificación del entorno:** Se recomienda `init.js` (Node.js, cross-platform).
> Existe también `harness-kit-install/templates/init.sh.template` como alternativa para entornos
> Bash puros (Linux/Mac sin Node). Usa el que corresponda al entorno del proyecto.

### 8. Scaffold de carpetas
Crea la estructura de carpetas vacías del proyecto (con .gitkeep).

### 9. Enlace de Skills al agente IA activo

**Paso 9.1 — Crear las carpetas del agente activo**

Tú (la IA que está ejecutando esta instalación) sabes qué modelo eres.
Crea ahora la carpeta que te corresponde si no existe:

- Si eres **Claude Code** → crea `.claude/` (ya debería existir; si no, créala)
- Si eres **Gemini** → crea `.gemini/`
- Si eres **otro modelo** → no necesitas crear nada extra

Además, crea **siempre** la carpeta `.agents/` independientemente del modelo.
Es el estándar abierto que usan OpenAI, Qwen, xAI, Llama y el resto de IAs.
Así cualquier agente futuro encontrará los skills sin configuración adicional.

```
Carpetas a crear (las que no existan):
  .agents/          ← SIEMPRE (estándar universal)
  .claude/          ← solo si eres Claude Code
  .gemini/          ← solo si eres Gemini
```

**Paso 9.2 — Ejecutar el script de enlace**

Copia `harness-kit-install/templates/harness-link-skills.js.template` a `scripts/harness-link-skills.js`
y ejecútalo:

```bash
node scripts/harness-link-skills.js
```

El script detecta las carpetas creadas en el paso anterior y genera un symlink
(o copia si el symlink falla) desde `.harness/skills/` hacia cada una:

| Carpeta detectada | Skills enlazados en |
|-------------------|---------------------|
| `.claude/` | `.claude/skills/` |
| `.gemini/` | `.gemini/skills/` |
| `.agents/` | `.agents/skills/` |

> Si en el futuro el usuario añade un nuevo agente IA al proyecto,
> basta con crear su carpeta raíz y volver a ejecutar este script.

### 10. Configuración de hooks e integración con IA (opcional pero recomendado)

**Si el proyecto usa Claude Code:**
- Copia `harness-kit-install/templates/.claude/settings.json.template` a `.claude/settings.json`
- Este hook ejecuta `node init.js` automáticamente en cada edición y al cerrar sesión
- Sin el hook, AGENTS.md ya instruye a la IA que ejecute `node init.js` al arrancar (Paso 1 del Handshake)

**Si el proyecto usa Gemini como IA principal o secundaria:**
- Copia `harness-kit-install/templates/GEMINI.md.template` a `GEMINI.md` en la raíz
- Adapta `{{PROJECT_NAME}}` y `{{DATE}}`
- El protocolo universal sigue siendo `AGENTS.md` — `GEMINI.md` solo añade contexto Gemini-específico

**Cualquier otra IA (Cursor, Copilot, etc.):**
- No requiere configuración adicional — el handshake está en `AGENTS.md` y es modelo-agnóstico

### 10. Capa de contexto (.harness/context/)

Usando las respuestas de las preguntas 19-24 del cuestionario (o inferidas del análisis), genera los 7 ficheros en `.harness/context/`:

**Ficheros ESTABLES** (generados una vez, raramente cambian):

- `.harness/context/projectbrief.md` — usar Q19 (problema), Q20 (usuarios), Q21 (fuera de alcance), Q22 (restricciones)
- `.harness/context/productContext.md` — usar Q19 (problema), Q20 (usuarios)
- `.harness/context/systemPatterns.md` — usar el stack técnico detectado, documentar D001 (elección de stack)
- `.harness/context/techContext.md` — usar el stack, comandos del proyecto, Q24 (variables de entorno)

**Ficheros VOLÁTILES** (actualizados con cada tarea):

- `.harness/context/activeContext.md` — fecha actual, estado "proyecto inicializado", próximo paso desde STATE.md
- `.harness/context/progress.md` — una fila por módulo respondido en Q23, todos con ⬜ (sin empezar) al inicio
- `.harness/context/plans-index.md` — vacío al inicio (tablas con "ninguno aún")

**Skills AI-agnósticos:**

- `.harness/skills/load-memory.md` — copiar desde `harness-kit-install/templates/skills/load-memory.md`
- `.harness/skills/update-memory.md` — copiar desde `harness-kit-install/templates/skills/update-memory.md`

Si alguna pregunta (19-24) no fue respondida, usar el placeholder `[pendiente de definir]`.

Verificar que `node init.js` muestra:
```
✅  Existe .harness/context/projectbrief.md
ℹ️   context/activeContext.md presente ✅
ℹ️   context/progress.md presente ✅
```

### 11. Índice de la memoria (.harness/context/index.md)

Crea `.harness/context/index.md` desde
`harness-kit-install/templates/context/index.md.template`: es el **mapa de la memoria**
(qué fichero leer, cuándo — FAST vs FULL LOAD — y quién lo actualiza).

> 💡 Si el usuario quiere además una **knowledge base de dominio** (conceptos,
> fuentes verificadas, playbooks), instala **harness-okf** — proyecto hermano e
> independiente que convive en `.harness/llm-wiki/` (ver su `prompts/01_init.md`).

## FASE D — Cierre

1. Haz un commit con todo lo creado
2. Muéstrame el árbol de ficheros generado (incluyendo `.harness/context/` y `.harness/skills/`)
3. Dime cuál es el primer agente que debo abrir y por qué

4. Confirma que `graphify-out/GRAPH_REPORT.md` existe (se generó en la Sección 3.5,
   obligatoria) y muéstraselo al usuario como resumen de la indexación del proyecto.

## Reglas que debes seguir durante todo el proceso

- Si en algún momento no tienes suficiente información → PREGUNTA, no inventes
- Adapta TODO al proyecto real — no uses nombres genéricos como "MiProyecto"
- Si ya existe un fichero, analízalo y complétalo en vez de sobreescribirlo
- Los criterios de verificación deben ser COMANDOS REALES que se puedan ejecutar
- El backlog debe reflejar el estado ACTUAL del proyecto (marca DONE lo que ya esté hecho)

¿Entendido? Empieza con la Fase A.
```
