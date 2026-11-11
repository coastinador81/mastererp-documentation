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

Crear exige `Idempotency-Key`; asignación y transiciones usan versión
optimista. Una ruta no se completa con paradas pendientes.

## Ejecución por el repartidor

El usuario debe estar vinculado a un perfil `ACTIVE` con acceso `LINKED`, tener
`transport.deliveries.execute` y pertenecer al mismo tenant y empresa. Database
comprueba que la ruta esté asignada a ese perfil; la UI no puede indicar otro
repartidor.

La experiencia operativa vive en la aplicación independiente **Repartidores**,
con la misma sesión Cognito y selección autorizada de tenant y empresa que las
demás aplicaciones Uonify. El Storefront público no expone capacidades internas.
E-commerce conserva la planificación de rutas; Repartidores administra perfiles
y permite al conductor vinculado ejecutar exclusivamente su propia ruta.

`DELIVERED` exige latitud, longitud y precisión del dispositivo. Database
compara la evidencia con las coordenadas inmutables del pedido y rechaza
precisión mayor a 100 metros o distancia mayor a 250 metros. `FAILED` exige un
motivo y devuelve el pedido a `READY`, sin ruta, para que Operaciones lo
replanifique.

La última parada terminal cierra la ruta y libera al repartidor en la misma
transacción. Repetir un resultado ya aplicado devuelve éxito idempotente. Cada
resultado queda auditado con actor, pedido, ruta y evidencia; Operaciones
consulta el resultado, pero no compone el cierre desde el frontend.
