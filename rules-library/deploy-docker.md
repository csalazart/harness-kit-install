# Reglas — Deploy con Docker / CI-CD
> Copiar a `.harness/rules/deploy.md` y adaptar al proyecto.

## Regla de oro (no negociable)
> **NUNCA** hacer deploy en producción sin confirmación explícita del director.
> Mostrar siempre el checklist y esperar respuesta afirmativa.

## Orden de entornos (obligatorio)
```
1. Local (Docker Compose)     ← desarrollo
2. Staging / Preview          ← validación previa
3. Producción                 ← 🚨 requiere confirmación explícita
```

## Dockerfile — buenas prácticas
```dockerfile
# Multi-stage build para imagen mínima
FROM node:22-alpine AS builder
WORKDIR /app
COPY package.json pnpm-lock.yaml ./
RUN pnpm install --frozen-lockfile --prod

FROM node:22-alpine AS runner
WORKDIR /app
COPY --from=builder /app/node_modules ./node_modules
COPY . .
# No root
USER node
EXPOSE 3000
CMD ["node", "dist/index.js"]
```

## Variables de entorno en producción
- Nunca en el Dockerfile ni en la imagen
- Usar secrets del CI/CD (GitHub Actions Secrets, AWS Secrets Manager, etc.)
- `.env.production` fuera del repositorio — nunca commitear
- Variables de configuración (no secretas): pueden ir en el docker-compose.yml

## Checklist pre-deploy producción
- [ ] Tests 100% pasando
- [ ] Build sin errores
- [ ] Variables de entorno de producción configuradas
- [ ] Backup de base de datos reciente
- [ ] Revisión del docker-compose / k8s manifests
- [ ] Rollback plan definido
- [ ] **CONFIRMACIÓN EXPLÍCITA DEL DIRECTOR**

## Registro de deploys
Cada deploy debe registrarse en `deployments/log.json`:
```json
{
  "date": "ISO timestamp",
  "environment": "production",
  "version": "v1.2.3",
  "deployer": "nombre",
  "image": "ghcr.io/usuario/proyecto:sha",
  "rollback_to": "sha_anterior"
}
```

## Criterio de verificación
- `docker build` sin errores
- `docker compose up` levanta todos los servicios
- Health check endpoint responde 200
- Deploy registrado en `deployments/log.json`
