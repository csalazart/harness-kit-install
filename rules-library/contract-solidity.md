# Reglas — Smart Contract Solidity (EVM)
> Copiar a `.harness/rules/contract.md` y adaptar al proyecto.

## Versión y compilación
- `pragma solidity ^0.8.24;` mínimo — nunca bajar de 0.8 (overflow checks automáticos)
- Activar optimizer: `runs: 200` para equilibrio deploy/runtime gas
- 0 warnings en `pnpm dlx hardhat compile` antes de avanzar
- **OpenZeppelin v5**: los imports cambiaron de ruta — usar `@openzeppelin/contracts/token/ERC20/...` (misma ruta pero la API interna cambió: `Ownable` ahora requiere `initialOwner` en el constructor)

## Seguridad (no negociables)
- **Checks-Effects-Interactions** en TODAS las funciones payable
- `nonReentrant` (ReentrancyGuard) en todas las funciones que transfieren valor
- `SafeERC20` para TODA transferencia de tokens ERC20
- Nunca `transfer()` para enviar ETH — usar `call{value: amount}("")`
- **Pull over push** para fees y pagos — receptor reclama, no se envían automáticamente
- Nunca guardar secretos on-chain — usar `keccak256(abi.encodePacked(secret))`

## Patrones de OpenZeppelin recomendados
- `ERC721URIStorage` para NFTs con metadata dinámica
- `Ownable` para control de admin
- `ReentrancyGuard` siempre en contratos que manejan valor
- `Pausable` si se necesita circuit breaker
- `AccessControl` si hay múltiples roles (más flexible que `Ownable`)

## Naming conventions
- Variables y parámetros: `camelCase`
- Eventos: `PascalCase` (ej: `ChestOpened`, `TokenDeposited`)
- Constantes: `UPPER_SNAKE_CASE`
- Structs e interfaces: `PascalCase`
- Funciones privadas/internas: `_camelCase` con underscore
- Modifiers: `camelCase` descriptivo (ej: `onlyChestOwner`, `onlyActive`)

## Estructura de un contrato limpio
```
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

// 1. Imports
// 2. Interfaces (si las hay)
// 3. Errores custom (más barato que require strings)
// 4. Eventos
// 5. Constantes
// 6. Enums
// 7. Structs
// 8. Variables de estado
// 9. Modifier
// 10. Constructor
// 11. Funciones externas
// 12. Funciones públicas
// 13. Funciones internas/privadas
// 14. View/pure functions
```

## Errores custom (preferir sobre require + string)
```solidity
error NotChestOwner(uint256 chestId, address caller);
error InvalidStatus(uint256 chestId, uint256 expected, uint256 actual);
// Uso: if (condition) revert NotChestOwner(chestId, msg.sender);
```

## Criterio de verificación
```bash
pnpm dlx hardhat compile               # 0 errores, 0 warnings
slither contracts/MiContrato.sol  # sin HIGH ni CRITICAL
pnpm dlx hardhat test                  # 0 fallos
pnpm dlx hardhat coverage              # >95% líneas
```
