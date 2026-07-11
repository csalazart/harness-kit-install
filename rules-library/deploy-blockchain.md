# Reglas — Deploy en Blockchain (EVM y Solana)
> Aplicar en cualquier proyecto de smart contracts antes y durante el despliegue.

---

## Orden obligatorio de despliegue

```
1. Tests locales (Hardhat Network / Anchor localnet) → todos en verde
2. Auditoría estática (Slither / cargo audit)        → sin críticos
3. Testnet                                           → validación completa
4. Mainnet                                           → solo con 1-3 OK
```

**Nunca saltar pasos.** Un paso fallido detiene el proceso.

---

## Variables de entorno — seguridad

- Claves privadas y API keys SIEMPRE en `.env` — nunca en el código
- `.env` en `.gitignore` — verificar antes de cada commit
- Usar `.env.example` con las claves sin valor como referencia
- Nombres descriptivos: `DEPLOYER_PRIVATE_KEY`, `ALCHEMY_API_KEY_POLYGON`

```bash
# .env.example
DEPLOYER_PRIVATE_KEY=
ALCHEMY_API_KEY_POLYGON=
ALCHEMY_API_KEY_BSC=
POLYGONSCAN_API_KEY=
FEE_RECIPIENT_ADDRESS=
```

---

## Scripts de despliegue (EVM — Hardhat)

- Un script por red si hay diferencias de configuración
- Un script genérico si la lógica es idéntica en todas las redes
- Siempre guardar la dirección desplegada en `deployments/{network}.json`
- Verificar el contrato en el explorer después del deploy

```javascript
// deployments/polygon.json
{
  "network": "polygon",
  "chainId": 137,
  "contractName": "MyContract",
  "address": "0x...",
  "deployedAt": "2026-01-01T00:00:00Z",
  "deployer": "0x...",
  "txHash": "0x..."
}
```

---

## Verificación en explorer (EVM)

```bash
# Hardhat + hardhat-etherscan
pnpm dlx hardhat verify --network polygon 0xCONTRACT_ADDRESS "arg1" "arg2"

# Con constructor args complejos — usar fichero de args
pnpm dlx hardhat verify --network polygon --constructor-args args.js 0xCONTRACT_ADDRESS
```

- Verificar SIEMPRE en testnet antes que en mainnet
- Guardar el link del contrato verificado en `deployments/{network}.json`

---

## Multi-chain (EVM)

- Mismo bytecode → mismo comportamiento garantizado en todas las redes EVM
- Diferente `chainId` → configurar en `hardhat.config.js` con variables de entorno
- Diferente dirección de contratos externos (USDC, oráculos) → parámetros del constructor, no hardcodeados
- Documentar las direcciones de dependencias por red en `deployments/dependencies.json`

```javascript
// hardhat.config.js
networks: {
  polygon: {
    url: `https://polygon-mainnet.g.alchemy.com/v2/${process.env.ALCHEMY_API_KEY_POLYGON}`,
    accounts: [process.env.DEPLOYER_PRIVATE_KEY],
    chainId: 137
  },
  bsc: {
    url: process.env.BSC_RPC_URL,
    accounts: [process.env.DEPLOYER_PRIVATE_KEY],
    chainId: 56
  }
}
```

---

## Deploy Solana (Anchor)

```bash
# Localnet
anchor test

# Devnet
anchor build
anchor deploy --provider.cluster devnet

# Mainnet
anchor build --verifiable    # build reproducible para verificación
anchor deploy --provider.cluster mainnet
```

- Guardar el Program ID en `Anchor.toml` y en `deployments/solana.json`
- Usar `anchor build --verifiable` en mainnet para builds reproducibles
- Verificar el programa en explorer.solana.com

---

## Checklist pre-mainnet

```
[ ] Todos los tests pasan (pnpm dlx hardhat test / anchor test)
[ ] Cobertura >90% de líneas
[ ] Slither / cargo audit sin issues críticos ni altos
[ ] Deploy y prueba completa en testnet
[ ] Funciones críticas probadas manualmente en testnet
[ ] Variables de entorno de mainnet configuradas y verificadas
[ ] feeRecipient / authority es una multisig (Gnosis Safe u equivalente)
[ ] ABI / IDL guardado en el repositorio
[ ] Dirección de deploy anotada en deployments/
[ ] Contrato verificado en explorer
```

---

## Gestión del deployer wallet

- Usar una wallet dedicada solo para deploys — no la wallet personal
- En mainnet: usar multisig como owner del contrato, no el deployer directamente
- El deployer solo necesita fondos para el gas del deploy inicial
- Transferir ownership a la multisig en la misma transacción o inmediatamente después

---

## Lo que NO hacer

- No desplegar en mainnet sin testnet previo completo
- No hardcodear el `DEPLOYER_PRIVATE_KEY` ni en scripts ni en config
- No reutilizar el mismo deployer EOA como owner permanente del contrato
- No olvidar verificar el contrato en el explorer — dificulta la auditoría y la confianza
- No ignorar warnings de Slither/cargo audit aunque sean de severidad baja
