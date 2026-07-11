# Reglas — Seguridad Web3 / Smart Contracts
> Copiar a `.harness/rules/security.md` y adaptar al proyecto.

## Vulnerabilidades críticas a verificar siempre

| Vulnerabilidad | Descripción | Mitigación |
|----------------|-------------|-----------|
| **Reentrancy** | Contrato externo llama de vuelta antes de terminar | `nonReentrant` + CEI pattern |
| **Integer Overflow** | Aritmética desbordada | Solidity ^0.8 previene automáticamente |
| **Front-running** | Bot copia tu tx antes de que sea incluida | Commit-reveal, slippage tolerance |
| **Oracle Manipulation** | Precio manipulado en el mismo bloque | TWAP oracles, múltiples fuentes |
| **Access Control** | Función crítica sin restricción de caller | `onlyOwner`, `onlyRole`, modifiers |
| **Timestamp Manipulation** | Miner puede ajustar ±15 segundos | No usar para lógica crítica de tiempo |
| **Delegatecall** | Ejecuta código en contexto del llamante | Solo con contratos de confianza |
| **Token malicioso** | ERC20 con callback malicioso | CEI pattern, `SafeERC20` |

## Checklist de auditoría pre-mainnet

### Herramientas estáticas
- [ ] Slither: `slither . --print human-summary` → sin HIGH/CRITICAL
- [ ] Mythril: `myth analyze contracts/MiContrato.sol` → sin vulnerabilidades críticas
- [ ] Solhint: `solhint 'contracts/**/*.sol'` → sin errores

### Patrones de seguridad implementados
- [ ] Checks-Effects-Interactions en todas las funciones payable
- [ ] `nonReentrant` en todas las funciones que transfieren valor
- [ ] `SafeERC20` para transferencias de tokens
- [ ] Pull over push para pagos/fees
- [ ] `call{value:}` en lugar de `transfer()` o `send()`
- [ ] Modifier `onlyOwner` / `onlyRole` en funciones admin
- [ ] Sin `selfdestruct` — **deprecado desde Solidity 0.8.18+ (EIP-6780)**: en mainnet ya solo borra el código si el contrato fue creado en la misma transacción; no transfiere saldo. Eliminar de contratos nuevos.
- [ ] Sin `tx.origin` para autorización (usar `msg.sender`)

### Testnet
- [ ] Deploy y prueba completa en testnet
- [ ] Prueba de ataque de reentrancy con contrato malicioso
- [ ] Prueba de todos los flujos con fondos reales de testnet
- [ ] Verificar que el contrato es verificable en el explorer

### Auditoría externa (recomendada para >$100K TVL)
- [ ] Code4rena / Sherlock / Cantina / auditor independiente
- [ ] Presupuesto estimado: $5.000–$30.000 según complejidad

## Criterio de verificación
```bash
slither contracts/             # sin HIGH ni CRITICAL
pnpm dlx hardhat test               # 0 fallos incluyendo tests de seguridad
pnpm dlx hardhat coverage           # >95% cobertura
```
