# E-commerce y Storefront API v1

Candidato no desplegado. Administración exige JWT y permisos
`digital_sales.ecommerce.read/write`:

| Método | Ruta | Uso |
| --- | --- | --- |
| GET/POST | `/v1/companies/{company_id}/stores` | Listar/crear tiendas |
| GET/PATCH | `/v1/companies/{company_id}/stores/{store_id}` | Consultar/configurar |
| GET | `/v1/storefront/catalog?domain={domain}` | Catálogo público activo |

El catálogo público no acepta una identidad administrativa. Exige
`X-Tenant-Id` y resuelve un dominio `ACTIVE` dentro de ese tenant; solo devuelve
marca/configuración, categorías activas y productos con disponibilidad positiva
del nodo configurado. No expone stock interno, usuarios ni permisos.

El storefront base conserva carrito local y emite `mastererp:checkout`; el
adaptador autenticado de Pedidos se integra en una fase posterior. Los módulos
opcionales se reciben en `store.modules`, sin acoplar la tienda a Repartidores.
