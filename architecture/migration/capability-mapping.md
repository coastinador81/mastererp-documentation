# Matriz de migración de capacidades

Estado: Fase 2 completada; contratos detallados pendientes de Fase 3.

Esta matriz convierte el inventario de Tu Mandadito en destinos concretos dentro de los cuatro repositorios MasterERP. No autoriza copiar código, desplegar recursos ni cambiar fuentes de verdad.

## 1. Reglas de mapeo

1. El destino se determina por propietario funcional, no por el repositorio o nombre técnico de origen.
2. Platform contiene frontend y backend; Infrastructure declara recursos; Database contiene modelo y migraciones; Documentation conserva contratos y operación.
3. Las rutas funcionales deben coincidir en las capas correspondientes.
4. Una fila `ADAPTAR` exige revisar contratos, autorización multi-tenant y modelo PostgreSQL.
5. Una fila `CONSOLIDAR` exige identificar duplicados y consumidores antes de retirar código anterior.
6. Una fila `DECIDIR` no se implementa hasta acordar el límite funcional indicado.

Acciones adicionales: `REMODELAR` conserva la semántica pero cambia persistencia e invariantes; `MANTENER` conserva temporalmente el origen; `RETIRAR` solo se ejecuta después de validar consumidores y completar el corte.

## 2. Plataforma Base y Seguridad

| Origen | Responsabilidad observada | Propietario MasterERP | Destino principal | Acción | Dependencias/nota |
| --- | --- | --- | --- | --- | --- |
| `tumandaito-backend/src/client/auth` | Login, registro, confirmación, refresh, cambio de clave y OAuth | Plataforma Base / Identidad | Platform `backend/modules/1.0-platform-base/1.2-platform-management/1.2.1-identity-access` | `ADAPTAR` | Comparar Cognito issuer/clientes; no identificar por email. |
| `tumandaito-backend/src/modules/workspace/auth` | Adapter de autenticación del workspace | Plataforma Base / Identidad | Mismo destino anterior o adapter de `0-uonify` | `CONSOLIDAR` | Determinar si envuelve o duplica auth cliente. |
| `tumandaito-backend/src/shared/jwt_claims.py` | Lectura y validación de claims | Seguridad compartida | Platform `backend/shared/tenant-security` | `ADAPTAR` | Debe validar actor, tenant, empresa y pertenencia. |
| `src/shared/admin_guard.py`, `admin_scope.py`, `administration_auth.py` | Autorización y alcance Admin | Plataforma Base / Seguridad | Platform shared security + contratos de cada módulo | `ADAPTAR` | Menú/claims no conceden permisos. |
| `templates/foundation/cognito`, `templates/admin/cognito` | Pools, clientes, grupos e IdP | Plataforma Base / Identidad | Infrastructure `modules/1.0-platform-base/.../identity-access/resources.yml` | `ADAPTAR` | No recrear recursos MasterERP existentes. |
| Módulos/menús del Admin | Habilitación visual de módulos | Plataforma Base / configuración tenant | Platform `1.2.3-tenant-configuration` y `1.3-app-shell` | `ADAPTAR` | UI no sustituye autorización API. |

## 3. Catálogo e Inventario

| Origen | Responsabilidad observada | Propietario MasterERP | Destino principal | Acción | Dependencias/nota |
| --- | --- | --- | --- | --- | --- |
| `src/admin/products` | CRUD administrativo de productos | 2.1.1 Productos | Platform `backend/modules/2.1-catalog/2.1.1-products` | `ADAPTAR` | Unificar con handler modular. |
| `src/modules/admin/catalog/products` | Adapter modular de productos | 2.1.1 Productos | Mismo destino | `CONSOLIDAR` | Rastrear wrapper versus implementación real. |
| `src/client/catalog/products` | Consulta de productos storefront | 2.1.1 Productos | API de Productos consumida por Digital Sales | `ADAPTAR` | Evitar catálogo paralelo en storefront. |
| `src/client/catalog/categories` | Listado/catálogo por categorías | 2.1.2 Categorías | Platform `backend/modules/2.1-catalog/2.1.2-categories` | `ADAPTAR` | Contrato compartido Admin/storefront. |
| `src/modules/admin/catalog/categories` | Adapter Admin de categorías | 2.1.2 Categorías | Mismo destino | `CONSOLIDAR` | Confirmar implementación detrás del adapter. |
| Campos de marcas/atributos en producto | Información comercial embebida | 2.1.3 Atributos / 2.1.4 Marcas | Carpetas funcionales correspondientes | `DECIDIR` | Inventariar modelo real antes de separar entidades. |
| `src/modules/admin/procurement/suppliers` | Proveedores operativos | Finanzas/Compras: 2.2.1.2.1 | No duplicar en Catálogo | `ADAPTAR` | El catálogo inicial menciona proveedores, pero registro vigente los ubica en Compras. |
| `src/admin/inventory`, `src/modules/admin/inventory` | Stock, ajustes y movimientos | 2.1.6 Inventario | Platform `backend/modules/2.1-catalog/2.1.6-inventory` | `ADAPTAR` | Preservar reservas/idempotencia; consolidar generaciones. |
| Pantallas Admin de productos/catálogo/inventario | Operación manual | Módulos anteriores | Platform `frontend/modules/2.1-catalog/...` | `ADAPTAR` | Integrar en espacio Uonify sin fijar mecanismo prematuramente. |
| Templates `modules/admin/catalog` e `inventory` | APIs/Lambdas de catálogo e inventario | Módulos anteriores | Infrastructure `modules/2.1-catalog/.../resources.yml` | `ADAPTAR` | Propiedad única; APIs compartidas no se duplican. |
| Claves Product/Inventory/Movement | Persistencia DynamoDB | 2.1.1 / 2.1.6 | Database `modules/2.1-catalog/...` + migraciones SQL | `REMODELAR` | Access patterns informan consultas; no copiar PK/SK. |

## 4. Operaciones

| Origen | Responsabilidad observada | Propietario MasterERP | Destino principal | Acción | Dependencias/nota |
| --- | --- | --- | --- | --- | --- |
| `src/admin/orders`, `src/client/orders` | Crear, listar, consultar, cancelar y cambiar estado | 2.3.1 Pedidos | Platform `backend/modules/2.3-operations/2.3.1-orders` | `ADAPTAR` | Primer corte recomendado; múltiples canales, un propietario. |
| `src/modules/admin/operations/orders` | Adapter/handler modular de pedidos | 2.3.1 Pedidos | Mismo destino | `CONSOLIDAR` | Comparar con handlers históricos antes de elegir base. |
| `src/projections/orders` | Proyecciones de lectura | 2.3.1 Pedidos | Integración técnica interna de Pedidos | `ADAPTAR` | PostgreSQL puede eliminar necesidad de proyección equivalente. |
| `src/modules/admin/operations/preparation` | Preparación e ítems | 2.3.2 Preparación | Platform `backend/modules/2.3-operations/2.3.2-preparation` | `ADAPTAR` | Depende de Pedidos e Inventario. |
| `src/admin/routes`, `src/admin/route_definitions` | Rutas operativas y definiciones | 2.3.3 Rutas | Platform `backend/modules/2.3-operations/2.3.3-routes` | `ADAPTAR` | Transporte consume; Operaciones conserva propiedad. |
| `src/modules/admin/logistics/routes` | Adapter modular de rutas | 2.3.3 Rutas | Mismo destino | `CONSOLIDAR` | “logistics” técnico no cambia propietario funcional. |
| `src/admin/nodes`, `src/modules/admin/nodes` | Nodos, cobertura, comercios, asignaciones | 2.3.4 Nodos | Platform `backend/modules/2.3-operations/2.3.4-nodes` | `ADAPTAR` | Consolidar handlers; aclarar asignaciones de drivers. |
| Pantallas Admin Pedidos/Preparación/Rutas/Nodos | Operación manual | 2.3 Operaciones | Platform `frontend/modules/2.3-operations/...` | `ADAPTAR` | Conservar operación manual. |
| Templates Admin operations/routes/nodes | Recursos AWS | 2.3 Operaciones | Infrastructure `modules/2.3-operations/.../resources.yml` | `ADAPTAR` | No crear stack por endpoint/Lambda. |
| Claves Order/Route/Node, GSI1–GSI3/GSI5 | Persistencia y consultas | 2.3 Operaciones | Database `modules/2.3-operations/...` | `REMODELAR` | Traducir estados, invariantes y consultas a SQL. |

## 5. Transporte y Repartidores

| Origen | Responsabilidad observada | Propietario MasterERP | Destino principal | Acción | Dependencias/nota |
| --- | --- | --- | --- | --- | --- |
| `repartidores-backend/src/admin/drivers` | CRUD de repartidores | 2.10.1 Repartidores | Platform `backend/modules/2.10-transport/2.10.1-drivers` | `CONSOLIDAR` | Fuente canónica actual del dominio. |
| `repartidores-backend/src/modules/admin/logistics/drivers` | Adapter Admin | 2.10.1 Repartidores | Mismo destino | `CONSOLIDAR` | Mantener contrato, eliminar wrapper duplicado si procede. |
| `repartidores-backend/src/driver/route` | Experiencia operativa de ruta | Rutas + Repartidores | API de Rutas consumida por canal driver | `ADAPTAR` | La ruta pertenece a Operaciones. |
| `repartidores-backend/src/driver/orders` | Consulta y entrega de pedidos | Pedidos + Repartidores | API de Pedidos/entrega autorizada | `ADAPTAR` | No copiar entidad Pedido a Transporte. |
| `repartidores-backend/src/shared` | Snapshot de plataforma | Múltiples propietarios | Shared real o módulos propietarios | `RETIRAR` gradualmente | Sustituir por contratos/imports empaquetables. |
| `repartidores-frontends/admin` | Administración de repartidores | 2.10.1 Repartidores | Platform `frontend/modules/2.10-transport/2.10.1-drivers` | `CONSOLIDAR` | Separar vistas de rutas/nodos según propietario. |
| `repartidores-frontends/client` | Portal web driver legacy | Canal Transporte | Destino pendiente | `DECIDIR` | Comparar vigencia con Expo. |
| `repartidores-frontends/customer-app` | Pantallas Expo driver | Canal Transporte | Destino móvil pendiente | `DECIDIR` | Mantener durante transición. |
| Repos Infrastructure/Database de Repartidores | Wiring mínimo/documentación | Infrastructure/Database centrales | Módulos 2.10 y 2.3 correspondientes | `CONSOLIDAR` | No crear repositorios MasterERP separados. |
| Driver/Route GSI4–GSI6 | Listado y ruta activa | 2.10.1 + 2.3.3 | Modelos SQL de sus propietarios | `REMODELAR` | Restricción de una ruta activa debe ser invariante transaccional. |

## 6. Digital Sales y canales

| Origen | Responsabilidad observada | Propietario MasterERP | Destino principal | Acción | Dependencias/nota |
| --- | --- | --- | --- | --- | --- |
| `src/client/store` | Configuración, zonas, horarios y búsqueda de dirección | 2.4.2 E-commerce | Platform backend Digital Sales | `ADAPTAR` | Geocoding es integración técnica, no dueño funcional. |
| `src/client/checkout` | Cotización/validación de checkout | 2.4.2 E-commerce | Platform backend Digital Sales | `ADAPTAR` | Consume Catálogo, Inventario, Pedidos y Pagos. |
| `frontends/client` | Storefront web | 2.4.2 E-commerce | Platform `frontend/storefront-web` y/o módulo E-commerce | `ADAPTAR` | Reutilizar APIs canónicas. |
| `frontends/landing` | Sitio público | 2.4.1 Website | Platform `frontend/modules/2.4-digital-sales/2.4.1-website` | `ADAPTAR` | Revisar relación con `storefront-web`. |
| `frontends/customer-app` | Canal móvil cliente | Digital Sales | Destino técnico pendiente | `DECIDIR` | La estructura MasterERP no define app móvil todavía. |
| `src/client/customers` | Perfil, direcciones y favoritos | Digital Sales / Clientes | Destino funcional pendiente | `DECIDIR` | Separar identidad, party/cliente y preferencias de canal. |
| Mock API y pruebas de contratos | Contrato storefront | Digital Sales/API contracts | Platform shared contracts/tests | `MIGRAR` | Solo contratos confirmados. |
| Hosting Client/Landing | S3, CloudFront, certificados y DNS | Digital Sales | Infrastructure, dueño funcional correspondiente | `ADAPTAR` | No imponer dominios finales ni reutilizar IDs físicos. |

## 7. Finanzas, Compras y Reporting

| Origen | Propietario MasterERP | Destino | Acción | Nota |
| --- | --- | --- | --- | --- |
| `src/admin/finance`, `accounting`, `sales` | 2.2 Finanzas | Platform backend Finance | `ADAPTAR` | Verificar funcionalidad real frente a stubs/adapters. |
| `src/modules/admin/finance/*` | Capacidades 2.2.1–2.2.4 | Carpetas financieras vigentes | `CONSOLIDAR` | Mapear invoicing, receivables, payables, cash y payments. |
| `src/admin/procurement`, `src/modules/admin/procurement` | 2.2.1.2 Gestión de compras | Carpetas de compras/proveedores | `ADAPTAR` | Preservar proveedores como maestro del dominio acordado. |
| `src/modules/admin/reporting/*` | 2.2.3 BI u otros propietarios | Pendiente por reporte | `DECIDIR` | Cada reporte consume contratos; no debe apropiarse de datos. |
| `src/modules/admin/catalog/taxes` | Finanzas/Administración fiscal | Pendiente | `DECIDIR` | El nombre técnico bajo catálogo no fija propietario. |

## 8. Documentación, pruebas y automatización

| Origen | Destino | Acción | Nota |
| --- | --- | --- | --- |
| `docs/flujo-operativo-pedidos.md` | Documentation product/operations | `MIGRAR` | Base funcional del primer corte. |
| `docs/pruebas-operaciones.md` | Documentation operations + pruebas Platform | `ADAPTAR` | Convertir evidencia por contrato MasterERP. |
| `integrations/*` | Documentation architecture/integrations | `ADAPTAR` | Separar decisiones vigentes de detalles Tu Mandadito. |
| `tumandaito-database/docs/*` | Database documentation y arquitectura | `ADAPTAR` | Conservar access patterns como insumo histórico. |
| Tests backend/frontend | Repositorio propietario del código migrado | `MIGRAR/ADAPTAR` | Mantener intención; cambiar fixtures y persistencia. |
| Workflows y scripts DEV | Repo propietario | `CONSOLIDAR` | Seleccionar; no copiar en bloque. |

## 9. Mapa de dependencias

```text
Plataforma Base / Seguridad
  └── habilita todos los módulos mediante identidad, tenant y permisos

Catálogo
  ├── Productos
  ├── Categorías
  └── Inventario

Digital Sales
  ├── consume Catálogo
  ├── consume Inventario
  ├── consume Pagos
  └── crea Pedidos

Operaciones
  ├── Pedidos
  │   ├── consume Catálogo/Inventario
  │   └── alimenta Preparación
  ├── Preparación
  ├── Nodos
  └── Rutas
      └── asigna Repartidores de Transporte

Transporte
  └── Repartidores consumen Rutas y Pedidos autorizados
```

## 10. Orden aprobado para planificación

| Ola | Capacidad | Motivo |
| ---: | --- | --- |
| 0 | Seguridad/contexto mínimo transversal | Ninguna API nueva puede confiar solo en UI o claims incompletos. |
| 1 | Pedidos | Prioridad operativa y centro de dependencias. |
| 2 | Catálogo/Productos | Fuente canónica para canales y pedidos. |
| 3 | Inventario | Reservas y disponibilidad dependen del catálogo y afectan pedidos. |
| 4 | Nodos | Contexto operativo para stock, preparación y distribución. |
| 5 | Repartidores | Consolidación del módulo externo en Transporte. |
| 6 | Preparación y Rutas | Completa el flujo operativo extremo a extremo. |
| 7 | Storefront/Digital Sales | Cambia de APIs legacy a capacidades canónicas. |
| 8 | Pagos, Finanzas y capacidades restantes | Se planifican con contratos funcionales específicos. |

## 11. Criterios de salida de la Fase 2

- [x] Matriz origen → capacidad → destino → acción creada.
- [x] Dependencias entre capacidades documentadas.
- [x] Propietarios funcionales y de datos asignados o marcados explícitamente como decisión pendiente.
- [x] Orden de migración y cortes verticales confirmado para planificación.

La Fase 3 deberá convertir la Ola 1 (Pedidos) en contratos concretos antes de implementar.
