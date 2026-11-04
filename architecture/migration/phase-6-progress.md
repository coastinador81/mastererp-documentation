# Fase 6 — datos y convivencia

Estado: cierre de datos en sombra DEV completo. Carga y reconciliación semántica
correctas; Tu Mandadito continúa como fuente de verdad durante la convivencia.

## Estrategia de convivencia elegida

Se descarta escritura dual DynamoDB/PostgreSQL: los modelos, regiones y
transacciones son distintos y una falla parcial podría duplicar reservas,
pedidos o entregas. Se usará por capacidad:

1. Tu Mandadito continúa como única fuente de verdad y escritura.
2. Exportación inmutable y fechada desde DynamoDB.
3. Transformación offline determinista, sin conexión a AWS/RDS.
4. Carga idempotente en MasterERP durante una ventana autorizada.
5. Shadow read/reconciliación sin responder tráfico desde MasterERP.
6. Pausa corta de escrituras de la capacidad, delta final y comparación.
7. Cutover explícito con rollback hacia Tu Mandadito durante la ventana acordada.

El corte es vertical: una capacidad puede cambiar de fuente sin obligar a cortar
todas las demás. Storefront y canales siempre consumen la API del propietario.

## Orden de oleadas

| Orden | Datos | Dependencia previa |
| ---: | --- | --- |
| 1 | Categorías, Productos | Tenant y actor mapeados |
| 2 | Sucursales/Nodos | Empresa y mapa Nodo → sucursal aprobados |
| 3 | Inventario | Productos y Nodos reconciliados; reservas activas identificadas |
| 4 | Repartidores | Nodos reconciliados; canal web/Expo puede seguir en origen |
| 5 | Clientes/Terceros | Empresa y permisos de Partes disponibles |
| 6 | Pedidos y Preparación | Clientes, productos, nodos e inventario disponibles |
| 7 | Rutas y paradas | Pedidos y repartidores reconciliados |
| 8 | Tiendas/Storefront | Catálogo e inventario ya canónicos |

## Herramientas implementadas

Database contiene `migration/phase6/transform_export.py`. Acepta JSONL plano o
DynamoDB JSON, exige un archivo de mapeo y genera IDs UUIDv5 estables, archivos
canónicos ordenados y manifiesto SHA-256. No consulta AWS ni escribe RDS.

`reconcile_dev_snapshot.py` compara el bundle firmado con una inspección DEV de
solo lectura y falla ante registros faltantes, inesperados o diferentes. La
ejecución real finalizó sin diferencias. Evidencia en
[phase-6-dev-evidence.md](phase-6-dev-evidence.md).

La primera oleada ya dispone de loader candidato por lotes para Categorías y
Productos. La evidencia y las puertas de promoción están en
[phase-6-wave-1-catalog.md](phase-6-wave-1-catalog.md). Esto completa la
implementación local de esa oleada, no la migración de datos real.

La segunda oleada dispone de loader candidato para Nodos e Inventario, con
bloqueo explícito de reservas activas. Evidencia en
[phase-6-wave-2-nodes-inventory.md](phase-6-wave-2-nodes-inventory.md).

La tercera oleada dispone de loader candidato para Repartidores no asignados a
rutas activas. Evidencia en [phase-6-wave-3-drivers.md](phase-6-wave-3-drivers.md).

La cuarta oleada dispone de loader candidato para Customer → Tercero/Cliente,
sin crear identidades paralelas. Evidencia en
[phase-6-wave-4-customers.md](phase-6-wave-4-customers.md).

La quinta oleada dispone de preflight y loader candidato para Pedidos activos y
reservas reconciliadas. Evidencia en
[phase-6-wave-5-active-orders.md](phase-6-wave-5-active-orders.md). Los pedidos
ya preparados, históricos o en ruta permanecen diferidos para evitar un segundo
consumo de stock.

La sexta oleada reconstruye Pedidos preparados, en ruta, entregados y cancelados
sin alterar el saldo final de Inventario. Evidencia en
[phase-6-wave-6-historical-orders.md](phase-6-wave-6-historical-orders.md).

La séptima oleada migra Rutas y coordina Pedidos/Repartidores sin fusionar sus
límites. Evidencia en [phase-6-wave-7-routes.md](phase-6-wave-7-routes.md).

La octava oleada migra configuración Storefront sin copiar Catálogo/Inventario.
Evidencia en [phase-6-wave-8-storefront.md](phase-6-wave-8-storefront.md).

El cierre de ingeniería y las puertas operativas están en
[phase-6-closure.md](phase-6-closure.md).

## Mapeos obligatorios

- `T001` no se convierte en UUID por inferencia: debe asociarse al tenant real.
- Cada Nodo fuente debe mapearse a una sucursal UUID de la empresa; Nodos no crea
  una identidad organizacional paralela.
- Productos, pedidos, rutas y repartidores conservan el ID anterior como
  referencia externa y reciben una identidad estable de migración.
- `PREPARED` legado se normaliza a `READY`; cualquier otro cambio de estado debe
  quedar en el reporte de excepciones.
- Inventario requiere nodo explícito. No se inventa un nodo para filas legacy.

## Resultado DEV

La carga incluyó Clientes, todos los estados de Pedidos, reservas, Preparación y
Rutas. Los conteos, claves, estados, líneas, totales, existencias y reservas
coincidieron con el bundle fuente. La sombra DEV no responde tráfico todavía.

## Criterio para marcar los puntos restantes

- **Migración repetible:** loader transaccional aprobado, dry-run y reejecución
  sobre una base desechable sin duplicados.
- **Comparación:** completada con export real y snapshot DEV, sin diferencias.
- **Fuente durante convivencia:** Tu Mandadito permanece como fuente de verdad.
- **Cutover posterior:** requiere infraestructura integrada, smoke/E2E, delta
  final, rollback probado y aprobación explícita del Product Owner.
