# Contrato de Repartidores v1

Estado: candidato funcional no desplegado. Propietario: `2.10.1-drivers`.

Repartidores conserva el perfil operativo del conductor, su disponibilidad,
estado de acceso y asignación a nodos. El vínculo con un usuario interno es
opcional. Cognito autentica, pero este módulo no crea usuarios Cognito ni guarda
claims. Rutas será propietaria de la ruta activa y de su ciclo de entrega.

El vínculo de acceso se realiza con el UUID de un usuario existente del tenant.
No crea identidades ni duplica Cognito. El usuario dedicado debe usar el rol
`tenant.driver`; `tenant.owner` también conserva acceso administrativo. El
backend vuelve a comprobar vínculo, rol/capacidad, empresa, nodo y propiedad de
la ruta en cada entrega.

Estados del perfil: `ACTIVE`, `INACTIVE`, `SUSPENDED`. Disponibilidad:
`OFFLINE`, `AVAILABLE`, `ON_ROUTE`. Acceso: `PENDING`, `LINKED`, `ERROR`,
`DISABLED`. No existe eliminación física en v1.

## API inicial

```text
GET   /v1/companies/{company_id}/drivers
GET   /v1/companies/{company_id}/drivers/{driver_id}
POST  /v1/companies/{company_id}/drivers
PATCH /v1/companies/{company_id}/drivers/{driver_id}
PUT   /v1/companies/{company_id}/nodes/{node_id}/drivers/{driver_id}
```

`POST` exige `Idempotency-Key`; `PATCH` exige `version`; la asignación recibe
`{"active": true|false}`. La lista admite `node_id`, `status` y `limit`.

Permisos: `transport.drivers.read/write/assign`, módulo `erp.transport` activo,
empresa y nodo pertenecientes al tenant. La UI obtiene la sesión del app-shell
en runtime y la API vuelve a validar identidad, tenant, módulo y permisos.

La experiencia **Mi ruta** consume el contrato de Rutas con
`transport.deliveries.execute`. Repartidores no escribe pedidos ni rutas y no
contiene una segunda implementación del cierre.
