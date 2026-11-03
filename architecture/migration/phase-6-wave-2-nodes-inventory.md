# Fase 6 — Oleada 2: Nodos e Inventario

Estado: loader candidato validado localmente; exportación/carga DEV pendientes.

## Diseño seguro

- Cada Nodo fuente debe estar en `node_map` y apuntar a una sucursal UUID ya
  existente dentro de la empresa destino.
- El loader consulta el estado actual mediante la API SQL, configura solo cuando
  existen diferencias y recorre únicamente transiciones válidas.
- Inventario resuelve el UUID real del Producto por su referencia externa usando
  una función `SECURITY DEFINER` con permiso `catalog.products.read`; no recibe
  SELECT ni DML directo sobre tablas.
- Existencias se escriben únicamente mediante `inventario_establecer`, con
  versión optimista, movimiento, actor, correlación e idempotencia.
- Los lotes son configurables de 1–1000 registros, 200 por defecto, y pueden
  retomarse después de una falla.

## Guardas

La prevalidación bloquea nodos sin sucursal, stock negativo, inventario con
ventas deshabilitadas y cualquier reserva activa (`reserved != 0`). Las reservas
se migrarán junto con Pedidos; convertirlas en stock libre sería una corrupción
operativa.

## Evidencia local

- Catálogo cargado como prerrequisito.
- Nodos e Inventario aplicados dos veces en PostgreSQL 17.9 desechable.
- Resultado estable: 1 Nodo, 1 Inventario, 1 movimiento y 4 eventos totales de
  Catálogo/Nodo; la segunda ejecución no añadió registros.
- Rol efectivo `mastererp_app`, sin DML directo y sin escrituras remotas.
- Migración 027 aplicada y reaplicada con las 15 pruebas SQL del modelo.
- Rechazo unitario de bundles con reservas activas.

Para DEV todavía se requiere export real, sucursales aprobadas, revisión del
bundle/SQL, snapshot, ejecución autorizada y reconciliación.
