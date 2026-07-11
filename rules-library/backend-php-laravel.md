# Reglas — Backend PHP Laravel
> Copiar a `.harness/rules/backend.md` y adaptar al proyecto.

## Versión y configuración
- Laravel **12.x** (o 11.x en proyectos en mantenimiento) + PHP **8.3+** (recomendado 8.4)
- `declare(strict_types=1);` en todos los ficheros propios
- Instalar Pint (formateador oficial Laravel) y Larastan (PHPStan para Laravel) desde el inicio
- Variables de entorno: `.env` + `.env.testing` — nunca commitear `.env` con valores reales
- `APP_DEBUG=false` y `APP_ENV=production` en producción — verificar antes de deploy

## Estructura de carpetas (Laravel estándar + limpieza)
```
app/
├── Http/
│   ├── Controllers/     ← delgados, solo HTTP in/out
│   ├── Requests/        ← Form Requests para validación (obligatorio)
│   ├── Resources/       ← API Resources para transformar respuestas
│   └── Middleware/      ← middleware personalizado
├── Models/              ← Eloquent models
├── Services/            ← lógica de negocio (sin Eloquent directo)
├── Repositories/        ← acceso a datos (opcional, para proyectos grandes)
├── Actions/             ← una acción = una operación de negocio (patrón Laravel moderno)
├── Events/              ← eventos del dominio
├── Listeners/           ← handlers de eventos
└── Exceptions/          ← excepciones tipadas + Handler personalizado
database/
├── migrations/
├── seeders/
└── factories/
tests/
├── Unit/
├── Feature/             ← tests HTTP con TestCase de Laravel
└── Integration/
```

## Laravel — convenciones clave

### Controladores (Resource Controllers)
```php
final class CofreController extends Controller
{
    public function __construct(
        private readonly CofreService $cofreService
    ) {}

    public function show(int $id): CofreResource
    {
        return new CofreResource(
            $this->cofreService->obtener($id)
        );
    }
}
```
- Controladores `final` y delgados — sin lógica de negocio
- Usar `php artisan make:controller --resource` para CRUD
- Siempre devolver API Resources (`JsonResource`) — nunca `Model` directo como JSON

### Form Requests (validación obligatoria)
```php
final class CrearCofreRequest extends FormRequest
{
    public function rules(): array
    {
        return [
            'nombre'   => ['required', 'string', 'max:100'],
            'tema'     => ['required', Rule::enum(TemaCofrE::class)],
            'rareza'   => ['required', 'in:Común,Raro,Épico,Legendario'],
        ];
    }
}
```
- **Nunca** validar en el controlador — siempre Form Request
- Autorización en `authorize()` del Form Request, no en el controlador

### Eloquent — buenas prácticas
```php
final class Cofre extends Model
{
    protected $fillable = ['nombre', 'tema', 'rareza', 'estado'];
    protected $casts = [
        'estado' => EstadoCofre::class,    // cast a enum
        'metadatos' => 'array',
    ];
    // Scopes para queries frecuentes
    public function scopeCerrados(Builder $query): Builder
    {
        return $query->where('estado', EstadoCofre::CERRADO);
    }
}
```
- `$fillable` explícito — nunca `$guarded = []` en producción
- Casts para tipos: usar enums, Carbon, arrays — no parsear manualmente
- Sin lógica de negocio en los Models — solo relaciones, scopes y casts
- Eager loading explícito (`with()`) — nunca dejar N+1 sin resolver

### Actions (patrón recomendado en Laravel moderno)
```php
final class AbrirCofreAction
{
    public function execute(Cofre $cofre, ?string $clave = null): void
    {
        // lógica de apertura — reutilizable desde Controller, Job, Command
    }
}
```
- Una Action por operación de negocio — más granular que los Services
- Se inyectan en controladores, jobs y commands por igual

### Jobs y Queues
- Procesos largos siempre en Jobs — nunca bloquear una request HTTP
- `ShouldQueue` para todo lo que no necesite respuesta inmediata
- Configurar `failed_jobs` table — siempre monitorear fallos

### Seguridad
- Rutas protegidas con middleware `auth:sanctum` o `auth:api`
- CSRF automático en rutas web — no deshabilitar
- Rate limiting con `RateLimiter` en `AppServiceProvider` o `RouteServiceProvider`
- Políticas (Policies) para autorización de recursos — nunca if/else de roles en controladores
- `Hash::make()` para contraseñas — nunca manualmente

### Migraciones
- Una migración = un cambio — nunca modificar migraciones ya ejecutadas en producción
- `php artisan migrate --pretend` antes de migrar en producción
- Siempre escribir el método `down()` correctamente

## Testing con Pest (recomendado) o PHPUnit
```php
// Pest — más expresivo para Laravel
it('opens a chest with correct key', function () {
    $cofre = Cofre::factory()->locked()->create();

    $response = $this->postJson("/api/cofres/{$cofre->id}/abrir", [
        'clave' => 'secreto',
    ]);

    $response->assertOk();
    expect($cofre->fresh()->estado)->toBe(EstadoCofre::ABIERTO);
});
```
- Usar `RefreshDatabase` o `LazilyRefreshDatabase` en tests con DB
- Factories para todos los modelos — nunca crear datos de test manualmente
- `Http::fake()` para mockear peticiones HTTP externas

## Criterio de verificación
```bash
php artisan config:clear && php artisan route:list  # sin errores de configuración
./vendor/bin/pint --test                            # formato correcto (sin cambios)
./vendor/bin/phpstan analyse                        # Larastan nivel 8, 0 errores
php artisan test                                    # 0 fallos
php artisan test --coverage --min=80               # >80% cobertura
```
