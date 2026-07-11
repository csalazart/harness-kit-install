# Reglas — Backend PHP Symfony
> Copiar a `.harness/rules/backend.md` y adaptar al proyecto.

## Versión y configuración
- Symfony **7.2** (LTS) + PHP **8.3+** (recomendado 8.4)
- `declare(strict_types=1);` en todos los ficheros
- Instalar solo los componentes necesarios — no usar `symfony/website-skeleton` si no hace falta todo
- Variables de entorno: `.env` + `.env.local` (nunca commitear `.env.local`)
- PHPStan nivel 8 + PHP_CodeSniffer PSR-12 desde el inicio

## Estructura de carpetas (Symfony estándar)
```
src/
├── Controller/          ← solo recibe Request, delega, devuelve Response
├── Service/             ← lógica de negocio inyectable
├── Repository/          ← extienden ServiceEntityRepository de Doctrine
├── Entity/              ← entidades Doctrine con anotaciones/atributos
├── Form/                ← Symfony Form Types
├── EventListener/       ← listeners y subscribers
├── Exception/           ← excepciones del dominio
└── DataTransferObject/  ← DTOs para entrada/salida de datos
config/
├── packages/            ← configuración por bundle
├── routes/              ← definición de rutas
└── services.yaml        ← inyección de dependencias
templates/               ← Twig (si hay vistas)
migrations/              ← migraciones Doctrine
tests/
├── Unit/
├── Integration/
└── Functional/          ← WebTestCase de Symfony
```

## Symfony — convenciones clave

### Controladores
```php
#[Route('/cofres', name: 'cofre_')]
final class CofreController extends AbstractController
{
    public function __construct(
        private readonly CofreService $cofreService
    ) {}

    #[Route('/{id}', name: 'show', methods: ['GET'])]
    public function show(int $id): JsonResponse
    {
        return $this->json($this->cofreService->obtener($id));
    }
}
```
- Controladores `final` y delgados — sin lógica de negocio
- Inyección en constructor, nunca `$this->get()` (deprecated)
- Usar atributos PHP 8 (`#[Route]`, `#[IsGranted]`) — no anotaciones YAML/XML

### Servicios e inyección de dependencias
- Todo servicio en `services.yaml` con autowiring activado
- Servicios inmutables: propiedades `readonly` cuando sea posible
- `#[Autowire]` para inyecciones específicas de parámetros/env vars
- Evitar el Service Locator — no inyectar el Container

### Doctrine ORM
```php
#[ORM\Entity(repositoryClass: CofreRepository::class)]
#[ORM\Table(name: 'cofres')]
class Cofre
{
    #[ORM\Id]
    #[ORM\GeneratedValue]
    #[ORM\Column]
    private ?int $id = null;

    #[ORM\Column(length: 50, enumType: EstadoCofre::class)]
    private EstadoCofre $estado = EstadoCofre::ABIERTO;
}
```
- Migraciones siempre con `doctrine:migrations:diff` — nunca `schema:update` en producción
- Repositorios extienden `ServiceEntityRepository` — sin SQL en controladores ni servicios
- Lazy loading consciente — usar `JOIN FETCH` cuando se sabe que se necesitarán relaciones

### Seguridad
- Configurar `security.yaml` con firewalls explícitos
- `#[IsGranted('ROLE_X')]` en controladores — nunca lógica de roles en servicios
- CSRF automático en formularios Symfony — no desactivar
- Validación con `symfony/validator` y constraints en los DTOs o entidades
- Passwords con `UserPasswordHasherInterface` — nunca manualmente

## Manejo de errores
- `EventSubscriber` para `KernelEvents::EXCEPTION` — respuesta uniforme de errores
- Excepciones del dominio mapeadas a HTTP status codes en el event listener
- Nunca capturar `\Throwable` o `\Exception` en lógica de negocio salvo para relanzar

## Testing
```bash
# Test unitario de un servicio
class CofreServiceTest extends TestCase { ... }

# Test funcional (HTTP)
class CofreControllerTest extends WebTestCase { ... }

# Test de integración con BD real
class CofreRepositoryTest extends KernelTestCase { ... }
```

## Criterio de verificación
```bash
php bin/console lint:container        # DI container válido
php bin/console doctrine:schema:validate  # Schema sincronizado
vendor/bin/phpstan analyse            # nivel 8, 0 errores
vendor/bin/phpunit                    # 0 fallos
vendor/bin/phpunit --coverage-text    # >80% cobertura
```
