# Prompt 07 — Actualización del Harness Kit v3 → v4 (grafo dual)
> Copia y pega este prompt en un proyecto que ya tiene el Harness Kit **v3** instalado
> (Lean, con `.harness/STATE.md`, `context/`, `rules/`, posiblemente `llm-wiki/` de
> harness-okf) y quiere pasar a **v4** (grafo dual: `codebase-memory-mcp` para código +
> graphify para conocimiento).
>
> **No uses `01_init.md` para este caso.** `01_init.md` asume terreno virgen o completa
> lo que falte sin más — no sabe distinguir "proyecto nuevo" de "proyecto con v3 ya
> funcionando cuyo backlog y decisiones no se pueden perder". Este prompt sí.
>
> **Caso puntual, no el flujo dominante.** La instalación normal de v4 (`01_init.md`)
> asume un proyecto sin harness-kit previo — es el caso dominante (~90-98%). Este
> prompt cubre el caso específico contrario: hay una instalación v3 real que
> preservar y actualizar, no borrar y empezar de cero.

---

```
ASUNTO: Actualizar este proyecto del Harness Kit v3 (Lean) al v4 (grafo dual)

Actúa como un Harness Migration Engineer. Tu misión es extraer todo lo que este
proyecto ya tiene de la instalación v3 (tareas, decisiones, memoria, reglas,
cerebro OKF si existe), respaldarlo, y luego construir la capa nueva de v4
(perfilado, grafo dual, MCP condicional) sin perder ni un dato de lo que ya
existía. Nunca elimines nada sin backup previo y confirmación explícita.

---

## MAPA DE DATOS — qué pasa de v3 a v4 y cómo

| Elemento v3 | ¿Cambia de formato en v4? | Qué hacer |
|---|---|---|
| `.harness/STATE.md` | No | Se preserva TAL CUAL — v4 no toca el formato del backlog |
| `.harness/context/*.md` (7 ficheros) | No | Se preserva TAL CUAL |
| `.harness/rules/*.md` | No | Se preserva TAL CUAL |
| `.harness/llm-wiki/` (si harness-okf instalado) | No | Se preserva TAL CUAL — v4 no cambia el cerebro OKF, solo hace que ahora SÍ se indexe (era el bug P1) |
| `.harness/workspaces/`, `.harness/logs/` | No | Se preservan TAL CUAL |
| `.harness/skills/` (skills propios del proyecto, p.ej. de harness-okf: `okf-ingest`, `okf-lint`, `okf-query`) | No | Se preserva TAL CUAL — es dato/capacidad del proyecto, no fontanería del kit v4 |
| `.harness/scripts/` (scripts propios del proyecto o de harness-okf, p.ej. `okf-check.*`, `okf-link-skills.*`) | No | Se preserva TAL CUAL — **no confundir con `scripts/` en la raíz** (esa sí es fontanería del kit, ver más abajo). Si tienes dudas sobre si un fichero de `.harness/scripts/` es del kit o del proyecto, pregunta antes de tocarlo |
| Documentos de decisión previos del proyecto (`docs/*.md`, `working-plan/*.md`, `plan/*.md`, `planes/*.md`) | No | **Se leen, no solo se cuentan** — pueden contener una decisión ya tomada sobre perfil/MCP que el paso de perfilado (FASE B) debe respetar o contrastar, no ignorar (ver FASE A punto 4) |
| `AGENTS.md` / `CLAUDE.md` | Sí, parcialmente | **Backup primero** (van a editarse por fusión, mismo riesgo que justifica el backup de STATE/context/rules), luego **fusionar**: el Paso 0.5 y la jerarquía de búsqueda se actualizan al enrutamiento por dominio; cualquier sección propia del proyecto (ajena al kit) se conserva íntegra. De paso, verifica que no queden placeholders `{{...}}` sin resolver de la instalación v3 original — repórtalo si los hay, no lo arregles sin preguntar |
| `.graphifyignore` (v3) | Sí, por completo | **Se sustituye** por la composición en 3 capas de v4 (Bloque A/B/C). El v3 casi seguro excluye `.harness/` (bug P1) — no se fusiona, se recompone desde cero |
| `graphify-out/` (si existe) | Se reconstruye | **No se preserva.** Se generó con un filtro con bug; regla ya establecida del kit — "ante conflicto, regenerar, no mergear" |
| `.mcp.json` (si existe, de otro uso previo) | Se revisa | Se **fusiona** con la entrada `codebase-memory` si el perfil la requiere; nunca se sobrescribe |
| `.harness/PROFILE.md` | Nuevo en v4 | No existía en v3 por definición — se crea con el perfilado calculado fresco |
| `init.js` / `init.sh` / `scripts/harness-*` / `scripts/query-graph.*` / `scripts/update-graph.*` | Sí | Se sustituyen por las versiones v4 — sin backup, son fontanería del kit, no datos del proyecto. **Comprueba `package.json` antes de decidir la extensión**: si tiene `"type": "module"`, los scripts de Node van en `.cjs`, no `.js` (un `.js` con `require()` en un paquete ESM rompe al ejecutarse) — ver FASE A punto 5 |
| `.claude/skills/graphify/`, `.agents/skills/graphify/` (si ya instalado desde v3) | Puede cambiar | Re-copia desde `harness-kit-install/templates/skills/graphify/` igual que el resto de fontanería — no lo des por "ya instalado, no tocar" si el contenido del skill cambió entre versiones del kit |
| `docs/GRAPHIFY_GUIDE.md`, `docs/AGENTS_REFERENCE.md` | Sí, parcialmente | Se actualizan/crean desde las plantillas v4; `docs/KNOWLEDGE_GRAPHS.md` es nuevo |

**Principio que ordena toda la tabla:** se hace backup y se preserva todo lo que
es *dato del proyecto* (STATE, context, rules, llm-wiki). Se sustituye sin
preguntar lo que es *fontanería del kit* (init.js, init.sh) porque es idéntico
en todos los proyectos. Se recompone desde cero lo que *cambia de mecanismo*
en v4 (`.graphifyignore`, `graphify-out/`) porque fusionar un filtro con bug
con uno correcto no tiene sentido.

---

## FASE A — Escáner de la instalación v3 existente

1. **Confirma que este prompt aplica:** debe existir `.harness/` **sin**
   `.harness/PROFILE.md`. Si ya existe `PROFILE.md`, este proyecto ya está en
   v4 (o a medio migrar) — detente y dilo, no hay nada que escanear.

   **No busques solo en el directorio donde te invocaron.** En un monorepo,
   `.harness/` puede vivir varios niveles por debajo (o por encima) de la
   raíz real del `.git/`. Antes de concluir "no aplica, no hay `.harness/`",
   busca en todo el árbol del repositorio (mismo criterio que `01_init.md`
   Fase A punto 0.a, cuarto caso). Si encuentras `.harness/` en una subcarpeta
   y no en la raíz que te pidieron analizar, **pregunta el alcance antes de
   seguir**: ¿se migra solo esa subcarpeta en su sitio, o se promueve
   `.harness/` a la raíz del monorepo? La respuesta cambia el perfilado y
   dónde viven `.mcp.json`/`.graphifyignore`/`migracion-v4/` — no lo decidas
   por tu cuenta, ni asumas que "no hay `.harness/` aquí" significa que este
   prompt no aplica.

2. **Inventario real** — para cada fila de la tabla de arriba, comprueba si
   existe y con qué tamaño/nº de ficheros. Verifica en particular:
   - ¿`.graphifyignore` (v3) excluye `.harness/`? (línea con `.harness/` bajo
     "CARPETAS DE CONTROL DE AGENTES") — si sí, es la evidencia de que la
     migración corrige un bug real, no un capricho de versión.
   - ¿`graphify-out/` existe? ¿Tiene `graph.json` ya construido con el filtro
     viejo?
   - ¿Hay contenido en `AGENTS.md`/`CLAUDE.md` que no venga de ninguna
     plantilla del kit (secciones propias del proyecto)? ¿Quedan placeholders
     `{{PROJECT_NAME}}`/`{{AREA}}`/etc. sin resolver de la instalación v3?
   - **Lee el contenido de `STATE.md` y de `llm-wiki/` (si existe), no solo
     cuentes que existen.** Distingue un backlog real (tareas concretas,
     decisiones con contexto) de andamiaje nunca operado (placeholders sin
     rellenar, `workspaces/`/`logs/` vacíos). Esto importa para el paso 7 del
     plan de la FASE B — preservar algo real no es lo mismo que preservar un
     placeholder, y hay que poder distinguirlos al reportar qué se conservó.

3. Produce una tabla de inventario:

```
| Elemento | Existe | Tamaño/Nº ficheros | Notas |
|---|---|---|---|
| .harness/STATE.md | Sí/No | ... | X tareas pendientes reales (no placeholders), Y decisiones abiertas |
| .harness/context/ | Sí/No | .../7 ficheros | ... |
| .harness/rules/ | Sí/No | N ficheros | ... |
| .harness/llm-wiki/ | Sí/No | N ficheros | harness-okf instalado / no; ¿poblado o vacío? |
| .harness/skills/ | Sí/No | N ficheros | ¿cuáles? |
| .harness/scripts/ | Sí/No | N ficheros | ¿del proyecto/harness-okf, o fontanería del kit mal ubicada? |
| .graphifyignore (v3) | Sí/No | — | ¿excluye .harness/? Sí/No |
| graphify-out/ | Sí/No | ... | ¿graph.json presente? |
| .mcp.json | Sí/No | — | ¿qué servidores declara? |
| AGENTS.md/CLAUDE.md | — | — | ¿secciones propias? ¿placeholders sin resolver? |
| package.json | Sí/No | — | ¿"type": "module"? (decide si los scripts van en .cjs o .js) |
```

4. **Busca documentos de decisión previos** — `docs/`, `working-plan/`,
   `plan/`, `planes/`, y cualquier `.md` de la raíz que no sea `AGENTS.md`/
   `CLAUDE.md`/`README.md`. Lee (no solo listes) los que traten sobre grafo
   de conocimiento, perfilado o `codebase-memory-mcp` — un proyecto v3 tiene
   historia, y puede que ya exista una decisión razonada sobre exactamente lo
   que la FASE B va a proponer desde cero. Si encuentras uno, cítalo en el
   inventario y contrasta su conclusión contra la propuesta automática del
   paso 6 antes de presentarla como si fuera la primera vez que se analiza.

5. **Comprueba `package.json`** (si existe): si declara `"type": "module"`,
   los scripts de Node de este proyecto (y los que instalará v4) deben usar
   extensión `.cjs`, no `.js` — un `.js` con `require()` en un paquete ESM
   falla con `ReferenceError: require is not defined in ES module scope` en
   cuanto se ejecuta. Verifica también qué extensión usa la instalación v3
   real (`init.js` vs `init.cjs`, `scripts/harness-*.js` vs `.cjs`) — si ya
   resolvió esto con `.cjs`, la v4 debe mantener el mismo criterio, no
   revertir a `.js` por defecto.

Muéstrame el inventario y **espera mi confirmación** antes de continuar.

---

## FASE B — Plan de migración

Con el inventario confirmado, genera un plan concreto:

```
PLAN DE ACTUALIZACIÓN v3 → v4
==============================

1. [CREAR ÁREA DE TRABAJO] migracion-v4/backup/ — copia de seguridad de todo
   lo que se va a tocar antes de tocarlo

2. [BACKUP] Copiar íntegros a migracion-v4/backup/:
   - .harness/STATE.md
   - .harness/context/*.md
   - .harness/rules/*.md
   - .harness/llm-wiki/ (si existe)
   - .harness/skills/ (si existe)
   - .harness/scripts/ (si existe)
   - AGENTS.md, CLAUDE.md (van a fusionarse en el paso 9 — se respaldan antes
     por el mismo motivo que STATE/context/rules: una fusión que sale mal
     puede perder contenido)
   - .graphifyignore (v3, como referencia histórica del bug)
   - .mcp.json (si existe)
   - graphify-out/graph.json + GRAPH_REPORT.md (si existen, como referencia —
     NO se reutilizan, solo quedan de constancia de "así estaba antes")

3. [EXTRAER RESUMEN] Generar migracion-v4/backup/RESUMEN_ESTADO_PREVIO.md con:
   - Tareas pendientes y completadas de STATE.md (recuento, no solo copia)
   - Decisiones activas y bloqueos de STATE.md
   - Un resumen de 3-5 líneas de qué sabía el proyecto en llm-wiki/ (si existe)
   Esto es una RED DE SEGURIDAD adicional a la copia íntegra del paso 2 — si algo
   en el paso 2 fallara o un fichero resultara estar corrupto, este resumen
   permite reconstruir el backlog a mano sin perder el trabajo.

   > Punto de no retorno: todo lo que sigue asume que el backup de los pasos
   > 1-3 ya está confirmado y completo. **Espera mi OK explícito aquí** antes
   > de tocar nada — es el gate de confirmación de esta FASE B, correspondiente
   > al paso "una vez que se confirme toda la información por el usuario y se
   > dé la orden" que describiste al pedir este mecanismo.

4. [ELIMINAR/CAMBIAR — datos y procesos viejos que ya no se van a usar]
   - `.graphifyignore` (v3): eliminar (si existe). No se fusiona con el nuevo
     — es el fichero con el bug (excluye `.harness/`); recomponerlo desde
     cero en 3 capas (§3.5.1.b de `01_init.md`) es más seguro que intentar
     arreglarlo.
   - `graphify-out/`: eliminar por completo (si existe; grafo construido con
     el filtro viejo — no se fusiona, se reconstruye).
   - Verificación v3 (`init.js`/`init.sh` **o** `init.cjs`, según lo que
     determinaste en la FASE A punto 5): eliminar la versión v3 (verifica
     solo "graphify existe", no el perfil ni el MCP). Si el proyecto usa
     `.cjs` y solo buscas `init.js`, no lo encontrarás y lo dejarás huérfano
     — bórralo por el nombre real que tenga, no por el nombre que esperas.

5. [CAMBIAR LOS SCRIPTS] Sustituir por las versiones v4 de
   `harness-kit-install/templates/`, **con la extensión que corresponda según la
   FASE A punto 5** (`.js` por defecto, `.cjs` si el proyecto tiene
   `"type": "module"` en `package.json` — mantén el mismo criterio que ya
   usaba la instalación v3, no lo reviertas a `.js` por costumbre):
   - `init.js.template` / `init.sh.template` → verificación por perfil
   - `harness-finish.js.template` → **cambió de contenido real en v4**: el
     disparador de `update-graph` ahora es filtrado por rutas de
     conocimiento, no incondicional. No es solo "reinstalar lo mismo" — si
     te saltas este paso, `harness-finish.js` seguiría regenerando el grafo
     en cada cierre de tarea aunque no haya cambiado documentación.
   - `harness-start.js.template`, `query-graph.ps1/sh.template`,
     `update-graph.ps1/sh.template` → sin cambios de lógica en v4, pero
     re-cópialos igualmente para que el proyecto quede en la misma revisión
     exacta del kit, no en una mezcla de v3+v4.
   - `.claude/skills/graphify/`, `.agents/skills/graphify/` (si ya estaban
     instalados desde v3): re-copia desde `harness-kit-install/templates/skills/graphify/`
     por el mismo motivo — no lo saltes solo porque "ya está instalado".

   > **Cuidado con la carpeta `scripts/` completa.** Este paso sustituye
   > ficheros concretos por nombre (los listados arriba), no vacía ni
   > repuebla la carpeta entera. Muchos proyectos guardan lógica de negocio
   > propia en `scripts/` (p. ej. `scripts/mint/`, `scripts/ipfs/`) junto a
   > la fontanería del kit — tócala fichero a fichero, nunca por carpeta
   > completa.

6. [INSTALACIÓN NUEVA] Con lo viejo fuera y los scripts al día, ejecuta lo
   que instala v4 desde cero:
   - [PERFILAR] Fase A punto 0 / Fase B punto 6 de `01_init.md` sobre el
     proyecto real (conteo de código, detección de vendorizado/infra si
     aplica, propuesta de perfil DOC/MIXTO/CODE) — espera mi confirmación
     del perfil igual que en una instalación nueva. **Si la FASE A punto 4
     encontró un documento de decisión previo sobre perfil/MCP, cítalo y
     contrasta tu propuesta contra él explícitamente** — no la presentes
     como si fuera la primera vez que el proyecto se plantea la pregunta.
   - `.graphifyignore` nuevo, compuesto en 3 capas.
   - `.harness/PROFILE.md`: crear con el perfilado de este paso (incluye
     `Kit-Version: 1.4.0` — ver `templates/PROFILE.md.template`).
   - `/graphify` para reconstruir `graphify-out/` desde cero.
   - Si el perfil lo requiere: `codebase-memory-mcp` (§3.5.2 de `01_init.md`).

7. [MIGRAR LOS DATOS DEL BACKUP] Aquí es donde vuelve la información
   respaldada en el paso 2-3 — pero **con matiz honesto, no automático**:
   - `STATE.md`, `.harness/context/*.md`, `.harness/rules/*.md`,
     `.harness/llm-wiki/`: **no cambian de formato entre v3 y v4** (a
     diferencia de la migración estructura-antigua→Lean de `06_migrate.md`,
     que sí transforma formatos). Por eso el "relleno con los datos del
     backup" aquí es una **restauración literal**, no una conversión: cópialos
     de vuelta desde `migracion-v4/backup/` tal cual, sin reescribir su
     contenido.
   - **Si prefieres no tocarlos en absoluto** (opción más segura, mismo
     resultado): sáltate la restauración y dejalos donde están — el paso 4
     de arriba deliberadamente NO los tocó. Backup + restauración solo
     aporta valor real si en el camino decides limpiar/reorganizar algo
     (tareas obsoletas, workspace abandonado) — pregúntame si ese es el caso
     antes de reescribir nada; si no, no hay motivo para mover ficheros que
     ya están bien donde están.

8. [CREAR Y RELLENAR TAREAS AL NUEVO FORMATO] Para v3→v4 esto es, con
   honestidad, **un paso sin trabajo real que hacer**: el formato de
   `STATE.md` (backlog con checkboxes) no cambia en v4. Verifica que el
   `STATE.md` restaurado/preservado sigue teniendo el mismo número de tareas
   `- [ ]`/`- [x]` que el inventario de la Fase A — es la comprobación, no
   una reescritura.

9. [FUSIONAR — nunca sobrescribir]
   - `AGENTS.md`: aplicar el nuevo Paso 0.5 (enrutamiento por dominio) y la
     sección de Mantenimiento; conservar cualquier sección propia del proyecto
     detectada en la Fase A
   - `.mcp.json` (si existía): añadir la entrada `codebase-memory` conservando
     lo que ya hubiera

10. [CREAR — nuevo en v4]
   - docs/KNOWLEDGE_GRAPHS.md (nuevo)
   - docs/GRAPHIFY_GUIDE.md, docs/AGENTS_REFERENCE.md: **fusionar** desde
     `harness-kit-install/templates/docs/*.template` con el mismo criterio que
     AGENTS.md (paso 9) — si ya existen con contenido v3, no dejarlos
     congelados en esa versión; actualiza su alcance/descripción sin perder
     contenido propio del proyecto que pudieran tener.
   - scripts/index-code.ps1 / .sh (solo si el perfil es CODE o MIXTO-con-MCP)
   - .claude/skills/doc-code-audit/, .agents/skills/doc-code-audit/ (solo si
     el perfil es CODE o MIXTO-con-MCP — mismo criterio que 01_init.md §5,
     para que una migración no entregue menos que una instalación nueva)
   - codebase-memory-mcp + .mcp.json (solo si el perfil lo requiere, siguiendo
     §3.5.2 de 01_init.md — incluida la advertencia de NUNCA usar el
     instalador oficial)
```

Muéstrame el plan y **espera mi OK** antes de ejecutar.

---

## FASE C — Ejecución de la migración

Tras recibir OK, ejecuta cada acción del plan en orden.

Reglas durante la ejecución:
- **El backup (pasos 1-3) va SIEMPRE primero**, antes de tocar nada del paso 4
  en adelante. Si el backup falla o queda incompleto, DETENTE y repórtalo —
  no continúes con sustituciones sobre datos sin respaldar.
- **Nunca sobrescribir** `STATE.md`, `context/`, `rules/`, `llm-wiki/` —
  si en algún momento un paso posterior parece que va a tocarlos, para y
  pregunta.
- **`.graphifyignore` y `graphify-out/` se recomponen, no se fusionan** — es
  la única excepción deliberada a "nunca sobrescribir", justificada porque
  ambos son mecanismo/artefacto derivado, no dato del proyecto (ver MAPA DE
  DATOS).
- Tras cada fichero creado/sustituido, confirma en el chat: `✅ {fichero} —
  {1 línea de qué se hizo}`.

---

## FASE D — Verificación y cierre

1. Ejecuta `node init.js` — debe terminar con `[HARNESS OK]`, ahora incluyendo
   la verificación de `.harness/PROFILE.md` y del grafo dual (Bloque 3 v4).
2. Muestra comparación antes/después:
   - `STATE.md`: mismo nº de tareas pendientes que en el inventario de la Fase A
   - `.graphifyignore`: `.harness/` ya NO excluido (corrección del bug P1)
   - `graphify-out/graph.json`: reconstruido, con god nodes documentales
     (puerta de calidad §3.5.1.d de `01_init.md`)
   - `.harness/PROFILE.md`: perfil declarado y decisión sobre el MCP
3. Si el perfil requiere MCP: verifica con `/mcp` que `codebase-memory`
   aparece, y aplica la puerta de calidad de §3.5.2.f (ficheros indexados del
   orden esperado, sin `node_modules` colado).

4. **Cierre del área de migración** (mismo patrón que `06_migrate.md` FASE D,
   aplicado aquí a `migracion-v4/` en vez de `migracion/`):

   Antes de proponer nada, **verifica que el backup sigue siendo redundante**:
   toda la información de `migracion-v4/backup/` debe seguir viva en su
   destino original (`STATE.md`, `context/`, `rules/`, `llm-wiki/` no se
   tocaron — deberían ser idénticos al backup salvo por lo que decidiste
   fusionar en `AGENTS.md`/`.mcp.json`).

   Si algo no cuadra, **detente y repórtalo** — no propongas eliminar nada.

   Si todo cuadra, ofrece las mismas tres opciones que `06_migrate.md`:
   a) **Archivar fuera del repo** (recomendado): comprimir `migracion-v4/` a
      `../<proyecto>-migracion-v4-<fecha>.zip` y eliminar la carpeta.
   b) **Eliminar** `migracion-v4/` directamente.
   c) **Conservarla** — añadir `migracion-v4/` al `.gitignore`.

   **Espera la decisión del usuario. Nunca elimines sin confirmación explícita.**

5. Haz commit: `"chore: upgrade harness-kit v3 → v4 (grafo dual)"`

---

## Reglas generales durante todo el proceso

- Si algo de la Fase A no encaja con el MAPA DE DATOS (un fichero v3 que no
  esperabas, una estructura distinta) → pregunta antes de decidir qué hacer
  con él. No lo fuerces a encajar en una fila de la tabla si no encaja de
  verdad.
- Si `.harness/` existe pero está a medias (instalación v3 incompleta,
  p.ej. sin `rules/`) → documenta el hueco en el inventario de la Fase A,
  pero no lo inventes ni lo completes con contenido genérico en el backup.
- Si detectas un ecosistema de agentes previo que NO es exactamente `.harness/`
  (ver `01_init.md` Fase A punto 0.a, "ecosistema equivalente bajo otro
  nombre") → este prompt no está diseñado para ese caso. Detente y repórtalo;
  no improvises una migración sobre una estructura que no es la que este
  prompt espera.
- Si no hay instalación v3 detectable (no debería llegar a este prompt, pero
  por si acaso) → informa y ofrece ejecutar `prompts/01_init.md` en su lugar.

¿Entendido? Empieza con la Fase A.
```
