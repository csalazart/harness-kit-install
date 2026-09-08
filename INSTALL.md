# Guía de Instalación — Harness Engineering Kit
> Para instalar el sistema en un proyecto nuevo o existente.
> Tiempo estimado: 5 minutos de tu parte + 20-45 minutos de la IA configurando todo.

---

## Lo que necesitas antes de empezar

- [ ] **Node.js** instalado en tu máquina → [nodejs.org](https://nodejs.org) (descarga la versión LTS)
- [ ] **Python 3.10+** instalado en tu máquina → [python.org](https://www.python.org/downloads/) (requerido por Graphify, obligatorio en este kit para que el proyecto quede indexado)
- [ ] **`codebase-memory-mcp`** — solo si tu proyecto resulta perfil CODE, o MIXTO y decides activarlo (lo decide el agente durante la instalación, ver PASO 4)
- [ ] **Claude Code** (u otro agente IA) abierto en tu proyecto
- [ ] La carpeta `harness-kit-install/` (esta misma carpeta que estás leyendo)

Verifica Node.js y Python abriendo una terminal y ejecutando:
```
node -v
python --version
```
Debe mostrar algo como `v20.x.x` y `Python 3.10+`. Si alguno da error, instálalo primero.

---

## PASO 1 — Copia esta carpeta dentro de tu proyecto

Copia la carpeta `harness-kit-install/` completa a la raíz de tu proyecto.

**Tu proyecto debe quedar así:**
```
mi-proyecto/
├── harness-kit-install/      ← esta carpeta aquí
├── src/              ← tu código (si ya existe)
├── package.json      ← tus archivos (si ya existen)
└── ...
```

> **Si tu proyecto está vacío**, no pasa nada — la IA construirá la estructura desde cero.

---

## PASO 2 — Abre tu agente IA en la carpeta del proyecto

Abre el chat de tu agente IA (Claude Code, Gemini, etc.) apuntando a la raíz de tu proyecto.

- **Claude Code:** abre la terminal en la carpeta del proyecto y escribe `claude`
- **Gemini / otro:** ábrelo normalmente y asegúrate de que esté viendo la carpeta del proyecto

---

## PASO 3 — Envía este mensaje al agente IA

Copia y pega **exactamente** este mensaje en el chat:

```
Lee el fichero harness-kit-install/prompts/01_init.md y sigue todas las instrucciones que contiene.
Empieza por la Fase A (descubrimiento) antes de hacer nada más.
```

Eso es todo lo que tienes que escribir. El agente hará el resto.

---

## PASO 4 — Responde las preguntas del agente

El agente te hará preguntas sobre tu proyecto. Son preguntas simples:

- ¿Cómo se llama tu proyecto?
- ¿Para qué sirve?
- ¿Qué tecnologías usa? (si no lo sabes exactamente, describe lo que puedes)
- ¿En qué estado está? (¿es nuevo? ¿ya tiene código?)

> Si ya tienes un `README.md`, `package.json` u otra documentación,
> díselo al agente — los leerá directamente y necesitará preguntarte menos cosas.

---

## PASO 5 — Espera a que el agente termine

El agente construirá toda la infraestructura automáticamente. Al terminar verás:

```
✅ [HARNESS OK]
```

Y tu proyecto tendrá esta estructura nueva:

```
mi-proyecto/
│
├── CLAUDE.md              ← instrucciones maestras para el agente IA
├── AGENTS.md              ← protocolo universal (funciona con cualquier IA)
├── init.js                ← verificador de salud del sistema
│
├── scripts/
│   ├── harness-start.js   ← crea una zona de trabajo por tarea
│   ├── harness-finish.js  ← archiva y cierra la zona de trabajo
│   └── harness-link-skills.js  ← enlaza los skills al agente activo
│
├── .harness/
│   ├── STATE.md           ← tu lista de tareas y decisiones (fuente de verdad)
│   ├── PROFILE.md         ← perfil del proyecto (DOC/MIXTO/CODE) y qué grafos se instalaron
│   ├── context/           ← contexto del proyecto para la IA
│   ├── skills/            ← habilidades disponibles para el agente
│   ├── rules/             ← reglas por área del proyecto
│   ├── workspaces/        ← zonas de trabajo temporales por tarea
│   └── logs/              ← historial de tareas completadas
│
├── docs/
│   └── KNOWLEDGE_GRAPHS.md ← qué indexa cada grafo, enrutamiento, coste, troubleshooting
│
├── graphify-out/          ← grafo de CONOCIMIENTO (.harness/, docs/, plan/) — siempre se instala
├── .mcp.json              ← servidor codebase-memory-mcp — solo si tu perfil es CODE, o MIXTO y lo activaste
│
├── .agents/skills/        ← skills accesibles para cualquier agente IA
├── .claude/skills/        ← skills accesibles para Claude Code (si lo usas)
│
└── harness-kit-install/           ← el kit original (no borrar)
```

---

## PASO 6 — Verifica que todo funciona

Ejecuta en la terminal:

```
node init.js
```

Debes ver `[HARNESS OK]` al final. Si ves errores, el propio agente te ayudará a resolverlos.

---

## ¿Y ahora qué?

A partir de aquí, **cada vez que abras el chat** con el agente:

1. El agente lee `CLAUDE.md` / `AGENTS.md` automáticamente
2. Ejecuta `node init.js` para verificar que todo está bien
3. Lee `.harness/STATE.md` para saber qué hay pendiente
4. Está listo para trabajar

Cuando quieras hacer algo, simplemente dile al agente qué necesitas.
Para tareas no triviales, el agente creará automáticamente una zona de trabajo aislada,
ejecutará el trabajo allí y lo archivará al terminar.

---

## Preguntas frecuentes

**¿Tengo que entender todo lo que crea la IA?**
No. El sistema está diseñado para que la IA lo gestione sola. Tú solo necesitas
leer `.harness/STATE.md` de vez en cuando para ver el estado de tu proyecto.

**¿Qué pasa si ya tengo un proyecto con código?**
Perfecto — la IA analizará lo que ya tienes y adaptará el sistema a tu código existente.
No borra ni modifica tu código.

**¿Funciona con agentes distintos a Claude Code?**
Sí. `AGENTS.md` es el protocolo universal. Funciona con Gemini, GPT, Cursor, Copilot,
y cualquier otro agente. Los skills se enlazan automáticamente a la carpeta correcta
de cada agente durante la instalación.

**¿Puedo reinstalar si algo salió mal?**
Sí. Vuelve al Paso 3 y dile al agente:
`"El harness no quedó bien instalado, por favor revisa y completa la instalación leyendo harness-kit-install/prompts/01_init.md"`

**¿Graphify es obligatorio?**
Sí, en los tres perfiles de proyecto. El agente lo instala automáticamente durante
la construcción del harness (Fase C del prompt de instalación) — es un requisito
para que `node init.js` reporte `[HARNESS OK]`. Solo necesitas tener Python 3.10+
instalado de antemano; el skill instala el paquete `graphifyy` por ti la primera
vez que ejecuta `/graphify`.

**¿Necesito Python para graphify?**
Sí. Graphify es una librería Python. Instálalo desde
[python.org](https://www.python.org/downloads/) **antes** de empezar el Paso 3 de
esta guía — el agente lo necesitará para completar la instalación. No usa npm/pnpm.

**¿`codebase-memory-mcp` es obligatorio?**
No, lo condicional es esto, no graphify. Este kit (v4) separa dos grafos: graphify
indexa el conocimiento del proyecto (siempre) y `codebase-memory-mcp` indexa el
código fuente. Si tu proyecto tiene poco código (perfil DOC), el agente no lo
instala — no aporta nada y el mismo grafo de conocimiento ya cubre ese código.
Si tiene mucho código (perfil CODE) se instala siempre; en el rango intermedio
(MIXTO) el agente te lo propone y tú decides. La decisión queda registrada en
`.harness/PROFILE.md`, y `docs/KNOWLEDGE_GRAPHS.md` explica el enrutamiento entre
ambos grafos con más detalle.

**¿Por qué mi proyecto no tiene `codebase-memory-mcp` instalado?**
Porque tu perfil es DOC (poco código, o un proyecto de infraestructura/configuración
sin lógica de aplicación que un LSP pueda resolver), o porque tu perfil es MIXTO y
decidiste no activarlo por el coste de contexto que añade en cada ventana. Ambos
casos son correctos, no un fallo de instalación — revisa `.harness/PROFILE.md`
para ver el motivo registrado.

---

> Kit versión 1.4.0 · Para más detalles técnicos ver `QUICKSTART.md`
