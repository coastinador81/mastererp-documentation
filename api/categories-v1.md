# Contrato de Categorías v1

Estado: candidato funcional no desplegado. Propietario: `2.1.2-categories`.

Categorías mantiene un árbol multi-tenant con slug estable, nombre, descripción,
imagen, orden y estado `ACTIVE|INACTIVE`. El slug no se renombra. La jerarquía
rechaza padres inexistentes, autorreferencias y ciclos.

```text
GET   /v1/catalog/categories
GET   /v1/catalog/categories/{category_slug}
POST  /v1/catalog/categories
PATCH /v1/catalog/categories/{category_slug}
```

`POST` exige `Idempotency-Key`; `PATCH` exige `version`. Permisos:
`catalog.categories.read/write` y módulo `erp.catalog` activo.

Durante la migración `productos.category_ref` sigue siendo texto para aceptar
datos heredados. La FK estricta se habilitará después del backfill y la
reconciliación, evitando romper el e-commerce actual antes del cutover.
