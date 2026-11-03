# Contrato de Marcas v1

Estado: candidato funcional no desplegado. Propietario: `2.1.4-brands`.

CRUD sin borrado físico para marcas con slug estable, nombre, descripción, logo,
sitio y estado. Rutas: `GET/POST /v1/catalog/brands` y
`GET/PATCH /v1/catalog/brands/{brand_slug}`. Escrituras usan idempotencia o
versión optimista y permisos `catalog.brands.read/write`.
