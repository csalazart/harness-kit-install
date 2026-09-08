# Prompt 06 — Migración al Harness Lean (estructura nueva)
> Copia y pega este prompt en un proyecto que usaba la estructura anterior del Harness.
> La IA escaneará los ficheros viejos, construirá un plan de migración y lo ejecutará tras tu confirmación.

---

```
ASUNTO: Migrar proyecto desde estructura Harness antigua a estructura Lean

Actúa como un Harness Migration Engineer. Tu misión es detectar los ficheros
de la estructura ANTIGUA del Harness en este proyecto y migrar su contenido
a la estructura LEAN actual, sin perder ninguna información relevante.

---

## MAPA DE DATOS — Referencia obligatoria

Antes de tocar nada, interioriza este mapa. Define exactamente qué leer y dónde escribir.

### GRUPO 1 — Tareas y backlog

| Fichero origen (viejo)              | Sección / campo                        | Destino (nuevo)                        | Transformación                                                              |
|-------------------------------------|----------------------------------------|----------------------------------------|-----------------------------------------------------------------------------|
| `.harness/feature_backlog.json`     | Array de tareas                        | `.harness/STATE.md` → Backlog          | Cada ítem → `- [ ] AREA: descripción` (o `- [x]` si status=done)           |
| `.harness/STATUS.md`                | Tareas pendientes                      | `.harness/STATE.md` → Backlog          | Extraer líneas de tareas, convertir al formato checkbox                     |
| `.harness/STATUS.md`                | Decisiones / bloqueos                  | `.harness/STATE.md` → Decisiones       | Copiar literalmente bajo la sección correspondiente                         |
| `.harness/CHECKPOINTS.md`           | Checkpoints pendientes                 | `.harness/STATE.md` → Backlog          | Cada checkpoint no completado → `- [ ] checkpoint: descripción`             |
| `.harness/CHECKPOINTS.md`           | Checkpoints completados                | `.harness/logs/SUMMARY.md`             | Una línea por checkpoint: `### FECHA — NOMBRE — ✅ COMPLETADO`              |

### GRUPO 2 — Trabajo en curso

| Fichero origen (viejo)              | Sección / campo                        | Destino (nuevo)                        | Transformación                                                              |
|-------------------------------------|----------------------------------------|----------------------------------------|-----------------------------------------------------------------------------|
| `.harness/current.md`               | Tarea activa (objetivo)                | `.harness/workspaces/{nombre}/orden.md`| Crear workspace con `node scripts/harness-start.js --task {nombre}`; pegar objetivo y criterios en orden.md |
| `.harness/current.md`               | Notas de trabajo / progreso            | `.harness/workspaces/{nombre}/session.md` | Pegar en sección "Diario de Ejecución"                                   |
| `.harness/scratchpad.md`            | Notas relacionadas con tarea activa    | `session.md` del workspace activo      | Solo lo relevante para la tarea en curso                                    |
| `.harness/scratchpad.md`            | Decisiones o hallazgos vigentes        | `.harness/STATE.md` → Decisiones       | Las que siguen siendo relevantes hoy                                        |
| `.harness/scratchpad.md`            | Notas temporales sin valor actual      | **Descartar**                          | No migrar ruido                                                             |

### GRUPO 3 — Historial

| Fichero origen (viejo)              | Sección / campo                        | Destino (nuevo)                        | Transformación                                                              |
|-------------------------------------|----------------------------------------|----------------------------------------|-----------------------------------------------------------------------------|
| `.harness/history.md`               | Entradas de tareas completadas         | `.harness/logs/SUMMARY.md`             | Convertir cada entrada al formato: `### FECHA — NOMBRE — ✅ COMPLETADO\nresumen\n` (más reciente primero) |
| `.harness/history.md`               | Sesiones con detalle relevante         | `.harness/logs/{FECHA}_{nombre}.md`    | Un fichero por sesión con detalle importante; descartar las triviales       |
| `.harness/SUMMARY.md` (si existe)   | Entradas existentes                    | `.harness/logs/SUMMARY.md`             | Mover tal cual, ajustar formato si es necesario                             |

### GRUPO 4 — Agentes y reglas

| Fichero origen (viejo)              | Destino (nuevo)                        | Transformación                                                              |
|-------------------------------------|----------------------------------------|-----------------------------------------------------------------------------|
| `.harness/rules/leader.md`          | `.harness/rules/orchestrator.md`       | Renombrar; adaptar secciones al template `AGENT_00_ORCHESTRATOR.template.md`; añadir sección "Cómo lanzar un sub-agente" con harness-start.js |
| `.harness/rules/implementer.md`     | `.harness/rules/{modulo}.md`           | Si hay reglas genéricas → crear un `specialist.md`; si hay reglas por área → dividir en un fichero por área |
| `.harness/rules/reviewer.md`        | `.harness/rules/git-reviewer.md`       | Renombrar; si el contenido es pobre, reemplazar por `harness-kit-install/rules-library/agent-git-reviewer.md` |
| Reglas técnicas de stack (si existen)| `.harness/rules/{stack}.md`           | Mantener; si no existen, copiar las relevantes de `harness-kit-install/rules-library/` |

### GRUPO 5 — Infraestructura y documentación

| Fichero origen (viejo)              | Destino (nuevo)                        | Transformación                                                              |
|-------------------------------------|----------------------------------------|-----------------------------------------------------------------------------|
| `init.sh`                           | `init.js`                              | Reemplazar con `harness-kit-install/templates/init.js.template` adaptado al stack detectado |
| `CLAUDE.md` (incompleto/antiguo)    | `CLAUDE.md`                            | Actualizar con `harness-kit-install/templates/CLAUDE.md.template`; preservar Descripción, Stack y Notas del dominio existentes |
| `AGENTS.md` (antiguo)               | `AGENTS.md`                            | Reemplazar con `harness-kit-install/templates/AGENTS.md.template` adaptado; el contenido relevante ya habrá migrado a las rules/ |
| `README.md` / `docs/`              | `CLAUDE.md` → sección Notas del dominio | Extraer: decisiones de arquitectura vigentes, convenciones del equipo, restricciones de negocio |

---

## FASE A — Escáner de estructura antigua

Ejecuta este inventario ANTES de tocar nada:

1. Busca los ficheros del MAPA DE DATOS en el proyecto (usa `ls`, `find` o `tree`).
2. Para cada fichero encontrado, lee su contenido completo.
3. Produce una tabla de inventario:

```
| Fichero encontrado | Tiene contenido | Notas (qué contiene en 1 línea) |
|--------------------|-----------------|----------------------------------|
| ...                | Sí / No         | ...                              |
```

4. Identifica si ya existe estructura NUEVA (`.harness/STATE.md`, `orchestrator.md`, etc.)
   — Si ya existe → modo MERGE (no sobreescribir, solo añadir lo que falta)
   — Si no existe → modo FULL MIGRATION

Muéstrame el inventario y espera mi confirmación antes de continuar.

---

## FASE B — Plan de migración

Con el inventario confirmado, genera un plan concreto:

```
PLAN DE MIGRACIÓN
=================
Modo: FULL MIGRATION | MERGE (indicar cuál)

ACCIONES:
1. [CREAR]   .harness/STATE.md — desde feature_backlog.json + STATUS.md (X tareas)
2. [CREAR]   .harness/logs/SUMMARY.md — desde history.md (X entradas)
3. [CREAR]   workspace activo para "{nombre}" — desde current.md
4. [RENOMBRAR] leader.md → orchestrator.md (con adaptaciones)
5. [REEMPLAZAR] reviewer.md → git-reviewer.md (desde rules-library)
6. [DESCARTAR] scratchpad.md (ruido sin valor actual)
7. ...

FICHEROS VIEJOS A ELIMINAR TRAS MIGRACIÓN:
- .harness/current.md
- .harness/history.md
- .harness/scratchpad.md
- .harness/feature_backlog.json
- .harness/STATUS.md
- .harness/CHECKPOINTS.md
- init.sh
- .harness/rules/leader.md
- .harness/rules/implementer.md
- .harness/rules/reviewer.md
```

Muéstrame el plan y espera mi OK antes de ejecutar.

---

## FASE C — Ejecución de la migración

Tras recibir OK, ejecuta cada acción del plan en orden.

Reglas durante la ejecución:
- **Nunca sobreescribir** un fichero nuevo que ya tenga contenido — hacer MERGE
- **Nunca descartar** sin confirmar primero con el usuario si hay duda
- **STATE.md**: primero escribe las tareas completadas al final con `[x]`, luego las pendientes con `[ ]`
- **SUMMARY.md**: entradas más recientes primero; si no hay fecha exacta, usa "YYYY-MM (aprox)"
- **orchestrator.md**: usar `harness-kit-install/templates/agents/AGENT_00_ORCHESTRATOR.template.md` como base; no perder reglas específicas del proyecto del leader.md viejo
- **Al crear workspaces**: ejecutar `node scripts/harness-start.js --task {nombre}` en vez de crear carpetas a mano

Tras cada fichero creado, confirma en el chat: `✅ {fichero} — {1 línea de qué contiene}`

---

## FASE D — Verificación y cierre

1. Ejecuta `node init.js` — debe terminar con `[HARNESS OK]`
2. Muestra el árbol final de `.harness/` y ficheros raíz
3. Verifica que STATE.md tiene al menos una tarea pendiente con `[ ]`
4. Elimina los ficheros viejos del plan (solo tras confirmación tuya)

5. **Cierre del área de migración**

   `migracion/` es un área de trabajo temporal: su `backup/` recolectó la
   información del sistema antiguo para volcarla en la estructura nueva. Si la
   migración se ha verificado, esa información ya vive en su destino definitivo
   y la carpeta es un duplicado.

   Antes de proponer nada, **verifica que el volcado está completo**:

   - `node init.js` termina en `[HARNESS OK]` (paso 1)
   - `.harness/STATE.md` contiene las tareas de `backup/02_TAREAS_PENDIENTES.md`
     y `backup/03_TAREAS_COMPLETADAS.md`
   - `.harness/context/` refleja lo que había en `backup/01_ESTADO_PROYECTO.md`
   - `.harness/rules/` recoge lo de `backup/04_MEMORIA_Y_REGLAS.md`
   - Ningún fichero de `backup/files/` contiene información que no esté ya
     representada en la estructura nueva

   Si algo no cuadra, **detente y repórtalo** — no propongas eliminar nada.

   Si todo cuadra, muestra al usuario el resumen (nº de ficheros que ocupa
   `migracion/`, qué se verificó) y ofrécele tres opciones:

   a) **Archivar fuera del repo** (recomendado): comprimir `migracion/` a
      `../<proyecto>-migracion-<fecha>.zip` y eliminar la carpeta.
   b) **Eliminar** `migracion/` directamente.
   c) **Conservarla** — en tal caso, añade `migracion/` al `.gitignore` del
      proyecto para que no siga versionándose.

   **Espera su decisión. Nunca elimines sin confirmación explícita.**

6. Haz commit: `"chore: migrate harness to lean workspace architecture"`

---

## Reglas generales durante todo el proceso

- Si un fichero viejo no está en el MAPA DE DATOS → pregunta antes de hacer algo con él
- Si el contenido de un fichero viejo está vacío o es trivial → proponer descartarlo
- Si hay conflicto entre datos viejos y nuevos → mostrar ambos y preguntar cuál prevalece
- Si no hay estructura vieja detectable → informar y ofrecer ejecutar `01_init.md` en su lugar

¿Entendido? Empieza con la Fase A.
```
