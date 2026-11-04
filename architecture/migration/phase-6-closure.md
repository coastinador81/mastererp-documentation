# Cierre de ingeniería — Fase 6 Datos y convivencia

Estado: implementación local completa. Promoción DEV, reconciliación real y
cutover pendientes de autorización operativa.

## Capacidades cubiertas

- Categorías y Productos.
- Nodos e Inventario.
- Clientes como Terceros con rol `cliente`.
- Repartidores como módulo separado.
- Pedidos activos, Preparación, reservas e históricos.
- Rutas, paradas y disponibilidad del Repartidor.
- Configuración Storefront sin duplicar capacidades canónicas.

Todos los loaders validan SHA-256, tenant, referencias, estados y dependencias;
generan SQL para revisión, usan funciones autorizadas bajo `mastererp_app`, son
reiniciables y se probaron dos veces en PostgreSQL desechable. No conectan por sí
mismos a AWS/RDS y no cambian la fuente de verdad.

## Puertas operativas abiertas

1. Aprobar y obtener una exportación inmutable real de DynamoDB.
2. Ejecutar transform/preflight y revisar excepciones de datos.
3. Cargar una sombra DEV durante una ventana autorizada.
4. Exportar snapshot MasterERP y reconciliar conteos, estados, totales y hashes.
5. Probar delta final, smoke/E2E y rollback.
6. Declarar cada vertical como nueva fuente de verdad mediante aprobación
   explícita. Hasta entonces Tu Mandadito conserva esa función.
