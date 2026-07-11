# Reglas — Seguridad Web2
> Copiar a `.harness/rules/security.md` y adaptar al proyecto.

## OWASP Top 10 — checks obligatorios antes de producción

| Vulnerabilidad | Mitigación |
|----------------|-----------|
| Injection (SQL, NoSQL, CMD) | ORM/queries parametrizadas, nunca interpolación directa |
| Broken Authentication | JWT con expiración corta + refresh tokens, rate limiting en login |
| XSS | Escapado automático en templates, CSP headers, sanitizar HTML externo |
| Insecure Direct Object Reference | Verificar que el usuario tiene permiso sobre el recurso solicitado |
| Security Misconfiguration | Helmet, CORS restrictivo, no exponer stack traces, deshabilitar X-Powered-By |
| Sensitive Data Exposure | HTTPS siempre, no logear datos sensibles, encriptar en reposo |
| Broken Access Control | Middleware de auth en TODAS las rutas protegidas, principio de mínimo privilegio |
| CSRF | SameSite cookies, tokens CSRF en formularios |
| Using Components with Known Vulnerabilities | `pnpm audit` / `pip audit` en CI, actualizar dependencias |
| Insufficient Logging | Logar intentos de auth fallidos, accesos a datos sensibles, errores 5xx |

## Checklist pre-producción

### Código
- [ ] Sin secrets hardcodeados (usar `git secrets` o `truffleHog`)
- [ ] `pnpm audit` sin vulnerabilidades HIGH/CRITICAL
- [ ] Validación de entrada en TODOS los endpoints
- [ ] Error messages no exponen información interna

### Configuración
- [ ] HTTPS configurado (nunca HTTP en producción)
- [ ] Headers de seguridad: CSP, HSTS, X-Frame-Options
- [ ] CORS con lista blanca explícita
- [ ] Rate limiting en endpoints de auth y públicos

### Acceso
- [ ] Contraseñas hasheadas con bcrypt/argon2 (nunca MD5/SHA1)
- [ ] Tokens con expiración configurada
- [ ] Logs de acceso y errores habilitados

## Criterio de verificación
```bash
pnpm audit --audit-level=high    # sin HIGH ni CRITICAL
# Ejecutar OWASP ZAP o similar en staging antes de producción
```
