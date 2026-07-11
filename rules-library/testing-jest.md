# Reglas — Testing con Jest / Vitest
> Copiar a `.harness/rules/testing.md` y adaptar al proyecto.

## Stack recomendado
- **Unit/Integration:** Jest (Node.js) o Vitest (Vite/Next.js — más rápido)
- **E2E:** Playwright (preferido) o Cypress
- **Mocks HTTP:** MSW (Mock Service Worker) — no mockear fetch/axios directamente
- **Cobertura:** V8 (Vitest) o Istanbul (Jest)

## Criterios de aprobación (no negociables)
```
✅ 0 tests fallando
✅ Cobertura de líneas  > 80%
✅ Cobertura de branches > 75%
✅ Tests E2E del happy path pasan
```

## Estructura de tests
```
src/
└── __tests__/          ← o co-ubicados como ComponentName.test.ts
    ├── unit/           ← funciones puras, utils, hooks
    ├── integration/    ← módulos que interactúan (ej: service + repository)
    └── e2e/            ← flujos completos de usuario
```

## Convenciones de nomenclatura
```typescript
describe("nombreDelModulo", () => {
  describe("nombreDelMetodo", () => {
    it("should [hacer algo] when [condición]", () => { ... })
    it("should [fallar con X] when [condición de error]", () => { ... })
  })
})
```

## Qué mockear y qué no
- ✅ Mockear: llamadas HTTP externas (MSW), bases de datos (factory), servicios externos
- ❌ No mockear: lógica de negocio propia, utils puros, código que estás testeando
- Regla: si el mock hace que el test sea inútil (mockeas lo que testeas), algo está mal

## Arrange-Act-Assert
```typescript
it("should calculate fee correctly", () => {
  // Arrange
  const amount = 1000
  const basisPoints = 20

  // Act
  const fee = calculateFee(amount, basisPoints)

  // Assert
  expect(fee).toBe(2)
})
```

## Criterio de verificación
```bash
pnpm dlx vitest run             # 0 fallos
pnpm dlx vitest run --coverage  # reporte de cobertura
pnpm dlx playwright test        # E2E pasan (si aplica)
```
