# Skill: update-memory
> Skill AI-agnóstico — compatible con Claude, Gemini, Qwen, Kimi y cualquier agente LLM
> Versión: 1.0

## Propósito
Actualizar los ficheros de contexto del proyecto al cerrar una sesión de trabajo, dejando el estado listo para la próxima sesión.

> Si se ejecutó `harness-finish.js` durante la sesión, `activeContext.md` ya fue actualizado automáticamente.
> Este skill verifica y complementa lo que el script no puede inferir (rama git, decisiones de arquitectura).

## PASO 1 — Ficheros volátiles (ejecutar siempre)

### 1.1 Actualizar activeContext.md
Abrir `.harness/context/activeContext.md` y actualizar:
- **Rama Git:** resultado de `git branch --show-current`
- **Último cambio:** descripción de lo que se hizo en esta sesión
- **Próximo paso:** el siguiente paso lógico según el trabajo realizado
- **Bloqueantes:** cualquier impedimento activo, o "Ninguno"
- Añadir un bullet al "Resumen de última sesión" con timestamp ISO y descripción breve

### 1.2 Actualizar progress.md
Abrir `.harness/context/progress.md` y revisar cada módulo que cambió:
- ⬜ → 🔄 si se empezó a trabajar en él
- 🔄 → ✅ si se completó
- cualquier estado → ❌ si quedó bloqueado
- Actualizar el porcentaje si cambió

### 1.3 Actualizar plans-index.md
Solo si en esta sesión se creó o completó un plan:
- Añadir fila en "Planes activos" si hay plan nuevo
- Mover a "Planes completados" si un plan se cerró

## PASO 2 — Ficheros estables (solo si aplica)

Preguntar al usuario: "¿Cambió la arquitectura, el stack o alguna dependencia en esta sesión?"

Si la respuesta es sí:
- Actualizar `systemPatterns.md` con la decisión tomada (añadir fila a la tabla de decisiones)
- Actualizar `techContext.md` con la dependencia o versión que cambió

> 💡 Si el proyecto tiene instalado **harness-okf** (`.harness/llm-wiki/`), el
> conocimiento de dominio se mantiene con sus propios skills (`okf-ingest`,
> `okf-lint`) — no es responsabilidad de este skill.

## Output final del skill

Mostrar resumen de lo actualizado:
```
Memory actualizado:
✅ activeContext.md — [descripción del cambio]
✅ progress.md — [módulos actualizados, o "sin cambios"]
[✅ systemPatterns.md — solo si aplica]
[✅ techContext.md — solo si aplica]

Próxima sesión comenzará en:
→ [próximo paso desde activeContext.md]
```

## Reglas del skill
- Nunca borrar entradas existentes del "Resumen de última sesión" — solo añadir al principio (máx 5 entradas)
- Nunca escribir valores reales de secrets o passwords en ningún fichero de contexto
- Si el usuario ejecutó `harness-finish.js`, `activeContext.md` ya fue actualizado — solo verificar y completar
