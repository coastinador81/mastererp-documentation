# Fase 7 — preflight de infraestructura DEV

Estado: preflight AWS en curso. Cuenta, región, inventario y propiedad única
confirmados; la revisión de change sets y el despliegue continúan pendientes.
Fase 6 cerró la carga y reconciliación de datos en sombra; el cutover ocurre
después de integrar y probar infraestructura.

## Evidencia local

| Control | Resultado | Límite |
| --- | --- | --- |
| Estructura de Infrastructure | `validate_structure.py` aprobado | No certifica el estado de AWS. |
| Propiedad/rutas de la migración | `validate_phase5.py`: 12 propietarios y 54 rutas | Solo verifica plantillas locales. |
| Región y cuenta | `us-west-1`, cuenta `277352717693`, usuario `andrea_madriz` | Confirmado mediante STS el 2026-11-04. |
| Repositorios | Database contiene carga/reconciliación publicada; Documentation registra la evidencia | Cada repositorio conserva su propio estado Git. |

El inventario actual, drift y límites de los próximos change sets están en
[evidencia DEV de Fase 7](phase-7-dev-inventory.md).

## Condiciones que permanecen bloqueantes

1. Las plantillas reorganizadas no son reemplazos directos de stacks existentes.
2. RDS, red, SQL y el migrador pertenecen a `mastererp-database`; no crear un
   segundo migrador ni modificar datos durante esta fase.
3. El Admin API conserva propiedad vigente en `mastererp-dev-onboarding`; los
   APIs Client y Agent siguen en `mastererp-dev-http-apis`. No aplicar un
   OpenAPI parcial ni recrear rutas, permisos o authorizers.
4. El drift actual está inventariado: 11 recursos eliminados en
   `mastererp-dev-onboarding` y un security group modificado en
   `mastererp-dev-data`. Ningún change set puede intentar corregirlos de forma
   implícita.

## Secuencia autorizable posterior

1. Consulta de solo lectura: identidad AWS, región, stacks, drift, APIs,
   authorizers, Lambdas, RDS y secretos referenciados, sin revelar valores.
2. Comparar el inventario real contra los propietarios locales y resolver toda
   diferencia antes de empaquetar artefactos.
3. Preparar un change set por stack propietario con parámetros y artefactos
   versionados; no ejecutar todavía.
4. Rechazar cualquier `Remove` o `Replacement`; revisar rutas, permisos,
   roles, CORS y referencias cruzadas.
5. Solo tras aprobación explícita, ejecutar el change set y conservar el plan
   de rollback compatible con el stack existente.

Las casillas 7-region y 7-owner tienen evidencia propia. 7-change y 7-deploy
continúan sin marcar hasta preparar, revisar y ejecutar planes seguros. La
fuente de verdad de Tu Mandadito no cambia.
