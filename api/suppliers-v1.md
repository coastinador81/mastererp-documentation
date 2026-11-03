# Contrato de Proveedores v1

Estado: candidato funcional no desplegado. Propietario: `2.1.5-suppliers`.

Proveedor reutiliza `catalogo.terceros` como identidad canónica y añade perfil
de abastecimiento, contacto, condiciones y relaciones producto-proveedor. No
duplica el tercero financiero. Rutas: `GET/POST /v1/catalog/suppliers` y
`GET/PATCH /v1/catalog/suppliers/{supplier_id}`. Permisos:
`catalog.suppliers.read/write`.
