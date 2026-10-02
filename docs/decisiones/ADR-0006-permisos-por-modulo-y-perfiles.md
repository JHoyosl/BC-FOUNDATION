# ADR-0006 — Permisos por módulo y perfiles a la medida

> **Estado:** Aceptada · **Fecha:** 2026-09-15 · **Ampliada:** 2026-10-02 · **Decide:** Jorge Hoyos

## Contexto

[`05-actores-y-roles.md`](../05-actores-y-roles.md) nació con **roles fijos**: diez roles, cada
uno con lo que puede y no puede hacer, y una matriz rol × módulo.

Al revisar la ficha de `inventario` el modelo no aguantó. La ficha usa figuras que `05` no tenía
(jefe de barra, cocinero de producción, bartender de turno), su matriz de permisos tiene una
columna "Jefe estación" que no es un rol, y al Supervisor le da operaciones que `05` no le
reconocía. Corregirlo rol por rol habría sido repetir el problema en cada módulo.

Estaba además abierta `PA-ACT-004`: *¿los roles son fijos o el negocio puede crear los suyos?*

## Decisión

**Cada módulo define sus permisos; cada negocio arma sus perfiles con ellos.**

### El modelo

1. **Catálogo de permisos por módulo.** Cada módulo tiene sus permisos, con código
   `PRM-<MOD>-<nnn>` (ej. `PRM-INV-001 — Crear insumo`). El catálogo lo define Barscode: un
   permiso abre una función que existe en el software, y un negocio no puede crear funciones.
   Si el usuario tiene el permiso, puede hacer la acción; si no, no.
2. **Perfiles del negocio.** El negocio crea perfiles y les asocia permisos. Barscode entrega
   **perfiles sugeridos** como base inicial; no determinan el detalle de cada negocio. Un usuario
   puede tener varios perfiles, y sus permisos **se suman**. No existen permisos que quiten.
3. **Perfil global o local.** Como los insumos (`RN-INV-038`): un perfil es `global` (del
   negocio, usable en todas las sedes) o `local` (de una sede).
4. **Asignación.** Vinculación del usuario con el negocio + perfil + sede + estaciones
   opcionales. Siempre por sede. No hay alcance por horario: dependería de `schedule` y rompería
   la autonomía. "Durante su turno" deja de ser un alcance del permiso. Qué es una vinculación lo
   decide [`ADR-0007`](ADR-0007-un-usuario-varias-vinculaciones.md).

### El catálogo es un árbol

```
Inventario                          ← módulo
├─ Insumos                          ← grupo
│   ├─ Ver           PRM-INV-…      ← permiso
│   ├─ Crear         PRM-INV-…
│   ├─ Modificar     PRM-INV-…
│   └─ Desactivar    PRM-INV-…
└─ Costos
    └─ Ver costos    PRM-INV-…      (sensible)
```

- **Tres niveles:** módulo → grupo → permiso. Solo los permisos tienen código; los grupos
  ordenan y no se verifican.
- **Marcar un grupo es un atajo, no un comodín.** Marca los permisos que el grupo tiene en ese
  momento. Un permiso que nazca después bajo ese grupo no entra solo al perfil.
- **El código no depende del lugar en el árbol.** Mover un permiso a otro grupo no lo cambia.

### Dónde se valida

- **El servidor verifica el permiso en toda operación,** venga de la pantalla, de la API, de una
  importación o de otro sistema. La pantalla muestra u oculta con esos mismos permisos, pero no
  es la que autoriza: lo que alguien no puede hacer en pantalla tampoco lo puede hacer por otra
  vía.
- **Un cambio de permisos rige desde la siguiente operación** de la persona, sin que cierre
  sesión.
- **El permiso se verifica en la acción que la persona realiza.** Lo que esa acción provoca en
  otros módulos se ejecuta como sistema: el mesero vende sin necesitar permisos de inventario.

### Lo que no se puede configurar

- **Propietario no es un perfil.** Siempre tiene todo y no se edita. Evita que un negocio se
  quede sin acceso.
- **Nadie autoriza su propia acción.** La autoriza otra persona presente que tenga el permiso de
  autorizarla. Con perfiles no hay jerarquía de roles.
- **Nadie asigna un permiso que no tiene,** salvo el Propietario. Evita que quien administra
  perfiles se dé acceso a sí mismo.
- Toda acción sensible queda auditada con usuario, negocio, sede, código de permiso y perfil.
- Un permiso de un módulo no habilitado no opera.

### Cómo evoluciona el catálogo

Los permisos van apareciendo a medida que avanza el desarrollo. Reglas:

Un permiso es **sensible** si expone importes o costos, autoriza, anula, reabre, configura o
administra usuarios y perfiles.

| Caso | Regla |
|---|---|
| **Permiso nuevo** | Lo recibe solo el Propietario. Los perfiles propios no cambian, aunque tengan marcado el grupo completo. Los sugeridos vinculados lo reciben si no es sensible. El administrador recibe aviso. |
| **Permiso que se divide** | El original se retira y nacen permisos nuevos que declaran de cuál nacen. Todo perfil que tenía el original recibe los nuevos, aunque sean sensibles. Nadie pierde acceso por una versión. |
| **Retirar, renombrar, fusionar** | Un código nunca se reutiliza. El retirado queda marcado con fecha y motivo y deja de operar. El nombre puede cambiar; el significado nunca se amplía. Los permisos no se fusionan. |
| **Dependencias** | El catálogo las declara. Asignar un permiso agrega las suyas, y una dependencia no se quita mientras quede en el perfil un permiso que la necesita. Una dependencia agregada en una versión posterior nunca es sensible. |
| **Módulo que se habilita o se quita** | Al habilitarlo aparecen su catálogo y sus perfiles sugeridos, sin tocar los propios. Al quitarlo, sus permisos quedan en los perfiles pero no operan, y vuelven igual si se habilita de nuevo. |
| **Perfil sugerido que mejora** | *Vinculado*: se actualiza con Barscode y no se edita; los cambios no sensibles se aplican solos; los sensibles y los que **quitan** un permiso esperan la aceptación del administrador. *Propio*: copia editable que nunca cambia sola, salvo por división o dependencia. Editar un sugerido es duplicarlo como propio. |
| **Auditoría y avisos** | Se audita cada acción sensible y cada cambio a perfiles y asignaciones. Debe poderse responder qué podía hacer una persona en una fecha. El administrador tiene una bandeja de novedades de permisos. |
| **Registro en el KB** | La §12 de cada ficha es el árbol de permisos del módulo, más la matriz de perfiles sugeridos. Ninguna función se construye sin su código en la ficha. |

Las reglas numeradas están en [`05-actores-y-roles.md`](../05-actores-y-roles.md):
`RN-ROL-008`…`027`.

### Reglas que dejan de ser fijas y pasan a ser permisos

- `RN-ROL-005` (el rol Estación nunca ve importes) → permiso "ver costos", que el perfil sugerido
  de Estación no trae. Un chef puede necesitarlo.
- `RN-INV-043` (quién crea insumos `global`) → permiso con alcance de negocio.

## Alternativas consideradas

| Alternativa | Por qué se descartó |
|---|---|
| Roles fijos definidos por Barscode | Era el modelo de `05`. No refleja que cada negocio reparte el trabajo distinto, y por sede. |
| Roles fijos, ampliando el rol Estación con permisos de inventario | Era la propuesta inicial para alinear la ficha con `05`. Resuelve `inventario` y deja el mismo problema para cada módulo siguiente. |
| Permisos con alcance por horario ("durante su turno") | Obliga a que `iam` dependa de `schedule`. Rompe `ADR-0001`. |
| Grupo del árbol como comodín (todo lo nuevo entra solo) | Una función sensible nueva llegaría a la gente sin que nadie la apruebe. |

## Consecuencias

**A favor**

- Cierra `PA-ACT-004`: los perfiles son a la medida.
- Cada negocio arma sus accesos por sede sin pedirle un rol nuevo a Barscode.
- La sección de permisos de cada ficha deja de depender de una lista de roles definida antes de
  conocer el módulo.
- Ninguna versión le quita ni le da acceso sensible a nadie sin que alguien del negocio lo vea.

**En contra**

- El catálogo de permisos crece con el desarrollo y hay que gobernar cómo cambia. Es trabajo que
  los roles fijos no tenían.
- Cambia mucho el módulo `iam`: perfiles, asignaciones, historial y bandeja de novedades.
- Cada ficha debe traer su árbol de permisos completo antes de entrar a desarrollo.

## Notas

**Para la capa técnica.** En palabras de Jorge: los permisos se cargan del backend al frontend
y se validan en los dos lados; si un usuario no puede editar en el front, tampoco lo podrá hacer
por API; la fuente de los permisos siempre es el backend. El KB lo registra en términos
funcionales (`RN-ROL-018`); el cómo se abre en `docs/tecnico/` cuando corresponda.

## Lo que queda abierto

- `PA-INV-016` — el árbol de permisos de Inventario y sus perfiles sugeridos.
- `PA-ALC-002` — con qué permisos opera un POS sin conexión, si el servidor no puede validar.

## Referencias

- [`docs/05-actores-y-roles.md`](../05-actores-y-roles.md) — reglas `RN-ROL-008`…`027` y perfiles sugeridos
- [`ADR-0007`](ADR-0007-un-usuario-varias-vinculaciones.md) — usuario y vinculación
- [`ADR-0001`](ADR-0001-modulos-autonomos.md) — autonomía de módulos
- [`docs/00-guia.md`](../00-guia.md) — identificador `PRM-<MOD>-<nnn>`
