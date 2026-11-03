# Inventario técnico de Tu Mandadito

Estado: inventario inicial revisado y aprobado para continuar a la matriz de migración.

Fecha de corte: 2026-11-03. Este documento registra evidencia local de los repositorios actuales. No declara que una carpeta esté desplegada ni modifica la fuente de verdad.

## 1. Alcance y método

Se inspeccionaron estructura, estado Git, manifiestos, aplicaciones, handlers, pruebas, plantillas AWS, workflows, documentación de datos y cableado del módulo Repartidores. Se excluyeron dependencias generadas (`node_modules`, `.next`, `dist`, `build`) y no se consultó ni modificó AWS.

Clasificación usada:

| Acción | Significado |
| --- | --- |
| `MIGRAR` | La responsabilidad encaja directamente en un módulo MasterERP. |
| `ADAPTAR` | La capacidad se conserva, pero requiere contrato, multi-tenancy o modelo nuevo. |
| `CONSOLIDAR` | Existe separada o duplicada y debe incorporarse al propietario MasterERP. |
| `MANTENER` | Sigue operando en Tu Mandadito durante la transición. |
| `RETIRAR` | Candidato a eliminación solo después de validar consumidores y completar el corte. |
| `DECIDIR` | Falta una decisión funcional o técnica que cambia el destino. |

## 2. Estado de repositorios

| Repositorio | Rama observada | Archivos rastreados aprox. | Estado local relevante |
| --- | --- | ---: | --- |
| `tumandaito-backend` | `dev` | 313 | Limpio. |
| `tumandaito-frontends` | `dev` | 821 | Contiene `Mejorar experiencia TuMandaito (copia)/` no versionado; queda fuera de la migración hasta decisión explícita. |
| `tumandaito-infrastructure` | `dev` | 98 | Limpio; conserva artefactos `.aws-sam`. |
| `tumandaito-database` | `dev` | 10 | Limpio; documentación DynamoDB, no migraciones ejecutables. |
| `tumandaito-documentation` | `dev` | 14 | Tiene documentación local no versionada de MasterERP y módulos; preservar. |
| `tumandaito-repartidores-backend` | `main` | 75 | Limpio; submódulo del workspace. |
| `tumandaito-repartidores-frontends` | `main` | 42 | Limpio; submódulo del workspace. |
| `tumandaito-repartidores-infrastructure` | `main` | 2 | Limpio; submódulo del workspace. |
| `tumandaito-repartidores-database` | `main` | 3 | Limpio; submódulo del workspace. |

Advertencias:

- Los repositorios de Repartidores no tienen la rama de trabajo `dev` activa; no se cambiará su rama como efecto del inventario.
- Los archivos locales no versionados pertenecen al usuario y no deben borrarse, sobrescribirse ni incorporarse automáticamente.
- El workspace actual referencia los cuatro repositorios de Repartidores como submódulos.

## 3. Backend principal

Ubicación: `tumandaito-backend`.

### 3.1 Métricas y estructura

- 241 archivos Python bajo `src/`.
- 114 archivos con `lambda_handler`.
- 46 archivos de pruebas `test_*.py`.
- Scripts de seed, backfill, reconciliación, diagnóstico y despliegue directo a DEV.
- No existe manifiesto de dependencias Python en el repositorio observado.

Hay dos organizaciones coexistentes:

1. Handlers históricos por canal y endpoint bajo `src/admin/` y `src/client/`.
2. Adapters modulares bajo `src/modules/`, con handlers por capacidad.

Esto puede representar wrappers, evolución parcial o duplicación. La migración deberá rastrear cada endpoint hasta su servicio/repositorio real antes de elegir código fuente.

### 3.2 Capacidades observadas

| Capacidad actual | Evidencia principal | Destino preliminar | Acción |
| --- | --- | --- | --- |
| Pedidos cliente y Admin | `src/client/orders`, `src/admin/orders`, `src/modules/admin/operations/orders` | `2.3-operations/2.3.1-orders` | `ADAPTAR` |
| Preparación | `src/modules/admin/operations/preparation`, repositorio compartido | `2.3-operations/2.3.2-preparation` | `ADAPTAR` |
| Rutas y definiciones | `src/admin/routes`, `src/admin/route_definitions`, `src/modules/admin/logistics/routes` | `2.3-operations/2.3.3-routes` | `ADAPTAR` |
| Nodos | `src/admin/nodes`, `src/modules/admin/nodes` | `2.3-operations/2.3.4-nodes` | `ADAPTAR` |
| Productos y categorías | `src/admin/products`, `src/client/catalog`, `src/modules/admin/catalog` | `2.1-catalog` | `ADAPTAR` |
| Inventario | `src/admin/inventory`, `src/modules/admin/inventory` | `2.1-catalog/2.1.6-inventory` | `ADAPTAR` |
| Checkout y tienda | `src/client/checkout`, `src/client/store` | `2.4-digital-sales/2.4.2-ecommerce` | `ADAPTAR` |
| Perfil, direcciones y favoritos | `src/client/customers` | Digital Sales y/o Administración Clientes | `DECIDIR` |
| Autenticación cliente | `src/client/auth`, `src/modules/workspace/auth` | Plataforma Base y Seguridad | `ADAPTAR` |
| Finanzas, ventas y contabilidad | `src/admin/finance`, `sales`, `accounting`, `src/modules/admin/finance` | `2.2-finance` | `ADAPTAR` tras verificar implementación real |
| Compras y proveedores | `src/admin/procurement`, `src/modules/admin/procurement` | `2.2.1-administration/gestion-compras` | `ADAPTAR` |
| Reporting | `src/modules/admin/reporting` | BI y reportes propietarios | `DECIDIR` |
| Asistente Admin | `src/modules/admin/assistant` | Uonify/Centro de Agentes | `DECIDIR` |
| Proyección de pedidos | `src/projections/orders` | Operaciones, integración técnica | `ADAPTAR` |

### 3.3 Código transversal

`src/shared/` contiene autorización administrativa, claims JWT, HTTP, claves y repositorios de catálogo, clientes, inventario, nodos, pedidos, preparación, rutas y repartidores. No debe copiarse como un paquete único: cada pieza debe asignarse al módulo propietario o a seguridad compartida.

Riesgos:

- Dependencia directa del esquema de claves DynamoDB.
- Responsabilidades de dominio concentradas en `shared`.
- Posible duplicación entre handlers históricos y modulares.
- Scripts DEV capaces de modificar datos o código desplegado.

## 4. Módulo Repartidores

### 4.1 Backend

Ubicación: `tumandaito-repartidores-backend`.

- 63 archivos Python.
- 13 handlers Lambda.
- 9 pruebas.
- CRUD Admin de repartidores, API de ruta/pedidos/entrega e incidencias.
- `src/shared/` es un snapshot de dependencias de la plataforma, no una biblioteca compartida versionada.

Destino: `2.10-transport/2.10.1-drivers` para la entidad y operación del repartidor. Pedidos continúan bajo Operaciones; una entrega o asignación deberá consumir contratos del módulo propietario.

Acción: `CONSOLIDAR`. Preservar comportamiento, pero eliminar gradualmente la copia de dependencias compartidas mediante contratos explícitos.

### 4.2 Frontends

Ubicación: `tumandaito-repartidores-frontends`.

- Admin: listado, alta, detalle, nodos, rutas e incidencias.
- Portal web legacy del repartidor: shell, pedidos, detalle y ruta.
- Expo: listado y detalle de pedidos del repartidor.

La plataforma consume estas pantallas mediante rutas delgadas y enlaces. Destino preliminar:

- Administración de repartidores → frontend `2.10.1-drivers`.
- Ruta y pedidos del repartidor → experiencia de Transporte consumiendo Operaciones.
- Portal legacy y Expo → `DECIDIR` cuál canal sigue vigente antes de consolidar.

### 4.3 Infrastructure y Database

Los repositorios separados contienen principalmente wiring mínimo, workflow de driver y documentación vacía. No son destinos MasterERP independientes. Sus responsabilidades se incorporarán en `mastererp-infrastructure` y `mastererp-database` cuando existan recursos/modelos reales aprobados.

## 5. Frontends de plataforma

Ubicación: `tumandaito-frontends`.

| Aplicación | Tecnología observada | Archivos aprox. | Destino preliminar | Acción |
| --- | --- | ---: | --- | --- |
| Admin | Next.js/React | 367 | `frontend/modules` dentro del espacio Uonify | `ADAPTAR` |
| Client | Next.js/React | 122 | Digital Sales / storefront | `ADAPTAR` |
| Customer app | Expo/React Native | 172 | Canal móvil por decidir en arquitectura MasterERP | `DECIDIR` |
| Landing | Next.js/React | 52 | Website/Digital Sales | `ADAPTAR` |
| Raíz legacy | Next.js/React | No separar aún | Posible antecesor/duplicado | `DECIDIR` |
| Mock API | Node contract tests | Pequeño | Pruebas de contrato | `MIGRAR` si siguen vigentes |

Dependencias destacadas de customer app: Expo Router, Auth Session, Location, Maps, AsyncStorage y WebBrowser. Estas capacidades no deben asumirse disponibles en el frontend web MasterERP.

Riesgos:

- Varias raíces Next.js y una copia de diseño no versionada.
- Rutas delgadas/symlinks hacia el módulo Repartidores.
- Contratos duplicados en clientes API y mappers.
- Producción y DEV comparten repositorio pero tienen pipelines distintos.

## 6. Infraestructura

Ubicación: `tumandaito-infrastructure`.

### 6.1 Inventario observado

- 39 workflows bajo `.github/workflows`.
- 32 recursos `AWS::Serverless::Function` y 40 `AWS::Serverless::Application` declarados en plantillas.
- 5 REST APIs, 122 métodos y 92 recursos API Gateway según conteo estático.
- 2 tablas DynamoDB declaradas.
- Cognito: pools, clientes, dominio, grupos e IdP.
- S3, CloudFront, certificados y Route53 para frontends.
- Scripts de CORS, diagnóstico, migración de inventario, seed y validación de change sets.

Dominios de plantillas:

- Admin: catálogo, inventario, logística, nodos, operaciones, finanzas, compras, perfil y reporting.
- Client: autenticación, catálogo, checkout, clientes, pedidos y tienda.
- Driver.
- Foundation: Cognito, DynamoDB y administración.
- Frontends, certificados, proyectos, superadmin y workspace.

### 6.2 Clasificación

| Grupo | Destino | Acción |
| --- | --- | --- |
| Lambdas y rutas funcionales | `mastererp-infrastructure/modules/<propietario>/resources.yml` | `ADAPTAR` |
| API Gateway compartidos | Propiedad compartida existente de MasterERP | `ADAPTAR`, sin recrear APIs |
| DynamoDB | No se traslada como modelo canónico PostgreSQL | `MANTENER` durante convivencia; luego `RETIRAR` |
| Cognito | Plataforma Base/Seguridad | `ADAPTAR` tras comparar pools, issuer y clientes |
| Hosting Admin/Client/Landing | Frontend/Digital Sales | `DECIDIR` convivencia y dominios |
| Workflows y scripts | CI/CD por repo propietario | `CONSOLIDAR`; no copiar los 39 sin depuración |

Riesgo crítico: MasterERP usa recursos existentes en `us-west-1`; Tu Mandadito documenta DEV en `us-west-2`. Ningún ARN, ID físico, stack, dominio o plantilla puede trasladarse automáticamente.

## 7. Datos

Ubicación: `tumandaito-database` y código de repositorios en backend.

### 7.1 Modelo observado

- Tabla core multi-tenant con `PK` y `SK`.
- GSI1: pedidos por tenant.
- GSI2: pedidos por estado.
- GSI3: pedidos por ruta.
- GSI4: repartidores por tenant.
- GSI5: rutas por tenant.
- GSI6: ruta activa de repartidor.
- Particiones para producto/inventario/movimientos, clientes/direcciones/punteros de pedidos, nodos, rutas y repartidores.
- Uso previsto de transacciones para inventario, asignaciones y cambios operativos.
- `Scan` está prohibido en patrones normales, aunque la documentación reconoce una excepción temporal de catálogo.

### 7.2 Acción

`ADAPTAR`. MasterERP usa PostgreSQL/RDS y autorización por contexto transaccional/RLS. Las claves DynamoDB y GSIs son evidencia de access patterns, no el diseño del esquema destino.

Antes de migrar datos se requiere:

- Traducir entidades, invariantes y consultas a modelo relacional.
- Conservar IDs externos cuando soporten trazabilidad.
- Definir tenant y empresa explícitamente.
- Diseñar idempotencia, auditoría y reconciliación.
- Crear migraciones repetibles y pruebas de aislamiento.

## 8. Documentación y QA

Ubicación: `tumandaito-documentation` y documentación distribuida.

Activos relevantes:

- Flujo operativo completo de pedidos, pagos, preparación, rutas, repartidores y nodos.
- Plan de pruebas de Operaciones con happy path y reglas críticas.
- Integraciones de Cognito/Google, Maps, notificaciones y tiempo real.
- Guías de cableado del módulo Repartidores.
- Documentación DynamoDB en `tumandaito-database`.

Acción: `MIGRAR` y `ADAPTAR` a `mastererp-documentation`, preservando evidencia histórica y actualizando nombres/propietarios. Las guías que prescriben cuatro repositorios por módulo quedarán históricas cuando se complete la consolidación.

## 9. Dependencias funcionales observadas

```text
Digital Sales / Customer
  ├── Catálogo y Productos
  ├── Inventario
  ├── Checkout y Pagos
  └── Pedidos

Pedidos
  ├── Clientes y direcciones
  ├── Catálogo / precios
  ├── Inventario / reservas
  ├── Preparación
  ├── Nodos
  ├── Rutas
  └── Repartidores / entrega

Transporte / Repartidores
  ├── Rutas (Operaciones)
  ├── Pedidos (Operaciones)
  ├── Nodos (Operaciones)
  └── Identidad y autorización
```

## 10. Duplicaciones y transiciones que deben resolverse

1. Handlers históricos versus handlers bajo `src/modules`.
2. `src/shared` del backend versus snapshot en Repartidores.
3. Portal web del repartidor versus experiencia Expo.
4. Next.js raíz versus `frontends/admin` y otros proyectos internos.
5. Infraestructura platform versus repositorio mínimo de Repartidores.
6. Documentación rastreada versus archivos locales no versionados.
7. Autorización actual basada en claims/guards versus contrato MasterERP de actor, tenant, empresa y permisos.

## 11. Decisiones pendientes para la matriz de migración

- Ubicación funcional de clientes e identidad comercial del storefront.
- Canal futuro del repartidor: web, Expo o ambos.
- Alcance de customer app dentro de la arquitectura frontend MasterERP.
- Tratamiento de impuestos, pagos y reporting entre Catálogo, Finanzas y Digital Sales.
- Qué handlers modulares sustituyen realmente a handlers históricos.
- Estrategia de convivencia DynamoDB/PostgreSQL por primera capacidad.
- Destino de dominios y hosting actuales.

## 12. Criterios de salida de la Fase 1

- [x] Frontends inventariados.
- [x] Backend y handlers inventariados.
- [x] Infraestructura y workflows inventariados.
- [x] Entidades, claves e índices inventariados.
- [x] Documentación y QA inventariados.
- [x] Componentes clasificados de forma preliminar.
- [x] Dependencias, duplicaciones, riesgos y decisiones pendientes documentados.
- [x] Inventario revisado y aprobado por el Product Owner.
