# Fase 8 — cierre de capacidad

Estado: en curso. Cutover y ventana de observación aprobados en DEV; aún falta
retirar la doble escritura legacy antes de declarar MasterERP fuente de verdad.

## Decisiones de producto confirmadas

- `app-dev.uonify.com` es la interfaz matriz integrada. Inicio, módulos y chat
  operativo coexisten; el chat se conserva en `/#/chat` y no se elimina.
- Los clientes Storefront migran al Cognito de MasterERP y deberán iniciar
  sesión nuevamente. La identidad se vincula solo por `issuer + sub`; queda
  prohibido asociar automáticamente cuentas por coincidencia de correo.

## Completado

- APIs canónicas desplegadas y smoke autenticado sobre el tenant Tumandaito.
- Conteos HTTP reconciliados con el snapshot firmado de Fase 6.
- Aislamiento cruzado probado con 403.
- Acceso QA temporal retirado y verificado.
- Storefront público lee el catálogo canónico en DEV.
- Delta del origen comprobado en cero mediante export consistente y SHA-256.
- Backup previo a Fase 8 restaurado y comparado en una tabla aislada: 296
  registros idénticos; tabla temporal eliminada y backup conservado.
- Uonify matriz publicado con Catálogo, Inventario, Pedidos, Nodos,
  Preparación, Rutas, Repartidores y Tiendas inyectados sin iframe. El E2E real
  aprobó Cognito, contexto, todas las lecturas canónicas y cero errores de
  navegador.
- Escrituras UI → API → RDS aprobadas en DEV para Tiendas, Repartidores y Nodos
  mediante guardados no-op auditados, sin cambiar valores funcionales.
- Cliente Cognito Storefront exclusivo desplegado con Authorization Code + PKCE,
  reautenticación obligatoria y tokens de 15 minutos.
- Identidad de cliente explícita desplegada en RDS y API: invitación de un solo
  uso almacenada solo como SHA-256, reclamación por `issuer + sub`, correo
  verificado usado únicamente como perfil y sin membresía administrativa.
- Authorizer y rutas Storefront desplegados sin reemplazar Lambda, APIs,
  integraciones ni datos. Los smokes reales aprobaron catálogo público 200,
  rechazo sin JWT 401, autorización administrativa hasta RDS y ID token
  Storefront verificado hasta RDS; los casos sintéticos terminaron en 404 sin
  crear filas de negocio y el acceso QA fue retirado.

Evidencia: [phase-8-dev-evidence.md](phase-8-dev-evidence.md).

## Dependencias que impiden declarar cutover

El checkout Storefront → Inventario → Pedidos fue promovido aditivamente y
aprobó E2E en DEV con Cognito Storefront e invitación explícita. Resuelve
tienda, cliente y precios en servidor, y crea reserva y pedido idempotentes en
una transacción. Tras autorización explícita, el índice candidato fue
promovido a la raíz DEV; sus assets permanecen bajo
`/mastererp-phase8-candidate/` para permitir rollback inmediato.

El inventario del 2026-11-05 encontró consumidores activos que aún usan los
contratos, Cognito y endpoints legacy de Tu Mandadito en `us-west-2`:

- Admin conserva clientes para `/catalog`, `/inventory`, `/operations`, nodos,
  rutas, preparación y repartidores del API anterior.
- Customer/Storefront conserva `EXPO_PUBLIC_CLIENT_API_URL`, autenticación del
  pool `tumandaito-dev-users` y checkout/pedidos legacy.
- Repartidores conserva su API y repositorios DynamoDB propios.
- Backend e infraestructura legacy siguen escribiendo
  `tumandaito-dev-core`; retirarlos ahora interrumpiría operación.

Por tanto, MasterERP aún no es la fuente de verdad operativa. Marcarlo o borrar
recursos en este estado sería incorrecto.

## Orden restante para el corte vertical

1. Checkout canónico promovido, probado y con tráfico DEV conmutado; el índice
   previo permanece respaldado en `mastererp-phase8-rollback/`.
2. Admin, Storefront y Repartidores ya aprobaron escrituras
   no-op auditadas y las lecturas de todos los módulos pasaron en navegador.
3. Delta final aprobado dentro de una pausa controlada: 296 de 296 registros y
   SHA-256 idéntico al snapshot firmado; 36 concurrencias restauradas.
4. Rollback de tráfico ensayado sobre el prefijo candidato: legacy y candidato
   se sirvieron con sus hashes exactos, sin tocar la raíz ni DNS.
5. Autorización explícita recibida y cutover aplicado sin cambios DNS; índice
   y assets públicos verificados con hash exacto y HTTP 200.
6. Ventana de observación aprobada durante 15 minutos después del checkout real:
   cero 4xx nuevos, 5xx, errores o throttling; Pedido y Reserva consistentes.
7. Retirar la doble escritura y, solo con cero consumidores, retirar Lambdas,
   APIs, tablas y código legacy.

El creador legacy `POST /orders` fue conmutado de forma reversible a HTTP 410
después de confirmar cero invocaciones desde el cutover. Las lecturas, la
cancelación y el seguimiento histórico permanecen disponibles. El Storefront
canónico continuó sirviendo catálogo HTTP 200. Falta repetir el delta firmado
bajo pausa final; el primer intento no inició porque el control de aprobación
AWS alcanzó su límite, por lo que ninguna concurrencia fue modificada.

La recuperación visual de TuMandadito queda explícitamente fuera de esta fase y
se abordará después de terminar la migración técnica.

La regla de archivos menores de 500 líneas se mantiene para todos los cambios.
