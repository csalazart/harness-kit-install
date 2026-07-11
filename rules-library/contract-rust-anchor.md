# Reglas — Smart Contracts Solana con Anchor
> Aplicar en proyectos Solana que usen el framework Anchor (Rust).
> Versión mínima: **Anchor 0.30+** (`@coral-xyz/anchor ^0.30`). Solana CLI 1.18+.

---

## Estructura del programa

- Un módulo por responsabilidad — no poner toda la lógica en `lib.rs`
- `lib.rs` solo declara el programa y las instrucciones (entry points)
- Lógica de negocio en módulos separados: `state.rs`, `instructions/`, `errors.rs`
- Tipos y structs en `state.rs`

```
programs/my-program/src/
├── lib.rs                  ← Declaración del programa e instrucciones
├── state.rs                ← Structs de cuentas y tipos
├── errors.rs               ← Códigos de error custom
└── instructions/
    ├── mod.rs
    ├── initialize.rs
    └── execute.rs
```

## Cuentas — validación con constraints

- Validar SIEMPRE las cuentas en la struct de contexto con `#[account(...)]`
- No asumir que una cuenta es válida si no tiene constraint explícito
- Usar `has_one` para verificar relaciones entre cuentas
- Usar `constraint` con expresión para validaciones custom

```rust
#[derive(Accounts)]
pub struct Execute<'info> {
    #[account(mut, has_one = authority)]
    pub vault: Account<'info, Vault>,

    #[account(mut)]
    pub authority: Signer<'info>,

    #[account(
        mut,
        constraint = recipient.key() != vault.key() @ ErrorCode::InvalidRecipient
    )]
    pub recipient: SystemAccount<'info>,

    pub system_program: Program<'info, System>,
}
```

## PDAs — Program Derived Addresses

- Derivar PDAs con seeds semánticos y descriptivos
- Documentar las seeds de cada PDA con un comentario
- Verificar el bump en la constraint: `seeds = [...], bump = account.bump`
- Guardar el bump en la cuenta al inicializarla para no recalcularlo

```rust
#[account(
    init,
    payer = payer,
    space = 8 + UserAccount::INIT_SPACE,
    seeds = [b"user", payer.key().as_ref()],
    bump
)]
pub user_account: Account<'info, UserAccount>,
```

## Errores custom

- Definir TODOS los errores en `errors.rs` con `#[error_code]`
- Mensajes de error descriptivos — ayudan en debugging y en el frontend
- Nunca usar `panic!` ni `unwrap()` en producción — manejar con `?` y errores custom

```rust
#[error_code]
pub enum ErrorCode {
    #[msg("The vault is already initialized")]
    AlreadyInitialized,
    #[msg("Insufficient funds to complete the operation")]
    InsufficientFunds,
    #[msg("The provided recipient is invalid")]
    InvalidRecipient,
}
```

## Instrucciones

- Una instrucción por responsabilidad — no combinar múltiples acciones en una
- Validar primero, mutar estado después (Checks-Effects-Interactions)
- Retornar `Result<()>` siempre — usar `?` para propagar errores
- Emitir eventos con `emit!()` para operaciones importantes

```rust
pub fn transfer_funds(ctx: Context<TransferFunds>, amount: u64) -> Result<()> {
    // Checks
    require!(ctx.accounts.vault.balance >= amount, ErrorCode::InsufficientFunds);

    // Effects
    ctx.accounts.vault.balance -= amount;

    // Interactions
    // ... CPI o transfer aquí

    emit!(FundsTransferred {
        from: ctx.accounts.vault.key(),
        amount,
    });

    Ok(())
}
```

## Espacio de cuentas

- Calcular el espacio exacto con `INIT_SPACE` o manualmente
- Añadir `8` bytes para el discriminador de Anchor
- Para strings: `4 + max_length` (4 bytes para la longitud)
- Para Vec: `4 + (element_size * max_elements)`

```rust
#[account]
#[derive(InitSpace)]
pub struct UserAccount {
    pub authority: Pubkey,    // 32
    pub balance: u64,         // 8
    pub bump: u8,             // 1
    #[max_len(50)]
    pub name: String,         // 4 + 50
}
// Space total: 8 (discriminador) + 32 + 8 + 1 + 54 = 103
```

## Testing

- Tests en TypeScript con `@coral-xyz/anchor ^0.30`
- Un fichero de test por instrucción o flujo principal
- Usar `provider.connection.confirmTransaction()` para esperar confirmaciones
- Derivar PDAs en los tests igual que en el programa
- Preferir `AnchorProvider.env()` para obtener el provider en local validator

```typescript
const [userPda, bump] = PublicKey.findProgramAddressSync(
  [Buffer.from("user"), wallet.publicKey.toBuffer()],
  program.programId
)
```

## Lo que NO hacer

- No usar `unwrap()` o `expect()` en código de producción — siempre `?`
- No almacenar datos sensibles en cuentas on-chain sin cifrado
- No ignorar el retorno de CPIs — verificar que se ejecutaron correctamente
- No reutilizar bumps sin verificarlos contra la cuenta almacenada
- No hacer operaciones aritméticas sin verificar overflow (usar `checked_add`, `checked_sub`)

## Criterio de verificación
```bash
cargo build-bpf                  # compilar programa sin errores
cargo clippy -- -D warnings      # sin warnings
anchor test                      # 0 fallos (levanta validator local automáticamente)
```
