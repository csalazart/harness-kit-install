# Reglas — Backend PHP puro (POO sin framework)
> Copiar a `.harness/rules/backend.md` y adaptar al proyecto.

## Versión y configuración
- PHP **8.3+** — usar siempre la última versión estable (8.4 es la actual)
- `declare(strict_types=1);` en TODOS los ficheros sin excepción
- PSR-12 como estándar de estilo de código
- Composer para gestión de dependencias — sin vendoring manual
- Autoloading PSR-4 en `composer.json`

## Estructura de carpetas recomendada
```
src/
├── Controller/         ← recibe la request, delega, devuelve respuesta
├── Service/            ← lógica de negocio (sin HTTP, sin DB directa)
├── Repository/         ← acceso a datos (solo SQL/ORM aquí)
├── Model/              ← entidades del dominio (sin lógica de infraestructura)
├── Exception/          ← excepciones tipadas del dominio
├── Interface/          ← contratos (interfaces PHP)
└── Util/               ← helpers puros sin side effects
public/
└── index.php           ← único punto de entrada (front controller)
config/
└── *.php               ← configuración por entorno
tests/
```

## POO — reglas estrictas
- Toda clase tiene **una sola responsabilidad** (SRP)
- Preferir **composición sobre herencia** — herencia máximo 1 nivel
- **Inyección de dependencias** en constructor — nunca `new ClaseExterna()` dentro de un servicio
- Interfaces para todo lo que pueda tener más de una implementación
- `final class` por defecto — solo quitar `final` si hay razón explícita
- Propiedades de clase siempre tipadas: `private string $nombre;`
- Sin métodos estáticos en lógica de negocio (dificultan testing)

## Tipos y declaraciones
```php
<?php
declare(strict_types=1);

namespace App\Service;

final class CalculadoraFee
{
    public function calcular(int $monto, int $basisPoints): int
    {
        return (int) ($monto * $basisPoints / 10000);
    }
}
```
- Tipos en TODOS los parámetros y retornos de función
- Nunca `mixed` salvo en capas de serialización/deserialización
- `?Tipo` solo cuando null es un valor válido del dominio, no para "no sé el tipo"
- Usar `enum` (PHP 8.1+) en lugar de constantes para estados y categorías

## Seguridad (no negociables)
- **Nunca** construir queries SQL con concatenación de strings — usar PDO con prepared statements
- Validar y sanitizar TODA entrada externa antes de procesarla
- `password_hash()` con `PASSWORD_DEFAULT` (elige el mejor algoritmo disponible) o `PASSWORD_ARGON2ID` — nunca MD5/SHA1/BCrypt explícito
- `htmlspecialchars()` al renderizar cualquier dato de usuario en HTML
- Sesiones: `session_regenerate_id(true)` tras login
- Variables de entorno via `$_ENV` o librería `vlucas/phpdotenv` — nunca hardcodeadas
- `error_reporting(0)` y `display_errors = Off` en producción — logar con Monolog

## Manejo de errores
```php
// Excepciones tipadas del dominio
final class CoffreNoEncontradoException extends \DomainException {}
final class SaldoInsuficienteException extends \RuntimeException {}

// Captura por tipo, nunca \Exception genérico en lógica de negocio
try {
    $servicio->procesar($id);
} catch (CoffreNoEncontradoException $e) {
    // manejar caso específico
}
```
- Sin `@` para suprimir errores — resolver la causa raíz
- Sin `die()` ni `exit()` en lógica de negocio

## Base de datos
- PDO con prepared statements siempre
- Transacciones explícitas para operaciones múltiples
- Repositorio como única capa que conoce SQL — los servicios no saben de tablas

## Criterio de verificación
```bash
composer run lint        # PHP_CodeSniffer PSR-12
composer run analyse     # PHPStan nivel 8 (máximo)
composer run test        # PHPUnit — 0 fallos
composer run test-coverage  # >80% cobertura
```
