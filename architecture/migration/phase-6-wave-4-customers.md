# Fase 6 — oleada 4: Clientes

Estado: candidato local completo; no ejecutado contra DEV ni promovido como
fuente de verdad.

## Alcance

- `CUSTOMER` se transforma determinísticamente a Tercero con rol `cliente`.
- `customerId` se conserva como código y referencia externa estable.
- El perfil queda limitado a la empresa explícita del archivo de configuración.
- No se crea una tabla Customer paralela ni se copia `cognitoSub` al maestro ERP.
- Clientes inactivos quedan bloqueados hasta aprobar una política de ciclo de vida.

## Seguridad y repetibilidad

El loader usa únicamente `administracion.party_write` bajo `mastererp_app`, con
tenant, actor, operación, correlación e idempotencia estable. La nueva función
de lectura `administracion.cliente_id_por_referencia_externa` exige alcance de
empresa y permiso de Partes; no concede DML.

## Evidencia local

El ensayo PostgreSQL 17.9 aislado aplicó dos veces Catálogo, Clientes,
Nodos/Inventario y Repartidores. Resultado: un tercero, un perfil de cliente,
sin duplicados; 8 eventos de auditoría totales y rol efectivo `mastererp_app`.
Las pruebas unitarias verifican hash de origen, determinismo y ausencia de DML
directo en el artefacto SQL.

## Puertas todavía cerradas

- No se ha usado una exportación real de DynamoDB.
- No se ha escrito en RDS/DEV.
- Email y teléfono permanecen fuera del Tercero mínimo hasta existir un contrato
  canónico para datos de contacto; los Pedidos conservan su snapshot histórico.
- Pedidos activos y reservas deben migrarse como una unidad consistente.
