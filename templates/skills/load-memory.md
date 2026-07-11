# Skill: load-memory
> Skill AI-agnóstico — compatible con Claude, Gemini, Qwen, Kimi y cualquier agente LLM
> Versión: 1.0

## Propósito
Cargar el contexto del proyecto al iniciar una sesión de trabajo, reconstruyendo el estado en el menor número de tokens posible.

## NIVEL 1 — FAST LOAD (ejecutar siempre al inicio de sesión)

Leer en orden estos 3 ficheros:
1. `.harness/context/activeContext.md` — ¿dónde estamos? ¿qué bloqueantes hay? ¿hay tarea en progreso?
2. `.harness/context/progress.md` — ¿en qué estado están los módulos?
3. `.harness/context/plans-index.md` — ¿qué planes hay activos?

Tras leer los 3, mostrar este resumen estructurado:

---
**Contexto cargado (FAST LOAD)**
- Proyecto: [nombre desde CLAUDE.md o AGENTS.md]
- Rama activa: [desde activeContext.md]
- Último cambio: [desde activeContext.md]
- Próximo paso: [desde activeContext.md]
- Bloqueantes: [desde activeContext.md, o "ninguno"]
- Módulos en progreso: [desde progress.md, formato "módulo (%)"]
- Planes activos: [desde plans-index.md, o "ninguno"]
---

## NIVEL 2 — FULL LOAD (bajo demanda)

Activar solo si el usuario pregunta sobre arquitectura, producto, stack o decisiones técnicas.

Leer en orden:
4. `.harness/context/projectbrief.md`
5. `.harness/context/productContext.md`
6. `.harness/context/systemPatterns.md`
7. `.harness/context/techContext.md`

Mostrar resumen adicional con: visión del proyecto, stack, última decisión arquitectónica.

## Reglas del skill
- No inventar información que no esté en los ficheros
- Si un fichero no existe, indicarlo y continuar con los demás
- No pasar a FULL LOAD sin que el usuario lo solicite explícitamente
- Si activeContext.md indica "workspace activo: X", avisar al usuario de que hay una tarea en curso
