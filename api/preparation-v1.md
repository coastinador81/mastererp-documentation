# Preparación API v1

Candidato no desplegado. JWT y `X-Tenant-Id` son obligatorios; la API vuelve a
autorizar tenant, empresa, nodo, módulo y permisos.

| Método | Ruta | Uso |
| --- | --- | --- |
| GET | `/v1/companies/{company_id}/preparations` | Listar |
| GET | `/v1/companies/{company_id}/orders/{order_id}/preparation` | Detalle |
| POST | `/v1/companies/{company_id}/orders/{order_id}/preparation/start` | Iniciar pedido `CONFIRMED` |
| PATCH | `/v1/companies/{company_id}/orders/{order_id}/preparation/items/{line_id}` | Marcar `DONE`/`PENDING` |
| POST | `/v1/companies/{company_id}/orders/{order_id}/preparation/complete` | Comprometer reserva y dejar `READY` |

Las escrituras usan versión optimista. Reabrir una línea exige `reason`.
Completar falla mientras exista una línea pendiente.
