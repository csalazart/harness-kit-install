*   **Preguntas de CÓDIGO → CodeGraph (MCP `codegraph`).** Nunca los scripts de graphify:
    el código no está en ese grafo.

    | Pregunta | Llamada |
    | --- | --- |
    | Arquitectura, flujo de llamadas, «¿cómo funciona X?», impacto de un cambio | `codegraph_explore(query="…")` — devuelve en una sola llamada el código fuente relevante, las rutas de llamada y el radio de impacto |
    | ¿Quién llama a X? / ¿Qué llama X? | `codegraph_explore` sobre X; o CLI de solo lectura: `codegraph callers X` / `codegraph callees X` |
    | ¿Qué se rompe si cambio X? | `codegraph_explore` sobre X; o CLI: `codegraph impact X` / `codegraph affected` |
    | Localizar por nombre / ver un símbolo | CLI: `codegraph query "X"` / `codegraph node X` (o `codegraph_search` / `codegraph_node` si el servidor las expone) |
    | Estado y frescura del índice | CLI: `codegraph status` |

    **Cinco reglas de uso** (el resto, en `docs/KNOWLEDGE_GRAPHS.md`):
    1. Antes de Read/Glob/Grep amplios, comprueba que existe `.codegraph/`. Si falta,
       créalo UNA vez con `codegraph init .` desde la raíz Git del proyecto (nunca en
       `$HOME` ni en carpetas temporales). Un índice ausente es el disparador para
       inicializar, no un motivo para saltarse CodeGraph.
    2. Por defecto el MCP solo anuncia `codegraph_explore`. Las demás herramientas
       (`codegraph_node`, `codegraph_search`, `codegraph_callers`, `codegraph_callees`,
       `codegraph_impact`, `codegraph_files`, `codegraph_status`) solo existen si se
       activaron con `CODEGRAPH_MCP_TOOLS`; si no están, usa la CLI de solo lectura.
    3. El MCP se auto-sincroniza con un watcher; la CLI lee el índice tal cual. Antes de
       consultar por CLI tras editar ficheros, ejecuta `codegraph sync -q .`.
    4. Antes de afirmar **ausencia** («no se usa», «es código muerto») contrasta con
       `codegraph callers`/`impact` y lee el rango real: un índice sin sincronizar o
       incompleto no prueba que algo no exista.
    5. Nunca ejecutes `codegraph uninit`, `codegraph install`, `codegraph uninstall` ni
       `codegraph upgrade`; `codegraph index` solo para recuperar un índice corrupto.

    > Si el perfil es **DOC** no hay motor de código: el código está en el grafo de
    > graphify (o se lee directamente — son pocos ficheros). Consultar
    > `.harness/PROFILE.md` en caso de duda.
