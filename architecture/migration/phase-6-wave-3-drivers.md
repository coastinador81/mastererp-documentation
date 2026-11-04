# Fase 6 — Oleada 3: Repartidores

Estado: loader ejecutado y reconciliado en DEV. Evidencia consolidada en
[phase-6-dev-evidence.md](phase-6-dev-evidence.md).

Repartidores permanece bajo Transporte. El transformador reúne perfiles DRIVER
y asociaciones `NODE_DRIVER`, traduce cada Nodo mediante el mapa de sucursales y
genera un perfil con vínculos explícitos. El loader usa exclusivamente
`repartidor_crear`, `repartidor_actualizar`, `repartidor_asignar_nodo` y lecturas
autorizadas bajo `mastererp_app`.

Los lotes son configurables y reiniciables. La creación usa idempotencia; antes
de actualizar o asignar se vuelve a leer el estado real para evitar respuestas
cacheadas antiguas y escrituras innecesarias.

## Guarda de coherencia

Un repartidor `ON_ROUTE` o con `routeId` no puede entrar en esta oleada. Debe
migrarse junto con Rutas y Pedidos para conservar la ruta activa, sus paradas y
las máquinas de estado. Los Nodos no mapeados también bloquean el bundle.

## Evidencia local

- Catálogo y Nodos cargados como prerrequisitos.
- Perfil y asociación a Nodo aplicados; loader completo repetido.
- Resultado estable: 1 repartidor y 1 asociación; sin duplicados en reejecución.
- Conteo conjunto estable `1|1|1|1|1|6`: Nodo, Inventario, movimiento,
  Repartidor, vínculo y eventos de auditoría.
- Sin DML directo, rol efectivo `mastererp_app` y cero escrituras remotas.

DEV requiere export real, revisión de excepciones activas, snapshot, ventana
autorizada, carga y reconciliación antes de considerar cualquier corte.
