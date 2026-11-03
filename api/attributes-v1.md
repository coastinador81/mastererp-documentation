# Contrato de Atributos v1

Estado: candidato funcional no desplegado. Propietario: `2.1.3-attributes`.

Define atributos reutilizables con código estable, tipo `TEXT`, `NUMBER`,
`BOOLEAN`, `SELECT` o `MULTISELECT`, obligatoriedad, unidad, opciones, orden y
categorías aplicables. El tipo y código son inmutables para no reinterpretar
valores históricos silenciosamente.

```text
GET   /v1/catalog/attributes
GET   /v1/catalog/attributes/{attribute_code}
POST  /v1/catalog/attributes
PATCH /v1/catalog/attributes/{attribute_code}
```

`POST` exige `Idempotency-Key`; `PATCH` exige `version`. Permisos:
`catalog.attributes.read/write` con `erp.catalog` activo. Los valores actuales
de productos permanecen temporalmente en `productos.atributos`; su validación y
backfill se activarán antes del cutover.
