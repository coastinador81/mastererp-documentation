# Rutas API v1

Rutas pertenece a Operaciones y solo referencia el perfil de Repartidores,
propiedad de Transporte. El contrato de ejecución del repartidor está
implementado como candidato y permanece pendiente de despliegue.

| Método | Ruta | Uso |
| --- | --- | --- |
| GET | `/v1/companies/{company_id}/routes[/{route_id}]` | Lista/detalle |
| POST | `/v1/companies/{company_id}/routes` | Crear con pedidos `READY` |
| PUT | `/v1/companies/{company_id}/routes/{route_id}/driver` | Asignar repartidor disponible |
| POST | `/v1/companies/{company_id}/routes/{route_id}/transitions` | Iniciar/completar/cancelar |
| POST | `/v1/companies/{company_id}/routes/{route_id}/stops/{order_id}` | `DELIVERED` o `FAILED` |
| GET | `/v1/companies/{company_id}/driver-route` | Ruta activa del repartidor autenticado |
| POST | `/v1/companies/{company_id}/routes/{route_id}/driver-deliveries/{order_id}` | Resultado de una parada propia |
| GET | `/v1/storefront/driver-route?domain={domain}` | Ruta propia desde Storefront |
| POST | `/v1/storefront/routes/{route_id}/driver-deliveries/{order_id}?domain={domain}` | Resultado propio desde Storefront |

Crear exige `Idempotency-Key`; asignación y transiciones usan versión
optimista. Una ruta no se completa con paradas pendientes.

## Ejecución por el repartidor

El usuario debe estar vinculado a un perfil `ACTIVE` con acceso `LINKED`, tener
`transport.deliveries.execute` y pertenecer al mismo tenant y empresa. Database
comprueba que la ruta esté asignada a ese perfil; la UI no puede indicar otro
repartidor.

La experiencia operativa vive en el Storefront autenticado como **Mis rutas**,
junto al carrito. El dominio resuelve internamente la tienda y empresa; el
cliente no selecciona `company_id`. El Admin de E-commerce conserva únicamente
planificación y consulta operativa.

`DELIVERED` exige latitud, longitud y precisión del dispositivo. Database
compara la evidencia con las coordenadas inmutables del pedido y rechaza
precisión mayor a 100 metros o distancia mayor a 250 metros. `FAILED` exige un
motivo y devuelve el pedido a `READY`, sin ruta, para que Operaciones lo
replanifique.

La última parada terminal cierra la ruta y libera al repartidor en la misma
transacción. Repetir un resultado ya aplicado devuelve éxito idempotente. Cada
resultado queda auditado con actor, pedido, ruta y evidencia; Operaciones
consulta el resultado, pero no compone el cierre desde el frontend.
