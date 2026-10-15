*   **Preguntas de CÓDIGO → herramientas MCP.** Nunca los scripts de graphify:
    el código no está en ese grafo.

    | Pregunta | Llamada |
    | --- | --- |
    | ¿Quién llama a X? | `trace_path(function_name="X", direction="inbound")` |
    | ¿Qué llama X? | `trace_path(direction="outbound")` |
    | Contexto completo de llamadas | `trace_path(direction="both")` |
    | Localizar por nombre | `search_graph(name_pattern=".*X.*")` |
    | Código muerto | `search_graph(max_degree=0, exclude_entry_points=true)` |
    | Impacto de mis cambios | `detect_changes()` |
    | Ver el código exacto | `get_code_snippet(qualified_name="…")` |
    | Estructura del proyecto | `get_architecture()` |
    | Búsqueda textual en código | `search_code()` — preferir sobre `Grep` |

    **Cuatro reglas de uso** (el resto, en `docs/KNOWLEDGE_GRAPHS.md`):
    1. `trace_path` exige el nombre **exacto** — descúbrelo antes con `search_graph`.
    2. `direction="outbound"` se pierde llamadas cross-service; ante duda usa `"both"`.
    3. Los resultados paginan de 50 en 50 — comprueba `has_more` y usa `offset`
       antes de concluir que algo no existe.
    4. Antes de afirmar **ausencia** («no se usa», «no existe», «es código muerto»)
       o exhaustividad, llama a `check_index_coverage` con las rutas implicadas.
       Un resultado limpio significa *«no hay hueco registrado»*, **no** prueba de
       completitud: si reporta cobertura parcial, saltada o desconocida, lee el
       rango real antes de responder.

    > Si el perfil es **DOC** no hay MCP: el código está en el grafo de
    > graphify (o se lee directamente — son pocos ficheros). Consultar
    > `.harness/PROFILE.md` en caso de duda.
