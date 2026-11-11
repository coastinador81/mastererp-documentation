# Contrato de Pedidos v1

Estado: contrato implementado en su corte operativo inicial; candidato no desplegado en MasterERP.

Aprobación de línea base: 2026-11-03. La promoción, migración histórica y el cambio de fuente de verdad permanecen como gates explícitos.

Propietario funcional: `2.3-operations/2.3.1-orders`.

Este documento define el límite de Pedidos antes de migrar código. Conserva las reglas verificadas de Tu Mandadito, las adapta al aislamiento de MasterERP y separa responsabilidades de Catálogo, Inventario, Preparación, Rutas, Transporte y Pagos.

## 1. Alcance

Pedidos es propietario de:

- identidad, número y ciclo de vida del pedido;
- líneas y snapshots comerciales confirmados al crear;
- totales y moneda registrados en el pedido;
- referencia al cliente, empresa, nodo y ventana de entrega;
- snapshot de entrega necesario para cumplir el pedido;
- referencia y estado operativo del pago, consumiendo el contrato de Pagos;
- historial de transiciones del pedido;
- vínculo con una ruta, sin apropiarse de la entidad Ruta;
- creación idempotente y cancelación coordinada.

Pedidos no es propietario de:

- productos, categorías, precios vigentes o disponibilidad: Catálogo/Inventario;
- reservas y movimientos físicos: Inventario;
- checklist de preparación: Preparación;
- definición y ejecución de rutas: Rutas;
- identidad/disponibilidad del repartidor: Transporte;
- validación financiera o liquidación: Pagos/Finanzas;
- identidad, membresía o permisos: Plataforma Base/Seguridad.

## 2. Entidades conceptuales

### 2.1 Order

| Campo | Regla |
| --- | --- |
| `id` | UUID interno MasterERP. |
| `externalId` | ID histórico opcional para trazabilidad de Tu Mandadito. |
| `orderNumber` | Identificador visible, único dentro de su alcance comercial definido. |
| `tenantId` | Obligatorio e inmutable. Unidad de aislamiento. |
| `companyId` | Obligatorio cuando el módulo/operación lo requiera; debe pertenecer al tenant. |
| `nodeId` | Nodo propietario de la preparación/entrega; debe pertenecer al mismo tenant y empresa aplicable. |
| `customerId` | Referencia al cliente/party autorizado. |
| `status` | Estado canónico del pedido. |
| `currency` | Moneda de registro de los totales. |
| `subtotal`, `deliveryFee`, `total` | Importes no negativos; `total = subtotal + deliveryFee` para el alcance actual. |
| `exchangeRateSnapshot` | Tasa, fuente y fecha efectiva cuando se registren importes convertidos. |
| `deliveryWindowId` | Referencia a la ventana seleccionada. |
| `deliveryMethod` | `realtime` o `scheduled` para compatibilidad inicial. |
| `routeId` | Referencia opcional a Rutas; no transfiere propiedad. |
| `version` | Control de concurrencia optimista. |
| `createdAt`, `updatedAt` | Timestamps de servidor. |

### 2.2 OrderItem

Cada línea pertenece a un solo pedido y conserva snapshot de:

- `productId`;
- nombre comercial;
- cantidad positiva;
- precio unitario;
- subtotal de línea.

El snapshot es histórico. Cambiar el producto después no reescribe pedidos existentes. El estado `PENDING/DONE` de preparación pertenece a Preparación y no debe convertirse en el estado principal de `OrderItem` dentro de Pedidos.

### 2.3 CustomerSnapshot

Conserva nombre y teléfono confirmados para la operación, además de `customerId`. No sustituye el maestro de clientes ni autoriza acceso al pedido.

### 2.4 DeliverySnapshot

Conserva etiqueta, dirección, zona, referencia opcional y coordenadas opcionales. Latitud y longitud deben estar ambas presentes o ambas ausentes. El código de confirmación de entrega es secreto operativo: nunca aparece en APIs del repartidor y solo se expone al cliente autorizado cuando el flujo lo requiere.

### 2.5 PaymentSnapshot

Pedidos conserva una proyección operativa del pago:

- método;
- moneda e importes relevantes;
- estado de validación;
- referencia manual cuando corresponda;
- validación/rechazo y actor.

Esta proyección no convierte a Pedidos en propietario de transacciones financieras. La evolución deberá consumir eventos o API del módulo de Pagos.

### 2.6 OrderStatusEvent

Evento inmutable con:

- pedido, tenant y empresa/nodo aplicables;
- estado anterior y nuevo;
- motivo y detalle opcional sanitizado;
- actor;
- timestamp de servidor;
- correlation ID.

No se guardan tokens, prompts, credenciales ni payloads completos en auditoría.

## 3. Estados del pedido

Estados canónicos:

| Estado | Significado | Propietario del cambio |
| --- | --- | --- |
| `PENDING` | Pedido recibido, pendiente de confirmación/pago aplicable. | Pedidos |
| `CONFIRMED` | Pedido aceptado para operación. | Pedidos/Admin o evento autorizado de Pago |
| `PREPARING` | Preparación iniciada. | Preparación |
| `READY` | Preparación completa y disponible para ruta. | Preparación |
| `OUT_FOR_DELIVERY` | Ruta iniciada con el pedido. | Rutas |
| `DELIVERED` | Entrega confirmada con evidencia válida. | Rutas/entrega autorizada |
| `CANCELLED` | Pedido cancelado y compensaciones aplicadas. | Pedidos |

`PREPARED` es únicamente un alias histórico de lectura. Las operaciones nuevas no lo escriben; durante migración se normaliza a `READY` o se conserva como valor externo con traducción explícita.

### 3.1 Transiciones permitidas

```text
PENDING ───────► CONFIRMED ───────► PREPARING ───────► READY
   │                 │                   │                │
   └──► CANCELLED    └──► CANCELLED      └──► CANCELLED   └──► OUT_FOR_DELIVERY ───► DELIVERED
```

Reglas:

- Confirmar exige pago apto para progreso operativo.
- Pago electrónico exige estado `VALIDATED`.
- Efectivo puede avanzar sin validación previa y permanece pendiente de cobro según Pagos.
- Finalizar Preparación exige todas las líneas completas y confirmación transaccional de Inventario.
- Iniciar Ruta exige pedido `READY`, ruta compatible y mismo nodo/alcance.
- Entregar exige repartidor/ruta autorizados, evidencia de ubicación y código correcto según el contrato de entrega.
- Cancelar desde `PENDING`, `CONFIRMED` o `PREPARING` exige compensación idempotente de la reserva/stock aplicable.
- `READY`, `OUT_FOR_DELIVERY` y `DELIVERED` no se cancelan mediante el contrato inicial; una excepción futura requiere flujo compensatorio específico.
- Repetir una transición ya completada puede devolver el mismo resultado solo cuando la operación sea idempotente y el fingerprint coincida.

## 4. Estados operativos del pago

| Estado | Uso inicial |
| --- | --- |
| `PENDING_VERIFICATION` | Pago móvil/transferencia esperando revisión. |
| `VALIDATED` | Pago manual aceptado; puede confirmar pedido. |
| `REJECTED` | Referencia rechazada; cliente puede reenviar una nueva. |
| `PENDING_COLLECTION` | Efectivo pendiente de cobro; permite progreso operativo. |

La referencia manual debe ser única dentro del tenant mientras este mecanismo exista. La garantía final pertenecerá a Pagos, no a una tabla paralela de Pedidos.

## 5. Operaciones canónicas

Los nombres son estables aunque el transporte o las URLs cambien durante integración.

### 5.0 Perfil implementado en el primer corte

El candidato previo a despliegue expone el subconjunto operativo Admin que
permite crear, listar, consultar y transicionar pedidos:

```text
GET  /v1/companies/{company_id}/orders
GET  /v1/companies/{company_id}/orders/{order_id}
POST /v1/companies/{company_id}/orders
POST /v1/companies/{company_id}/orders/{order_id}/transitions
```

La creación recibe snapshots e importes ya resueltos por capacidades de
Catálogo/Inventario y exige `inventory_reservation_ref`. El corte rechaza crear
sin reserva y no escribe stock. Historial, dashboard, rutas de cliente, pagos y
asignación de ruta permanecen fuera de este perfil inicial; no deben anunciarse
como disponibles hasta implementar sus propietarios.

| Operación | Actor | Efecto |
| --- | --- | --- |
| `orders.create` | Cliente autorizado | Valida nodo, productos, precios y stock; crea pedido y reserva inventario de forma atómica. |
| `orders.list_own` | Cliente autorizado | Lista únicamente pedidos del cliente activo. |
| `orders.get_own` | Cliente autorizado | Devuelve detalle/seguimiento propio. |
| `orders.cancel_own` | Cliente autorizado | Cancela si estado y política lo permiten. |
| `orders.resubmit_payment_reference` | Cliente autorizado | Reenvía referencia después de rechazo. |
| `orders.list` | Usuario operativo | Lista por alcance de tenant/empresa/nodo y filtros autorizados. |
| `orders.get` | Usuario operativo | Consulta detalle dentro de su alcance. |
| `orders.dashboard` | Usuario operativo | Agrega contadores usando el mismo alcance/filtros del listado. |
| `orders.history` | Usuario autorizado | Consulta eventos sanitizados. |
| `orders.tracking` | Usuario autorizado | Consulta proyección persistida; no inventa ETA. |
| `orders.confirm` | Usuario operativo | Confirma un pedido apto. |
| `orders.cancel` | Usuario operativo | Cancela con razón y compensación. |
| `orders.validate_payment` | Usuario autorizado de pagos | Valida o rechaza referencia manual. |
| `orders.assign_route` | Usuario operativo | Solicita a Rutas vincular pedido `READY`. |

### 5.1 Rutas actuales de compatibilidad

Durante la transición, los adapters pueden conservar:

#### Cliente

- `POST /orders`
- `GET /orders`
- `GET /orders/{orderId}`
- `POST /orders/{orderId}/cancel`
- `PATCH /orders/{orderId}/payment-reference`

#### Admin

- `GET /operations/orders`
- `GET /operations/orders/dashboard`
- `GET /operations/orders/{orderId}`
- `GET /operations/orders/{orderId}/history`
- `GET /operations/orders/{orderId}/tracking`
- `PATCH /operations/orders/{orderId}/status`
- `POST /operations/orders/{orderId}/payment-validation`
- `POST /operations/orders/{orderId}/cancel`
- `PATCH /operations/orders/{orderId}/route`

Estas rutas no fijan la URL pública final de MasterERP. El contrato OpenAPI y el versionado HTTP se definirán al integrar el API propietario existente.

## 6. Creación de pedido

### 6.1 Entrada lógica

```json
{
  "clientRequestId": "compatibility-request-id",
  "nodeId": "node-reference",
  "customer": {
    "customerId": "optional-customer-reference",
    "name": "Nombre",
    "phone": "+58..."
  },
  "delivery": {
    "label": "Casa",
    "addressLine": "Dirección",
    "zone": "Zona",
    "reference": "Opcional",
    "latitude": 10.0,
    "longitude": -66.0,
    "method": "realtime"
  },
  "slotId": "delivery-slot-reference",
  "payment": {
    "method": "mobile_payment",
    "reference": "123456",
    "currency": "VES"
  },
  "items": [
    {"productId": "product-reference", "quantity": 1}
  ]
}
```

Compatibilidad verificada:

- `clientRequestId`: 8–64 caracteres.
- máximo 30 productos distintos; líneas repetidas se consolidan.
- cantidad entera positiva.
- métodos: `mobile_payment`, `bank_transfer`, `cash`.
- referencia electrónica: 4–40 dígitos.
- efectivo: `VES` o `USD`.
- coordenadas válidas y suministradas en pareja.
- `nodeId` debe coincidir en todas las fuentes temporales donde aparezca.

Objetivo MasterERP:

- actor, tenant y empresa se obtienen de contexto autenticado, no del cuerpo;
- `nodeId` se valida dentro del tenant/empresa y contra permisos;
- precios, nombres, tasa e importes se calculan en servidor;
- nunca se confía en totales enviados por el cliente;
- se acepta `Idempotency-Key` canónica; el adapter transforma temporalmente `clientRequestId`.

### 6.2 Resultado

Creación nueva devuelve éxito de creación; replay idempotente devuelve el recurso previamente creado. Incluye ID/número, estado, cliente, importes, moneda, ventana, líneas y timestamps. Nunca devuelve el código secreto de entrega en una respuesta no autorizada.

## 7. Listado, filtros y acciones agrupadas

Filtros actuales que deben conservar semántica inicialmente:

- `status`;
- búsqueda textual;
- fecha desde/hasta en `YYYY-MM-DD`;
- método de pago;
- orden por fecha, más recientes o más antiguos;
- cursor keyset compuesto por fecha e ID;
- límite 1–100, predeterminado 25.

El backend vuelve a autorizar tenant y empresa en cada página. El cursor se
interpreta con los filtros y el orden de la misma petición; el frontend no
descarga un subconjunto fijo para simular búsqueda completa.

`POST /v1/companies/{company_id}/orders/bulk-transitions` recibe entre 1 y 50
pares `id/version`, un único estado (`CONFIRMED` o `CANCELLED`) y motivo. Todo el
lote se ejecuta en la misma transacción: cualquier autorización, versión o
precondición inválida revierte todas las transiciones.

## 8. Autenticación y autorización

Cada operación debe:

1. verificar identidad externa mediante `issuer + sub`;
2. resolver usuario interno activo;
3. validar membresía activa en tenant;
4. validar módulo `2.3-operations` habilitado;
5. validar empresa activa cuando aplique;
6. validar permiso de la operación;
7. limitar nodos/recursos al alcance autorizado;
8. establecer contexto transaccional PostgreSQL solo después de autenticar;
9. verificar que todo recurso leído o escrito pertenece al mismo tenant/empresa.

Permisos semánticos requeridos, con código final pendiente de Seguridad:

- leer pedidos propios;
- crear/cancelar pedido propio;
- leer pedidos operativos;
- confirmar/cancelar pedidos;
- validar pagos manuales;
- asignar pedidos a rutas;
- leer historial/auditoría.

Para recursos fuera de alcance, responder como no encontrado cuando revelar su existencia produzca fuga de información. Los filtros del frontend no conceden acceso.

## 9. Idempotencia y concurrencia

Operaciones de escritura externas requieren:

- clave de idempotencia de 1–128 caracteres;
- alcance por actor, tenant y operación;
- fingerprint canónico del request relevante;
- bloqueo transaccional de la clave;
- replay solo si actor, tenant, operación y fingerprint coinciden;
- conflicto si la clave se reutiliza con otro request;
- respuesta persistida sin datos sensibles.

La creación coordina atómicamente pedido, líneas, número, reserva de inventario y referencia manual cuando aplique. Si Inventario permanece temporalmente en otro sistema, la atomicidad distribuida deberá resolverse mediante operación idempotente y compensación explícita; no se simula éxito.

Actualizaciones usan `version` o condición equivalente para impedir pérdida de escrituras concurrentes.

## 10. Errores

Envelope objetivo:

```json
{
  "ok": false,
  "code": "ORDER_NOT_FOUND",
  "message": "Mensaje sanitizado",
  "correlationId": "optional-correlation-id",
  "details": {}
}
```

| HTTP | Código | Significado |
| ---: | --- | --- |
| 400 | `INVALID_REQUEST`, `INVALID_FILTER` | Forma, formato o filtro inválido. |
| 401 | `UNAUTHENTICATED` | Identidad ausente/inválida. |
| 403 | `FORBIDDEN` | Actor autenticado sin permiso. |
| 404 | `ORDER_NOT_FOUND`, `PRODUCT_NOT_FOUND`, `INVENTORY_NOT_FOUND`, `NODE_NOT_FOUND` | Recurso inexistente o no visible. |
| 409 | `INVALID_TRANSITION`, `IDEMPOTENCY_CONFLICT`, `PRODUCT_CHANGED`, `PRODUCT_INACTIVE`, `INSUFFICIENT_STOCK`, `PAYMENT_CONFLICT`, `NODE_UNAVAILABLE` | Conflicto de estado o concurrencia. |
| 422 | `VALIDATION_ERROR` | Regla de entrada semántica no satisfecha. |
| 500 | `INTERNAL_ERROR` | Fallo interno sanitizado. |
| 502/503 | `DEPENDENCY_ERROR` | Dependencia real no disponible; nunca convertir en éxito. |

No se exponen detalles SQL, claves, stack traces, secretos ni existencia de recursos fuera de alcance.

## 11. Auditoría

Eventos mínimos:

- `order.created`;
- `order.confirmed`;
- `order.cancelled`;
- `order.preparation_started`;
- `order.ready`;
- `order.route_assigned`;
- `order.out_for_delivery`;
- `order.delivered`;
- `order.payment_validated`;
- `order.payment_rejected`;
- `order.payment_reference_resubmitted`.

Cada evento conserva tenant, actor, pedido, acción, timestamp, correlation ID, transición y referencias necesarias. Los detalles libres tienen tamaño limitado y sanitización.

## 12. Integraciones y eventos

| Dependencia | Contrato requerido |
| --- | --- |
| Catálogo | Obtener producto activo y snapshot comercial. |
| Inventario | Reservar, liberar y comprometer cantidades idempotentemente. |
| Preparación | Iniciar/completar y solicitar transición a `PREPARING/READY`. |
| Rutas | Asignar pedido y transicionar a `OUT_FOR_DELIVERY`. |
| Transporte | Autorizar actor conductor sin transferir propiedad de Pedido/Ruta. |
| Pagos | Validar/rechazar/cobrar y publicar estado operativo. |
| Auditoría | Registrar acciones sanitizadas e inmutables. |

No se asume consistencia mediante acceso directo a tablas ajenas. Cada módulo usa servicio, evento o función SQL autorizada con propietario explícito.

## 13. Herramientas MCP

No se publica ninguna herramienta MCP por la existencia de este documento.

Candidatas posteriores a API y permisos estables:

- `operations.orders.get`;
- `operations.orders.list`;
- `operations.orders.confirm`;
- `operations.orders.cancel`.

Cada herramienta deberá reutilizar exactamente la autorización del API, declarar esquema estricto, ser auditable y requerir aprobación humana para acciones sensibles conforme al Centro de Agentes. Un agente nunca accede directamente a PostgreSQL.

## 14. Compatibilidad y rollout

1. Tu Mandadito/DynamoDB sigue siendo fuente de verdad hasta aprobar el corte de Pedidos.
2. Los adapters conservan temporalmente rutas y envelopes actuales.
3. IDs históricos se guardan como referencias externas, no como claves relacionales primarias obligatorias.
4. `PREPARED` se acepta solo al leer registros antiguos.
5. No se habilita escritura doble sin outbox/compensación, reconciliación y propietario definidos.
6. El cambio de fuente de verdad exige pruebas de aislamiento, transiciones, inventario, pagos, idempotencia y rollback.

## 15. Pendientes funcionales

- Alcance exacto y unicidad de `orderNumber` por tenant/empresa/nodo.
- Política de cancelación por parte del cliente frente a Admin.
- Modelo definitivo de cliente/party para e-commerce.
- Contrato definitivo de Pagos y cobro en efectivo.
- Política de retención de historial e idempotencia.
- Moneda base por empresa y reglas de redondeo.
- Si la ventana de entrega pertenece a Digital Sales, Operaciones o configuración de Nodo.
- Códigos definitivos de permisos en Seguridad.
- URLs/versionado OpenAPI finales sobre el API compartido existente.

## 16. Criterios para promover a DEV

- [x] Entidades y propietarios del perfil inicial definidos.
- [x] Estados y transiciones cubiertos por pruebas.
- [x] Seguridad tenant/empresa y permisos aplicada en PostgreSQL.
- [x] Modelo SQL, funciones controladas y bloqueo de DML probados.
- [x] Reserva de Inventario exigida como precondición opaca.
- [x] Idempotencia y concurrencia optimista probadas.
- [x] Rutas HTTP iniciales e infraestructura candidatas.
- [ ] Artefactos publicados y change set revisado.
- [ ] Smoke/E2E DEV y rollback ensayados.
- [ ] Backfill, reconciliación y estrategia de cutover aprobados.
