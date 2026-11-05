# Evidencia DEV — Fase 8

Fecha: 2026-11-05. Cuenta `277352717693`, región `us-west-1`.

## Smoke autenticado

Se ejecutó `mastererp-infrastructure/scripts/verify_phase8_tumandaito.py`
contra las APIs Admin y Client reales. El runner obtuvo un access token Cognito
sin persistir credenciales, creó una membresía temporal para el actor QA
sintético, realizó únicamente lecturas y retiró esa membresía en `finally`.

Resultado: 14 controles aprobados.

| Capacidad | Resultado HTTP | Conteo comparado |
| --- | ---: | ---: |
| Categorías | 200 | 11 |
| Productos | 200 | 27 |
| Nodos | 200 | 5 |
| Inventario | 200 | 42 |
| Repartidores | 200 | 6 |
| Pedidos | 200 | 9 |
| Preparación | 200 | lista válida |
| Rutas | 200 | 3 |
| Tiendas | 200 | 1 |
| Storefront público | 200 | catálogo válido |

Los conteos se compararon en la misma ejecución con el snapshot de solo lectura
de Fase 6, identificado por SHA-256
`6a6d9c5030bd7c63070a91876b9114b0d8d36553cddf8c15bc103589084a212d`.
Un tenant ajeno recibió 403. Antes y después de la membresía temporal,
Tumandaito también devolvió 403 para el actor QA; esto verifica la limpieza.

## Incidente controlado del runner

La primera ejecución fue interrumpida después de crear la membresía temporal.
El helper se endureció para recuperar únicamente la única relación entre el
usuario QA sintético, identificado por email e issuer, y el tenant exacto
`tumandaito`. La relación huérfana se retiró, se verificó 403 y luego el smoke
completo aprobó. No se modificaron datos de negocio ni membresías reales.

## Límite de la evidencia

Esto confirma contratos de lectura, autenticación, autorización, aislamiento y
paridad de conteos en sombra. No confirma delta final, escrituras end-to-end,
conmutación de frontends, rollback operativo ni retiro de recursos legacy.
Esos puntos siguen siendo puertas obligatorias antes del cutover.

## Delta y restauración del origen

El 2026-11-05 se repitió la exportación consistente de `T001` desde la cuenta
origen `941751053509`, región `us-west-2`. Se seleccionaron 296 de 296 registros
y el SHA-256 volvió a ser exactamente
`6a6d9c5030bd7c63070a91876b9114b0d8d36553cddf8c15bc103589084a212d`.
Por tanto, no existía delta desde la carga reconciliada de Fase 6 al momento de
la comprobación.

Se creó el backup
`tumandaito-dev-core-before-mastererp-phase8-20261105`, estado `AVAILABLE`, ARN
terminado en `backup/01793889299911-969f4647`. Se restauró en la tabla aislada
`tumandaito-dev-core-phase8-rollback-drill-20261105`; el verificador leyó 296
registros y confirmó el mismo SHA-256. La tabla temporal fue eliminada después
del ensayo; el backup permanece recuperable.

Esta prueba acredita recuperación de los datos DynamoDB. Aún falta ensayar la
conmutación de tráfico de los canales y ejecutar escrituras E2E antes de marcar
completa la puerta conjunta de pruebas/datos/rollback.
