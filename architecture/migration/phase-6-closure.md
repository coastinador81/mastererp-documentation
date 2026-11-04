# Cierre de ingeniería — Fase 6 Datos y convivencia

Estado: cerrada para datos y convivencia en sombra DEV. El cutover no forma
parte de este cierre y permanece reservado para el cierre de capacidad.

Actualización 2026-11-04: se verificaron las cuentas separadas de origen y
destino, se crearon respaldos, se cargó la sombra DEV mediante el migrador
oficial y se reconcilió mediante inspección de solo lectura. Los nueve dominios
comparados coincidieron sin faltantes ni inesperados. Ver
[evidencia DEV](phase-6-dev-evidence.md) y la
[guía de verificación reproducible](phase-6-verification.md).

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

## Controles completados

1. Exportación inmutable real de DynamoDB y respaldos antes/después.
2. Transformación, preflight y excepciones auditadas.
3. Carga idempotente de la sombra DEV.
4. Snapshot lógico de solo lectura y reconciliación semántica exacta.

## Siguiente fase

Infraestructura y DEV deben habilitar smoke/E2E sin cambiar tráfico. Después se
probarán delta final y rollback. La declaración de MasterERP como fuente de
verdad permanece en Fase 8 y requiere aprobación explícita.
