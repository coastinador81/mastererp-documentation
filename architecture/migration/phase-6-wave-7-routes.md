# Fase 6 — oleada 7: Rutas

Estado: ejecutado y reconciliado en sombra DEV. Ver
[evidencia consolidada](phase-6-dev-evidence.md).

Los Pedidos con `routeId` se reconstruyen hasta `READY`. El loader de Rutas
resuelve Pedidos y Repartidores por referencia externa, crea las paradas, asigna
el Repartidor y reproduce `IN_ROUTE`, entregas y `COMPLETED`. La responsabilidad
de disponibilidad permanece en Transporte y se inyecta mediante funciones
autorizadas; no se fusiona con Rutas.

El preflight bloquea rutas vacías, Nodo/Driver incompatibles, propiedad cruzada
y pedidos cancelados dentro de una ruta hasta que exista un contrato explícito
de entrega fallida.

Evidencia: una Ruta terminada fue aplicada dos veces en PostgreSQL 17.9
desechable. Resultado único: Ruta `COMPLETED`, parada y Pedido `DELIVERED`,
Repartidor `AVAILABLE`, inventario final `10/0`, rol `mastererp_app`, sin DML en
el loader ni escrituras remotas.
