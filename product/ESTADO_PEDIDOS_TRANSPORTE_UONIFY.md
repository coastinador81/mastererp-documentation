# Estado consolidado: Pedidos, Transporte y experiencia Uonify

Fecha de corte: 18 de noviembre de 2026.
Rama de trabajo: `dev` en los cuatro repositorios MasterERP.

Este documento separa deliberadamente lo **publicado en DEV**, lo
**implementado y validado solo en el workspace** y lo **pendiente**. La
existencia de código o documentación local no acredita que una capacidad esté
desplegada.

## Resumen ejecutivo

MasterERP se está consolidando como un solo producto modular: `dev.uonify.com`
es la entrada principal y Finanzas, Ecommerce, Transporte, Nodos y las demás
aplicaciones son superficies conectadas con autenticación común, autorización
real del backend y ciclos de publicación independientes.

Pedidos ya dispone del núcleo operativo, contratos autorizados, historial y
notas persistentes, cancelación coordinada, directorio paginado desde backend,
acciones agrupadas transaccionales y lectura del estado financiero. Transporte
y Repartidores ya tienen contratos y aplicación independiente candidata, pero
`transporte-dev.uonify.com` no está aprovisionado: el workflow llegó hasta la
creación de infraestructura y falló porque el usuario de GitHub Actions no
tiene permiso de `cloudformation:DescribeStacks` en `us-east-1`.

La convergencia visual también está avanzada localmente: existen tokens
corporativos y una base administrativa compartida que eliminan la dependencia
de Ecommerce y Transporte sobre `finance.css`. Su publicación está retenida
por ejecuciones huérfanas de GitHub Actions, no por un fallo conocido de build.

## Estado por frente

| Frente | Publicado o comprobado en DEV | Implementado/validado localmente | Falta para cierre |
| --- | --- | --- | --- |
| Pedidos | Flujo operativo y APIs existentes; checkout crea pedidos; navegación Pedido → Ruta; estado financiero visible; comprobante operativo PDF; devoluciones administrativas hasta recepción e inventario | Contrato de devoluciones separado, autorización por permisos y conciliación financiera endurecida | Smoke autenticado del ciclo de devolución; documentos fiscales y nota de crédito pertenecen a Finanzas |
| Devoluciones | Solicitud, decisión, recepción, reintegro apto a inventario, expediente e historial desplegados en DEV | El cierre exige referencias verificables de Tesorería y nota de crédito del mismo tenant, empresa, cliente y pedido | Finanzas debe emitir/vincular los registros fiscales y monetarios reales para pasar de `RECEIVED` a `COMPLETED` |
| Notas internas | Contrato y persistencia real implementados | Documentación alineada | Smoke final con identidades autorizadas después de publicar |
| Cancelación | Cancelación coordinada de pedido, reserva y venta implementada con autorización y auditoría | Cobertura y documentación alineadas | Validación E2E del artefacto publicado, incluidos conflictos y compensaciones |
| Historial | Historial real de transiciones expuesto por backend y mostrado en la ficha | Lectura autorizada y estados visuales alineados | Validación E2E de actor, fecha, motivo y orden cronológico |
| Pagos | Finanzas/Tesorería valida o rechaza; Pedidos solo lee el resultado | Textos y contrato alineados: espera, validado, rechazado y pendiente de cobro | Finanzas es propietaria del mantenimiento posterior; Pedidos solo requiere probar la integración publicada |
| Rutas | Operaciones planifica; Pedido abre su ruta; listado ordenado por fecha | Contrato de ejecución del repartidor consolidado | Smoke de inicio, incidencias, entrega y cierre automático de ruta |
| Repartidores | CRUD, ficha, nodos, vehículo, licencia, rutas y entregas; backend restringe ejecución a perfil vinculado | Aplicación independiente Transporte/Repartidores compila y consume contratos reales | Aprovisionar hosting autorizado, publicar y validar “Mi ruta” con un conductor real |
| Transporte | Código fuente en Platform y recursos/pipeline separados diseñados | Build y pruebas focalizadas correctas; comparte header y base administrativa | Crear rol de aprovisionamiento con mínimo privilegio; completar ACM/CloudFront/S3/DNS por workflow; no hacerlo manualmente |
| Sistema visual | Finanzas sirve como referencia administrativa y Uonify como identidad principal | `design-tokens.css`, `admin-application.css`, `ApplicationHeader`, estados y carga compartidos | Publicar Ecommerce; revisión visual escritorio/móvil; terminar sustitución gradual de valores locales |
| Uonify | `dev.uonify.com` es entrada principal con cuenta, espacios, chat y lanzador | Arquitectura y documentación alineadas con aplicaciones por subdominio | Validar continuidad SSO real entre todos los subdominios y conectar lanzadores solo cuando cada app esté disponible |
| Nodos | Capacidad existente dentro del entorno administrativo | Identificado como futura aplicación independiente y visión de sucursal | Definir dominio DEV, límites de la app, pipeline y experiencia sin duplicar Inventario, Pedidos o Rutas |

## Pedidos: alcance cerrado y pendientes reales

### Ya resuelto

- Directorio con búsqueda, filtros y paginación del servidor, sin limitar la
  experiencia a los primeros 100 registros cargados localmente.
- Orden cronológico descendente en listados operativos.
- Ficha con cliente, entrega, pago, productos, totales, progreso e historial.
- Notas internas persistentes por API; no dependen del almacenamiento local del
  navegador.
- Cancelación coordinada y auditable de pedido, reserva y venta.
- Acciones agrupadas mediante un comando transaccional: se aplican todas o
  ninguna.
- La interfaz niega escritura cuando `runtime.permissions` no está disponible;
  no muestra controles por una suposición permisiva.
- El estado financiero es de solo lectura. Operaciones no puede validar,
  rechazar ni inventar pagos.
- Pagos electrónicos usan `PENDING_VERIFICATION`, `VALIDATED` y `REJECTED`;
  efectivo o punto de venta conserva `PENDING_COLLECTION`.
- Navegación directa desde un pedido hasta su ruta cuando existe `route_ref`.
- Comprobante de entrega operativo en PDF, derivado de evidencia real y rotulado
  expresamente como no fiscal.
- Devoluciones administrativas parciales con solicitud, autorización o rechazo,
  recepción, condición de mercancía, reintegro controlado a Inventario e
  historial inmutable.
- La ficha ordena Productos, Historial, Devoluciones e Intentos; las tres
  cronologías operativas comparten un acordeón inicialmente cerrado.
- Los pedidos nuevos preservan en su snapshot nombre, correo y teléfono
  disponibles dentro del mismo tenant y empresa.

### Responsabilidad entre Operaciones y Finanzas

Finanzas → Administración → Tesorería → Cobros es la única propietaria de la
revisión financiera. Debe consultar, validar o rechazar, registrar actor, fecha,
motivo y evidencia, y coordinar la creación de la venta y confirmación del
pedido cuando corresponda.

Pedidos solo muestra:

- “En espera de validación de Finanzas” para `PENDING_VERIFICATION`.
- “Pago validado por Finanzas” para `VALIDATED`.
- “Pago rechazado por Finanzas” para `REJECTED`.
- “Pendiente de cobro” para efectivo o punto de venta.

No ofrece “Confirmar pedido” cuando el pago requiere revisión financiera.

### Pendientes fuera del cierre operativo básico

- Emitir documentos fiscales reales. La descarga actual debe continuar
  rotulada “NO ES UNA FACTURA FISCAL” hasta integrar el propietario fiscal.
- Integrar desde Finanzas la factura y la nota de crédito fiscal reales. Pedidos
  no las fabrica ni acepta referencias financieras libres.
- Ejecutar un smoke autenticado completo de devolución con las identidades de
  Operaciones, Almacén y Finanzas.
- Realizar smoke autenticado en DEV del corte local después de su publicación.

### Devoluciones: contrato publicado en DEV

El flujo es administrativo y está aislado por `tenant_id + company_id`:

1. Un usuario con permiso solicita cantidades de un pedido entregado.
2. Un usuario autorizado aprueba o rechaza el expediente.
3. Almacén registra recepción, condición, nodo y cantidad apta para reventa.
4. Inventario registra movimiento `RETURN` solo por unidades recibidas aptas.
5. Finanzas crea o vincula el egreso real y la nota de crédito correspondiente.
6. El expediente pasa a `COMPLETED` únicamente cuando ambos registros
   financieros han sido verificados en la misma empresa, cliente y pedido.

Estados: `REQUESTED`, `AUTHORIZED`, `REJECTED`, `RECEIVED` y `COMPLETED`.
Todos los cambios generan eventos inmutables con actor, fecha y correlación, y
se incorporan a la cronología consolidada del pedido.
La interfaz de Pedidos cubre los pasos 1 a 4 y muestra `Pendiente de Finanzas`
después de la recepción; no expone campos para inventar identificadores de
reembolso o nota de crédito. La solicitud directa desde Storefront no forma
parte de esta etapa.

La interfaz descuenta de cada línea las cantidades comprometidas en expedientes
no rechazados. Sólo ofrece el remanente y oculta «Nueva devolución» cuando el
pedido ya no contiene unidades retornables; el backend vuelve a validar la
regla para cubrir concurrencia y clientes desactualizados.

Pedidos, Rutas, Mi ruta y Repartidores comparten notificaciones flotantes,
visibles desde cualquier posición de scroll y persistentes hasta cierre manual.
Los códigos operativos conocidos, como `invalid_transition`, se presentan como
mensajes accionables para el usuario sin perder el código contractual del API.

## Transporte, Rutas y Repartidores

### Límites de propiedad

- **Operaciones/Rutas** posee la ruta, sus paradas, transiciones y cierre.
- **Pedidos** posee el pedido y conserva el vínculo a la ruta.
- **Transporte/Repartidores** posee el perfil, disponibilidad, licencia,
  vehículo, acceso y asociación del usuario conductor.
- La aplicación Repartidores consume Rutas y Pedidos mediante APIs autorizadas;
  no duplica sus entidades ni su lógica.

### Flujo del conductor

1. El usuario entra en la aplicación independiente Repartidores.
2. Cognito autentica y el backend resuelve un perfil `ACTIVE` y `LINKED`.
3. “Mi ruta” muestra exclusivamente la ruta asignada al conductor autenticado.
4. El conductor inicia su ruta, consulta paradas y pedidos, registra incidencias
   y confirma entregas con el contrato autorizado.
5. Una parada fallida queda auditada y devuelve el pedido a `READY`, sin ruta,
   para que Operaciones lo reprograme.
6. La última parada terminal completa la ruta y libera al repartidor dentro de
   la misma operación transaccional.

Los propietarios del tenant pueden administrar perfiles y rutas si sus permisos
reales lo permiten, pero no deben suplantar al conductor para ejecutar una
entrega.

### Estado de publicación

- Dominio canónico previsto: `transporte-dev.uonify.com`.
- El código de la aplicación y el workflow de publicación están preparados
  localmente.
- La aplicación compila y las pruebas focalizadas de acceso, ruta y Cognito
  pasaron localmente.
- El aprovisionamiento anterior falló antes de crear el certificado porque el
  principal de GitHub Actions carece de `cloudformation:DescribeStacks` en
  `us-east-1`.
- No se debe crear ACM, CloudFront, S3 o DNS manualmente. Hace falta un rol de
  aprovisionamiento mínimo y ejecutar el workflow autorizado.

## Uonify y sistema visual compartido

### Dirección vigente

`dev.uonify.com` es el producto principal. El chat, las vistas manuales y las
aplicaciones independientes utilizan los mismos contratos y permisos. El chat
no accede directamente a la base de datos y ningún menú concede autorización.

Cognito resuelve autenticación; cada API vuelve a validar tenant, empresa,
tienda, nodo, dominio y permiso. La sesión compartida entre subdominios debe
comprobarse como comportamiento desplegado, no presumirse por compartir marca.

### Fundamentos visuales

- Azul principal `#0178F7` y acción `#006AE1`.
- Aqua reservado para IA o estado en vivo, no como acción genérica.
- Neutros, estados semánticos, espaciado de 8 px, radios 4/8/12/16 y movimiento
  120/200/320 ms definidos como tokens compartidos.
- Inter es la tipografía administrativa y de Uonify; Pedidos conserva Arimo por
  el patrón aprobado de Ecommerce. Storefront y móvil pueden tener composición
  propia sin crear otra identidad.
- Tablas administrativas: 14 px regular, encabezado `#F4F7FB`, bordes
  `#E8EDF3`, fondo general `#F6F8FC` y superficies blancas.
- Header, estados, carga y base administrativa pertenecen a
  `frontend/shared/ui`; ningún módulo nuevo debe copiar `finance.css` ni
  importar internamente la aplicación Uonify.

### Trabajo local pendiente de publicar

- Tokens corporativos neutrales en `frontend/shared/ui/design-tokens.css`.
- Base administrativa compartida en `frontend/shared/ui/admin-application.css`.
- Header común consumido por Finanzas, Ecommerce y Transporte.
- Ecommerce y Transporte separados del archivo histórico `finance.css`.
- Pedidos, `StatusBadge` y `LoadingState` migrados parcialmente a `--uon-*`.
- Workflows de Finanzas/Ecommerce preparados para observar cambios en UI
  compartida; workflow independiente de Transporte preparado localmente.

Falta una revisión visual autenticada de Ecommerce en escritorio y móvil para
confirmar que no quede una regla local sobreescribiendo el header compartido.

## Bloqueo operativo de GitHub Actions

La publicación se mantiene detenida por 12 ejecuciones huérfanas creadas durante
un incidente de GitHub Actions el 11 de noviembre de 2026: diez en Platform y
dos en Database. No tienen jobs, siguen como `queued`, la cancelación forzada
devuelve HTTP 409 y la eliminación devuelve HTTP 403. El presupuesto de Actions
no está agotado.

Este bloqueo debe escalarse a GitHub Support. No se recomienda generar pushes
solo para intentar liberar la cola. Hasta resolverlo, los cambios locales deben
permanecer sin publicar y conservarse sin sobrescribir trabajo existente.

## Orden recomendado para continuar

1. Resolver o retirar las ejecuciones huérfanas con GitHub Support.
2. Revisar el diff acumulado y volver a ejecutar las pruebas focalizadas.
3. Publicar Ecommerce y validar Pedidos y el header compartido con una identidad
   autorizada.
4. Crear el rol mínimo de aprovisionamiento de Transporte y ejecutar su workflow
   autorizado.
5. Probar “Mi ruta” de extremo a extremo: inicio, incidencia, entrega, cierre de
   ruta y liberación del repartidor.
6. Probar en DEV el ciclo administrativo de devolución y conectar desde
   Finanzas la emisión/vinculación real del reembolso y nota de crédito.
7. Diseñar Nodos como aplicación independiente con visión de sucursal, sin
   duplicar capacidades canónicas.
8. Preparar una fuente neutral de tokens para web y móvil, generando las
   representaciones de cada canal en lugar de mantener paletas manuales.

## Evidencia y referencias

- [Producto Uonify conectado](../architecture/PRODUCTO_UONIFY_CONECTADO.md)
- [Contrato de Pedidos](../api/orders-v1.md)
- [Contrato de Rutas](../api/routes-v1.md)
- [Contrato de Repartidores](../api/drivers-v1.md)
- [Validación financiera](FLUJO_VALIDACION_PAGOS_ECOMMERCE.md)
- `mastererp-platform/frontend/modules/2.3-ecommerce/2.3.1-orders/README.md`
- `mastererp-platform/frontend/modules/2.10-transport/README.md`
