# Evidencia DEV — Fase 8

Fecha: 2026-11-05. Cuenta `277352717693`, región `us-west-1`.

Decisión posterior del Product Owner: Uonify conserva Chat operativo y módulos
en una misma interfaz matriz. Storefront migra a Cognito MasterERP con nuevo
inicio de sesión y sin vinculación automática por correo. Esta decisión define
el trabajo restante; no acredita por sí sola checkout ni cutover.

## Cognito e identidad de clientes Storefront

El cliente `mastererp-dev-storefront-web` quedó desplegado en el pool MasterERP
con Authorization Code + PKCE, sin secreto, callback exclusivo de
`tienda-dev.tumandaito.com`, tokens de 15 minutos y refresh de un día. No se
modificó el cliente Uonify ni se eliminó el chat.

Database `406fe4c` aplicó el esquema 039 en RDS DEV con preflight, dos pasadas
del modelo y 23 pruebas aprobadas; correlación
`e558cf87-f801-4d0a-8d4b-c09e05b96302`. La invitación guarda únicamente el
SHA-256 del secreto y la reclamación enlaza por `issuer + sub`; el correo es
perfil verificado, nunca llave de coincidencia, y no se crea `user_tenants`.

Platform `1f04243` e Infrastructure `3b7f2f0` añadieron el flujo HTTP. El change
set `phase8-storefront-identity-1f04243` agregó un authorizer y dos rutas;
Lambda e integraciones se actualizaron sin reemplazo. Solo se reemplazó el
permiso invocable para ampliar su ARN desde catálogo a `/v1/storefront/*`.
El stack finalizó `UPDATE_COMPLETE`.

Smokes efectivos:

- catálogo público: HTTP 200, 11 categorías y 17 productos disponibles;
- reclamación sin JWT: HTTP 401;
- invitación administrativa con cliente inexistente: HTTP 404 desde RDS, sin
  crear invitación;
- reclamación con ID token real del cliente Storefront y correo verificado,
  pero invitación inexistente: HTTP 404 desde RDS, sin crear identidad;
- el atributo QA temporal volvió a `email_verified=false` y su acceso al tenant
  Tumandaito volvió a HTTP 403.

Esto prueba Cognito, ambos authorizers, API Gateway, Lambda y el contrato SQL sin
contaminar datos de negocio. El checkout canónico sigue pendiente.

## Smoke autenticado

Se ejecutó `mastererp-infrastructure/scripts/verify_phase8_tumandaito.py`
contra las APIs Admin y Client reales. El runner obtuvo un access token Cognito
sin persistir credenciales, creó una membresía temporal para el actor QA
sintético, realizó únicamente lecturas y retiró esa membresía en `finally`.

Resultado: 14 controles aprobados.

| Capacidad | Resultado HTTP | Conteo comparado |
| --- | ---: | ---: |
| Categorías | 200 | 11 |
| Productos | 200 | 27 |
| Nodos | 200 | 5 |
| Inventario | 200 | 42 |
| Repartidores | 200 | 6 |
| Pedidos | 200 | 9 |
| Preparación | 200 | lista válida |
| Rutas | 200 | 3 |
| Tiendas | 200 | 1 |
| Storefront público | 200 | catálogo válido |

Los conteos se compararon en la misma ejecución con el snapshot de solo lectura
de Fase 6, identificado por SHA-256
`6a6d9c5030bd7c63070a91876b9114b0d8d36553cddf8c15bc103589084a212d`.
Un tenant ajeno recibió 403. Antes y después de la membresía temporal,
Tumandaito también devolvió 403 para el actor QA; esto verifica la limpieza.

## Incidente controlado del runner

La primera ejecución fue interrumpida después de crear la membresía temporal.
El helper se endureció para recuperar únicamente la única relación entre el
usuario QA sintético, identificado por email e issuer, y el tenant exacto
`tumandaito`. La relación huérfana se retiró, se verificó 403 y luego el smoke
completo aprobó. No se modificaron datos de negocio ni membresías reales.

## Límite de la evidencia

Esto confirma contratos de lectura, autenticación, autorización, aislamiento y
paridad de conteos en sombra. No confirma delta final, escrituras end-to-end,
conmutación de frontends, rollback operativo ni retiro de recursos legacy.
Esos puntos siguen siendo puertas obligatorias antes del cutover.

## Delta y restauración del origen

El 2026-11-05 se repitió la exportación consistente de `T001` desde la cuenta
origen `941751053509`, región `us-west-2`. Se seleccionaron 296 de 296 registros
y el SHA-256 volvió a ser exactamente
`6a6d9c5030bd7c63070a91876b9114b0d8d36553cddf8c15bc103589084a212d`.
Por tanto, no existía delta desde la carga reconciliada de Fase 6 al momento de
la comprobación.

Se creó el backup
`tumandaito-dev-core-before-mastererp-phase8-20261105`, estado `AVAILABLE`, ARN
terminado en `backup/01793889299911-969f4647`. Se restauró en la tabla aislada
`tumandaito-dev-core-phase8-rollback-drill-20261105`; el verificador leyó 296
registros y confirmó el mismo SHA-256. La tabla temporal fue eliminada después
del ensayo; el backup permanece recuperable.

Esta prueba acredita recuperación de los datos DynamoDB. Aún falta ensayar la
conmutación de tráfico de los canales y ejecutar escrituras E2E antes de marcar
completa la puerta conjunta de pruebas/datos/rollback.

## Uonify: recorrido modular integrado

El 2026-11-05 se publicó en `app-dev.uonify.com` el app-shell matriz con
Productos y Pedidos mediante composición de código fuente, sin iframe ni copia
de pantallas. Ambos conservan su build independiente y comparten el mismo
componente con Uonify. El menú exige módulo y permiso; cada petición vuelve a
ser autorizada por API/PostgreSQL.

El preflight de navegador detectó que el API Admin permitía únicamente el
header financiero `x-mastererp-tenant-id`, mientras las APIs canónicas usan
`x-tenant-id`. CloudFormation modificó solo `CorsConfiguration` de
`AdminHttpApi`, sin reemplazo. Se conservaron los identificadores del API y sus
recursos. El preflight real devuelve 204 y autoriza ambos headers únicamente
para los orígenes existentes.

`scripts/validate_phase8_uonify.cjs` ejecutó navegador real con Chrome y una
membresía QA sintética temporal. Resultado aprobado:

- login Cognito y contexto Tumandaito;
- Productos visible y 27 registros obtenidos por HTTP 200;
- Pedidos visible y 9 registros obtenidos por HTTP 200;
- navegación dentro de `app-dev.uonify.com`, sin errores de navegador;
- membresía temporal retirada al finalizar.

En recorridos posteriores el mismo runner confirmó en una sola sesión:

| Módulo | Resultado observado |
| --- | ---: |
| Productos | 27 |
| Categorías | 11 |
| Atributos | 0, contrato HTTP válido |
| Marcas | 0, contrato HTTP válido |
| Proveedores de catálogo | 8 |
| Nodos | 5 |
| Inventario | 42 en 5 nodos |
| Pedidos | 9 |
| Preparación | 5 |
| Rutas | 3 |
| Repartidores | 6 |
| Tiendas | 1 |

Todos se abrieron desde `app-dev.uonify.com`, con Cognito y contexto compartido,
sin iframe y sin errores de navegador. La membresía QA temporal se retiró en
cada ejecución. Esta evidencia acredita lecturas; las escrituras reversibles y
el checkout autenticado siguen siendo puertas separadas.

El mismo runner, activado explícitamente en modo escritura, guardó sin cambiar
valores funcionales la configuración vigente de una Tienda, un Repartidor y un
Nodo. Los tres comandos recorrieron formulario Uonify → API autorizada → RDS y
respondieron HTTP 200; versiones y auditoría avanzaron como en una escritura
real. La membresía QA volvió a retirarse. Esto acredita escritura administrativa
y de Transporte, pero no reemplaza la prueba pendiente de checkout Storefront.
