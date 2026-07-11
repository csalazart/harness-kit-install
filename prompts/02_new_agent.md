# Prompt 02 — Crear un nuevo agente especializado
> Usa este prompt cuando necesites añadir un nuevo rol/área al proyecto ya harnessado.

---

```
ASUNTO: Crear nuevo agente especializado para el Harness

Necesito añadir un nuevo agente especializado al proyecto.

## Información que necesito darte (completa lo que aplique):

- **Nombre del agente:** [ej: "Agente 10 — Analytics"]
- **Número/ID:** [ej: AGENT_10]
- **Área de trabajo:** [ej: "Análisis de datos on-chain y dashboards"]
- **Ficheros clave que debe conocer:** [ej: "scripts/analytics/, docs/06_REDES.md"]
- **Depende de:** [ej: "necesita que el contrato esté desplegado — Agente 09"]
- **Otros agentes dependen de él:** [ej: "Agente 05 (portal) necesita los datos"]
- **Stack tecnológico específico:** [ej: "Dune Analytics, The Graph, GraphQL"]
- **Criterio de verificación:** [ej: "dashboard accesible en /analytics"]

## Lo que debes hacer:

1. Leer .harness/STATE.md para entender el estado actual del proyecto
2. Leer .harness/rules/orchestrator.md para ver el mapa de agentes existente
3. Crear .harness/rules/{modulo}.md usando la plantilla
   harness-kit/templates/agents/AGENT_SPECIALIST.template.md
4. Actualizar .harness/rules/orchestrator.md:
   - Añadir el nuevo agente al mapa de agentes
   - Añadir las rutas de routing relevantes
   - Actualizar el grafo de dependencias
5. Copiar las reglas técnicas relevantes desde rules-library/ a .harness/rules/
6. Añadir las tareas del nuevo agente en .harness/STATE.md (sección Backlog de Tareas)
7. Hacer commit con el mensaje: "agente: añadir AGENT_{N} — {area}"

## Reglas

- El agente nuevo debe tener protocolo de inicio Y protocolo de cierre
- Debe tener criterios de verificación concretos (comandos ejecutables)
- Debe tener límites claros: "Lo que NO gestiona este agente"
- Si el agente bloquea o desbloquea a otros, debe quedar reflejado en el grafo de AGENT_00
```
