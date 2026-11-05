# Fase 8 — cierre de capacidad

Estado: en curso. Contratos de lectura, autorización y aislamiento confirmados
en DEV; cutover todavía no autorizado ni ejecutado.

## Completado

- APIs canónicas desplegadas y smoke autenticado sobre el tenant Tumandaito.
- Conteos HTTP reconciliados con el snapshot firmado de Fase 6.
- Aislamiento cruzado probado con 403.
- Acceso QA temporal retirado y verificado.
- Storefront público lee el catálogo canónico en DEV.
- Delta del origen comprobado en cero mediante export consistente y SHA-256.
- Backup previo a Fase 8 restaurado y comparado en una tabla aislada: 296
  registros idénticos; tabla temporal eliminada y backup conservado.
- Uonify matriz publicado con Productos y Pedidos inyectados desde sus mismos
  componentes; E2E real aprobó Cognito, contexto, 27 Productos y 9 Pedidos.

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

1. Completar la inyección en Uonify de Categorías, Inventario, Nodos,
   Preparación, Rutas y Repartidores; Productos y Pedidos ya están integrados.
   Resolver además la transición de identidad Storefront sin vincular por email.
2. Probar lecturas y escrituras E2E de Admin, Storefront y Repartidores.
3. Repetir el export de delta dentro de la pausa final de escrituras; el ensayo
   previo ya produjo diferencia cero.
4. Ensayar rollback de tráfico; la restauración de datos ya fue aprobada.
5. Obtener autorización explícita del Product Owner y conmutar tráfico.
6. Observar la ventana acordada; solo después retirar Lambdas, APIs, tablas y
   código legacy sin consumidores.

La regla de archivos menores de 500 líneas se mantiene para todos los cambios.
