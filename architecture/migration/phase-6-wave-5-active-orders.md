# Fase 6 — oleada 5: Pedidos activos y reservas

Estado: `PENDING`, `CONFIRMED` y `PREPARING` ejecutados y reconciliados en
sombra DEV; no promovidos como fuente de verdad. Ver
[evidencia consolidada](phase-6-dev-evidence.md).

## Regla de inventario

Tu Mandadito mantiene `reserved` mientras el pedido está activo. Al quedar
`READY`, el stock ya fue consumido. El preflight suma las líneas de todos los
pedidos activos por Nodo/Producto y exige igualdad exacta con `reserved`. Una
diferencia bloquea toda generación de SQL.

## Implementación

- Normaliza `RECEIVED` → `PENDING` e `IN_ROUTE` → `OUT_FOR_DELIVERY`.
- Carga el stock fuente inicialmente sin reservas.
- Crea la reserva mediante `catalogo.inventario_reservar`.
- Crea el Pedido con la referencia real de esa reserva.
- Reproduce `CONFIRMED` y `PREPARING` mediante transiciones autorizadas.
- Conserva el avance `DONE/PENDING` de líneas en Preparación.
- Usa hashes, correlación, lotes e idempotencia estable; no contiene DML directo.

## Evidencia local

PostgreSQL 17.9 aislado recibió dos veces el mismo bundle. Resultado exacto:
un Pedido, una línea, una reserva, dos unidades reservadas y dos movimientos de
Inventario (`SET` y `RESERVE`). Rol efectivo `mastererp_app`; cero escrituras
remotas.

## Diferido deliberadamente

`READY`, `OUT_FOR_DELIVERY`, `DELIVERED` y `CANCELLED` pertenecen a la siguiente
sub-oleada. Reproducirlos como activos descontaría stock una segunda vez. Su
loader debe reconstruir la base previa, ejecutar consumo/liberación y terminar
con el saldo fuente exacto antes de habilitar shadow reads.
