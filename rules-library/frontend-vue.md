# Reglas — Frontend Vue.js 3
> Copiar a `.harness/rules/frontend.md` y adaptar al proyecto.

## Versión y configuración
- Vue **3.5+** con **Composition API** — nunca Options API en código nuevo
- TypeScript obligatorio — sin ficheros `.js` en `src/`
- Vite como bundler — no Vue CLI (deprecado)
- `vue-tsc` para verificación de tipos en templates
- ESLint con `@vue/eslint-config-typescript` + Prettier desde el inicio

### Vue 3.5 — novedades relevantes
- `useTemplateRef()` sustituye a `ref` para referencias DOM tipadas: `const el = useTemplateRef<HTMLInputElement>('myInput')`
- `defineProps` con desestructuración reactiva sin perder reactividad: `const { count = 0 } = defineProps<{ count?: number }>()`
- `onWatcherCleanup()` para limpiar efectos dentro de watchers
- `useId()` para generar IDs únicos accesibles (SSR-safe)

## Estructura de carpetas
```
src/
├── assets/              ← imágenes, fuentes, estilos globales
├── components/
│   ├── ui/              ← componentes primitivos (Button, Input, Modal)
│   └── features/        ← componentes de negocio por dominio
├── composables/         ← lógica reutilizable (useXxx.ts)
├── layouts/             ← layouts de página
├── pages/               ← vistas (una por ruta)
├── router/
│   └── index.ts         ← Vue Router
├── stores/              ← Pinia stores
├── types/               ← tipos TypeScript compartidos
└── utils/               ← helpers puros sin side effects
```

## Composition API — convenciones

### Script Setup (obligatorio)
```vue
<script setup lang="ts">
import { ref, computed, onMounted } from 'vue'
import { useCofre } from '@/composables/useCofre'

// Props tipadas
const props = defineProps<{
  cofreId: number
  readonly?: boolean
}>()

// Emits tipados
const emit = defineEmits<{
  opened: [cofreId: number]
  closed: []
}>()

const { cofre, abrir, isLoading } = useCofre(props.cofreId)
</script>
```
- `<script setup>` siempre — nunca `setup()` en Options API
- Props y emits con TypeScript genéricos — sin `PropType`
- Sin `this` — no existe en Composition API

### Composables (patrón `useXxx`)
```typescript
// composables/useCofre.ts
export function useCofre(cofreId: number) {
  const cofre = ref<Cofre | null>(null)
  const isLoading = ref(false)

  async function abrir(clave?: string) {
    isLoading.value = true
    try {
      cofre.value = await cofreService.abrir(cofreId, clave)
    } finally {
      isLoading.value = false
    }
  }

  onMounted(() => { /* cargar cofre */ })

  return { cofre: readonly(cofre), isLoading, abrir }
}
```
- Un composable por responsabilidad
- Devolver `readonly()` para estado que no debe mutarse desde fuera
- Los composables no conocen componentes — son lógica pura reutilizable

## Pinia (estado global)
```typescript
// stores/cofreStore.ts
export const useCofreStore = defineStore('cofre', () => {
  const cofres = ref<Cofre[]>([])

  const cofresAbiertos = computed(() =>
    cofres.value.filter(c => c.estado === 'OPEN')
  )

  async function cargarTodos() {
    cofres.value = await cofreService.listar()
  }

  return { cofres: readonly(cofres), cofresAbiertos, cargarTodos }
})
```
- Stores en formato **Setup Store** (función) — no Options Store
- Solo en store el estado verdaderamente global — no abusar
- Acciones asíncronas directamente en el store, sin mutations

## Templates
```vue
<template>
  <!-- v-if antes de v-for si ambos están en el mismo elemento -->
  <!-- Nunca v-if + v-for en el mismo elemento — usar <template> wrapper -->
  <template v-if="!isLoading">
    <CofreCard
      v-for="cofre in cofres"
      :key="cofre.id"
      :cofre="cofre"
      @opened="onOpened"
    />
  </template>
  <LoadingSpinner v-else />
</template>
```
- `key` obligatorio en `v-for` — nunca usar el índice del array como key
- Eventos en `kebab-case` en el template: `@chest-opened`
- Props en `camelCase` en el script, `kebab-case` en el template

## Vue Router
- Rutas con nombres — nunca navegar por path hardcodeado
- Navigation guards para proteger rutas autenticadas
- Lazy loading de páginas: `component: () => import('@/pages/Cofre.vue')`

## Seguridad
- Nunca usar `v-html` con datos de usuario — XSS garantizado
- Si `v-html` es inevitable, sanitizar con DOMPurify primero
- No exponer claves API en el frontend — usar variables de entorno solo para `VITE_PUBLIC_*`

## Criterio de verificación
```bash
pnpm run build          # Vite build sin errores TypeScript
pnpm run type-check     # vue-tsc --noEmit sin errores
pnpm run lint           # ESLint sin errores
pnpm run test           # Vitest — 0 fallos
```
