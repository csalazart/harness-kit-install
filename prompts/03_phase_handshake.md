# Prompt 03 — Protocolo Handshake (inicio de sesión)
> Ejecutar al inicio de CADA sesión de trabajo. Garantiza que el agente arranca con el contexto correcto.

---

```
ASUNTO: Protocolo Handshake — Inicio de sesión

Soy el Agente [NÚMERO] — [NOMBRE]. Antes de hacer cualquier cosa, ejecuta este protocolo:

## 1. Verificar el entorno

Ejecuta:
  node init.js

- Si termina con [HARNESS OK] → continuar
- Si termina con [HARNESS FAIL] → STOP. Resolver los errores antes de continuar.

## 2. Leer el estado del proyecto

Lee en este orden:
a) .harness/STATE.md → backlog de tareas pendientes y decisiones tomadas
b) .harness/workspaces/ → ¿hay algún workspace activo sin cerrar?
   - Si existe un workspace → leer su orden.md para entender qué estaba en progreso
c) .harness/rules/[mi-area].md → mis reglas y contexto específico

## 3. Identificar la tarea

Basándote en STATE.md y el workspace asignado (si existe), dime:
- ¿Cuál es mi tarea asignada?
- ¿Cuáles son los criterios de aceptación de orden.md?
- ¿Hay decisiones críticas que me bloquean?

## 4. Confirmación antes de actuar

Muéstrame:
- La tarea que vas a ejecutar (nombre + objetivo)
- Los criterios de aceptación que usaré para saber que está hecha
- Cualquier dependencia o bloqueo detectado

Espera mi confirmación antes de empezar a trabajar.

---
CONTEXTO DEL PROYECTO: [pegar aquí el CLAUDE.md o indicar que ya está cargado]
```
