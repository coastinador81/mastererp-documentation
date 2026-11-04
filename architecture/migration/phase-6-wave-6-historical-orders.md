# Fase 6 — oleada 6: Pedidos preparados e históricos

Estado: `READY`, `OUT_FOR_DELIVERY`, `DELIVERED` y `CANCELLED` ejecutados y
reconciliados en sombra DEV; no promovidos como fuente de verdad. Ver
[evidencia consolidada](phase-6-dev-evidence.md).

## Estrategia

El stock fuente de un pedido preparado ya refleja su consumo. El loader calcula
una base temporal sumando exclusivamente las cantidades consumidas, reconstruye
reserva, Pedido y Preparación mediante funciones canónicas, y vuelve a consumir
esas cantidades. Los cancelados reservan y liberan sin cambiar la existencia.

La preparación de la base exige inventario nuevo (`version=0`) y usa una clave
idempotente estable. Una reejecución obtiene la respuesta almacenada y no vuelve
a incrementar stock. Un destino parcialmente poblado con otra procedencia falla
por versión en lugar de mezclar saldos.

## Evidencia local

PostgreSQL 17.9 aislado recibió dos veces cuatro pedidos: preparado, en ruta,
entregado y cancelado. Resultado exacto: cuatro Pedidos, cuatro líneas, cuatro
reservas, nueve movimientos y estados finales esperados. El inventario regresó
a `stock=10`, `reserved=0`. Rol efectivo `mastererp_app`, sin DML en el loader
ni escrituras remotas.

## Resultado

Junto con la oleada de Pedidos activos, todos los estados fuente admitidos ya
tienen una ruta de migración local repetible. El cutover continúa bloqueado hasta
usar una exportación real, comparar snapshots DEV y probar delta/rollback.
