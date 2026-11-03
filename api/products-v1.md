# Contrato de Productos v1

Estado: perfil administrativo implementado como candidato no desplegado.

Propietario: `2.1-catalog/2.1.1-products`.

## Límite

Productos es dueño de la identidad, código, nombre, descripción, precio de
catálogo, estado comercial, referencia de categoría, imagen y atributos de la
ficha maestra. No es dueño de stock, reservas, movimientos, disponibilidad por
nodo, categorías ni precios históricos de pedidos.

Inventario enlaza por `product_id`; Pedidos conserva snapshots de nombre y
precio. Storefront consumirá este catálogo por API y lo combinará con
disponibilidad autorizada de Inventario, sin mantener una copia paralela.

## Estados

- `ACTIVE`: publicable cuando las demás capacidades lo permitan.
- `INACTIVE`: suspendido administrativamente.
- `DRAFT`: ficha incompleta/no publicada.
- `DISCONTINUED`: retirado; no se elimina para preservar referencias.

Un estado comercial activo no implica existencia ni disponibilidad.

## API administrativa inicial

```text
GET   /v1/products
GET   /v1/products/{product_id}
POST  /v1/products
PATCH /v1/products/{product_id}
```

Filtros iniciales: `status`, `search` y `limit` entre 1–100.

La creación exige `Idempotency-Key`. La edición exige `version`; un escritor
obsoleto recibe conflicto. Identidad, tenant y actor provienen del contexto
autenticado. El navegador nunca envía stock ni identificadores internos al
crear.

## Campos del perfil inicial

| Campo | Regla |
| --- | --- |
| `id` | UUID generado por servidor. |
| `code` | Código `PRD-*` generado por servidor y único por tenant. |
| `name` | Obligatorio, 1–200 caracteres. |
| `description` | Hasta 2.000 caracteres. |
| `price` | Positivo, hasta 6 decimales. No sustituye snapshots históricos. |
| `status` | Uno de los cuatro estados canónicos. |
| `category_ref` | Referencia temporal; Categorías será su propio vertical. |
| `image_url` | HTTPS o ruta `/images/`; no contiene credenciales. |
| `attributes` | Objeto JSON de metadatos comerciales, nunca stock. |
| `external_reference` | ID histórico opcional para backfill/reconciliación. |
| `version` | Concurrencia optimista. |

## Seguridad

- API Gateway verifica JWT.
- Backend resuelve `issuer + sub` a usuario interno.
- PostgreSQL exige tenant activo, módulo `erp.catalog` habilitado y permisos
  `catalog.products.read` o `catalog.products.write`.
- `mastererp_app` no posee DML directo sobre `catalogo.productos`.
- El frontend obtiene el token del proveedor de sesión del app-shell en tiempo
  de ejecución; ningún token se materializa en el bundle.

## Convivencia

Tu Mandadito/DynamoDB continúa como fuente de verdad. El futuro backfill debe
mapear `productId` a `external_reference`, validar duplicados por tenant y
reconciliar conteos, estados y precios antes del cutover. No se crea inventario
implícito durante ese backfill.

## Evidencia

- Modelo aplicado dos veces en PostgreSQL 17.9 desechable.
- Alta/replay idempotente, edición, versión obsoleta, filtros y ausencia de DML
  directo probados en SQL.
- Seis pruebas unitarias del adapter HTTP.
- ZIP Lambda determinista e interfaz Vite compilados localmente.
- Recursos CloudFormation validados como JSON y por estructura local.
