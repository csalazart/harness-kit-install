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

0. **Exploración automatizada (ejecuta esto SIEMPRE, no lo condiciones a "si existe algo").**
   En el **~90-98% de los casos no habrá ningún harness previo** — este paso tiene que
   funcionar bien en terreno virgen, no solo detectar instalaciones ya hechas.

   a) **Detección de estado previo** — comprueba si ya existen `.harness/`,
      `graphify-out/`, `.mcp.json`, `.harness/PROFILE.md`. **Si `PROFILE.md`
      existe, lee su campo `Kit-Version` primero — es la fuente de verdad
      explícita, no adivines la versión por qué ficheros hay o faltan.** Si
      `Kit-Version` es menor que la de este prompt (1.4.0), este proyecto
      necesita un upgrade dirigido, no una instalación nueva — detente y dilo
      (el mecanismo concreto depende de qué versión declare; hoy solo existe
      el camino v3→v4 documentado abajo). Si `PROFILE.md` no existe todavía,
      no hay sello que leer — usa los heurísticos de abajo, que son los que
      cubren toda instalación anterior a que este campo empezara a existir.
      Reporta cuál de estos casos aplica antes de continuar:
      - Ninguno existe → caso dominante, instalación desde cero.
      - `.harness/` existe sin `PROFILE.md` → instalación del kit v3. **No
        continúes con este prompt.** Dilo explícitamente y ofrece ejecutar
        `prompts/07_upgrade_v3_to_v4.md` en su lugar — ese prompt extrae y
        respalda el `STATE.md`/`context/`/`rules/`/`llm-wiki/` existentes
        antes de tocar nada, cosa que este prompt (`01_init.md`) no hace. No
        asumas que puedes sobrescribir la instalación v3.
      - `graphify-out/` existe sin `.harness/` → grafo huérfano de un uso suelto
        de graphify, se puede reconstruir sin conflicto.
      - **Ecosistema equivalente bajo otro nombre o nivel** — no te fíes solo de
        los nombres exactos de arriba. Busca también: `.agent/` (u otra carpeta
        de convención distinta) con su propio `init.js`, `AGENTS.md`/`CLAUDE.md`/
        `GEMINI.md`, skills propias o un `.mcp.json` ya configurado; o un
        `.harness/` real que vive en un nivel de carpeta distinto al que estás
        analizando (monorepo donde el `.git/` de verdad está uno o más niveles
        por encima o por debajo). Si encuentras esto, trátalo igual que el caso
        "`.harness/` sin `PROFILE.md`": repórtalo explícitamente y pregunta cómo
        proceder — nunca asumas que es terreno virgen solo porque no se llama
        `.harness/` o no está en la raíz que te pidieron analizar.

   b) **Conteo automático de código fuente** — cuenta los ficheros de código
      (extensiones: `.ts .tsx .js .jsx .mjs .cjs .vue .py .sol .rs .go .php
      .java .c .cpp .cs .rb`), **excluyendo** dependencias (`node_modules/`,
      `vendor/`, `.venv/`, `venv/`), artefactos de build (`dist/`, `build/`,
      `.next/`, `.nuxt/`, `out/`, `artifacts/`, `target/`) y generados
      (`typechain/`, `.nuxt/types`). Muestra el desglose por extensión. Este
      conteo alimenta directamente el perfilado de la Fase B punto 6 — no lo
      estimes a ojo.

      **Excluye también la fontanería del propio harness** si el proyecto ya
      tiene una instalación previa (caso "`.harness/` existe sin `PROFILE.md`"
      del punto 0.a): `scripts/harness-*`, `scripts/index-code.*`,
      `scripts/query-graph.*`, `scripts/update-graph.*`, `init.js`, `init.cjs`,
      `init.sh`, `.harness/scripts/`. Son plantillas del kit, no código del
      proyecto — igual que ya las excluye `.graphifyignore.base.template`
      (Bloque A.5). Sin esta exclusión, cualquier proyecto con el kit ya
      instalado infla su propio conteo con sus scripts de arranque/cierre y
      puede saltar de perfil sin que el código real del proyecto haya cambiado.

      **Desconfía de cualquier subcarpeta que por sí sola aporte una fracción
      grande del conteo total** (como referencia: más de un tercio). Puede ser
      código vendorizado o de referencia — plantillas de terceros, tutoriales,
      bootcamps, ejemplos copiados — que no es el código del proyecto aunque
      viva fuera de `node_modules/`. Señales para confirmarlo (no hace falta
      que se cumplan todas):
      - Tiene su propio `package.json` con un `name` que no coincide con el
        proyecto, o su propio README que se describe como plantilla/tutorial/
        ejemplo/bootcamp.
      - Nada en el resto del repositorio la importa o referencia (sin
        `import`/`require` hacia ella, sin aparecer en `tsconfig.json` paths).

      Si detectas esta situación, **no la excluyas en silencio ni la cuentes
      sin más**: muestra ambos conteos (bruto y sin esa carpeta) y pregunta al
      usuario si es código propio o material de referencia — el mismo criterio
      de "carpeta ambigua" que ya aplica el Bloque C de 3.5.1.b, pero aquí
      importa antes, porque puede cambiar el perfil resultante.

   c) **Medición de documentación existente** — cuenta los `.md` fuera de las
      carpetas excluidas, y si hay `docs/`, wikis o carpetas de planes ya
      pobladas. No decide el perfil, pero informa cuánto costará en tokens el
      primer build de graphify (referencia ya medida: ~44.677 tokens para
      38.787 palabras) — para que la confirmación de la Fase B sea informada.

   d) **Caso infraestructura** — si el conteo de código de aplicación es bajo
      pero predomina contenido de infraestructura/configuración
      (`docker-compose.yml`, manifiestos Terraform/Ansible, scripts de sistema
      en volumen, `Dockerfile` repetidos), repórtalo como tal en vez de forzarlo
      a una de las 3 categorías por el conteo bruto — ahí "el código" son
      manifiestos de configuración, no lógica que un LSP resuelva.

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

6. **Perfilado del proyecto (determina qué grafos se instalan):**

   Usa el conteo automático de la Fase A punto 0.b (ficheros de código,
   excluyendo dependencias, build y generados).

   | Ficheros de código | Perfil | Grafos a instalar |
   | --- | --- | --- |
   | 0 – 20 (o predomina infraestructura/configuración, ver 0.d) | **DOC**   | graphify (conocimiento + código) |
   | 21 – 100  | **MIXTO** | graphify (conocimiento) + MCP opcional |
   | > 100     | **CODE**  | graphify (conocimiento) + MCP |

   Muestra el conteo, el perfil propuesto y el desglose por extensión.
   **Espera confirmación del usuario** — puede corregir el perfil si conoce
   el proyecto mejor que el conteo (p. ej. un repo con poco código pero
   crítico, o uno con mucho código autogenerado).

   Si el conteo es bajo pero el proyecto es predominantemente infraestructura
   o configuración (Fase A punto 0.d), propón perfil **DOC**: no hay símbolos
   de aplicación que un LSP resuelva, así que el MCP no aporta nada — pero
   sigue siendo una propuesta sujeta a confirmación, no una regla automática.

   Si el perfil es MIXTO, explica el compromiso: el MCP añade un coste fijo
   de contexto en CADA ventana a cambio de resolución semántica precisa.
   Recomienda instalarlo solo si el proyecto tiene más de 50 ficheros o usa
   TypeScript/Python con imports por alias.

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

> Si `package.json` declara `"type": "module"`, usa extensión `.cjs` en vez de
> `.js` para estos dos ficheros (mismo motivo que la Sección 7 explica para
> `init.js`).

*   **Scripts de Soporte de Graphify:**
    Copia `harness-kit-install/templates/update-graph.ps1.template` a `scripts/update-graph.ps1`
    Copia `harness-kit-install/templates/update-graph.sh.template` a `scripts/update-graph.sh`
    Copia `harness-kit-install/templates/query-graph.ps1.template` a `scripts/query-graph.ps1`
    Copia `harness-kit-install/templates/query-graph.sh.template` a `scripts/query-graph.sh`

*   **Scripts de Soporte de codebase-memory-mcp** (solo perfiles CODE y MIXTO-con-MCP):
    Copia `harness-kit-install/templates/index-code.ps1.template` a `scripts/index-code.ps1`
    Copia `harness-kit-install/templates/index-code.sh.template` a `scripts/index-code.sh`

### 3.5. Instalación de los grafos de conocimiento (OBLIGATORIO)

Este harness separa dos dominios: el CÓDIGO lo indexa `codebase-memory-mcp`
y el CONOCIMIENTO (documentación + memoria del harness) lo indexa `graphify`.
El perfil decidido en la Fase B determina qué se instala.

#### 3.5.0 — Registrar el perfil

Crea `.harness/PROFILE.md` copiando
`harness-kit-install/templates/PROFILE.md.template` y rellena: **Kit-Version (1.4.0, fijo
para esta versión del kit)**, perfil (DOC/MIXTO/CODE), conteo de ficheros de
código, fecha, y si el MCP se instala o no y por qué.

> Este fichero NO es opcional. `init.js`/`init.sh` lo leen para saber qué
> verificar. Sin él no se puede distinguir "el MCP no está porque el perfil es
> DOC" de "el MCP no está porque la instalación falló".

#### 3.5.1 — Graphify (SIEMPRE, en los tres perfiles)

**Requisito previo: Python 3.10+.** Ver `docs/GRAPHIFY_GUIDE.md`.

```bash
python --version
```

Si no lo tienes:
- Descárgalo desde [python.org](https://www.python.org/downloads/) (versión LTS)
- O usa `winget install Python.Python.3.12` (Windows)
- O `brew install python` (Mac)

a) Copia el skill:
   ```
   harness-kit-install/templates/skills/graphify/ → .claude/skills/graphify/
   harness-kit-install/templates/skills/graphify/ → .agents/skills/graphify/
   ```

b) **Compón** el `.graphifyignore` en tres capas — **NO** es una copia
   directa de plantilla:

   **B.1 — Núcleo.** Copia `harness-kit-install/templates/.graphifyignore.base.template`
   tal cual a `.graphifyignore`. **No lo edites**: contiene los secretos, la
   salida del grafo, la memoria volátil y la fontanería. Ninguna de esas
   reglas depende del proyecto.

   **B.2 — Fragmentos del stack.** Del stack detectado en la Fase A, copia bajo
   el BLOQUE B los fragmentos que correspondan de
   `harness-kit-install/graphifyignore-library/`, cada uno precedido de
   `# ── de graphifyignore-library/<fichero>` para que su origen quede
   trazable.
   - Perfil **CODE**: incluye `_code-sources.txt` y quédate solo con las
     extensiones de los lenguajes que el proyecto usa realmente — el MCP
     se instala siempre en este perfil (3.5.2), así que excluir el código de
     graphify no lo deja sin cobertura.
   - Perfil **DOC**: **omite `_code-sources.txt`** — con menos de 20 ficheros de
     código (o siendo un proyecto de infraestructura), el mismo grafo cubre
     ambos dominios y no hay MCP que tome el relevo.
   - Perfil **MIXTO**: **omite `_code-sources.txt` por defecto.** En este
     perfil el MCP es opcional y su instalación se decide en 3.5.2, *después*
     de este paso — si aquí ya excluyeras el código de graphify "por si acaso"
     se instala el MCP, y luego se decide que no, el código quedaría sin
     cobertura en ningún grafo (bug real, detectado simulando este caso en dos
     proyectos reales). Deja que graphify cubra el código como opción segura
     por defecto. Si 3.5.2 termina instalando el MCP, vuelve aquí, añade
     `_code-sources.txt` al Bloque B y reconstruye graphify — no antes.
   - **Caso infraestructura (Fase A punto 0.d):** si el perfil DOC se propuso
     porque el proyecto es de infraestructura/configuración, **no copies
     fragmentos que excluyan sus propios manifiestos** — en particular, **no**
     copies `docker.txt` si el proyecto es principalmente `docker-compose.yml`/
     `Dockerfile` de gestión (eso excluiría justo el contenido que el punto
     0.d acaba de declarar como la sustancia real a indexar). El criterio es:
     un fragmento del Bloque B nunca debe excluir lo que la Fase A ya
     identificó como conocimiento del proyecto, aunque ese fragmento exista
     para otros stacks donde ese mismo patrón sí es ruido (p. ej. un
     `docker-compose.yml` de desarrollo en un proyecto de aplicación normal).

   **B.3 — Reglas propias.** Recorre las carpetas de primer nivel no cubiertas
   por A y B y clasifícalas:

   | Situación | Acción |
   | --- | --- |
   | Assets, datos generados, multimedia | excluir |
   | Documentación, planes, specs, decisiones | **NO excluir** |
   | Ambigua (nombre poco claro o contenido mixto) | **PREGUNTAR al usuario** |

   Para cada carpeta ambigua muestra nombre, nº de ficheros, extensiones
   predominantes y una línea de qué parece contener; pregunta si es conocimiento
   del proyecto o material auxiliar.

   Con los ficheros de texto plano grandes de la raíz (`*.txt`, `*.md` sueltos):
   **nunca los excluyas por tamaño**. Un `.txt` de 20 KB puede ser ruido o puede
   ser la especificación funcional del producto. Pregunta siempre.

   Cada regla del BLOQUE C lleva un comentario con su motivo, **incluidas las
   decisiones de NO excluir** que hayas confirmado con el usuario. Nunca añadas
   una regla «por si acaso».

   Muestra el fichero compuesto al usuario antes de continuar.

c) Ejecuta `/graphify` y **verifica que termina**. Deben existir:
   - `graphify-out/.graphify_python`
   - `graphify-out/graph.json`
   - `graphify-out/GRAPH_REPORT.md`

d) **PUERTA DE CALIDAD — no continuar sin pasarla.** Abre
   `graphify-out/GRAPH_REPORT.md` y comprueba los god nodes:

   | Resultado | Significado | Acción |
   | --- | --- | --- |
   | Documentos (`systemPatterns.md`, `STATE.md`, ficheros de `docs/`) | correcto | continuar |
   | Ficheros de código (`.ts`, `.vue`, `.sol`…) en perfil CODE/MIXTO | ⛔ el `.graphifyignore` no se aplicó | revisar y reconstruir |
   | El corpus reporta menos ficheros que los `.md` de `.harness/` | ⛔ `.harness/` sigue excluido | revisar sección A del ignore |
   | God nodes sobre el propio graphify ("Pipeline de graphify", "--update Incremental...") en vez de abstracciones del proyecto | ⛔ `.harness/skills/graphify/` no se excluyó (verificado en producción: 2 de 10 god nodes fueron documentación del skill, no del proyecto) | confirmar que Bloque A.5 del `.graphifyignore` sigue teniendo `.harness/skills/graphify/` y reconstruir |

   Reporta el resultado al usuario antes de seguir.

e) Configura el versionado **selectivo** del grafo. Añade al `.gitignore`:

   ```gitignore
   graphify-out/*
   !graphify-out/graph.json
   !graphify-out/GRAPH_REPORT.md
   !graphify-out/.graphify_labels.json
   !graphify-out/manifest.json
   ```

   Y al `.gitattributes` (créalo si no existe):

   ```gitattributes
   graphify-out/graph.json    -diff -merge
   graphify-out/manifest.json -diff -merge
   ```

   Explica el motivo al usuario: el grafo de conocimiento cuesta tokens de
   reconstruir, así que viaja con el repo. `.graphify_python` y `.graphify_root`
   contienen rutas absolutas de esta máquina y **nunca** se versionan.

#### 3.5.2 — codebase-memory-mcp (solo perfiles CODE y MIXTO-con-MCP)

a) Comprueba si el binario ya está disponible:
   ```bash
   codebase-memory-mcp --version
   ```

b) Si no lo está, **no ejecutes el instalador oficial. Instala el binario a mano.**

   > ⛔ **`--skip-config` no es fiable.** El instalador oficial delega en el
   > binario descargado (`install -y --force --dir=… --skip-config`), pero
   > releases publicados pueden ignorar esos flags en silencio y configurar
   > 8-9 agentes globalmente (Claude Code, Gemini CLI, VS Code, Cursor, etc.)
   > pese a habérselo pedido explícitamente. Detalle verificado en
   > `info-references/codebase-memory-mcp-referencia.md`.

   **Instalación manual (dos comandos, cero escrituras de configuración):**

   ```bash
   mkdir -p "$LOCALAPPDATA/Programs/codebase-memory-mcp"
   cp <ruta-al-exe> "$LOCALAPPDATA/Programs/codebase-memory-mcp/"
   ```

   Y añadir esa carpeta al PATH de usuario. En POSIX, equivalente en `~/.local/bin`.

   > ⛔⛔ **NUNCA invoques `codebase-memory-mcp install`.** En versiones donde
   > el subcomando no valida flags desconocidos, cualquier errata en un flag
   > (incluido `--help`) puede ejecutar una instalación completa. El binario es
   > autocontenido: `install` solo aporta la autoconfiguración de agentes que
   > precisamente no queremos. Copiar el `.exe` y declararlo en el `.mcp.json`
   > del proyecto es todo lo necesario.
   >
   > Si en algún momento hay que comprobar el estado de configuración de
   > agentes, el único comando seguro es `codebase-memory-mcp install --dry-run -n`.
   > Para revertir una instalación accidental, ver el procedimiento de limpieza
   > por fichero en `info-references/codebase-memory-mcp-referencia.md`.

c) Crea o **fusiona** `.mcp.json` en la raíz desde
   `harness-kit-install/templates/.mcp.json.template`. Si ya existe, **añade la
   entrada conservando los servidores existentes** — nunca sobrescribas el
   fichero.

   > ⚠️ Si usas ruta absoluta en Windows, **forward slashes obligatorios**.
   > Con backslashes el JSON es inválido (`\D`, `\c` no son escapes válidos)
   > y el servidor no arranca. Valida siempre:
   > `node -e "JSON.parse(require('fs').readFileSync('.mcp.json','utf8'))"`

d) Pide al usuario que reinicie el agente y verifique con `/mcp` que
   `codebase-memory` aparece.

e) **Mide el coste de contexto.** Pide `/context` antes y después. Anota el
   delta en `.harness/PROFILE.md`. Si el perfil es MIXTO y el delta resulta
   desproporcionado, propón desinstalar y quedarse solo con graphify.

f) Indexa el código y aplica la **PUERTA DE CALIDAD**:

   | Métrica | Esperado | Alarma |
   | --- | --- | --- |
   | Ficheros indexados | del orden del conteo de la Fase A/B | > 10× → entran dependencias |
   | Tiempo | segundos | minutos → ídem |

   Si entran dependencias: `delete_project` y reindexar apuntando a las
   carpetas de fuentes concretas.

g) Verifica que el lenguaje principal se parsea de verdad:
   `search_graph("<símbolo real conocido del proyecto>")`.
   Solidity, Vue SFC y los imports remotos de Deno son casos donde la
   cobertura no está garantizada — si falla, anótalo en `.harness/PROFILE.md`
   y considera devolver ese lenguaje al grafo de graphify.

> Nota: graphify es Python puro (no usa npm/pnpm) y `codebase-memory-mcp` es
> un binario estático sin dependencias de runtime. Son independientes entre sí.

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
- `harness-kit-install/templates/docs/KNOWLEDGE_GRAPHS.md.template` a `docs/KNOWLEDGE_GRAPHS.md`
- Si el perfil es CODE o MIXTO-con-MCP, copia también
  `harness-kit-install/templates/skills/doc-code-audit/SKILL.md` a
  `.claude/skills/doc-code-audit/SKILL.md` y `.agents/skills/doc-code-audit/SKILL.md`

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

> ⚠️ **Comprueba `package.json` antes de nombrar el fichero.** Si declara
> `"type": "module"`, el proyecto es ESM puro y un `init.js` con `require()`
> falla con `ReferenceError: require is not defined in ES module scope` en
> cuanto se ejecuta. En ese caso, crea el fichero como `init.cjs` en su lugar
> (mismo contenido de la plantilla, solo cambia la extensión) y usa `node
> init.cjs` en el resto de esta guía y en `AGENTS.md`. Aplica el mismo
> criterio a `scripts/harness-start.js`/`harness-finish.js`/
> `harness-link-skills.js` si el proyecto es ESM.

Verifica que al ejecutar `node init.js` (o `node init.cjs`, según lo anterior) termina con `[HARNESS OK]` antes de continuar.

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
