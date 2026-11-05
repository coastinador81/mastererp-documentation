# Evidencia DEV — Fase 7 Infraestructura

Fecha: 2026-11-04. Alcance exclusivo: cuenta `277352717693`, región
`us-west-1`, ambiente DEV. Todas las consultas fueron de inventario o detección
de drift; no se creó, actualizó ni eliminó ningún recurso.

## Identidad y recursos

STS confirmó el usuario `andrea_madriz`. Se encontraron diez stacks MasterERP
DEV en estado estable: ocho `IN_SYNC` y dos con drift conocido.

| Recurso compartido | Estado y propietario físico |
| --- | --- |
| RDS `mastererp-dev-data` | Disponible; stack `mastererp-dev-data`, propiedad de Database. |
| Migrador `mastererp-dev-database-migrator` | Lambda oficial de Database; no duplicarla en Infrastructure. |
| Admin API `qx8nmypdhi` | Stack `mastererp-dev-onboarding`; conservar API y IDs físicos. |
| Client API `btqqku792c` | Stack `mastererp-dev-http-apis`. |
| Agent API `ebn01q74hh` | Stack `mastererp-dev-http-apis`. |
| Administración | Stack `mastererp-dev-administration`, hijo de `mastererp-dev-finance`. |

El Admin API posee el authorizer JWT `mastererp-dev-admin-jwt`, ID `49et65`,
con issuer Cognito de `us-west-1`. Los candidatos verticales deben reutilizarlo;
no deben crear otro API ni publicar sus rutas administrativas sin JWT.

La correspondencia de recursos críticos con stacks no mostró propietarios
físicos duplicados. Las declaraciones eliminadas que CloudFormation aún conserva
en onboarding son drift, no recursos activos ni autorización para recrearlos.

## Resultado de drift

`mastererp-dev-finance-frontend`, `mastererp-dev-finance`,
`mastererp-dev-administration`, `mastererp-dev-application-data-access`,
`mastererp-dev-identity`, `mastererp-dev-uonify-google-connection`,
`mastererp-dev-uonify-hosting` y `mastererp-dev-http-apis`: `IN_SYNC`.

`mastererp-dev-data`: `DRIFTED`, un recurso. `DatabaseSecurityGroup` contiene
accesos PostgreSQL añadidos para un runner y dos security groups. El RDS está
marcado público, sin cifrado en reposo, sin deletion protection y con un día de
retención. Corregir estas características puede cambiar conectividad o exigir
reemplazo; queda fuera de un despliegue funcional implícito y requiere un plan
de endurecimiento separado.

`mastererp-dev-onboarding`: `DRIFTED`, once recursos eliminados: cinco Lambdas
históricas y seis permisos asociados. Un update basado en la plantilla actual
podría recrearlos. Los tres handlers vigentes de onboarding, el Admin API y sus
rutas se conservan.

## Puerta para change sets

1. No actualizar `mastererp-dev-data` ni `mastererp-dev-onboarding` con sus
   plantillas actuales.
2. Componer los módulos verticales en stacks propietarios nuevos o plantillas
   de adopción que solo añadan recursos y reutilicen referencias existentes.
3. Empaquetar artefactos desde commits publicados y usar claves S3 por hash.
4. Preparar un change set por propietario sin ejecutarlo.
5. Rechazar `Remove`, `Replacement=True`, cambios al Admin API compartido,
   authorizer, rutas ajenas, RDS, secretos o red.
6. Conservar plantilla, parámetros, artefactos y evidencia previa para rollback.

Los próximos candidatos son Catálogo, Operaciones, Transporte y Storefront.
Antes del despliegue deben pasar validación CloudFormation, revisión de permisos,
smoke autenticado y comprobación de aislamiento tenant/empresa.

## Change sets preparados

Código empaquetado desde Platform `2bb8e1449c4dfb55272b9b1ec46f503ce3613bf8`;
composición y preparador desde Infrastructure
`47b5afae80934e3c475af2b7a1b95b231d3246cd`. Los doce ZIP incluyen binarios
`psycopg` para Python 3.12 arm64 y fueron publicados en el bucket privado con
claves que contienen commit, módulo y SHA-256.

| Stack nuevo | Change set | Revisión |
| --- | --- | --- |
| `mastererp-dev-catalog` | `phase7-catalog-47b5afae8093` | 55 Add |
| `mastererp-dev-operations` | `phase7-operations-47b5afae8093` | 39 Add |
| `mastererp-dev-transport` | `phase7-transport-47b5afae8093` | 10 Add |
| `mastererp-dev-digital-sales` | `phase7-digital-sales-47b5afae8093` | 12 Add |

Los cuatro planes iniciales finalizaron `CREATE_COMPLETE`: 116 `Add`, ningún
`Modify`, `Remove` o reemplazo. Storefront conecta su ruta pública a Client API
mediante una integración separada; las operaciones administrativas usan Admin
API con el authorizer JWT compartido.

El preparador publicado no implementa `execute-change-set`. Antes de ejecutar
cada plan se debe conservar su registro, observar el stack hasta estabilizar,
verificar Lambdas y rutas exactas y realizar smoke autenticado. Si falla la
creación de un stack nuevo, permitir el rollback de CloudFormation, inspeccionar
eventos y no reintentar ampliando permisos.

## Resultado del despliegue

El primer smoke de Catálogo detectó que los ZIP no incluían PyJWT, cryptography
ni las claves públicas fijadas de Cognito. La secuencia se detuvo antes de
Operaciones. Platform `b785d6c` corrigió el empaquetado reproducible y
`b4fca28` corrigió el orden de validación identidad/tenant en cinco adaptadores.
No se modificaron datos ni se ampliaron permisos.

Los tres planes pendientes se regeneraron desde Platform
`b4fca288ce4010006f3ae34a6e37eca3719eaaa4` e Infrastructure
`d93249bd34c8f1dc944b59fa1a64da65ec1b3930`. Los placeholders eliminados
estaban `REVIEW_IN_PROGRESS` y contenían cero recursos.

| Stack | Estado | Recursos | Smoke técnico |
| --- | --- | ---: | --- |
| `mastererp-dev-catalog` | `CREATE_COMPLETE` | 55 | 6 Lambdas; sin JWT → 401 |
| `mastererp-dev-operations` | `CREATE_COMPLETE` | 39 | 4 Lambdas; sin JWT → 401 |
| `mastererp-dev-transport` | `CREATE_COMPLETE` | 10 | Repartidores; sin JWT → 401 |
| `mastererp-dev-digital-sales` | `CREATE_COMPLETE` | 12 | Admin → 401; Storefront valida entrada |

Los cuatro stacks quedaron `IN_SYNC`, sin recursos fallidos. HTTP real confirmó
401 en Admin API. Client API expone `GET /v1/storefront/catalog` sin JWT; las
cuatro rutas administrativas de tiendas conservan JWT `49et65`.

Rollback: cada dominio es un stack nuevo e independiente. Ante fallo durante
creación se permite rollback de CloudFormation y se preservan artefactos por
commit/SHA. Una retirada posterior requiere change set y evidencia de Fase 8.
