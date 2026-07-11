# Reglas — Testing con Hardhat
> Aplicar en proyectos de smart contracts EVM que usen Hardhat + Chai + ethers.js v6.

---

## Estructura de tests

- Un fichero de test por contrato: `ContractName.test.js`
- Agrupar con `describe` por función o flujo: `describe("openChest", () => {})`
- Cada `it` prueba UNA sola cosa — nombre descriptivo en presente: `"reverts if caller is not owner"`
- Orden dentro de cada `describe`: happy path primero, casos de error después

```javascript
describe("NombreContrato", () => {
  describe("nombreFuncion", () => {
    it("does X when Y", async () => { ... })      // happy path
    it("reverts if Z", async () => { ... })        // error cases
  })
})
```

## Fixtures — estado limpio por test

- Usar `loadFixture` de Hardhat para reiniciar estado entre tests
- Definir una fixture por escenario base — no compartir estado mutable entre tests
- Deploy del contrato siempre dentro de la fixture, nunca en `beforeEach` manual

```javascript
const { loadFixture } = require("@nomicfoundation/hardhat-toolbox/network-helpers")

async function deployFixture() {
  const [owner, user1, user2] = await ethers.getSigners()
  const Contract = await ethers.getContractFactory("MyContract")
  const contract = await Contract.deploy()
  return { contract, owner, user1, user2 }
}

it("does something", async () => {
  const { contract, owner } = await loadFixture(deployFixture)
  // ...
})
```

## ethers.js v6 — sintaxis correcta

```javascript
// Unidades
ethers.parseEther("1.0")          // en vez de ethers.utils.parseEther
ethers.formatEther(amount)         // en vez de ethers.utils.formatEther
ethers.parseUnits("100", 6)        // para tokens con decimales

// Wallets y signers
const [owner, user] = await ethers.getSigners()
await contract.connect(user).transfer(...)

// BigInt — ethers v6 usa BigInt nativo
const amount = 100n                // BigInt literal
expect(balance).to.equal(100n)
```

## Testear eventos (emit)

```javascript
await expect(contract.mint(user.address))
  .to.emit(contract, "Transfer")
  .withArgs(ethers.ZeroAddress, user.address, tokenId)
```

## Testear reverts

```javascript
// Revert con mensaje custom
await expect(contract.connect(user).adminFunction())
  .to.be.revertedWith("Ownable: caller is not the owner")

// Revert con error custom (Solidity custom errors)
await expect(contract.connect(user).adminFunction())
  .to.be.revertedWithCustomError(contract, "UnauthorizedCaller")
  .withArgs(user.address)

// Revert sin mensaje
await expect(contract.badCall())
  .to.be.reverted
```

## Testear balances y cambios de estado

```javascript
// Cambio de balance ETH
await expect(contract.withdraw())
  .to.changeEtherBalance(owner, ethers.parseEther("1.0"))

// Cambio de balance ERC20
await expect(contract.claimTokens())
  .to.changeTokenBalance(token, user, 100n)
```

## Cobertura mínima recomendada

- Funciones: 100% — toda función pública/external tiene al menos 1 test
- Líneas: >90%
- Branches: >85% — cubrir todos los `require` y `if` relevantes
- Ejecutar: `pnpm dlx hardhat coverage`

## Escenarios de ataque a cubrir

- Llamada desde dirección no autorizada (onlyOwner, onlyRole, modifier custom)
- Reentrancy: llamar dos veces antes de que el estado se actualice
- Integer overflow/underflow (menos crítico en Solidity ^0.8, pero verificar lógica)
- Manipulación de timestamp si el contrato depende de `block.timestamp`
- Frontrunning si el contrato tiene funciones sensibles al orden

## Lo que NO hacer

- No usar `beforeEach` con estado mutable compartido — usar `loadFixture`
- No hardcodear addresses de contratos desplegados en tests — usar las del deploy
- No hacer `console.log` en tests de producción — solo en debugging temporal
- No testear solo el happy path — los casos de error son igual de importantes
- No ignorar warnings del compilador antes de correr los tests
