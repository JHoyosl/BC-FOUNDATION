# Actores, permisos y perfiles

> **Estado:** 🟡 Borrador · **Dueño:** Jorge Hoyos · **Actualizado:** 2026-10-02

Los **actores** son quienes interactúan con Barscode. Los **permisos** son lo que el software
deja hacer, módulo por módulo. Los **perfiles** son los conjuntos de permisos que cada negocio
arma y asigna a su staff, siempre **en una sede**.

El modelo lo deciden [`ADR-0006`](decisiones/ADR-0006-permisos-por-modulo-y-perfiles.md)
(permisos y perfiles) y [`ADR-0007`](decisiones/ADR-0007-un-usuario-varias-vinculaciones.md)
(una persona, varios negocios). "Rol" deja de ser un término del KB; el archivo conserva su
nombre para no romper enlaces.

---

## 1. Actores

### 1.1 Staff del tenant

| Actor | Qué busca | Módulos que toca | Dispositivo típico |
|---|---|---|---|
| **Dueño** | Saber si el negocio gana. Decidir carta, precios, expansión. | Reportes, Catálogo, Tenancy, todos en lectura | Web, móvil |
| **Administrador de sede** | Que el local funcione hoy. Cuadrar caja, cubrir turnos, que no falte producto. | Todos los de su sede | Web |
| **Cajero** | Cobrar rápido y que cuadre. | POS, Caja, Pagos | Terminal / tablet |
| **Mesero** | Tomar pedidos y atender mesas sin perder tiempo. | POS, Salón y mesas | Móvil / tablet |
| **Cocina / Barra** | Ver qué preparar y en qué orden. | KDS | Pantalla fija |
| **Bodeguero / Compras** | Que el stock del sistema sea el stock real. | Inventario, Compras | Web / móvil |
| **Jefe de personal** | Cubrir turnos y controlar asistencia. | Schedule | Web |

### 1.2 Clientes del local

| Actor | Qué busca | Cómo entra |
|---|---|---|
| **Cliente anónimo** | Pedir sin esperar, pagar sin esperar, entretenerse. | Escanea el QR de la mesa. No crea cuenta. |
| **Cliente registrado** | Lo anterior, más historial, puntos e identidad de juego persistente. | Cuenta de Barscode. |
| **Anfitrión de mesa** | Coordinar el pedido y la cuenta del grupo. | Es el primero en abrir la sesión de mesa, o quien el grupo designe. |

> El **cliente anónimo es el caso por defecto**, no la excepción. Todo flujo del lado cliente
> debe funcionar completo sin cuenta, salvo lo que explícitamente requiera identidad
> persistente (puntos, ranking histórico).

### 1.3 Actores de la plataforma

| Actor | Qué hace |
|---|---|
| **Operador Barscode** | Personal de Barscode: alta de tenants, soporte, planes. No accede a datos de negocio salvo autorización explícita (`PA-ACT-003`). |
| **Sistema externo** | Pasarela de pago, proveedor de facturación electrónica, sistema contable del tenant. |

---

## 2. Permisos y perfiles

### 2.1 Cómo funciona

```
Usuario (la persona)
  └── Vinculación con un tenant
        └── Asignación:  perfil  +  sede  +  estaciones (opcional)
                           │
                           └── permisos del árbol de cada módulo
```

- **Permiso.** Cada módulo trae su árbol: módulo → grupo → permiso. Solo los permisos tienen
  código (`PRM-INV-001`). El árbol de cada módulo vive en la §12 de su ficha.
- **Perfil.** Lo arma el negocio con los permisos que quiera. Barscode entrega perfiles
  sugeridos como punto de partida.
- **Asignación.** Une a una persona con un perfil en una sede. Una persona puede tener varios
  perfiles; sus permisos se suman.
- **Propietario.** No es un perfil: tiene todo en su tenant y no se edita.

### 2.2 Reglas

**Modelo**

- `RN-ROL-008` — Cada módulo define un árbol de permisos de tres niveles: módulo, grupo y permiso. Solo los permisos tienen código (`PRM-<MOD>-<nnn>`). El catálogo lo define Barscode; un negocio no crea permisos.
- `RN-ROL-009` — El negocio crea perfiles y les asocia permisos. Un perfil es `global` (del tenant) o `local` (de una sede). Tener varios perfiles suma permisos; no existen permisos que quiten.
- `RN-ROL-010` — Marcar un grupo en un perfil marca los permisos que el grupo tiene en ese momento. No es un comodín: un permiso que nazca después bajo ese grupo no entra solo al perfil.
- `RN-ROL-011` — Una asignación une una vinculación (`RN-ARQ-017`), un perfil, una sede y, opcionalmente, estaciones. Quien tiene asignación en la sede A no opera la sede B sin otra asignación. No hay alcance por horario.
- `RN-ROL-012` — El Propietario no es un perfil: tiene todos los permisos de su tenant, en todas las sedes, y no se edita.

**Lo que no se puede configurar**

- `RN-ROL-013` — Nadie autoriza su propia acción. Una acción que exige autorización la autoriza otra persona presente que tenga el permiso de autorizarla (código o confirmación). El registro guarda a los dos: quien ejecuta y quien autoriza.
- `RN-ROL-014` — Nadie asigna un permiso que no tiene, salvo el Propietario.
- `RN-ROL-015` — Un permiso es **sensible** si expone importes o costos, autoriza, anula, reabre, configura o administra usuarios y perfiles.
- `RN-ROL-016` — Toda acción sensible queda registrada con usuario, tenant, sede, fecha, motivo, código de permiso y perfil por el que se tuvo. Los cambios a perfiles y asignaciones también se registran. Debe poderse reconstruir qué podía hacer una persona en una fecha dada.
- `RN-ROL-017` — Un permiso de un módulo que el tenant no tiene habilitado no opera. Queda en el perfil y vuelve a operar si el módulo se habilita de nuevo.

**Dónde se valida**

- `RN-ROL-018` — El servidor verifica el permiso en toda operación, venga de la pantalla, de la API, de una importación o de otro sistema. La pantalla muestra u oculta con esos mismos permisos, pero no es la que autoriza.
- `RN-ROL-019` — Un cambio de permisos rige desde la siguiente operación de la persona, sin que cierre sesión.
- `RN-ROL-020` — El permiso se verifica en la acción que la persona realiza. Los efectos de esa acción en otros módulos se ejecutan como sistema y no exigen permisos de esos módulos.

**Cómo evoluciona el catálogo**

- `RN-ROL-021` — Un permiso nuevo lo recibe solo el Propietario. Los perfiles propios no cambian; los sugeridos vinculados lo reciben si no es sensible. El administrador recibe aviso.
- `RN-ROL-022` — Un permiso que se divide se retira, y nacen permisos nuevos que declaran de cuál nacen. Todo perfil que tenía el original recibe los nuevos, aunque sean sensibles.
- `RN-ROL-023` — Un código de permiso nunca se reutiliza. Un permiso retirado queda marcado con fecha y motivo y deja de operar. El nombre puede cambiar; el significado no se amplía. Los permisos no se fusionan. Moverlo en el árbol no cambia su código.
- `RN-ROL-024` — El catálogo declara las dependencias entre permisos. Asignar un permiso agrega las suyas, y una dependencia no se quita mientras quede en el perfil un permiso que la necesita. Una dependencia agregada en una versión posterior nunca es sensible.
- `RN-ROL-025` — Un perfil sugerido se usa *vinculado* o como *propio*. Vinculado: se actualiza con Barscode y no se edita; los cambios no sensibles se aplican solos; los sensibles y los que quitan un permiso esperan la aceptación del administrador. Propio: copia editable que solo cambia por `RN-ROL-022` y `RN-ROL-024`. Editar un sugerido lo duplica como propio.
- `RN-ROL-026` — El administrador tiene una bandeja de novedades de permisos: nuevos, divididos, retirados, cambios en perfiles sugeridos y cambios pendientes de aceptar.

**Identificación**

- `RN-ROL-007` — El staff se autentica siempre. No existe operación anónima del lado negocio. Se admite autenticación rápida (PIN) para operación en terminal compartida, pero la identidad queda registrada.
- `RN-ROL-027` — El PIN de una terminal compartida pertenece a la vinculación: es distinto en cada tenant.

**Derogadas** (modelo de roles fijos, anterior a `ADR-0006`)

- `RN-ROL-001` — *(derogada por `RN-ROL-011`)* ~~Todo rol se asigna a un miembro del staff en una sede concreta. No existen roles sin sede, salvo Propietario y Solo lectura a nivel tenant.~~
- `RN-ROL-002` — *(derogada por `RN-ROL-009` y `RN-ROL-011`)* ~~Un mismo miembro del staff puede tener roles distintos en sedes distintas.~~
- `RN-ROL-003` — *(derogada por `RN-ROL-016`)* ~~Toda acción sensible (anulación, descuento, ajuste de inventario, reapertura de mesa, apertura de cajón) queda registrada con usuario, fecha y motivo.~~
- `RN-ROL-004` — *(derogada por `RN-ROL-013`)* ~~Una acción que un rol no puede hacer por sí mismo puede ejecutarse con autorización de un rol superior presente (código o confirmación). El registro guarda a los dos: quien ejecuta y quien autoriza.~~
- `RN-ROL-005` — *(derogada por `ADR-0006`: pasa a ser el permiso "ver costos")* ~~El rol Estación nunca ve importes. La cocina no necesita saber precios.~~
- `RN-ROL-006` — *(derogada por `RN-ROL-017`)* ~~Los permisos disponibles dependen de los módulos habilitados en el plan. Un tenant sin módulo Compras no tiene rol Compras.~~

### 2.3 Perfiles sugeridos

Punto de partida que Barscode entrega. Son los antiguos roles fijos; ya no son regla. **Qué
permisos exactos trae cada uno se define en la §12 de cada ficha**, módulo por módulo.

| Perfil sugerido | Trae | No trae |
|---|---|---|
| **Administrador de sede** | Operar y configurar su sede, ver todos sus reportes, anular ventas, autorizar descuentos, cerrar caja. | Cambiar el plan. |
| **Supervisor de turno** | Autorizar anulaciones y descuentos, reabrir mesas, ver reportes del turno. | Modificar carta, precios ni configuración. |
| **Cajero** | Abrir y cerrar su caja, cobrar, emitir documento fiscal, registrar medios de pago. | Anular después del cobro sin autorización, ver reportes de otros turnos. |
| **Mesero** | Abrir mesas, tomar pedidos, enviar comandas, trasladar mesas, pedir la cuenta. | Cobrar, anular, aplicar descuentos. |
| **Estación (cocina/barra)** | Ver comandas de su estación, marcarlas en preparación y listas. | Ver importes, ver la cuenta, tomar pedidos. |
| **Bodega** | Registrar entradas, salidas, traslados, mermas y conteos. Ver costos. | Ver ventas ni cuentas. |
| **Compras** | Gestionar proveedores, crear órdenes de compra, recibir mercancía. | Operar POS. |
| **Recursos / Personal** | Crear turnos, asignar personal, controlar asistencia. | Ver información financiera. |
| **Solo lectura / Contador** | Ver reportes y exportar. | Modificar cualquier cosa. |

El alcance lo da la asignación: la sede y, si aplica, las estaciones. "Durante su turno" ya no es
parte del perfil de Supervisor, porque no hay alcance por horario.

---

## 3. Matriz actor × módulo

Lectura: **C** crea · **L** lee · **M** modifica · **—** sin acceso

| Módulo | Propietario | Admin sede | Supervisor | Cajero | Mesero | Estación | Bodega | Compras | Personal |
|---|---|---|---|---|---|---|---|---|---|
| Catálogo | CLM | CLM | L | L | L | — | L | L | — |
| Salón y mesas | CLM | CLM | LM | L | LM | — | — | — | — |
| POS | CLM | CLM | CLM | CLM | CL | — | — | — | — |
| Caja | L | CLM | LM | CLM | — | — | — | — | — |
| KDS | L | L | L | — | L | LM | — | — | — |
| Pedidos QR | L | LM | L | L | L | — | — | — | — |
| Inventario | L | CLM | L | — | — | L | CLM | L | — |
| Compras | L | CLM | — | — | — | — | L | CLM | — |
| Schedule | L | CLM | L | — | L | L | — | — | CLM |
| Reportes | L | L | L (turno) | L (turno) | — | — | L (stock) | L | L (personal) |
| Gamecenter | CLM | CLM | L | — | — | — | — | — | — |
| Tenancy / Plan | CLM | — | — | — | — | — | — | — | — |
| IAM | CLM | CLM (su sede) | — | — | — | — | — | — | — |

> Esta matriz es **anterior a `ADR-0006`** y no es regla. Queda como referencia general de qué
> perfil sugerido toca qué módulo. Cada ficha la reemplaza, para su módulo, con su árbol de
> permisos y su matriz de perfiles sugeridos. La columna Propietario describe al actor Dueño,
> que no opera; el Propietario puede todo (`RN-ROL-012`).

---

## 4. Preguntas abiertas

- `PA-ACT-001` — ¿Existe la figura de "mesero" separada del "cajero" en el segmento objetivo, o en bares pequeños la misma persona hace todo? Si es lo segundo, los perfiles sugeridos deben traer uno combinado.
- `PA-ACT-002` — ¿El mesero puede cobrar? En Colombia es común que el mesero cobre en la mesa. Si es así, el perfil sugerido de Mesero necesita permiso de cobro.
- `PA-ACT-003` — ¿Bajo qué condiciones el operador Barscode puede ver datos de un tenant (soporte)? Necesita política explícita y registro.
- `PA-ACT-005` — ¿Cómo se autentica el staff en una terminal compartida: PIN, tarjeta, usuario y contraseña?
- `PA-ACT-006` — ¿El "anfitrión de mesa" es un concepto real del producto (con poderes sobre la cuenta del grupo) o solo una forma de hablar? Afecta `pedidos-qr` y `pagos`.

### Cerradas

| Id | Resolución |
|---|---|
| `PA-ACT-004` | **Resuelta** (2026-09-15): los perfiles son a la medida. → `ADR-0006` |
| `PA-ACT-007` | **Resuelta** (2026-10-02): un permiso nuevo lo recibe solo el Propietario. → `RN-ROL-021` |
| `PA-ACT-008` | **Resuelta** (2026-10-02): al dividirse, el original se retira y los perfiles reciben los nuevos. → `RN-ROL-022` |
| `PA-ACT-009` | **Resuelta** (2026-10-02): los códigos no se reutilizan ni se fusionan. → `RN-ROL-023` |
| `PA-ACT-010` | **Resuelta** (2026-10-02): dependencias declaradas en el catálogo. → `RN-ROL-024` |
| `PA-ACT-011` | **Resuelta** (2026-10-02): los permisos de un módulo no habilitado quedan y no operan. → `RN-ROL-017` |
| `PA-ACT-012` | **Resuelta** (2026-10-02): perfil sugerido vinculado o propio. → `RN-ROL-025` |
| `PA-ACT-013` | **Resuelta** (2026-10-02): auditoría con código de permiso y bandeja de novedades. → `RN-ROL-016`, `RN-ROL-026` |
| `PA-ACT-014` | **Resuelta** (2026-10-02): la §12 de cada ficha es el árbol de permisos del módulo. → [`_plantilla.md`](modulos/_plantilla.md) |
