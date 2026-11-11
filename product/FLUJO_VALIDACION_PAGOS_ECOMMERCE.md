# Validación de pagos de Ecommerce

## Propiedad

Storefront captura el método, la referencia y los datos del pagador. Finanzas → Administración → Tesorería → Cobros es la única propietaria de validar o rechazar pagos electrónicos. Operaciones → Pedidos consume el resultado como solo lectura y no puede modificarlo.

## Flujo

1. Efectivo o punto de venta queda `PENDING_COLLECTION` y puede avanzar según la política de cobro en entrega.
2. Pago móvil o transferencia crea el pedido `PENDING` con pago `PENDING_VERIFICATION`.
3. Storefront informa «En espera de validación de Finanzas».
4. Finanzas consulta la bandeja por tenant y empresa, revisa referencia, receptor, cliente, total y evidencia.
5. Finanzas registra motivo, evidencia, actor y fecha y decide `VALIDATED` o `REJECTED`.
6. `VALIDATED` crea la venta y confirma el pedido dentro de la misma transacción. Un fallo revierte toda la decisión.
7. `REJECTED` conserva el pedido pendiente y bloquea su confirmación operativa.
8. Pedidos muestra el estado financiero, sin controles para alterarlo.

## Autorización y seguridad

La API exige Cognito verificado, tenant, empresa y `administration.receipts.validate`; Database vuelve a comprobar módulo Finanzas, permiso, empresa y nodo. Las decisiones usan versión optimista, clave de idempotencia, correlación y auditoría. `administration.sales.write` continúa siendo obligatorio para crear la venta. No hay datos simulados ni validación confiada al navegador.

## Publicación coordinada

El código fuente requiere la migración 054, el artefacto de Tesorería y sus rutas JWT. El push no sustituye el workflow autorizado de Database ni despliega manualmente Lambdas o CloudFormation.
