# Evidencia DEV — Fase 6 Datos y convivencia

Fecha: 2026-11-04. Entorno: cuenta MasterERP `277352717693`, región
`us-west-1`, base `mastererp-dev-data`. Fuente: cuenta Tu Mandadito
`941751053509`, región `us-west-2`, tabla `tumandaito-dev-core`, tenant `T001`.

## Protección y trazabilidad

- Backup previo: `tumandaito-dev-core-before-mastererp-phase6-20261104`.
- Backup reconciliado: `tumandaito-dev-core-phase6-reconciled-20261104`.
- Snapshot RDS previo: `mastererp-dev-before-phase6-20261104`.
- SHA-256 fuente: `6a6d9c5030bd7c63070a91876b9114b0d8d36553cddf8c15bc103589084a212d`.
- La carga aceptó únicamente cuatro artefactos con hashes fijados y rechazó
  DML/DDL directo. Se ejecutó mediante el migrador oficial, no desde un agente.
- La inspección posterior abrió `BEGIN READ ONLY` y cerró con `ROLLBACK`.

## Resultado de carga

| Artefacto | SHA-256 | Resultado |
| --- | --- | --- |
| Catálogo | `5f71a6bc3b340d057cdda6dae45ec639d9ab28ff06fbfbcb95201593f1d7a2a2` | Correcto |
| Clientes | `2a6c2d3666f653a544b6fa8fb68389b8bdcc3dd99b0a2c502f2cbcfdb8a1dac0` | Correcto |
| Operaciones | `dce9404d080bfd4fa4d1bfef4bf158614ee7fff80c07ad4058c1c20e35f32ccb` | Correcto |
| Storefront | `dfbbf3b5f11773dd24c3853569f8d6b6ad064defc7707db2d9f3a804097a239e` | Correcto |

## Reconciliación semántica

| Capacidad | Fuente | Destino | Fingerprint coincide |
| --- | ---: | ---: | --- |
| Categorías | 11 | 11 | Sí |
| Productos | 27 | 27 | Sí |
| Clientes | 17 | 17 | Sí |
| Nodos | 5 | 5 | Sí |
| Inventarios | 42 | 42 | Sí |
| Repartidores | 6 | 6 | Sí |
| Pedidos | 9 | 9 | Sí |
| Rutas | 3 | 3 | Sí |
| Tiendas | 1 | 1 | Sí |

Los fingerprints comparan claves técnicas, estados, stock, reservas, moneda,
subtotales, tarifas, totales, cantidad de líneas, nodos, repartidores, paradas y
dominios. Resultado: `ok: true`, sin faltantes ni registros inesperados. Los 9
pedidos contienen 12 líneas; Inventario conserva stock total `914` y reservado
total `6`.

Se auditaron y excluyeron 10 inventarios legacy sin Producto correspondiente,
además de 3 inventarios globales sustituidos por inventario por Nodo y 2 rutas
vacías. No se inventaron productos ni se ocultaron reservas activas.

## Límite de esta evidencia

La sombra DEV está íntegra, pero todavía no atiende tráfico. Esta evidencia no
acredita smoke/E2E de APIs, delta final, restauración operativa ni cutover.
Durante la convivencia Tu Mandadito continúa como fuente de verdad. La
formalización de MasterERP corresponde al cierre de capacidad después de la
integración de infraestructura.

