# Contrato de Inventario v1

Estado: candidato funcional no desplegado. Propietario: `2.1.6-inventory`.

Inventario es dueño de existencias, reservas, disponibilidad y movimientos por
tenant, empresa, nodo y producto. No modifica la ficha de Producto ni la
configuración de Nodo. `available = stock - reserved` es calculado por servidor.

## API inicial

```text
GET  /v1/companies/{company_id}/nodes/{node_id}/inventory
GET  /v1/companies/{company_id}/nodes/{node_id}/inventory/{product_id}
PUT  /v1/companies/{company_id}/nodes/{node_id}/inventory/{product_id}/stock
POST /v1/companies/{company_id}/nodes/{node_id}/inventory/reservations
POST /v1/companies/{company_id}/inventory/reservations/{reservation_id}/finalize
```

Los ajustes y reservas exigen idempotencia. Los cambios usan versión optimista.
Una reserva bloquea filas en orden estable, valida disponibilidad y aumenta
`reserved` atómicamente. `RELEASE` libera reservado; `COMMIT` reduce reservado y
stock. Repetir la finalización devuelve el estado final sin doble consumo.

Los movimientos `SET`, `ADJUST`, `RESERVE`, `RELEASE` y `COMMIT` son inmutables
y conservan actor, motivo, correlación y referencia. La aplicación no posee DML
directo. Permisos: `catalog.inventory.read`, `.write` y `.reserve` con módulo
`erp.catalog` activo y nodo perteneciente a la empresa.

Tu Mandadito continúa como fuente de verdad. Backfill y reconciliación ocurren
en Fase 6; no se despliega ni se cambia tráfico en este corte.
