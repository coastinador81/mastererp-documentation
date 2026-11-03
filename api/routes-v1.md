# Rutas API v1

Candidato no desplegado. Rutas pertenece a Operaciones y solo referencia el
perfil de Repartidores, propiedad de Transporte.

| Método | Ruta | Uso |
| --- | --- | --- |
| GET | `/v1/companies/{company_id}/routes[/{route_id}]` | Lista/detalle |
| POST | `/v1/companies/{company_id}/routes` | Crear con pedidos `READY` |
| PUT | `/v1/companies/{company_id}/routes/{route_id}/driver` | Asignar repartidor disponible |
| POST | `/v1/companies/{company_id}/routes/{route_id}/transitions` | Iniciar/completar/cancelar |
| POST | `/v1/companies/{company_id}/routes/{route_id}/stops/{order_id}` | `DELIVERED` o `FAILED` |

Crear exige `Idempotency-Key`; asignación y transiciones usan versión
optimista. Una ruta no se completa con paradas pendientes.
