# Skill: load-memory
> Skill AI-agnóstico — compatible con Claude, Gemini, Qwen, Kimi y cualquier agente LLM
> Versión: 2.0 (Optimizado para contexto)

## Propósito
Cargar el contexto del proyecto al iniciar una sesión de trabajo, reconstruyendo el estado en el menor número de tokens posible.

---

## NIVEL 1 — FAST LOAD (Ejecutar siempre al inicio de sesión)

Leer en orden estos ficheros:
1. `.harness/context/activeContext.md` — ¿dónde estamos? ¿qué bloqueantes hay? ¿hay tarea en progreso?
2. `.harness/context/progress.md` — ¿en qué estado están los módulos?

> ⚡ **FAST LOAD EXTREME:** Por defecto, lee únicamente estos 2 ficheros si no necesitas navegar planes o tareas.
> Si necesitas conocer los planes del proyecto, lee opcionalmente:
> 3. `.harness/context/plans-index.md` — ¿qué planes hay activos?

Tras leer los ficheros de FAST LOAD, muestra este resumen estructurado:

```
Contexto cargado: [proyecto] | Rama: [rama] | Próximo: [próximo paso] | Bloqueantes: [ninguno/lista]
```

---

## NIVEL 1.5 — MAPA DEL PROYECTO (Bajo Demanda)

Si necesitas orientarte sobre la organización de los ficheros y dependencias del código:
- **Navegación ultra-ligera (Recomendado):** Lee `graphify-out/GRAPH_REPORT_MIN.md` para entender de un vistazo los módulos principales (*Community Hubs*) y ficheros críticos (*God Nodes*).
- **Listado completo:** Lee `graphify-out/GRAPH_REPORT.md` solo si necesitas una vista detallada de todo el grafo.
- **Búsqueda quirúrgica:** Ejecuta `graphify query "<pregunta>"` en lugar de escanear directorios enteros.

---

## NIVEL 2 — FULL LOAD (Bajo Demanda de Arquitectura/Negocio)

Activar solo si el usuario pregunta sobre arquitectura profunda, decisiones técnicas o diseño de producto.

Leer en orden:
4. `.harness/context/projectbrief.md`
5. `.harness/context/productContext.md`
6. `.harness/context/systemPatterns.md`
7. `.harness/context/techContext.md`

Mostrar resumen adicional con: visión del proyecto, stack, última decisión arquitectónica.

---

## Reglas del skill
- No inventar información que no esté en los ficheros.
- Si un fichero no existe, indicarlo en silencio y continuar con los demás.
- No pasar a FULL LOAD sin que el usuario lo solicite explícitamente.
- Si activeContext.md indica "workspace activo: X", avisar al usuario de que hay una tarea en curso.
