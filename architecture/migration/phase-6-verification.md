# Cómo verificar la Fase 6

Esta guía permite repetir los controles sin cambiar datos. La Fase 6 acredita
la carga y reconciliación de la sombra DEV; no acredita todavía APIs, pantallas,
tráfico real ni cutover. Esas pruebas requieren la integración de Fase 7 y las
puertas de Fase 8.

## 1. Validación local rápida

Desde `mastererp-database`:

```bash
python3 -m venv /tmp/mastererp-phase6-venv
/tmp/mastererp-phase6-venv/bin/pip install \
  -r migrations/lambda/requirements-package.txt
python3 scripts/validate_structure.py
PYTHONPATH=migrations/lambda:scripts \
  /tmp/mastererp-phase6-venv/bin/python -m unittest \
  scripts.test_phase6_validation \
  scripts.test_reconcile_dev_snapshot \
  scripts.test_package_migrator \
  scripts.test_migrator_failures
git diff --check
```

Estos controles comprueban estructura, contrato exacto de inspección, uso de
transacción de solo lectura con `ROLLBACK`, comparación determinista,
empaquetado del inspector y rechazo de artefactos no autorizados. No necesitan
credenciales AWS ni escriben en RDS.

El entorno se crea en `/tmp`, fuera del repositorio, y puede eliminarse al
terminar. Si las dependencias ya están instaladas en otro entorno virtual, se
puede usar su intérprete conservando el mismo `PYTHONPATH`.

## 2. Integración local desechable

Con Docker disponible, ejecutar los validadores de los loaders:

```bash
python3 scripts/validate_phase6_catalog_loader.py
python3 scripts/validate_phase6_nodes_inventory_loader.py
python3 scripts/validate_phase6_drivers_loader.py
python3 scripts/validate_phase6_active_orders_loader.py
python3 scripts/validate_phase6_historical_orders_loader.py
python3 scripts/validate_phase6_routes_loader.py
python3 scripts/validate_phase6_storefront_loader.py
```

Cada validador levanta PostgreSQL desechable, aplica el modelo, ejecuta dos
veces el loader y comprueba que la segunda ejecución no duplique registros. No
usar una base compartida para estas pruebas.

## 3. Inspección DEV en solo lectura

El inspector desplegado acepta solamente la cuenta, región, función, base,
empresa y SHA de fuente fijados. Generar un snapshot nuevo así:

```bash
aws lambda invoke \
  --function-name mastererp-dev-database-migrator \
  --profile andrea_madriz_mastererp \
  --region us-west-1 \
  --cli-binary-format raw-in-base64-out \
  --payload '{"action":"inspect_phase6","name":"Tumandaito","source_sha256":"6a6d9c5030bd7c63070a91876b9114b0d8d36553cddf8c15bc103589084a212d"}' \
  /tmp/mastererp-phase6-snapshot.json
```

Comprobar que la respuesta tenga `status: succeeded` y `read_only: true`:

```bash
jq '{status,read_only,source_sha256,counts}' \
  /tmp/mastererp-phase6-snapshot.json
```

La Lambda inicia `BEGIN READ ONLY` y termina con `ROLLBACK`; no es una carga ni
un despliegue.

## 4. Reconciliación exacta

Conservar fuera de Git el bundle canónico firmado que produjo la carga. No
reconstruirlo desde datos que hayan cambiado, porque dejaría de ser la misma
evidencia. Compararlo con el snapshot recién obtenido:

```bash
python3 migration/phase6/reconcile_dev_snapshot.py \
  --bundle /ruta/al/bundle-firmado \
  --snapshot /tmp/mastererp-phase6-snapshot.json \
  --report /tmp/mastererp-phase6-reconciliation.json
```

El comando termina con código `0` únicamente si `ok` es `true`. Deben coincidir
los conteos y fingerprints de Categorías, Productos, Clientes, Nodos,
Inventarios, Repartidores, Pedidos, Rutas y Tiendas, y `differences` debe ser
vacío.

## 5. Prueba funcional pendiente

Cuando Fase 7 conecte infraestructura y servicios, se probarán por API y UI:

1. autenticación, tenant, empresa y permisos;
2. lectura de catálogo, existencias, pedidos, rutas y Storefront;
3. operaciones manuales sin IA y herramientas autorizadas para IA;
4. aislamiento multi-tenant, errores e idempotencia;
5. smoke/E2E, delta final y rollback antes del cutover.

Hasta completar esas puertas, Tu Mandadito sigue siendo la fuente de verdad.
La evidencia ya obtenida está en
[phase-6-dev-evidence.md](phase-6-dev-evidence.md).
