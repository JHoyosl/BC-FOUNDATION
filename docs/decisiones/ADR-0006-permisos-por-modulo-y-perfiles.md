# ADR-0006 — Permisos por módulo y perfiles a la medida

> **Estado:** Aceptada (modelo base) · **Fecha:** 2026-09-15 · **Decide:** Jorge Hoyos
> **Registrada en el KB:** 2026-10-02. La evolución del catálogo de permisos sigue abierta
> (`PA-ACT-007`…`014`) y la aplicación a los documentos está pendiente.

## Contexto

[`05-actores-y-roles.md`](../05-actores-y-roles.md) nació con **roles fijos**: diez roles, cada
uno con lo que puede y no puede hacer, y una matriz rol × módulo.

Al revisar la ficha de `inventario` el modelo no aguantó. La ficha usa figuras que `05` no tiene
(jefe de barra, cocinero de producción, bartender de turno), su matriz de permisos tiene una
columna "Jefe estación" que no es un rol, y al Supervisor le da operaciones que `05` no le
reconoce. Corregirlo rol por rol habría sido repetir el problema en cada módulo.

Estaba además abierta `PA-ACT-004`: *¿los roles son fijos o el negocio puede crear los suyos?*

## Decisión

**Cada módulo define sus permisos; cada negocio arma sus perfiles con ellos.**

1. **Catálogo de permisos por módulo.** Cada módulo tiene sus permisos, con código
   `PRM-<MOD>-<nnn>` (ej. `PRM-INV-001 — Crear insumo`). El catálogo lo define Barscode: un
   permiso abre una función que existe en el software, y un negocio no puede crear funciones.
2. **Perfiles del negocio.** El negocio crea perfiles y les asocia permisos. Barscode entrega
   **perfiles sugeridos** como base inicial; no determinan el detalle de cada negocio. Un usuario
   puede tener varios perfiles.
3. **Perfil global o local.** Como los insumos (`RN-INV-038`): un perfil es `global` (del
   negocio, usable en todas las sedes) o `local` (de una sede).
4. **Asignación.** Usuario + perfil + sede + estaciones opcionales. La asignación siempre es por
   sede (`RN-ARQ-012`). No hay alcance por horario: dependería de `schedule` y rompería la
   autonomía. "Durante su turno" deja de ser un alcance del permiso.
5. **Los permisos van apareciendo a medida que avanza el desarrollo.** Por eso el catálogo
   necesita reglas de evolución propias (ver *Lo que queda abierto*).

### Lo que no se puede configurar

- **Propietario no es un perfil.** Siempre tiene todo y no se edita. Evita que un negocio se
  quede sin acceso.
- **Nadie autoriza su propia acción.** `RN-ROL-004` pasa de "un rol superior presente" a "otra
  persona presente con el permiso de autorizar esa acción": con perfiles no hay jerarquía.
- Toda acción sensible queda auditada **con el código de permiso usado** (`RN-ROL-003`).
- Un permiso de un módulo no contratado no opera (`RN-ROL-006`).

### Reglas que dejan de ser fijas y pasan a ser permisos

- `RN-ROL-005` (el rol Estación nunca ve importes) → permiso "ver costos", que el perfil sugerido
  de Estación no trae. Un chef puede necesitarlo.
- `RN-INV-043` (quién crea insumos `global`) → permiso con alcance de negocio.

## Alternativas consideradas

| Alternativa | Por qué se descartó |
|---|---|
| Roles fijos definidos por Barscode | Es el modelo de `05`. No refleja que cada negocio reparte el trabajo distinto, y por sede. |
| Roles fijos, ampliando el rol Estación con permisos de inventario | Era la propuesta inicial para alinear la ficha con `05`. Resuelve `inventario` y deja el mismo problema para cada módulo siguiente. |
| Permisos con alcance por horario ("durante su turno") | Obliga a que `iam` dependa de `schedule`. Rompe `ADR-0001`. |

## Consecuencias

**A favor**

- Cierra `PA-ACT-004`: los perfiles son a la medida.
- Cada negocio arma sus accesos por sede sin pedirle un rol nuevo a Barscode.
- La sección de permisos de cada ficha deja de depender de una lista de roles definida antes de
  conocer el módulo.

**En contra**

- El catálogo de permisos crece con el desarrollo y hay que gobernar cómo cambia: permisos
  nuevos, divididos, retirados. Es trabajo que los roles fijos no tenían.
- Cambia mucho el módulo `iam`.
- Deja obsoletas partes de documentos ya escritos (ver *Pendiente de aplicar*).

## Lo que queda abierto

La **evolución del catálogo de permisos** se presentó el 2026-09-15 en ocho puntos, cada uno con
su recomendación, y espera respuesta. Viven como `PA-ACT-007`…`PA-ACT-014` en
[`05-actores-y-roles.md`](../05-actores-y-roles.md).

El **catálogo de permisos de Inventario** y sus perfiles sugeridos son `PA-INV-016` en la
[ficha](../modulos/inventario.md).

## Pendiente de aplicar

Esta decisión todavía no está reflejada en:

- `05-actores-y-roles.md` §2 y §3 — roles fijos y matriz → perfiles sugeridos.
- `04-glosario.md` — "Rol" → "Perfil" y "Permiso" (los dos términos ya están agregados).
- `03-principios.md` — `RN-ARQ-012`.
- `05-actores-y-roles.md` — `RN-ROL-004` y `RN-ROL-005`.
- `modulos/_plantilla.md` §12 y `modulos/inventario.md` §3, §12 y `RN-INV-043`.

## Referencias

- [`docs/05-actores-y-roles.md`](../05-actores-y-roles.md) — actores, roles y `PA-ACT-007`…`014`
- [`ADR-0001`](ADR-0001-modulos-autonomos.md) — autonomía de módulos
- [`docs/00-guia.md`](../00-guia.md) — identificador `PRM-<MOD>-<nnn>`
