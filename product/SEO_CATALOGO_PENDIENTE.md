# Tarea pendiente: SEO del catálogo por tienda

Estado: pendiente. No implementar dentro del cierre funcional de Pedidos.

## Decisión recomendada

El producto base continúa siendo un maestro compartido por tenant. La
publicación y configuración SEO deben pertenecer a cada tienda; no deben
guardarse solamente en frontend ni mezclarse con precio, inventario u ofertas.

La primera entrega recomendada es una vertical completa de SEO de Producto por
tienda:

1. Database: crear una proyección comercial con clave
   `tenant_id + tienda_id + producto_id`, FKs reales, RLS, funciones
   controladas, concurrencia optimista, auditoría y pruebas SQL.
2. Backend: exponer lectura y escritura administrativa mediante un contrato
   autorizado existente o una ampliación explícita; el catálogo público debe
   resolver la proyección por tenant y dominio.
3. Admin: incorporar una sección SEO por tienda con slug, meta title, meta
   description, indexación y vista previa, conservando Arimo y los patrones
   visuales vigentes.
4. Storefront: usar ruta por slug y generar title, meta description, canonical,
   robots, Open Graph y datos estructurados de Product desde datos reales.
5. Migración: preparar un backfill dry-run para los productos actualmente
   visibles y comparar conteos antes de cambiar el filtro público.

## Contrato propuesto para Producto

- Publicación: `published`, `indexable`.
- Ruta: `slug`, estable y único por tienda.
- SEO: `meta_title`, `meta_description`.
- Social opcional: `og_title`, `og_description`, `og_image_url`.
- Control: `version`, actor, auditoría y timestamps.
- Canonical: derivado por el servidor como
  `https://{dominio}/product/{slug}`; no se persiste una URL absoluta libre.
- Precio, oferta y disponibilidad: calculados por sus capacidades propietarias;
  no pertenecen a esta proyección.

Categorías y Marcas deben recibir después tablas equivalentes con FKs reales,
no una tabla polimórfica sin integridad referencial. El slug actual de Categoría
y Marca se conserva como referencia interna estable del tenant; el slug público
puede variar por tienda.

## Alcance por entidad

| Entidad | Tratamiento pendiente |
| --- | --- |
| Productos | SEO y publicación por tienda; primera vertical. |
| Categorías | Página pública real, slug y metadata por tienda. |
| Marcas | Publicación por tienda, metadata, logo y página de marca. |
| Tienda/inicio | Campos tipados de title, description, indexación e imagen social. |
| Website/CMS | Contrato independiente dentro de Website; no usar JSON libre como sustituto. |
| Atributos | Sin SEO propio inicialmente; pueden alimentar datos estructurados de Producto. |
| Proveedores | Sin página indexable hasta decisión comercial expresa. |
| Ofertas | Contrato separado por tenant + tienda + producto + vigencia/descuento. |

## Organización de las fichas administrativas

La sección `Organización` no debe mostrarse igual para todas las entidades:

- Producto: categoría principal, marca y futuras categorías adicionales.
- Categoría: padre y orden.
- Atributo: orden y categorías aplicables.
- Marca: sustituir la sección vacía por SEO y Publicación por tienda.
- Proveedor: usar Contacto y Condiciones comerciales.
- Tienda: separar Dominio y nodo, Publicación y SEO.

## Referencia funcional

PrestaShop administra meta title, meta description y friendly URL en Productos,
Categorías y Marcas. También distingue activación/indexación, canonical y
redirecciones. MasterERP debe adoptar esas capacidades respetando su propio
aislamiento multi-tenant y multi-tienda, sin copiar el modelo interno de
PrestaShop ni trasladar autorización al frontend.

