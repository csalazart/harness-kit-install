# Prompt 05 — Resolver decisiones críticas bloqueantes
> Usa este prompt cuando haya decisiones en STATE.md que bloquean el avance.

---

```
ASUNTO: Resolver decisión crítica bloqueante

Hay una decisión pendiente que está bloqueando el avance del proyecto.
Necesito tu ayuda para resolverla de forma informada.

## La decisión a resolver

ID: [D00X]
Descripción: [pegar de .harness/STATE.md sección "Decisiones y Bloqueos Activos"]
Impacta a: [agentes impactados]

## Lo que necesito de ti

1. Lee los ficheros relevantes para esta decisión:
   [indicar qué ficheros tienen contexto — ej: docs/arquitectura.md, .harness/rules/backend.md]

2. Preséntame las opciones disponibles en una tabla:
   | Opción | Descripción | Ventajas | Desventajas | Cuándo elegir |
   |--------|-------------|----------|-------------|---------------|

3. Dame tu recomendación con justificación técnica de 2-3 líneas

4. Explícame los impactos concretos de la decisión:
   - ¿Qué cambia en el código?
   - ¿Qué cambia en otros agentes?
   - ¿Es reversible después?

5. Espera mi decisión final

## Después de que yo decida

1. Actualizar .harness/STATE.md:
   - Mover la decisión de "Decisiones y Bloqueos Activos" a una nueva sección "Decisiones Tomadas"
   - Anotar el valor decidido y la fecha
   - Anotar los impactos cruzados resultantes

2. Actualizar .harness/rules/{módulo}.md si la decisión implica una nueva regla permanente para ese agente

3. Informar qué agente debe actuar primero como consecuencia de esta decisión
```
