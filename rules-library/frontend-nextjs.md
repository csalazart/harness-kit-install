# Reglas — Frontend Next.js
> Copiar a `.harness/rules/frontend.md` y adaptar al proyecto.

## Versión y configuración
- Next.js **15+** (recomendado: **16**) con App Router — no usar Pages Router en proyectos nuevos
- **Node.js 20.9.0+** mínimo (Next.js 16 requiere Node.js 20.9+; recomendado 22 LTS)
- **TypeScript 5.1+** obligatorio — sin archivos `.js` en `src/`
- Modo estricto en `tsconfig.json`: `"strict": true`
- Usar `next.config.ts` (TypeScript) en lugar de `next.config.js`
- ESLint + Prettier configurados antes de escribir código
- **sharp** instalado como dependencia para `next/image` (eliminado squoosh en v15)

## Turbopack
- Turbopack es el bundler por defecto en dev desde Next.js 15 (`next dev --turbopack`)
- En Next.js 16, Turbopack es estable y default para builds también
- No configurar webpack manualmente salvo plugins sin soporte en Turbopack

## Estructura de carpetas (App Router)
```
src/
├── app/                    ← rutas (carpeta = ruta)
│   ├── layout.tsx          ← layout raíz
│   ├── page.tsx            ← página principal
│   └── (grupo)/            ← grupos de rutas sin segmento en URL
├── components/
│   ├── ui/                 ← componentes primitivos (botones, inputs)
│   └── features/           ← componentes de negocio
├── hooks/                  ← custom hooks
├── lib/                    ← utilidades, helpers, configuración
├── types/                  ← tipos TypeScript compartidos
└── styles/                 ← CSS global (mínimo, preferir Tailwind)
```

## Convenciones de componentes
- Un componente por fichero
- Nombre del fichero = nombre del componente en PascalCase
- `"use client"` solo cuando sea necesario (eventos, hooks del browser)
- Por defecto, todo es Server Component
- Props tipadas con `interface` o `type` en el mismo fichero si son simples

## APIs dinámicas — ASYNC desde Next.js 15 ⚠️

`cookies()`, `headers()`, `draftMode()`, `params` y `searchParams` son **Promises** en v15+.
Siempre usar `await`:

```typescript
// ❌ Next.js 14 — ya no funciona
const theme = cookies().get('theme')
const { id } = params

// ✅ Next.js 15+
const theme = (await cookies()).get('theme')
const { id } = await params
```

Aplica tanto en Server Components como en Route Handlers y middleware.

## Caching — cambio de comportamiento en v15 ⚠️

En Next.js 14 el fetch hacía `force-cache` por defecto.  
En **Next.js 15+** el comportamiento por defecto es **`no-store`** (sin caché).

```typescript
// ❌ En v14 esto se cacheaba implícitamente — en v15 NO
const data = await fetch('/api/data')

// ✅ Caché explícito con directiva (recomendado v15+)
'use cache'
export default async function Page() { ... }

// ✅ O controlar por fetch individual
const data = await fetch('/api/data', { cache: 'force-cache' })
const fresh = await fetch('/api/live', { cache: 'no-store' })

// ✅ Revalidación por tiempo
const data = await fetch('/api/data', { next: { revalidate: 3600 } })
```

### Directiva `use cache`
- Marcar un Server Component o función como cacheable: poner `'use cache'` al inicio
- Tiempo por defecto: 15 minutos
- Revalidar: `cacheTag('nombre')` + `revalidateTag('nombre')`
- Route Handlers GET tampoco se cachean por defecto — usar `export const revalidate = 3600` si hace falta

## Partial Prerendering (PPR)
- En Next.js 15: experimental, activar con `experimental: { ppr: 'incremental' }`
- En Next.js 16: estable, usar `cacheComponents` en `nuxt.config.ts`
- Combinar partes estáticas y dinámicas en la misma ruta con `<Suspense>`

## Estado y datos
- Estado del servidor: React Server Components + fetch nativo
- Estado del cliente: `useState` / `useReducer` para local, Zustand para global
- Mutaciones: Server Actions (no API routes para mutations internas)
- Queries complejas: SWR o TanStack Query

## Formularios — `next/form`
```typescript
import Form from 'next/form'

// Navegación client-side, prefetch automático, reset tras submit
<Form action="/search">
  <input name="q" />
  <button type="submit">Buscar</button>
</Form>
```
Usar `next/form` en lugar de `<form>` HTML para formularios que navegan a otra ruta.

## Middleware (Next.js 16)
- En Next.js 16 el archivo se llama `proxy.ts` (antes `middleware.ts`)
- Exportar función `proxy()` en lugar de `middleware()`
- Ejecutar codemods al migrar: `npx @next/codemod`

## React 19
- Next.js 15+ usa React 19 con soporte completo de Actions, `use()`, y el React Compiler
- El **React Compiler** (Next.js 16) aplica memoización automática — no es necesario añadir `useMemo`/`useCallback` manualmente salvo casos muy específicos
- `useOptimistic` para actualizaciones optimistas en Server Actions

## Performance
- Imágenes: siempre `<Image>` de Next.js con `sharp` instalado — nunca `<img>` para assets propios
- Fuentes: `next/font` para cargar fuentes
- Lazy loading: `dynamic()` para componentes pesados
- Formularios con navegación: `next/form` en lugar de `<form>` nativo

## Seguridad — Server Actions
- Next.js 15+ genera endpoints no-adivinables para Server Actions
- Elimina automáticamente las acciones no referenciadas del bundle
- No exponer lógica sensible en Client Components que llamen a Server Actions

## Migración v14 → v15 (codemod disponible)
```bash
npx @next/codemod@canary upgrade latest
```
Migra automáticamente `cookies()`, `headers()`, `params` a versiones async.

## Criterio de verificación
```bash
pnpm run build     # sin errores TypeScript ni build errors
pnpm run lint      # sin errores ESLint
pnpm run test      # 0 fallos (si hay tests)
```
