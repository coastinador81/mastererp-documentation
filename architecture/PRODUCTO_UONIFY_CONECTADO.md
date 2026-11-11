# Uonify y MasterERP como un solo producto conectado

Estado: decisión vigente de producto y arquitectura frontend.

`dev.uonify.com` es el producto principal y la puerta general: cuenta, espacios,
chat con Uony y lanzador de aplicaciones. Finanzas, Ecommerce, Transporte,
Nodos y las demás capacidades no son productos aislados; son superficies del
mismo producto modular con despliegue y rollback independientes.

## Experiencia y operación

- Un flujo puede iniciarse desde el chat o desde la vista manual propietaria.
- Ambas superficies usan las mismas APIs, permisos, datos y auditoría.
- Uony actúa mediante herramientas o APIs autorizadas; nunca accede directamente
  a la base de datos ni convierte un fallo real en un resultado simulado.
- Cognito autentica y el backend revalida tenant, empresa, tienda, nodo, módulo
  y permiso según cada contrato.
- Los subdominios no comparten tokens por URL ni obtienen permisos por aparecer
  en el lanzador.

## Sistema visual

El manual corporativo de Uonify define los fundamentos comunes. La experiencia
aprobada de `dev.uonify.com` es la referencia principal de marca. Chat,
administración, storefront y móvil pueden usar composiciones diferentes, pero
comparten colores, tipografía, espaciado, radios, movimiento, accesibilidad y
componentes compatibles.

La fuente técnica compartida comienza en
`mastererp-platform/frontend/shared/ui/design-tokens.css`. Ninguna aplicación
debe importar código interno de `0-uonify/0.2-app` ni copiar una paleta local.
Los tokens no cambian contratos, permisos, datos ni estado de despliegue.

## Transición

`app-dev.uonify.com` conserva el shell administrativo histórico y su evidencia
de Fase 8. No es la puerta maestra vigente. La migración visual y de componentes
será gradual: fundamentos, componentes base, shell y finalmente pantallas,
manteniendo cada aplicación funcional durante el proceso.
