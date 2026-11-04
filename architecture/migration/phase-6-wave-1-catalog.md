# Fase 6 — Oleada 1: Categorías y Productos

Estado: loader ejecutado y reconciliado en DEV. Evidencia consolidada en
[phase-6-dev-evidence.md](phase-6-dev-evidence.md).

## Diseño

1. El transformador offline valida tenant y genera bundle ordenado con SHA-256.
2. El loader verifica el hash esperado, hashes de archivos, slugs, referencias,
   unicidad, estados, precios y atributos antes de emitir SQL.
3. Categorías se confirman primero. Productos se dividen en lotes configurables
   de 1–1000 registros (200 por defecto).
4. Cada lote usa transacción, advisory lock, `mastererp_app`, tenant, actor,
   correlación y contexto de operación.
5. Solo se llaman `catalogo.categoria_crear` y `catalogo.producto_crear`; el
   artefacto no contiene `INSERT`, `UPDATE` ni `DELETE` directo.
6. Las claves idempotentes dependen del hash fuente, tipo e ID externo. Una
   reejecución retoma lotes incompletos sin duplicar los confirmados.

## Evidencia local

- Transformación repetida con hashes idénticos.
- Rechazo de tenant incorrecto y hash fuente diferente.
- SQL generado inspeccionado sin DML directo.
- Aplicación completa dos veces en PostgreSQL 17.9 desechable.
- Resultado tras ambas ejecuciones: 1 categoría, 1 producto y 2 eventos de
  auditoría; la segunda ejecución no duplicó datos ni auditoría.
- Rol efectivo: `mastererp_app`; escrituras remotas: ninguna.
- Loader: 113 líneas; todos los archivos de esta oleada cumplen el máximo 500.

## Para promover a DEV

Se requiere export real inmutable, configuración de tenant/actor aprobada,
revisión del bundle y del SQL emitido, ventana autorizada, snapshot previo,
ejecución supervisada y reconciliación. Este documento no concede esa aprobación.
