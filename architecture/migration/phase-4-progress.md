# Progreso de Fase 4 — Migración por cortes verticales

Estado: cerrado como candidato local no desplegado. Evidencia final en [phase-4-closure.md](phase-4-closure.md).

## Preparación, Rutas y E-commerce/Storefront

Preparación consume de forma controlada la reserva de Inventario y deja el
pedido listo. Rutas referencia Repartidores sin absorberlo y ofrece toda la
operación manual de asignación, paradas y entrega. E-commerce configura tiendas
por tenant/dominio/nodo; el storefront base consume Catálogo e Inventario por API
y declara módulos opcionales sin copiarlos.

## Categorías — corte vertical completo antes de despliegue

Categorías incorpora un árbol por tenant con slug estable, jerarquía segura,
orden, imagen y activación. El vínculo estricto con productos queda diferido al
backfill para conservar compatibilidad con referencias heredadas.

## Atributos — corte vertical completo antes de despliegue

Atributos incorpora definiciones tipadas, opciones y alcance por categorías.
Los valores JSON actuales de Productos se conservan hasta reconciliar códigos y
habilitar la validación estricta durante el backfill.

## Catálogo restante — Marcas y Proveedores

Marcas aporta su maestro independiente. Proveedores reutiliza Terceros como
identidad canónica y añade exclusivamente datos de abastecimiento. Con estos
cortes, Productos, Categorías, Atributos, Marcas, Proveedores e Inventario
completan el candidato funcional de Catálogo previo a despliegue y backfill.

## Repartidores — corte vertical completo antes de despliegue

Transporte conserva el perfil operativo, disponibilidad, acceso y asignación a
nodos. El usuario interno es un vínculo opcional; Cognito no se aprovisiona desde
la Lambda de negocio y Rutas será propietaria de recorridos y entregas activas.

El corte incluye SQL autorizado e idempotente, API/Lambda, cinco rutas JWT,
operación manual y contrato. Despliegue, conciliación de datos heredados e
integración al app-shell permanecen para las fases posteriores.

## Inventario y Nodos — cortes verticales completos antes de despliegue

Nodos modela el perfil operativo de una sucursal existente, sin duplicar la
identidad organizacional. Inventario enlaza nodo y producto, registra stock,
reservado, disponible y movimientos, y aporta reservas atómicas consumibles por
Pedidos. Las APIs, Lambdas, recursos y pantallas permanecen separados por dueño.

Se probaron creación/configuración de nodo, transición, alta de stock, reserva,
consumo, rechazo por insuficiencia y ausencia de DML directo. El despliegue,
backfill y cutover siguen reservados para las fases 6–8.

## Productos — corte vertical completo antes de despliegue

Productos quedó separado de Inventario: ficha maestra, precio comercial, estado,
imagen, categoría referenciada y atributos viven en `catalogo.productos`; stock,
reservas y disponibilidad no forman parte de sus tablas ni respuestas.

Incluye SQL controlado, API administrativa, Lambda empaquetable, cuatro rutas
JWT, UI manual y contrato. La UI obtiene la sesión del app-shell en runtime; no
incluye tokens en variables Vite. El siguiente vertical es Inventario.

## Pedidos — corte vertical completo antes de despliegue

Los cuatro repositorios contienen una misma capacidad canónica:

- `mastererp-database`: tablas multi-tenant, líneas, eventos, creación
  idempotente, lectura y transiciones mediante funciones controladas;
- `mastererp-platform/backend`: dominio, repositorio SQL, adapter HTTP y paquete
  Lambda reproducible;
- `mastererp-infrastructure`: Lambda privada y cuatro rutas JWT del API Admin;
- `mastererp-platform/frontend`: operación manual para listar, filtrar, revisar
  detalle y ejecutar transiciones;
- `mastererp-documentation`: contrato, decisiones y evidencia de migración.

Rutas implementadas:

```text
GET  /v1/companies/{company_id}/orders
GET  /v1/companies/{company_id}/orders/{order_id}
POST /v1/companies/{company_id}/orders
POST /v1/companies/{company_id}/orders/{order_id}/transitions
```

## Decisiones de límite

- La sucursal de Organización es la referencia operativa inicial; no se creó
  una copia de Nodos dentro de Pedidos.
- Cliente referencia `catalogo.terceros` con rol `cliente`.
- Inventario conserva la propiedad del stock: Pedidos exige una referencia
  opaca de reserva y nunca modifica existencias directamente.
- Cognito/API Gateway autentica; identidad interna, tenant, empresa, módulo y
  permisos vuelven a comprobarse en backend/PostgreSQL.
- La operación manual sigue disponible aunque en el futuro existan agentes.

## Evidencia local

- PostgreSQL 17.9 desechable, sin red ni puertos: dos aplicaciones consecutivas
  del modelo y todas las pruebas SQL aprobadas.
- 14 pruebas Python aprobadas para dominio y API.
- ZIP Lambda determinista generado e inspeccionado localmente.
- UI Vite compilada correctamente, sin vulnerabilidades reportadas por npm.
- `validate_structure.py` y `git diff --check` correctos.

## Qué significa “Pedidos terminado”

Está terminado el candidato migrable y verificable previo a promoción. Tu
Mandadito continúa como fuente de verdad porque no se autorizó despliegue,
migración histórica, tráfico dual ni corte. Para operar en DEV aún se requiere
una ventana explícita de promoción:

1. empaquetar dependencias arm64 y publicar artefacto;
2. aplicar `015_operations_orders.sql` mediante invocación explícita;
3. crear/revisar change set CloudFormation y desplegar;
4. integrar la UI en el app-shell;
5. ejecutar smoke/E2E con tenant de prueba;
6. ensayar backfill y reconciliación antes del cutover.

Esos pasos cambian estado externo y no se ejecutan como efecto implícito de la
migración de código.
