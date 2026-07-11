# Reglas — Backend Python (FastAPI / Django)
> Copiar a `.harness/rules/backend.md` y adaptar al proyecto.

## Stack recomendado
- **Runtime:** Python **3.12+** (recomendado 3.13, LTS activo)
- **Framework API:** FastAPI (async, alta performance) o Django REST Framework
- **ORM:** SQLAlchemy 2.0 (FastAPI) o Django ORM
- **Validación:** Pydantic v2 (FastAPI nativo) o DRF Serializers
- **Gestión de dependencias:** **`uv`** (recomendado — rápido, reemplaza Poetry + pip en 2025) o Poetry
- **Variables de entorno:** python-dotenv + Pydantic BaseSettings

## Estructura de carpetas (FastAPI)
```
src/
├── api/
│   └── v1/
│       ├── routes/      ← endpoints por recurso
│       └── deps.py      ← dependencias (auth, db session)
├── core/
│   ├── config.py        ← configuración via Pydantic Settings
│   └── security.py      ← JWT, hashing
├── models/              ← modelos SQLAlchemy
├── schemas/             ← Pydantic schemas (request/response)
├── services/            ← lógica de negocio
├── repositories/        ← acceso a datos
└── utils/               ← helpers
```

## Seguridad (no negociables)
- Validación con Pydantic en TODOS los endpoints — nunca acceder a body sin schema
- Usar `secrets.compare_digest` para comparar tokens (timing-safe)
- Contraseñas: `bcrypt` o `argon2` — nunca MD5/SHA1
- SQL: siempre ORM o queries parametrizadas — nunca f-strings en queries
- CORS configurado explícitamente con lista blanca de orígenes
- Rate limiting: slowapi (FastAPI) o django-ratelimit

## Convenciones Python
- Type hints en TODAS las funciones
- Docstrings solo si el código no se explica solo
- f-strings para interpolación (nunca `.format()` o `%`)
- `async/await` en FastAPI — nunca mezclar sync/async sin `run_in_executor`
- Imports: stdlib → terceros → propios (isort)

## Criterio de verificación
```bash
# Con uv (recomendado)
uv run pytest                  # 0 fallos
uv run mypy src/               # sin errores de tipos
uv run ruff check src/         # sin errores de linting

# Con Poetry (alternativa)
poetry run pytest
poetry run mypy src/
poetry run ruff check src/
```
