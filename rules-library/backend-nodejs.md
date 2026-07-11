# Reglas — Backend Node.js (Express / Fastify)
> Copiar a `.harness/rules/backend.md` y adaptar al proyecto.

## Stack recomendado
- **Runtime:** Node.js **22 LTS** (activo LTS desde Oct 2024) — mínimo 20 LTS
- **Framework:** Fastify **5.x** (preferido por performance) o Express
- **Lenguaje:** TypeScript siempre
- **ORM:** Prisma (PostgreSQL/MySQL) o Mongoose (MongoDB)
- **Validación:** Zod para schemas de entrada/salida
- **Auth:** JWT + refresh tokens o Auth.js (v5) / Better Auth

## Estructura de carpetas
```
src/
├── routes/          ← definición de rutas (un fichero por recurso)
├── controllers/     ← lógica de cada endpoint
├── services/        ← lógica de negocio (sin HTTP)
├── repositories/    ← acceso a datos (sin lógica de negocio)
├── middleware/      ← auth, logging, error handling
├── schemas/         ← Zod schemas de validación
├── types/           ← tipos TypeScript compartidos
└── utils/           ← helpers puros (sin side effects)
```

## Seguridad (no negociables)
- Validar TODA entrada de usuario con Zod antes de procesar
- Sanitizar antes de queries a DB (Prisma/Mongoose lo hacen automáticamente)
- Rate limiting en endpoints públicos (express-rate-limit / @fastify/rate-limit)
- Helmet para headers de seguridad HTTP
- CORS configurado explícitamente — nunca `origin: '*'` en producción
- Variables de entorno via `.env` — nunca hardcodeadas
- Nunca exponer stack traces en respuestas de error en producción

## Convenciones de API REST
```
GET    /recursos          ← listar
GET    /recursos/:id      ← obtener uno
POST   /recursos          ← crear
PUT    /recursos/:id      ← reemplazar completo
PATCH  /recursos/:id      ← actualizar parcial
DELETE /recursos/:id      ← eliminar
```
- Respuestas: siempre JSON `{ data: ..., error: null }` o `{ data: null, error: { message, code } }`
- Status codes: 200 OK, 201 Created, 400 Bad Request, 401 Unauthorized, 403 Forbidden, 404 Not Found, 500 Internal

## Manejo de errores
- Error handler global — nunca try/catch en cada controller
- Errores tipados: `class AppError extends Error { constructor(message, statusCode, code) }`
- Logging: Winston o Pino (nunca `console.log` en producción)

## Criterio de verificación
```bash
pnpm run build     # TypeScript compila sin errores
pnpm test          # 0 fallos
pnpm run lint      # sin errores ESLint
```
