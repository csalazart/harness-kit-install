# Reglas — Frontend Nuxt 3
> Copiar a `.harness/rules/frontend.md` y adaptar al proyecto.
> Nuxt 3 extiende Vue 3 — estas reglas complementan las de frontend-vue.md.

## Versión y configuración
- **Nuxt 4** (o Nuxt 3.x en modo compatibilidad) + Vue **3.5+** + TypeScript obligatorio
- `nuxt.config.ts` como única fuente de configuración — sin `vite.config.ts` paralelo
- Módulos oficiales preferidos: `@nuxtjs/tailwindcss`, `@pinia/nuxt`, `@nuxt/image`, `@nuxtjs/i18n`
- ESLint con `@nuxt/eslint` (módulo oficial) desde el inicio
- `strict: true` en `tsconfig.json`

### Nuxt 4 — cambios importantes
- Nueva estructura de carpetas: el código de la app vive en `app/` en lugar de la raíz
- `app/router.options.ts` para customizar el router (antes `router.options.ts` en raíz)
- `useAsyncData` y `useFetch` ahora deduplicados por defecto (no se llaman dos veces en SSR+cliente)
- `compatibilityVersion: 4` en `nuxt.config.ts` para activar comportamientos de v4 en proyectos v3:
  ```typescript
  export default defineNuxtConfig({ future: { compatibilityVersion: 4 } })
  ```
- Migration guide disponible: `npx nuxi upgrade --force`

## Estructura de carpetas (Nuxt auto-import)
```
├── assets/              ← estilos globales, fuentes, imágenes procesadas por Vite
├── components/          ← auto-importados por Nuxt (PascalCase o carpeta/Nombre)
│   ├── ui/              ← Button.vue, Modal.vue, Input.vue
│   └── features/        ← CofreCard.vue, WalletConnector.vue
├── composables/         ← auto-importados (useXxx.ts)
├── layouts/             ← default.vue, auth.vue, etc.
├── middleware/          ← auth.ts, guest.ts (route middleware)
├── pages/               ← file-based routing automático
│   ├── index.vue
│   ├── cofres/
│   │   ├── index.vue    →  /cofres
│   │   └── [id].vue     →  /cofres/:id
├── plugins/             ← inicialización de librerías externas
├── public/              ← archivos estáticos (no procesados)
├── server/              ← API routes y server-side code
│   ├── api/             →  /api/* endpoints
│   └── middleware/      ← middleware del servidor
├── stores/              ← Pinia stores (auto-importados con @pinia/nuxt)
└── types/               ← tipos TypeScript globales
```

## Rendering — elegir la estrategia correcta

| Estrategia | Cuándo usar | Config |
|-----------|-------------|--------|
| **SSR** (por defecto) | SEO importante, datos dinámicos | por defecto |
| **SSG** | Contenido estático, blog, docs | `nitro: { prerender: { routes: ['/'] } }` |
| **SPA** | Dashboard interno, sin SEO | `ssr: false` |
| **Híbrido** | Algunas rutas SSR, otras SPA | `routeRules` en `nuxt.config.ts` |

```typescript
// nuxt.config.ts — ejemplo híbrido
export default defineNuxtConfig({
  routeRules: {
    '/': { prerender: true },        // SSG
    '/cofres/**': { ssr: true },     // SSR
    '/dashboard/**': { ssr: false }  // SPA
  }
})
```

## Composables de Nuxt (usar los nativos)
```typescript
// Fetch de datos — preferir useFetch sobre $fetch en componentes
const { data: cofre, pending, error } = await useFetch(`/api/cofres/${id}`, {
  key: `cofre-${id}`,          // clave de caché
  transform: (r) => r.data,    // transformar respuesta
})

// Para mutaciones o fetch condicional
const { execute } = await useLazyFetch('/api/cofres', { immediate: false })

// Estado compartido entre servidor y cliente
const contador = useState('contador', () => 0)
```
- `useFetch` / `useAsyncData` para SSR — nunca `fetch()` directamente en `setup()`
- `useState` para estado que debe hidratarse entre servidor y cliente
- `useRuntimeConfig` para variables de entorno — nunca `import.meta.env` directamente

## Server Routes (API interna)
```typescript
// server/api/cofres/[id].get.ts
export default defineEventHandler(async (event) => {
  const id = getRouterParam(event, 'id')
  const cofre = await cofreService.obtener(Number(id))
  if (!cofre) throw createError({ statusCode: 404, message: 'Cofre no encontrado' })
  return cofre
})
```
- Naming convention: `[nombre].[metodo].ts` → `cofres.get.ts`, `cofres.post.ts`
- `defineEventHandler` siempre — tipado automático
- `createError` para errores HTTP — nunca `throw new Error()`
- Validación de body con `readValidatedBody(event, schema.parse)` usando Zod

## Middleware de rutas
```typescript
// middleware/auth.ts
export default defineNuxtRouteMiddleware((to) => {
  const { isAuthenticated } = useAuthStore()
  if (!isAuthenticated.value) {
    return navigateTo('/login')
  }
})

// En la página que lo necesita:
definePageMeta({ middleware: 'auth' })
```

## Plugins
```typescript
// plugins/miLibreria.client.ts  ← solo en el cliente
// plugins/miLibreria.server.ts  ← solo en el servidor
// plugins/miLibreria.ts         ← ambos

export default defineNuxtPlugin(() => {
  // inicialización
  return {
    provide: { miHelper: () => '...' }  // accesible como $miHelper
  }
})
```

## Variables de entorno
```typescript
// nuxt.config.ts
export default defineNuxtConfig({
  runtimeConfig: {
    // Solo servidor (nunca llegan al cliente)
    apiSecretKey: process.env.API_SECRET_KEY,
    // Público (llega al cliente — usar con cuidado)
    public: {
      apiBase: process.env.NUXT_PUBLIC_API_BASE,
    }
  }
})
```
- `runtimeConfig.public.*` → accesible en cliente y servidor
- `runtimeConfig.*` (sin public) → **solo servidor** — aquí van los secrets
- Prefijo de env vars: `NUXT_` para server, `NUXT_PUBLIC_` para public

## SEO
```vue
<script setup lang="ts">
useSeoMeta({
  title: () => `Cofre ${cofre.value?.nombre}`,
  description: 'Abre tu Monster Treasure Chest',
  ogImage: cofre.value?.imagenUrl,
})
</script>
```
- `useSeoMeta` en todas las páginas — nunca `<Head>` manual
- Títulos dinámicos con funciones reactivas `() => valor.value`

## Seguridad
- Nunca poner secrets en `runtimeConfig.public`
- Validar en server routes — el cliente no es de confianza
- Usar `useRequestHeaders` con cuidado — no reenviar todos los headers
- CSP configurable en `nuxt.config.ts` via `security` module (`nuxt-security`)

## Criterio de verificación
```bash
pnpm run build          # nuxt build sin errores TypeScript
pnpm run typecheck      # nuxi typecheck sin errores
pnpm run lint           # ESLint sin errores
pnpm run test           # Vitest — 0 fallos
pnpm dlx nuxt analyze       # revisar tamaño de bundles (opcional)
```
