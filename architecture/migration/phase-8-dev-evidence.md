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
contaminar datos de negocio. El checkout canónico se acreditó posteriormente,
como registra la sección siguiente.

## Checkout Storefront canónico

El 2026-11-05 se aplicó la migración 040 con preflight, dos pasadas del modelo y
la batería SQL aprobada. El change set
`phase8-storefront-checkout-20261105` agregó únicamente la ruta
`POST /v1/storefront/checkout` y actualizó Lambda/integraciones sin reemplazos.
La ruta rechaza solicitudes sin JWT con 401. El API Client autoriza en DEV el
origen de la tienda y solo GET/POST/OPTIONS con headers canónicos.

El Storefront MasterERP se publicó aditivamente en
`/mastererp-phase8-candidate/`; no se reemplazó la raíz legacy, no hubo DNS ni
cutover. Un navegador real completó Cognito Authorization Code + PKCE, reclamó
una invitación de un solo uso y creó el pedido `ORD-A5A55BB18CB6` (USD 6,50).
RDS confirmó invitación `CLAIMED`, pedido `PENDING` y reserva de Inventario
`ACTIVE`, con una unidad del producto canónico. La reclamación repetida devolvió
200 con `replayed=true` y no creó otro pedido.

La identidad QA volvió a `email_verified=false`, no recibió membresía de tenant
y el helper temporal de fixture fue retirado del migrador. El smoke posterior
aprobó las 14 comprobaciones de lectura, aislamiento y limpieza. El único 404
observado por el navegador fue el `favicon.ico` del dominio administrado de
Cognito; Storefront y sus APIs no produjeron errores.

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

Esto confirma contratos de lectura, autenticación, autorización, aislamiento,
paridad de conteos y, con las comprobaciones posteriores, checkout E2E, delta
final y mecanismo de rollback de tráfico. No confirma cutover real, ventana de
observación ni retiro de recursos legacy; requieren autorización explícita.

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

Esta prueba acredita recuperación de los datos DynamoDB. El checkout E2E y el
ensayo aislado de rollback de tráfico se aprobaron posteriormente.

## Delta final y rollback de tráfico

El cierre repitió el exportador determinista durante una pausa controlada de
los 36 consumidores Lambda que referencian `tumandaito-dev-core`. El export
consistente seleccionó 296 de 296 registros de `T001` y produjo exactamente el
SHA-256 firmado
`6a6d9c5030bd7c63070a91876b9114b0d8d36553cddf8c15bc103589084a212d`.
Todas las concurrencias fueron restauradas a sus valores previos y verificadas.

El rollback de tráfico se ensayó sin cutover sobre
`/mastererp-phase8-candidate/index.html`: se sirvió temporalmente el índice
legacy y se confirmó su SHA-256
`06001961069a364966d275c80ac74b28f6cd9cd6e17a893dc5a5be1a1f124d7d`;
luego se restauró el candidato y se confirmó su SHA-256
`d18fd0ceeb5147160525bc2163d8e5c203f77f58a1dcf59bff8d1c6f2ad9647c`.
La raíz legacy permaneció HTTP 200 y no hubo cambios de DNS.

## Cutover DEV autorizado

Con autorización explícita, el índice legacy se respaldó en
`mastererp-phase8-rollback/index-pre-cutover-20261105.html` y el índice
candidato se promovió a la raíz de `tmi-dev-client`. La invalidación
`I2504DQPRVP400LUPXXZ9QAIWG` de la distribución `E2PLU72V9YKM01` terminó
correctamente. El índice servido por `tienda-dev.tumandaito.com` coincidió con
el candidato mediante SHA-256
`d18fd0ceeb5147160525bc2163d8e5c203f77f58a1dcf59bff8d1c6f2ad9647c`;
JavaScript, CSS y `/#/chat` respondieron HTTP 200. No se cambió DNS ni se
eliminaron recursos legacy.

## Ventana de observación posterior al cutover

La identidad real `andrea.madriz@eticayservicios.com` reclamó la invitación de
un solo uso y creó el pedido `ORD-3695A0984CD7`. RDS confirmó estado
`CONFIRMED`, total USD 5,20, una línea y reserva de Inventario
`904aeef8-2b97-42a4-bd52-0f6e870e22d6` en estado `ACTIVE`.

Durante los 15 minutos posteriores se tomaron muestras de API Gateway y
CloudWatch. Después de la reclamación hubo cero 4xx nuevos, cero 5xx, cero
errores y cero throttling en Ecommerce; Inventario y Pedidos también reportaron
cero errores. No se observaron pedidos parciales ni reservas huérfanas.

La revisión y recuperación visual de TuMandadito fue diferida por decisión de
producto hasta después de finalizar la migración técnica.

## Retiro de la creación legacy

CloudWatch confirmó cero invocaciones posteriores al cutover en
`tumandaito-client-dev-checkout` y `tumandaito-client-dev-orders`. La inspección
del artefacto desplegado verificó que `/checkout/quote` y `/checkout/validate`
son lecturas; el único creador legacy dentro del corte vertical era
`POST /orders`.

Su integración se conmutó de forma reversible de Lambda proxy a MOCK y se
republicó el stage `dev`. El smoke devolvió HTTP 410 con
`LEGACY_CHECKOUT_RETIRED`; el catálogo MasterERP permaneció HTTP 200 con 17
productos. No se eliminaron rutas, Lambdas, datos ni permisos, y las operaciones
históricas permanecen intactas. La plantilla SAM conserva el mismo guard para
evitar que un despliegue posterior reactive la creación legacy.

## Clasificación y retiro de doble escritura

Las 36 funciones con referencia a `tumandaito-dev-core` quedaron clasificadas:

| Clasificación | Funciones |
| --- | --- |
| Escritura del corte retirada | `inventory-stock`, `inventory-adjustments`, `procurement-receiving`, `operations-orders`, `operations-preparation`, `order-projector`, `logistics-routes`, `logistics-drivers`, `nodes-assignments`, `client-driver`, `client-orders` |
| Lectura efectiva | `client-store-config`, `inventory-movements`, `client-checkout`, `client-catalog`, `client-tracking`, `nodes-monitoring`, `reporting-dashboard`, `reporting-operations`, `reporting-finance` |
| Fuera del corte | `client-customers`, `nodes-management`, `catalog-categories`, `finance-receivables`, `finance-cash`, `catalog-products`, `nodes-coverage`, `nodes-merchants`, `procurement-purchases`, `finance-payables`, `finance-payments`, `client-auth`, `catalog-taxes`, `procurement-suppliers`, `finance-invoicing`, `profile-management` |

Los ocho handlers administrativos de la primera fila quedaron en concurrencia
0. `client-orders` conserva GET, pero crear, cancelar y registrar referencia de
pago responden 410. `client-driver` conserva GET de ruta y pedidos, pero inicio
de ruta, incidentes y estado de entrega responden 410. El event source mapping
`978e23ef-12ea-47f0-bce7-bab5a1f7bd88` del proyector quedó `Disabled`.

El delta definitivo se ejecutó durante una pausa reversible de las 36 Lambdas.
Seleccionó 296/296 registros y produjo nuevamente
`6a6d9c5030bd7c63070a91876b9114b0d8d36553cddf8c15bc103589084a212d`.
Al terminar, las 26 funciones no retiradas recuperaron su concurrencia previa;
solo los ocho handlers retirados permanecieron en 0.

Los cinco guards Client devolvieron 410 mediante `test-invoke-method`. Los
smokes públicos finales devolvieron 200 para Storefront, Uonify, `/#/chat` y el
catálogo canónico con 17 productos; `POST /orders` legacy devolvió 410. Esto
formaliza MasterERP como fuente de verdad para nuevos checkouts, reservas y
pedidos sin eliminar recursos legacy.

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
y de Transporte; la prueba separada de checkout Storefront quedó aprobada en la
promoción aditiva descrita arriba.
