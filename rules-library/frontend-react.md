# Reglas — Frontend React
> Aplicar en cualquier proyecto que use React (con o sin Next.js, Vite, CRA).
> Versión mínima: **React 18**. Proyectos nuevos: **React 19**.

---

## React 19 — novedades relevantes

- **Actions**: funciones async en formularios via `useActionState` y `useFormStatus`
- **`use()`**: leer Promises o Context directamente dentro de render — sustituye casos de `useEffect` para fetch inicial
- **React Compiler**: memoización automática (disponible como plugin de Babel/Vite) — no añadir `useMemo`/`useCallback` manualmente cuando el compilador esté activo
- **`useOptimistic`**: actualizaciones optimistas sin librerías extra
- Hydration errors mejorados con mensajes más claros

## Componentes

- Usar siempre **componentes funcionales** — no usar class components
- Un componente por fichero — nombre del fichero = nombre del componente (PascalCase)
- Props con TypeScript o PropTypes — nunca componentes sin tipado en proyectos de producción
- Componentes pequeños y enfocados — si supera ~150 líneas, dividir en subcomponentes
- No lógica de negocio dentro del JSX — extraer a variables o funciones antes del return

## Hooks

- Seguir las **Rules of Hooks**: solo en el nivel superior, solo en componentes o custom hooks
- Prefijo `use` obligatorio para custom hooks: `useAuth`, `useCart`, `useFetch`
- `useEffect` solo para sincronizar con sistemas externos (APIs, DOM, timers)
- No usar `useEffect` para transformar datos que se pueden calcular en el render
- `useMemo` y `useCallback` solo cuando el profiler confirme un problema de rendimiento — no por defecto (el React Compiler los añade automáticamente cuando está activo)

## Estado

- Estado local (`useState`) para estado de UI que no necesitan otros componentes
- Context API para estado compartido entre pocos componentes del mismo árbol
- Zustand o Redux Toolkit para estado global complejo o compartido ampliamente
- Nunca mutar el estado directamente — siempre crear un nuevo objeto/array

## Naming

- Componentes: `PascalCase` — `UserCard`, `ProductList`
- Custom hooks: `camelCase` con prefijo `use` — `useUserData`
- Handlers de eventos: prefijo `handle` — `handleSubmit`, `handleClick`
- Props booleanas: prefijo `is` o `has` — `isLoading`, `hasError`

## Estructura de ficheros

```
src/
├── components/        ← Componentes reutilizables (UI puro)
├── pages/ (o app/)    ← Páginas / rutas
├── hooks/             ← Custom hooks
├── context/           ← Providers de Context API
├── store/             ← Estado global (Zustand/Redux)
├── services/          ← Llamadas a APIs externas
├── utils/             ← Funciones puras de utilidad
└── types/             ← Tipos TypeScript compartidos
```

## Rendimiento

- Lazy loading de rutas con `React.lazy` + `Suspense`
- Imágenes con atributo `loading="lazy"`
- Listas largas: usar virtualización (react-window, react-virtual)
- No pasar objetos/arrays literales como props si causan re-renders innecesarios

## Testing

- Tests con **React Testing Library** — testear comportamiento, no implementación
- No testear detalles de implementación (estado interno, nombres de métodos)
- Queries en orden de preferencia: `getByRole` → `getByLabelText` → `getByText` → `getByTestId`
- `userEvent` en vez de `fireEvent` para simular interacciones reales
- Mocks solo para llamadas de red — usar `msw` (Mock Service Worker)

## Lo que NO hacer

- No usar `index.js` como nombre de componente — dificulta el debugging
- No hacer fetch directamente en componentes — usar custom hooks o servicios
- No hardcodear URLs de API en componentes — usar variables de entorno
- No usar `any` en TypeScript salvo caso excepcional documentado

## Criterio de verificación
```bash
pnpm run build          # build sin errores
pnpm run lint           # ESLint sin errores
pnpm run test           # 0 fallos
```
