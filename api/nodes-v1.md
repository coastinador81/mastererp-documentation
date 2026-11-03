# Contrato de Nodos v1

Estado: candidato funcional no desplegado. Propietario: `2.3.4-nodes`.

Un Nodo es el perfil operativo 1:1 de una sucursal de Organización. La sucursal
conserva identidad, código, nombre, empresa y país; Nodos conserva estado,
aceptación de pedidos, cobertura, capacidad, horarios, tarifas y configuración.
Configurar un nodo exige una sucursal activa existente: no se duplica ni se
crea identidad organizacional desde Operaciones.

Estados: `DRAFT`, `ACTIVE`, `PAUSED`, `SUSPENDED`. No hay eliminación física.
Pausar o suspender deshabilita aceptación de pedidos. Las transiciones usan
versión optimista, motivo y auditoría.

## API inicial

```text
GET  /v1/companies/{company_id}/nodes
GET  /v1/companies/{company_id}/nodes/{node_id}
PUT  /v1/companies/{company_id}/nodes/{node_id}
POST /v1/companies/{company_id}/nodes/{node_id}/transitions
```

Permisos: `operations.nodes.read/write`, módulo `erp.operations` activo y
empresa/sucursal del tenant. Nodos no escribe inventario, pedidos, rutas ni
repartidores; esos módulos lo referencian por ID.
