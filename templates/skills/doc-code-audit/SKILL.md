# Skill: doc-code-audit

> Carea la documentación del proyecto (grafo de conocimiento, graphify) contra el código
> real (grafo de código, `codebase-memory-mcp`). Solo existe porque el proyecto tiene los
> dos grafos — es la capacidad nueva del modelo dual (ver `docs/KNOWLEDGE_GRAPHS.md`).

## Disponibilidad por perfil

Lee `.harness/PROFILE.md` primero.

- **CODE y MIXTO-con-MCP:** aplica completa.
- **DOC (o MIXTO sin MCP instalado):** **no aplica** — no hay dos grafos que carear.
  Dilo explícitamente al usuario y detente; no intentes un careo parcial ni falles en silencio.

## Alcance

- Sin argumentos → `.harness/rules/` + `docs/`
- Con un argumento (ruta) → esa ruta

## Algoritmo

1. **Determinar alcance** (ver arriba).

2. **Extraer afirmaciones verificables del grafo de conocimiento:**
   ```
   query-graph.ps1|sh "¿qué símbolos, módulos, funciones o rutas del código menciona <alcance>?"
   ```
   Recoger, por cada afirmación: el identificador mencionado y su `fichero:línea` en la documentación.

3. **Verificar cada identificador contra el grafo de código:**
   - `search_graph(<identificador>)` — localizarlo.
   - `get_code_snippet(<identificador>)` — solo si hay duda sobre la firma exacta.

4. **Clasificar cada afirmación:**
   - ✅ **CONFIRMADA** — existe, donde dice la doc, con la firma descrita.
   - ⚠️ **DESPLAZADA** — existe pero en otra ubicación o con otra firma.
   - ❌ **FANTASMA** — la doc lo menciona, el código no lo tiene.
   - 🔍 **HUÉRFANO** — el código lo tiene, ninguna doc lo menciona (solo con `--orphans`).

5. **Informe**, ordenado por severidad (FANTASMA > DESPLAZADA > HUÉRFANO):
   ```
   | Afirmación | doc:línea | Veredicto | Ubicación real |
   ```

## Requisitos de rigor (heredados de la disciplina de veracidad del kit)

- **Nunca** marcar CONFIRMADA sin un resultado real de `search_graph`.
- Citar siempre `fichero:línea` de la doc **y** la ubicación real del código.
- El informe va primero. Las correcciones a la documentación, solo bajo aprobación explícita del usuario — esta skill nunca edita documentación por su cuenta.

## Tabla de vocabulario — requisito crítico

En proyectos donde la documentación está en un idioma y el código en otro (habitual:
doc en español de producto, identificadores en inglés), el careo **falla en silencio**
por desajuste léxico entre lo que dice la doc y cómo se llama el símbolo real.

Antes del paso 2, genera (o actualiza si ya existe) una tabla de traducción por proyecto
en `.harness/context/` con los pares término-de-doc ↔ identificador-de-código que vayas
encontrando. Expande la pregunta del paso 2 contra ese vocabulario antes de la travesía
del grafo — no asumas que una traducción literal basta.

## Niveles de evidencia

Aplica el mismo principio que `codebase-memory-mcp` (ver `docs/KNOWLEDGE_GRAPHS.md` §6):
**cobertura limpia ≠ prueba de completitud.** Un `check_index_coverage` limpio en las
rutas auditadas significa que no hay hueco registrado, no que el careo es exhaustivo.
Si el índice reporta cobertura parcial o desconocida en el alcance auditado, decláralo
en el informe como limitación, no lo omitas.
