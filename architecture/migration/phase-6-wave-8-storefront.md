# Fase 6 — oleada 8: Storefront

Estado: ejecutado y reconciliado en sombra DEV. Ver
[evidencia consolidada](phase-6-dev-evidence.md).

Se migra únicamente configuración `MERCHANT` a Tiendas. Cada comercio requiere
un mapa explícito hacia Nodo y dominio; no se inventan dominios. Catálogo,
Productos e Inventario no se copian: Storefront consume las capacidades
canónicas mediante la API pública.

La configuración conserva branding, pagos, ventanas y módulos habilitados. Una
misma fuente puede producir varias Tiendas cuando el archivo de mapeo declara
varios Nodos/dominios.

Evidencia: dos aplicaciones en PostgreSQL 17.9 produjeron una sola Tienda
`ACTIVE`, dominio estable y versión 2 sin incrementos posteriores. Rol efectivo
`mastererp_app`, sin DML en el loader ni escrituras remotas.
