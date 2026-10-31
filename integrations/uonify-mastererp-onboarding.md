# Contrato Uonify ↔ MasterERP (onboarding)

Uonify es transversal; no depende del despliegue de `mastererp-dev-admin-api`.

## Invocación autorizada (fase futura)

1. Usuario autenticado en Uonify (identidad propia).
2. Uonify obtiene un token de servicio o delegación acotada al tenant.
3. Uonify llama operaciones equivalentes a:
   - crear sesión de onboarding,
   - registrar progreso,
   - listar módulos,
   - asignar módulos,
   - completar sesión.
4. La activación final sigue siendo **interna** (`mastererp-dev-onboarding-provisioner`).

## Reutilización de código

- Importar `tenant_onboarding.services.*` desde un adaptador en `uonify-core` (sin copiar SQL).
- PostgreSQL sigue siendo la fuente de verdad; Uonify no persiste historial conversacional en RDS.

## Pendiente

- Desplegar `uonify-dev-api`.
- Definir scopes OAuth/M2M entre Uonify y MasterERP.
- Mapear herramientas MCP (`tenant.setup`) a los mismos servicios.
