# Migración de Tu Mandadito a MasterERP

Estado: Fase 7 cerrada; Fase 8 en curso, sin cutover todavía.

Este directorio conserva el plan, las decisiones y la evidencia de la migración gradual de Tu Mandadito hacia MasterERP. El [tablero de seguimiento](index.html) ofrece el checklist local; este documento define los acuerdos versionados.

Cierres: [Fase 3](phase-3-closure.md), [Fase 4](phase-4-closure.md) y
[Fase 5](phase-5-closure.md).

Trabajo actual: [Fase 8 — cierre de capacidad](phase-8-progress.md).
Evidencia DEV actual: [smoke autenticado de Fase 8](phase-8-dev-evidence.md).
Primera oleada: [Categorías y Productos](phase-6-wave-1-catalog.md).
Segunda oleada: [Nodos e Inventario](phase-6-wave-2-nodes-inventory.md).
Tercera oleada: [Repartidores](phase-6-wave-3-drivers.md).

## Workspace y repositorios oficiales

El workspace local se encuentra en `/home/andrea/proyectos/mastererp` y contiene únicamente estos repositorios de producto:

| Repositorio | Responsabilidad |
| --- | --- |
| `mastererp-platform` | Frontend, backend, módulos y código compartido. |
| `mastererp-infrastructure` | Recursos funcionales AWS, APIs y configuración de despliegue. |
| `mastererp-database` | RDS, SQL, migraciones, seeds y pruebas de datos. |
| `mastererp-documentation` | Producto, arquitectura, contratos y operación. |

La rama de trabajo es `dev` en los cuatro repositorios. No se crean repositorios por tenant, cliente, módulo o submódulo.

## Fuentes de verdad durante la transición

La migración es incremental y se controla por capacidad funcional:

1. Mientras una capacidad no haya completado su corte, sus repositorios actuales de Tu Mandadito siguen siendo la fuente de verdad de código, operación y datos.
2. Crear una carpeta, README, contrato, adaptador o implementación parcial en MasterERP no cambia por sí solo la fuente de verdad.
3. MasterERP se convierte en fuente de verdad de una capacidad únicamente después de validar contratos, autorización, aislamiento multi-tenant, datos, pruebas, infraestructura, rollback y aprobación del corte.
4. El cambio de fuente de verdad debe quedar registrado en la documentación de la capacidad y en el inventario/matriz de migración.
5. El código y los recursos anteriores solo se retiran después del corte aprobado y de comprobar que no existen consumidores activos.

## Reglas de trabajo

- Leer el `AGENTS.md`, README padres y documentos obligatorios del repositorio afectado antes de editar.
- Mantener rutas funcionales equivalentes en Platform frontend/backend, Infrastructure y Database.
- Migrar por capacidades y cortes verticales; no copiar repositorios completos dentro de MasterERP.
- Conservar el lenguaje y framework reales cuando no exista una transición aprobada.
- Definir una sola propiedad por dato y recurso. Los recursos compartidos pertenecen al padre funcional común más cercano.
- No interpretar un directorio, README o `resources.yml` vacío como funcionalidad implementada.
- No copiar identificadores, secretos, nombres físicos o configuración AWS de Tu Mandadito a MasterERP sin análisis explícito.
- No ejecutar SQL, despliegues, cambios DNS, eliminaciones, commits o push como consecuencia implícita del inventario.
- Mantener la operación manual de cada módulo aunque posteriormente exista una interfaz de IA.
- Los agentes acceden mediante APIs o herramientas MCP autorizadas del módulo propietario; nunca directamente a la base de datos.

## Diferencias que requieren control explícito

| Tema | Tu Mandadito actual | MasterERP vigente | Regla de migración |
| --- | --- | --- | --- |
| Datos | Principalmente DynamoDB | PostgreSQL/RDS con aislamiento multi-tenant | Diseñar modelo y migración; no traducir claves literalmente. |
| Región DEV | `us-west-2` según la documentación actual | `us-west-1` según los repositorios MasterERP | No reutilizar recursos o ARN automáticamente. |
| Organización | Repositorios de plataforma y Repartidores separados | Cuatro repositorios con módulos internos | Consolidar por capacidad, no por copia física. |
| Autorización | Contratos actuales de Tu Mandadito | Cognito autentica; backend/PostgreSQL autorizan | Cada API revalida actor, tenant, empresa y permisos. |
| Frontend | Admin, aplicaciones y rutas existentes | Espacio común Uonify con integración progresiva | No imponer mecanismo de composición sin diseño aprobado. |

## Criterios de salida de la Fase 0

- [x] Los cuatro repositorios oficiales están disponibles en el workspace general.
- [x] Los cuatro repositorios usan la rama `dev` y siguen su remoto correspondiente.
- [x] Se revisaron las instrucciones, README, arquitectura, estándares y recursos relevantes.
- [x] Se documentó qué sistema conserva la fuente de verdad durante la transición.
- [x] Se fijaron reglas para evitar despliegues, movimientos o retiros implícitos.
- [x] El tablero de seguimiento está disponible y persiste sus checks localmente.

## Próxima fase

La Fase 1 generará un inventario verificable de los repositorios actuales de Tu Mandadito. Cada elemento se clasificará como `migrar`, `adaptar`, `reemplazar`, `mantener temporalmente`, `retirar` o `decisión pendiente`, sin mover código todavía.

Documento de trabajo: [inventario técnico de Tu Mandadito](tumandaito-inventory.md).

Matriz resultante: [mapeo de capacidades y destinos](capability-mapping.md).

Primer contrato vertical: [Pedidos v1](../../api/orders-v1.md).

Cierre aprobado: [Fase 3 — Contratos y límites](phase-3-closure.md).

Implementación en curso: [progreso de Fase 4](phase-4-progress.md).
