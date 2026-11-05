# Fase 8 — cierre de capacidad

Estado: en curso. Contratos de lectura, autorización y aislamiento confirmados
en DEV; cutover todavía no autorizado ni ejecutado.

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

1. Implementar checkout conectado con Inventario y Pedidos canónicos; la
   transición Cognito y el aprovisionamiento explícito de identidad ya están
   desplegados y probados.
2. Probar checkout Storefront; Admin y Repartidores ya aprobaron escrituras
   no-op auditadas y las lecturas de todos los módulos pasaron en navegador.
3. Repetir el export de delta dentro de la pausa final de escrituras; el ensayo
   previo ya produjo diferencia cero.
4. Ensayar rollback de tráfico; la restauración de datos ya fue aprobada.
5. Obtener autorización explícita del Product Owner y conmutar tráfico.
6. Observar la ventana acordada; solo después retirar Lambdas, APIs, tablas y
   código legacy sin consumidores.

La regla de archivos menores de 500 líneas se mantiene para todos los cambios.
