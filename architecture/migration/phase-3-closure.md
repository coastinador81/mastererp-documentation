# Cierre de Fase 3 — Contratos y límites

Fecha: 2026-11-03.

Estado: completada y aprobada como línea base para planificación.

## Entregables aceptados

- Inventario técnico de Tu Mandadito aprobado.
- Matriz de capacidades, propietarios y destinos aprobada para planificación.
- Contrato de Pedidos v1 definido como primer corte vertical.
- Entidades y propietarios delimitados.
- Estados de pedido y pago separados.
- Operaciones, compatibilidad HTTP y errores documentados.
- Autenticación, autorización, aislamiento y auditoría definidos como obligaciones.
- Idempotencia y concurrencia definidas como requisitos de implementación.
- Integraciones y herramientas MCP candidatas documentadas sin publicarlas.

## Línea base aprobada

La implementación futura debe respetar [Pedidos v1](../../api/orders-v1.md). Un cambio que altere propiedad funcional, estados, autorización, modelo de idempotencia o límites entre módulos requiere actualizar el contrato y registrar la decisión antes de implementar el comportamiento afectado.

## Gates pendientes

La aprobación de la línea base no resuelve automáticamente:

- unicidad definitiva de `orderNumber`;
- política de cancelación del cliente;
- modelo cliente/party;
- contrato definitivo de Pagos;
- moneda base y redondeo;
- propietario de ventanas de entrega;
- códigos finales de permisos;
- URLs y OpenAPI definitivos.

Estos gates pueden resolverse de forma incremental. Solo bloquean la operación o componente al que afectan; no invalidan el contrato completo ni autorizan inventar una respuesta.

## Condición para iniciar Fase 4

Antes de escribir código del corte de Pedidos se debe preparar un plan de implementación acotado que identifique:

1. qué operación se implementará primero;
2. qué gates necesita resolver esa operación;
3. qué repositorios y archivos cambiarán;
4. cómo convivirá con DynamoDB/Tu Mandadito;
5. pruebas, rollback y evidencia requeridos.

Fase 4 no iniciada por este cierre.
