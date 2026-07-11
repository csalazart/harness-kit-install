# Prompt 04 — Protocolo Hand-off (cierre de sesión)
> Ejecutar SIEMPRE antes de terminar una sesión. Garantiza que el workspace queda archivado correctamente.

---

```
ASUNTO: Protocolo Hand-off — Cierre de sesión

Hemos terminado el trabajo de esta sesión. Ejecuta el protocolo de cierre:

## 1. Verificación final

Ejecuta el criterio de verificación de lo que acabas de implementar:
[El agente debe saber cuál es — está en orden.md sección "Criterios de aceptación"]

  node init.js

- Si termina con [HARNESS OK] → continuar con el cierre
- Si termina con [HARNESS FAIL] → NO hacer hand-off. Resolver el problema primero.

## 2. Completar session.md

Rellena la sección "Reporte Final" del workspace con:
- **Status:** COMPLETADO | BLOQUEADO
- **Commit:** mensaje sugerido para el commit
- Impactos cruzados detectados (si los hay)
- Próxima acción recomendada

Marca cada criterio cumplido en orden.md: - [ ] → - [x]

## 3. Commit

Hacer git commit con mensaje descriptivo:
  git commit -m "feat([area]): [descripción breve de lo completado]"

## 4. Notificar al orquestador

Escribe una sola línea al director:
  "done → tarea [nombre] lista para cierre"

El orquestador ejecutará:
  node scripts/harness-finish.js --task [nombre]

Esto archivará session.md en .harness/logs/, marcará la tarea en STATE.md y limpiará el workspace.

## 5. Informe de cierre

Dime:
- ✅ Qué se completó
- 🔜 Próximo agente a abrir y por qué
- ⚡ Impactos cruzados detectados que el orquestador debe conocer
```
