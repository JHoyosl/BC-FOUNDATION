# Módulo: `inventario` — Inventario

> **Estado:** 🟡 Borrador · **Dueño:** Jorge Hoyos · **Orden:** 1 (en curso) · **Actualizado:** 2026-09-14

Esta ficha es la **referencia** del KB: define la forma que deben tener todas las demás.

---

## 1. Objetivo

Mantener, en todo momento, cuánto insumo hay en cada bodega de una sede, cuánto cuesta y por
qué cambió — de modo que el negocio pueda costear lo que vende, detectar fugas y reponer a
tiempo.

---

## 2. Contrato de autonomía

| | |
|---|---|
| **Funciona solo** | Catálogo de insumos (global por tenant o local por sede) con unidades, dimensiones y empaques · bodegas · existencias · entradas, salidas, traslados, ajustes y mermas manuales · control de envases abiertos · fichas técnicas (recetas) y costeo · órdenes de producción interna · costo promedio ponderado · lotes y vencimientos · conteos físicos · alertas de mínimo · valorización del inventario · trazabilidad completa de movimientos. Un negocio que ya tiene POS de otro proveedor puede contratar solo Inventario y usarlo entero. |
| **Degradación** | **Sin `pos`:** las salidas por venta se registran manualmente o por importación de un archivo de ventas. **Sin `compras`:** las entradas se registran como entrada manual con costo digitado, sin orden de compra ni proveedor formal (queda un campo libre de proveedor). **Sin `catalogo`:** las recetas se definen contra un "producto vendible" declarado localmente por Inventario, no contra el catálogo comercial. **Sin `reportes`:** Inventario entrega sus propios informes básicos. |
| **Consume** | De `pos`: ventas confirmadas (producto, cantidad, modificadores, sede, fecha) → para descontar insumos vía receta. De `compras`: recepciones de mercancía (insumo, cantidad, costo, proveedor) → para ingresar stock. De `catalogo`: identidad de los productos vendibles → para asociar recetas. De `tenancy`: sedes activas. De `iam`: usuarios y permisos. **Todos con alternativa manual.** |
| **Expone** | Existencia actual de un insumo en una bodega · costo unitario vigente de un insumo · costo calculado de una receta · disponibilidad para preparar un producto (cuántas unidades alcanzan los insumos) · operación de descontar/ingresar cantidad con referencia de origen · alertas de stock bajo o agotado · valorización del inventario a una fecha. |

---

## 3. Actores

| Actor | Qué hace aquí |
|---|---|
| **Bodeguero** | Registra entradas, salidas, traslados y mermas. Ejecuta conteos físicos. Es el usuario principal. |
| **Administrador de sede** | Configura insumos, bodegas, mínimos y recetas. Aprueba ajustes. Revisa diferencias de conteo. |
| **Jefe de barra / cocina** | Registra mermas de su estación. Consulta existencias. |
| **Dueño** | Consulta valorización, costos, mermas y diferencias. No opera. |
| **Compras** | Consulta existencias y mínimos para decidir qué pedir. |
| **Sistema (`pos`)** | Genera salidas automáticas por venta. |
| **Sistema (`compras`)** | Genera entradas automáticas por recepción. |
| **Cocinero / bartender de producción** | Ejecuta órdenes de producción: almíbares, salsas, infusiones, despiece. |

---

## 4. Conceptos propios

- **Insumo** — Cosa que se compra, se almacena y se consume. Tiene unidad de medida y costo.
  *No es lo mismo que producto* (lo que se vende). Ver [glosario](../04-glosario.md).
- **Bodega** — Lugar de almacenamiento dentro de una sede. El stock siempre es *de un insumo
  en una bodega*, nunca "de la sede".
- **Unidad base** — Unidad en la que se lleva el stock de un insumo (ml, g, unidad). Define
  su **dimensión**: volumen, masa o conteo.
- **Dimensión** — Volumen, masa o conteo. **No existe conversión entre dimensiones.**
- **Empaque** — Forma concreta en que se compra o maneja un insumo, que declara cuánto
  contiene *en la unidad base del insumo*: un tetrapak de crema = 1030 g.
- **Envase en servicio** — Envase ya abierto del que se está sirviendo.
- **Método de control de envase abierto** — Cómo se verifica lo que queda en un envase
  empezado: `unidad`, `nivel`, `peso` o `apertura`. Se configura por insumo.
- **Movimiento** — Único mecanismo por el que cambia una existencia.
- **Receta (ficha técnica)** — Insumos y cantidades que consume un producto vendible **o un
  insumo producido internamente**.
- **Orden de producción** — Documento que transforma unos insumos en otro insumo distinto.
- **Merma** — Pérdida sin venta, con motivo obligatorio.
- **Conteo físico** — Verificación manual del stock contra el sistema.
- **Costo promedio ponderado (CPP)** — Método de valoración: cada entrada recalcula el costo
  unitario promedio del insumo en la bodega.
- **Kardex** — Historial cronológico de movimientos de un insumo en una bodega, con saldo y
  costo después de cada uno.

---

## 5. Entidades

### 5.1 Insumo
Qué se compra y se consume.

- **Ámbito**: `global` (del tenant, compartido por todas las sedes) o `local` (de una sede)
- Código interno y nombre
- Categoría (licores, cervezas, alimentos, desechables, aseo…)
- **Unidad base** de medida, y con ella su **dimensión** (volumen, masa o conteo)
- **Empaques** con los que se compra o se maneja (ver 5.2)
- ¿Exige control de lote y vencimiento? (opcional, por insumo)
- **Método de control de envase abierto** (ver 5.3)
- Activo / inactivo
- Cuenta contable de referencia (opcional, para exportación)

### 5.2 Unidad de medida y empaques

> Resuelve `PA-INV-003` (2026-09-14).

Una unidad de medida tiene nombre, símbolo y **dimensión**: volumen, masa o conteo.

**No existe conversión entre dimensiones.** No hay densidad, no hay factor genérico
ml↔g. Un insumo vive en una sola dimensión y ahí se queda.

Lo que sí existe es el **empaque**: una forma concreta en que ese insumo se compra o se
maneja, que declara **cuánto contiene expresado en la unidad base del insumo**.

```
Insumo: Crema de leche
  Unidad base: g   (dimensión: masa)
  Empaques:
    · Tetrapak  = 1030 g      ← se declara el peso real del envase de 1 L
    · Caja      = 12 tetrapak = 12360 g

Insumo: Ron añejo
  Unidad base: ml  (dimensión: volumen)
  Empaques:
    · Botella 750 = 750 ml
    · Caja        = 12 botellas = 9000 ml
```

La crema se compra en tetrapak de 1 litro y se consume en gramos — y aun así nunca se
convierte volumen a masa. Se declara **una vez** cuánto pesa ese tetrapak concreto, y a
partir de ahí todo es masa.

**Por qué así:** la densidad es una propiedad del producto, no del insumo genérico. Dos
cremas distintas pesan distinto. Declarar el contenido del empaque real es más exacto que
aplicar una densidad teórica, y no obliga a nadie a hacer cuentas.

### 5.3 Método de control de envase abierto

> Resuelve `PA-INV-010` (2026-09-14).

Cada insumo declara **cómo se controla lo que queda en un envase empezado**. Es
configurable por insumo porque no compensa el mismo rigor para un whisky de 300.000 COP que
para una caja de cerveza.

| Método | Cómo funciona | Para qué sirve |
|---|---|---|
| `unidad` | El envase es indivisible. Se descuenta entero al venderse. | Cerveza en botella, gaseosa en lata, producto empacado |
| `nivel` | El stock se lleva en la unidad base y el conteo estima el nivel a ojo (⅓, ½, ¾ de envase). | El caso general de barra. Sin equipo extra |
| `peso` | El stock se lleva en la unidad base y el conteo **pesa** el envase abierto; se descuenta la tara. | Licor premium, insumos caros de cocina |
| `apertura` | Al abrirse, el envase se descuenta completo y pasa a estado *en servicio*. | Cuando el descuadre interno no importa o no se puede medir |

**En todos los métodos, la venta descuenta según la receta.** Lo que cambia es **cómo se
verifica** en el conteo físico, y por tanto qué tan fina es la diferencia que el sistema
puede detectar.

Un insumo con método `peso` declara además la **tara** de su envase (cuánto pesa vacío).

### 5.4 Envase en servicio
Solo para insumos con método `nivel`, `peso` o `apertura`. Representa un envase concreto
que ya se abrió.

- Insumo, bodega, empaque de origen
- Fecha y responsable de apertura
- Contenido teórico restante (calculado por las ventas)
- Contenido verificado en el último conteo, y cuándo
- Lote y vencimiento, si el insumo los controla
- Estado: `en servicio` · `agotado` · `descartado`

### 5.5 Bodega
Lugar de almacenamiento. Pertenece a una **sede**.

- Nombre, tipo (bodega principal, barra, cocina, nevera)
- ¿Permite venta directa desde ella? (define de dónde descuenta el POS)
- Responsable
- Activa / inactiva

### 5.6 Existencia
La cantidad de un insumo en una bodega. **No es una entidad que se edita**: es el resultado
acumulado de los movimientos.

- Insumo + bodega
- Cantidad en unidad base
- Costo unitario promedio vigente (importe con moneda)
- Stock mínimo y máximo configurados
- Fecha del último movimiento

### 5.7 Movimiento de inventario
El hecho que cambia la existencia. **Es el corazón del módulo.**

- Tipo: `entrada` · `salida` · `traslado` · `ajuste` · `merma`
- Subtipo/motivo (ver 5.8)
- Insumo, bodega origen y/o destino
- Cantidad y unidad en que se registró + cantidad convertida a unidad base
- Costo unitario y costo total (importe con moneda)
- Origen: `manual` · `pos` · `compras` · `conteo` · `importacion`
- Referencia de origen (id de venta, de recepción, de conteo)
- Responsable, fecha y hora, nota
- Estado: `borrador` · `confirmado` · `anulado`
- Lote y vencimiento (si el insumo los controla)

### 5.8 Motivos de movimiento
Catálogo configurable por tenant. Por defecto:

| Tipo | Motivos |
|---|---|
| Entrada | Compra · Devolución de cliente · Producción interna · Traslado entrante · Saldo inicial |
| Salida | Venta · Consumo interno · Cortesía · Traslado saliente · Devolución a proveedor |
| Merma | Rotura · Vencimiento · Derrame · Deterioro · Robo · Error de preparación |
| Ajuste | Diferencia de conteo · Corrección de registro |

### 5.9 Receta (ficha técnica)
Relación de un producto vendible con los insumos que consume.

- Producto vendible (del `catalogo`, o declarado localmente si no existe)
- Líneas: insumo, cantidad, unidad, ¿es opcional?
- Rendimiento (cuántas porciones produce)
- Merma esperada en % (ej. 5% de pérdida al porcionar)
- Costo calculado (derivado, no digitado)
- Vigencia desde / hasta
- Variantes por modificador (ej. "doble" duplica el licor)

### 5.10 Conteo físico
- Sede, bodega, fecha, responsable
- Alcance: total · por categoría · selectivo
- Líneas: insumo, cantidad teórica (congelada al iniciar), cantidad contada, diferencia
- Estado: `abierto` · `en conteo` · `cerrado` · `anulado`
- Ajustes generados al cerrar

### 5.11 Alerta de stock
- Insumo, bodega, tipo (`bajo mínimo` · `agotado` · `sobre máximo` · `por vencer`)
- Fecha de generación, estado (activa / atendida)

---

### 5.12 Orden de producción

> Resuelve `PA-INV-006` (2026-09-14).

Documento que registra la transformación de unos insumos en otro insumo distinto: almíbar,
salsa madre, infusión, despiece de una pieza de carne.

- Sede, bodega, fecha, responsable
- **Insumo producido** y cantidad obtenida (rendimiento real)
- **Receta de producción** aplicada (ver 5.9; una receta puede producir un insumo en lugar
  de un producto vendible)
- **Consumos**: insumos y cantidades realmente utilizadas
- **Subproductos**, si los hay: el despiece produce varios insumos a la vez, y el costo se
  reparte entre ellos según la proporción declarada
- Costo total consumido y **costo unitario resultante** (derivado)
- Merma de producción: diferencia entre rendimiento esperado y real
- Estado: `borrador` · `confirmada` · `anulada`

**Qué hace al confirmarse:** genera en un solo acto las salidas de los insumos consumidos y
la entrada del insumo producido, todas referenciando la orden. El insumo producido entra
con su **costo real**, no estimado.

---

## 6. Estados y transiciones

### 6.1 Movimiento

```
  crear
    │
    ▼
┌──────────┐   confirmar   ┌─────────────┐   anular    ┌──────────┐
│ borrador │──────────────►│ confirmado  │────────────►│ anulado  │
└────┬─────┘               └─────────────┘             └──────────┘
     │ descartar                  │                          ▲
     ▼                            │ NO se edita              │
  (eliminado)                     └──────────────────────────┘
                                   solo se corrige con
                                   un movimiento de ajuste
```

- **borrador** → no afecta existencias. Solo lo ve quien lo creó.
- **confirmado** → afecta existencias. **Inmutable.**
- **anulado** → genera automáticamente un movimiento inverso. El original se conserva.
- Los movimientos de origen `pos` y `compras` nacen **confirmados**; no pasan por borrador.

### 6.2 Conteo físico

```
┌─────────┐  iniciar  ┌────────────┐  cerrar   ┌─────────┐
│ abierto │──────────►│ en conteo  │──────────►│ cerrado │
└────┬────┘           └─────┬──────┘           └─────────┘
     │                      │                        │
     │ anular               │ anular                 └─► genera ajustes
     ▼                      ▼                            (inmutable)
 ┌─────────┐          ┌─────────┐
 │ anulado │◄─────────│ anulado │
 └─────────┘          └─────────┘
```

Al pasar a **en conteo** se congela la cantidad teórica de cada línea. Al **cerrar**, cada
diferencia genera un movimiento de ajuste confirmado. Un conteo cerrado no se reabre.

### 6.3 Orden de producción

```
┌──────────┐  confirmar  ┌─────────────┐   anular   ┌──────────┐
│ borrador │────────────►│ confirmada  │───────────►│ anulada  │
└────┬─────┘             └─────────────┘            └──────────┘
     │ descartar               │                          ▲
     ▼                         │ genera en un solo acto:  │
 (eliminada)                   │  · salidas de consumos   │
                               │  · entrada del producido │
                               │  · merma de producción   │
                               └──────────────────────────┘
                                 anular = movimientos inversos
```

Confirmar es **todo o nada** (`RN-INV-060`): si un insumo no alcanza, la orden se rechaza
entera y no queda medio aplicada.

### 6.4 Envase en servicio

```
   abrir envase
        │
        ▼
┌──────────────┐  contenido llega a 0   ┌──────────┐
│ en servicio  │───────────────────────►│ agotado  │
└──────┬───────┘                        └──────────┘
       │ merma (rotura, vencimiento)
       ▼
┌──────────────┐
│ descartado   │  ← genera merma por el contenido restante
└──────────────┘
```

---

## 7. Reglas de negocio

### Existencias y movimientos

- `RN-INV-001` — La existencia de un insumo en una bodega es siempre el resultado de sus movimientos confirmados. No se edita directamente.
- `RN-INV-002` — Todo movimiento pertenece a exactamente una sede y a un tenant.
- `RN-INV-003` — Todo movimiento confirmado registra responsable, fecha, hora, motivo y origen.
- `RN-INV-004` — Un movimiento confirmado es inmutable: no se edita ni se elimina.
- `RN-INV-005` — Una corrección se hace con un movimiento de ajuste que referencia al original y explica el motivo.
- `RN-INV-006` — Anular un movimiento confirmado genera un movimiento inverso, no borra el original.
- `RN-INV-007` — Un traslado es un solo movimiento con bodega origen y destino; nunca dos movimientos independientes. Si falla una parte, falla todo.
- `RN-INV-008` — No se permite traslado entre bodegas de sedes distintas. Eso es salida en una y entrada en otra, con motivo explícito.
- `RN-INV-009` — Toda cantidad se almacena convertida a la unidad base del insumo, conservando además la unidad en que se registró.

### Stock negativo

- `RN-INV-010` — Por defecto el stock no puede quedar negativo: el sistema rechaza la salida.
- `RN-INV-011` — El tenant puede habilitar *stock negativo permitido* por sede. Con esa opción activa, la salida se registra, el stock queda negativo y se genera una alerta.
- `RN-INV-012` — Una venta registrada por `pos` **nunca se bloquea por falta de stock**. Si no hay existencia, la salida se registra igual y se genera una alerta de inconsistencia. El inventario nunca impide vender.

### Costeo

- `RN-INV-013` — El método de valoración es costo promedio ponderado, calculado por insumo y por bodega.
- `RN-INV-014` — Cada entrada recalcula el costo promedio: `nuevo CPP = (stock × CPP anterior + cantidad × costo entrada) / (stock + cantidad)`.
- `RN-INV-015` — Las salidas se valoran al CPP vigente en el momento del movimiento, y ese costo queda registrado en el movimiento. Un cambio posterior de costo no reescribe movimientos pasados.
- `RN-INV-016` — Todo importe de costo lleva moneda explícita (`RN-ARQ-021`).
- `RN-INV-017` — Una entrada con costo cero es válida (donación, cortesía del proveedor) pero requiere motivo y afecta el promedio.
- `RN-INV-018` — Los costos de flete o impuestos no descontables se pueden distribuir en la entrada, aumentando el costo unitario. La regla de qué impuesto es descontable la resuelve el artefacto de país, no este módulo.

### Recetas y consumo

- `RN-INV-019` — Un producto vendible sin receta no descuenta inventario. No es un error; es una configuración válida (ej. un cover).
- `RN-INV-020` — Al confirmarse una venta, se descuenta cada insumo de la receta, multiplicado por la cantidad vendida y ajustado por la merma esperada.
- `RN-INV-021` — Los modificadores pueden alterar el consumo. La receta declara qué modificador altera qué línea y en qué proporción.
- `RN-INV-022` — El descuento se hace de la bodega marcada como *de venta* para la estación que prepara el producto. Si no hay ninguna definida, de la bodega por defecto de la sede.
- `RN-INV-023` — El costo de una receta es derivado (suma de sus líneas al CPP vigente). Nunca se digita.
- `RN-INV-024` — Cambiar una receta no altera el costo de las ventas ya registradas.
- `RN-INV-025` — Una receta puede incluir otra receta como línea (sub-receta: una base, un almíbar). No se admiten ciclos.

### Mermas

- `RN-INV-026` — Toda merma exige motivo del catálogo y responsable. No existe merma sin motivo.
- `RN-INV-027` — La merma se valora al CPP vigente y queda registrada como pérdida.
- `RN-INV-028` — El tenant puede exigir autorización de un rol superior para mermas que superen un umbral configurable de importe o cantidad.

### Conteos

- `RN-INV-029` — Al iniciar un conteo se congela la cantidad teórica de cada línea. Los movimientos que ocurran durante el conteo no alteran esa cifra congelada, pero sí el stock; la diferencia se calcula contra la cifra congelada.
- `RN-INV-030` — Al cerrar un conteo, cada diferencia genera un movimiento de ajuste confirmado con motivo *diferencia de conteo*.
- `RN-INV-031` — Un conteo cerrado no se reabre ni se modifica. Un error se corrige con un nuevo conteo.
- `RN-INV-032` — El tenant puede exigir autorización para cerrar un conteo cuya diferencia total supere un umbral configurable.

### Lotes y vencimientos

- `RN-INV-033` — Un insumo marcado con control de lote exige lote y fecha de vencimiento en toda entrada.
- `RN-INV-034` — Las salidas de insumos con lote se sugieren por vencimiento más próximo primero (FEFO), pero el usuario puede elegir otro lote con motivo.
- `RN-INV-035` — El sistema alerta de lotes próximos a vencer con la antelación configurada por sede.

### Alertas

- `RN-INV-036` — Cuando la existencia cruza el mínimo hacia abajo, se genera alerta de stock bajo. Se cierra automáticamente al superar el mínimo.
- `RN-INV-037` — Las alertas son por insumo y bodega, no por sede.

### Ámbito de los insumos

- `RN-INV-038` — Un insumo tiene ámbito `global` (del tenant, disponible en todas las sedes) o `local` (de una sede concreta).
- `RN-INV-039` — El ámbito se define al crear el insumo. Un insumo `local` puede promoverse a `global`; un `global` **no** puede degradarse a `local` si tiene movimientos en más de una sede.
- `RN-INV-040` — Las existencias, los costos y los movimientos son **siempre por sede y bodega**, incluso para un insumo `global`. Compartir la ficha no significa compartir el stock.
- `RN-INV-041` — El costo promedio ponderado se calcula por insumo **y bodega**. Un insumo `global` puede tener costos distintos en cada sede, y eso es correcto.
- `RN-INV-042` — Un insumo `local` no puede usarse en una receta `global`. Si una receta compartida lo necesita, el insumo debe promoverse a `global`.
- `RN-INV-043` — Solo el Propietario o un Administrador con alcance de tenant crea o modifica insumos `global`. Un Administrador de sede solo crea insumos `local` de su sede.
- `RN-INV-044` — Un insumo `global` no se elimina mientras tenga existencias o movimientos en cualquier sede. Se desactiva.

### Unidades, dimensiones y empaques

- `RN-INV-045` — Todo insumo tiene exactamente una unidad base, y esa unidad determina su dimensión: volumen, masa o conteo.
- `RN-INV-046` — **No existe conversión entre dimensiones.** No hay densidad ni factor genérico ml↔g. Una operación que la requiera se rechaza.
- `RN-INV-047` — Un empaque declara su contenido **expresado en la unidad base del insumo**. Un tetrapak de crema declara 1030 g, no 1 litro.
- `RN-INV-048` — Un empaque puede componerse de otro empaque (una caja son 12 botellas), y su contenido total se deriva. No se admiten ciclos.
- `RN-INV-049` — Toda cantidad registrada en un empaque se convierte a la unidad base al confirmarse, conservando el empaque en que se registró.
- `RN-INV-050` — El contenido declarado de un empaque puede corregirse, pero el cambio **no reescribe movimientos pasados**: los ya confirmados conservan la cantidad con que se registraron.

### Envases abiertos

- `RN-INV-051` — Cada insumo declara su método de control de envase abierto: `unidad`, `nivel`, `peso` o `apertura`. Por defecto, `unidad`.
- `RN-INV-052` — En todos los métodos, **la venta descuenta según la receta**. El método solo determina cómo se verifica el remanente en el conteo físico.
- `RN-INV-053` — Con método `apertura`, abrir un envase genera una salida por su contenido completo y crea un envase en servicio. Las ventas posteriores de ese insumo no vuelven a descontar stock hasta que se abra el siguiente.
- `RN-INV-054` — Con método `peso`, el insumo declara la tara de su envase. El conteo registra el peso bruto y el sistema deriva el contenido restando la tara.
- `RN-INV-055` — Con método `nivel`, el conteo registra una fracción del envase (⅓, ½, ¾) y el sistema la convierte a la unidad base. Se asume imprecisión: la diferencia resultante no dispara alerta salvo que supere el umbral configurado.
- `RN-INV-056` — Un envase en servicio pertenece a una bodega concreta. Trasladarlo entre bodegas es un movimiento de traslado por su contenido restante.
- `RN-INV-057` — Puede haber varios envases en servicio del mismo insumo en la misma bodega. El conteo los verifica uno por uno.

### Producción interna

- `RN-INV-058` — Una receta puede producir un **insumo** en lugar de un producto vendible. Esa receta es una *receta de producción*.
- `RN-INV-059` — El insumo producido entra al inventario únicamente a través de una orden de producción confirmada. No se puede ingresar como entrada manual con motivo *producción*.
- `RN-INV-060` — Al confirmarse una orden de producción se generan, en un solo acto e indivisible, las salidas de los insumos consumidos y la entrada del insumo producido. Si falla una parte, no se aplica ninguna.
- `RN-INV-061` — El costo unitario del insumo producido se deriva: costo total de los insumos consumidos dividido entre el rendimiento real. Nunca se digita.
- `RN-INV-062` — Cuando una orden produce varios insumos a la vez (despiece), el costo total se reparte entre ellos según la proporción declarada en la receta de producción. La suma de las proporciones es 100%.
- `RN-INV-063` — La diferencia entre el rendimiento esperado y el real se registra como merma de producción, con su costo.
- `RN-INV-064` — Una orden de producción confirmada es inmutable. Se corrige anulándola, lo que genera los movimientos inversos.
- `RN-INV-065` — Un insumo producido puede a su vez ser insumo de otra producción. No se admiten ciclos.
- `RN-INV-066` — Si un insumo consumido no tiene existencia suficiente, la orden **se rechaza** — a diferencia de la venta, que nunca se bloquea (`RN-INV-012`). Producir es una decisión interna que sí puede esperar.

### Disponibilidad y conexión

- `RN-INV-067` — Inventario **no tiene modo sin conexión**. Sin conexión no registra movimientos, no los encola localmente y no muestra existencias.
- `RN-INV-068` — Un movimiento que llega tarde porque su módulo de origen estuvo sin conexión se registra con **dos fechas**: la del hecho original y la de su recepción. El descuento se aplica al recibirse.
- `RN-INV-069` — Un movimiento con fecha de hecho anterior al cierre de un conteo físico ya cerrado **no altera ese conteo**. Se registra, afecta el stock actual y genera una alerta de llegada tardía para que alguien lo revise.

---

## 8. Historias de usuario

### `HU-INV-001` — Registrar entrada de mercancía

> Como **bodeguero** quiero registrar la mercancía que llega para que el stock refleje lo
> que realmente tengo y el costo quede actualizado.

- `CA-1` — Dado un insumo con stock 10 a CPP $20.000 COP, cuando registro entrada de 10 unidades a $30.000 COP, entonces el stock queda en 20 y el CPP en $25.000 COP.
- `CA-2` — Dado un insumo con unidad base *ml* y unidad de compra *botella (750 ml)*, cuando registro entrada de 2 botellas, entonces el stock aumenta en 1.500 ml.
- `CA-3` — Dado un insumo con control de lote, cuando intento confirmar una entrada sin lote ni vencimiento, entonces el sistema la rechaza indicando los campos faltantes.
- `CA-4` *(rechazo)* — Dado que soy un usuario con rol Mesero, cuando intento registrar una entrada, entonces el sistema me lo niega por falta de permiso.

### `HU-INV-002` — Descontar stock automáticamente al vender

> Como **administrador de sede** quiero que el inventario se descuente solo con cada venta
> para no llevar el control a mano.

- `CA-1` — Dado un producto *Cuba Libre* con receta (50 ml ron, 200 ml gaseosa) y stock suficiente, cuando el POS confirma la venta de 2 unidades, entonces se descuentan 100 ml de ron y 400 ml de gaseosa de la bodega de venta, en un movimiento con origen `pos` y referencia a la venta.
- `CA-2` — Dado el mismo producto con modificador *doble*, cuando se vende 1 con ese modificador, entonces se descuentan 100 ml de ron y 200 ml de gaseosa.
- `CA-3` — Dado un producto sin receta, cuando se vende, entonces no se genera ningún movimiento de inventario y no se reporta error.
- `CA-4` *(rechazo/alerta)* — Dado un insumo con stock 30 ml, cuando se confirma una venta que consume 50 ml, entonces la venta **no se bloquea**, el stock queda en −20 ml y se genera una alerta de inconsistencia.
- `CA-5` — Dado que el módulo `pos` no está instalado, cuando el bodeguero registra una salida manual con motivo *venta*, entonces el sistema la acepta igual que la automática.

### `HU-INV-003` — Registrar una merma

> Como **jefe de barra** quiero registrar una botella rota para que el stock refleje la
> realidad y la pérdida quede atribuida.

- `CA-1` — Dado un insumo con stock 12 y CPP $40.000 COP, cuando registro merma de 1 con motivo *rotura*, entonces el stock queda en 11, el movimiento registra pérdida de $40.000 COP, mi usuario y la hora.
- `CA-2` *(rechazo)* — Cuando intento confirmar una merma sin motivo, entonces el sistema la rechaza.
- `CA-3` *(rechazo)* — Dado un umbral de autorización de $100.000 COP, cuando registro una merma de $150.000 COP sin autorización de un supervisor, entonces el movimiento queda en borrador y no afecta el stock.
- `CA-4` *(rechazo)* — Dado stock 0 y stock negativo deshabilitado, cuando intento registrar merma de 1, entonces el sistema la rechaza indicando existencia insuficiente.

### `HU-INV-004` — Hacer un conteo físico

> Como **administrador de sede** quiero contar físicamente la bodega para corregir las
> diferencias y saber cuánto se está perdiendo.

- `CA-1` — Cuando inicio un conteo de la bodega Barra, entonces el sistema congela la cantidad teórica de cada insumo del alcance.
- `CA-2` — Dado un insumo con teórico 40 y contado 37, cuando cierro el conteo, entonces se genera un ajuste de −3 con motivo *diferencia de conteo* y el stock queda en 37.
- `CA-3` — Dado que durante el conteo se vendieron 2 unidades de ese insumo, cuando cierro el conteo, entonces la diferencia se calcula contra el teórico congelado (40), no contra el stock actual.
- `CA-4` *(rechazo)* — Dado un conteo cerrado, cuando intento modificarlo, entonces el sistema lo impide y sugiere crear un conteo nuevo.

### `HU-INV-005` — Saber qué reponer

> Como **encargado de compras** quiero ver qué está por agotarse para pedir a tiempo.

- `CA-1` — Dado un insumo con mínimo 10 y stock 8, entonces aparece en la lista de reposición con la cantidad sugerida hasta el máximo.
- `CA-2` — Cuando el stock vuelve a superar el mínimo, entonces la alerta se cierra sola.
- `CA-3` — Dado que el módulo `compras` no está instalado, entonces la lista de reposición sigue disponible y es exportable.

### `HU-INV-006` — Conocer el costo de lo que vendo

> Como **dueño** quiero saber cuánto me cuesta cada producto para fijar precios con criterio.

- `CA-1` — Dada una receta con 50 ml de ron a $100 COP/ml y 200 ml de gaseosa a $5 COP/ml, entonces el costo de la receta es $6.000 COP.
- `CA-2` — Dada una merma esperada del 5% en la receta, entonces el costo se calcula sobre la cantidad con merma incluida.
- `CA-3` — Cuando el CPP del ron cambia por una entrada nueva, entonces el costo de la receta se actualiza para ventas futuras, y las ventas ya registradas conservan su costo original.
- `CA-4` *(rechazo)* — Dado un insumo de la receta sin costo registrado, entonces el costo de la receta se marca como incompleto y el sistema señala cuál insumo falta.

### `HU-INV-007` — Trasladar entre bodegas

> Como **bodeguero** quiero pasar producto de la bodega principal a la barra.

- `CA-1` — Dado stock 50 en Principal y 5 en Barra, cuando traslado 10, entonces queda 40 y 15, en un solo movimiento con origen y destino.
- `CA-2` — El costo unitario viaja con el traslado: la bodega destino recalcula su CPP con el costo de origen.
- `CA-3` *(rechazo)* — Cuando intento trasladar a una bodega de otra sede, entonces el sistema lo rechaza indicando que debe registrarse como salida y entrada.

### `HU-INV-008` — Auditar qué pasó con un insumo

> Como **administrador de sede** quiero ver el historial completo de un insumo para entender
> una diferencia.

- `CA-1` — Cuando abro el kardex de un insumo en una bodega, entonces veo todos sus movimientos en orden cronológico, con saldo y CPP después de cada uno.
- `CA-2` — Cada movimiento muestra responsable, motivo, origen y referencia (venta, recepción, conteo).
- `CA-3` — Un movimiento anulado aparece junto a su movimiento inverso; ninguno desaparece.

---

### `HU-INV-009` — Producir un almíbar

> Como **bartender** quiero registrar el almíbar que preparo para que su costo sea real y
> los cocteles que lo usan queden bien costeados.

- `CA-1` — Dada una receta de producción (1000 g azúcar + 1000 ml agua → 1600 ml almíbar) y existencias suficientes, cuando confirmo la orden con rendimiento real de 1550 ml, entonces se generan las salidas de azúcar y agua, la entrada de 1550 ml de almíbar y una merma de producción de 50 ml.
- `CA-2` — El costo unitario del almíbar se deriva: costo total consumido ÷ 1550 ml. No lo digito yo.
- `CA-3` *(rechazo)* — Dado que hay 800 g de azúcar y la receta pide 1000 g, cuando intento confirmar, entonces la orden **se rechaza entera** y no se consume nada. Producir sí puede esperar, a diferencia de vender.
- `CA-4` — Dado un despiece que produce lomo y recortes, cuando confirmo la orden, entonces el costo total se reparte entre ambos según la proporción declarada, y la suma es el costo de la pieza original.
- `CA-5` *(rechazo)* — Cuando intento ingresar almíbar como entrada manual con motivo *producción*, entonces el sistema lo rechaza e indica que debe hacerse con una orden de producción.

### `HU-INV-010` — Controlar una botella empezada

> Como **jefe de barra** quiero saber cuánto queda realmente en las botellas abiertas para
> detectar si se está sirviendo de más.

- `CA-1` — Dado un ron con método `peso` y tara de 500 g, cuando en el conteo registro un peso bruto de 890 g, entonces el sistema deriva 390 ml de contenido y calcula la diferencia contra el teórico.
- `CA-2` — Dado el mismo ron con teórico 450 ml y verificado 390 ml, entonces se genera un ajuste de −60 ml con motivo *diferencia de conteo*.
- `CA-3` — Dado un insumo con método `nivel` y una diferencia dentro del umbral configurado, entonces **no** se genera alerta: el método asume imprecisión.
- `CA-4` — Dada una cerveza con método `unidad`, cuando se vende una, entonces se descuenta una unidad y no existe envase en servicio.
- `CA-5` — Dado un insumo con método `apertura`, cuando abro una botella de 750 ml, entonces se descuentan 750 ml del stock y se crea un envase en servicio; las ventas siguientes no vuelven a descontar hasta la próxima apertura.
- `CA-6` *(rechazo)* — Cuando intento registrar un peso bruto menor que la tara declarada, entonces el sistema lo rechaza indicando que el envase no puede pesar menos que vacío.

### `HU-INV-011` — Compartir insumos entre sedes

> Como **dueño de dos sedes** quiero definir los insumos una sola vez y aun así ver el costo
> real de cada local.

- `CA-1` — Dado un insumo `global`, cuando lo creo en el tenant, entonces está disponible en todas las sedes sin volver a crearlo.
- `CA-2` — Dado ese insumo con entradas a distinto precio en dos sedes, entonces cada sede mantiene su propio costo promedio, y eso no es un error.
- `CA-3` — Dado un insumo que solo usa una sede, cuando lo creo como `local`, entonces no aparece en las demás sedes.
- `CA-4` *(rechazo)* — Cuando intento usar un insumo `local` en una receta `global`, entonces el sistema lo rechaza y ofrece promover el insumo a `global`.
- `CA-5` *(rechazo)* — Dado que soy Administrador de una sede, cuando intento crear un insumo `global`, entonces el sistema me lo niega por falta de alcance.

---

## 9. Casos límite y errores

| Situación | Comportamiento definido |
|---|---|
| Venta que deja stock negativo | Se registra igual + alerta (`RN-INV-012`). |
| Venta de un producto sin receta | No genera movimiento, no es error (`RN-INV-019`). |
| Insumo de la receta inactivo | La venta descuenta igual; se genera alerta de configuración. |
| Dos movimientos simultáneos sobre la misma existencia | Se procesan en serie; el CPP se recalcula en el orden de confirmación. Nunca se pierde un movimiento. |
| Sin conexión | Inventario no opera: no registra, no encola, no muestra existencias (`RN-INV-067`). Lo que otro módulo haya encolado se aplica al reconectar, con fecha de hecho y fecha de recepción (`RN-INV-068`). |
| Conversión entre dimensiones (ml → g) | **Siempre rechazada** (`RN-INV-046`). El caso real se resuelve declarando el contenido del empaque en la unidad base: un tetrapak = 1030 g. |
| Entrada con costo cero | Permitida con motivo (`RN-INV-017`). |
| Insumo con stock en una bodega que se desactiva | No se permite desactivar una bodega con stock ≠ 0. Debe trasladarse o ajustarse antes. |
| Receta con ciclo (A contiene B que contiene A) | Rechazada al guardar (`RN-INV-025`). |
| Cambio de unidad base de un insumo con movimientos | Prohibido. Debe crearse un insumo nuevo. |
| Movimiento que llega después de cerrado un conteo | Se registra y afecta el stock actual, pero no altera el conteo cerrado. Genera alerta de llegada tardía (`RN-INV-069`). |
| Orden de producción sin existencia suficiente | Rechazada entera (`RN-INV-066`). A diferencia de la venta, producir sí puede esperar. |
| Envase con método `nivel` y diferencia pequeña en el conteo | No dispara alerta si está dentro del umbral configurado: el método asume imprecisión (`RN-INV-055`). |
| Insumo `local` usado en una receta `global` | Rechazado al guardar. Hay que promover el insumo a `global` (`RN-INV-042`). |
| Despiece que rinde menos de lo esperado | La diferencia se registra como merma de producción con su costo (`RN-INV-063`). |
| Importación masiva con filas inválidas | Se importan las válidas, se reporta línea por línea lo rechazado. No se importa parcialmente un movimiento. |

---

## 10. Interfaces funcionales

| Interfaz | Dirección | Con quién | Qué información | Si el otro no existe |
|---|---|---|---|---|
| Venta confirmada | ◄ entra | `pos` | Producto, cantidad, modificadores, sede, estación, fecha, referencia | Salida manual con motivo *venta*, o importación de archivo de ventas |
| Recepción de mercancía | ◄ entra | `compras` | Insumo, cantidad, unidad, costo, proveedor, lote, referencia | Entrada manual con costo digitado |
| Identidad de productos | ◄ entra | `catalogo` | Producto vendible y sus modificadores | Inventario declara localmente los productos a los que asocia recetas |
| Sedes y bodegas activas | ◄ entra | `tenancy` | Sedes del tenant | No aplica: módulo de plataforma obligatorio |
| Usuarios y permisos | ◄ entra | `iam` | Identidad y rol del responsable | No aplica: módulo de plataforma obligatorio |
| Existencia de un insumo | ► sale | cualquiera | Cantidad y costo en una bodega y fecha | — |
| Disponibilidad de un producto | ► sale | `pos`, `pedidos-qr`, `catalogo` | Cuántas unidades alcanzan los insumos de su receta | — |
| Costo de receta | ► sale | `catalogo`, `reportes` | Costo unitario derivado | — |
| Alertas de stock | ► sale | `notificaciones`, `compras` | Insumo, bodega, tipo de alerta | Se consultan desde el propio módulo |
| Valorización | ► sale | `reportes` | Valor del inventario a una fecha, por sede y bodega | Informe propio del módulo |

> **Nota:** ninguna de estas interfaces implica una decisión técnica. Describen qué
> información cruza, no cómo.

---

## 11. Dependencias de localización

| Qué | Interfaz de país | Por qué no vive aquí |
|---|---|---|
| Moneda y formato de los costos | Moneda y formato | Inventario no sabe que la moneda es COP ni que no lleva decimales. |
| Impuestos incluidos en el costo de compra | Cálculo de tributos | Si el IVA de compra es descontable o va al costo lo decide el país, no el inventario. |
| Identificación del proveedor en una entrada | Identificación fiscal | NIT es un concepto colombiano. |
| Exigencia de soporte documental de una merma | Documento fiscal | Algunos regímenes exigen documentar bajas de inventario. |

**Este módulo no contiene ninguna tasa, porcentaje legal ni formato de documento.**
(`RN-ARQ-020`)

---

## 12. Permisos

| Acción | Propietario | Admin sede | Supervisor | Bodega | Compras | Jefe estación | Cajero/Mesero |
|---|---|---|---|---|---|---|---|
| Ver existencias | ✔ | ✔ | ✔ | ✔ | ✔ | ✔ (su estación) | — |
| Ver costos y valorización | ✔ | ✔ | — | ✔ | ✔ | — | — |
| Crear/editar insumos y bodegas | ✔ | ✔ | — | — | — | — | — |
| Registrar entrada | ✔ | ✔ | — | ✔ | ✔ | — | — |
| Registrar salida / traslado | ✔ | ✔ | — | ✔ | — | — | — |
| Registrar merma | ✔ | ✔ | ✔ | ✔ | — | ✔ | — |
| Autorizar merma sobre umbral | ✔ | ✔ | ✔ | — | — | — | — |
| Crear/editar recetas | ✔ | ✔ | — | — | — | ✔ (propuesta) | — |
| Ejecutar conteo | ✔ | ✔ | ✔ | ✔ | — | — | — |
| Cerrar conteo | ✔ | ✔ | — | — | — | — | — |
| Anular movimiento | ✔ | ✔ | — | — | — | — | — |
| Ejecutar orden de producción | ✔ | ✔ | ✔ | ✔ | — | ✔ | — |
| Anular orden de producción | ✔ | ✔ | — | — | — | — | — |
| Crear/editar insumos `global` | ✔ | — | — | — | — | — | — |
| Crear/editar insumos `local` | ✔ | ✔ | — | — | — | — | — |
| Abrir envase / registrar peso en conteo | ✔ | ✔ | ✔ | ✔ | — | ✔ | — |
| Configurar mínimos y umbrales | ✔ | ✔ | — | — | — | — | — |

---

## 13. Reportes que entrega

- **Kardex** de un insumo en una bodega: movimientos, saldo y CPP tras cada uno.
- **Existencias actuales** por bodega, con valorización.
- **Valorización del inventario** a una fecha (sede, bodega, categoría).
- **Mermas** por periodo, motivo, responsable y estación, con importe.
- **Diferencias de conteo** por periodo, con tendencia por insumo.
- **Consumo teórico vs. real** — el reporte que detecta fugas: lo que las ventas dicen que
  se consumió contra lo que el conteo dice que falta.
- **Rotación** de insumos: cuáles se mueven y cuáles llevan meses quietos.
- **Reposición sugerida**: insumos bajo mínimo y cantidad hasta el máximo.
- **Costo de recetas** y su evolución en el tiempo.
- **Producción interna**: qué se produjo, cuánto rindió frente a lo esperado y a qué costo.
- **Envases abiertos**: cuántos hay en servicio, desde cuándo y con cuánto contenido teórico.
- **Próximos a vencer**, por lote.

---

## 14. Configuración

| Parámetro | Nivel | Por defecto |
|---|---|---|
| Permitir stock negativo | Sede | No |
| Método de valoración | Tenant | Costo promedio ponderado |
| Bodega de venta por defecto | Sede | La primera bodega marcada *de venta* |
| Umbral de autorización de merma | Sede | Sin umbral |
| Umbral de autorización de cierre de conteo | Sede | Sin umbral |
| Días de antelación de alerta de vencimiento | Sede | 30 |
| Motivos de movimiento | Tenant | Catálogo por defecto (5.8) |
| Categorías de insumo | Tenant | Catálogo por defecto |
| Exigir conteo antes del cierre de mes | Sede | No |
| Método de control de envase abierto por defecto | Tenant | `unidad` |
| Umbral de diferencia aceptable con método `nivel` | Sede | Por definir (`PA-INV-012`) |
| Ámbito por defecto al crear un insumo | Tenant | `global` |
| Exigir autorización para confirmar orden de producción | Sede | No |

---

## 15. Fuera de alcance de este módulo

| No hace | Lo hace |
|---|---|
| Precios de venta | `catalogo` |
| Órdenes de compra y proveedores formales | `compras` |
| Cuentas por pagar | Fuera del producto (software contable) |
| Producción y transformación compleja (despiece, maduración) | Ver `PA-INV-006` |
| Trazabilidad sanitaria / HACCP | Fuera del producto por ahora |
| Inventario de activos fijos (mesas, equipos) | Fuera del producto |
| Contabilización de los movimientos | Fuera del producto; se exporta |

---

## 16. Preguntas abiertas

### Cerradas en la sesión del 2026-09-14

| Id | Resolución |
|---|---|
| `PA-INV-001` | **Maestro común + locales.** Un insumo tiene ámbito `global` (tenant) o `local` (sede). Existencias y costos siempre por sede y bodega. → `RN-INV-038`…`044` |
| `PA-INV-002` | **Costo promedio ponderado**, confirmado. Sin FIFO ni costo estándar. |
| `PA-INV-003` | **Prohibida la conversión entre dimensiones.** No hay densidad. El caso real se resuelve con **empaques que declaran su contenido en la unidad base**: un tetrapak de crema = 1030 g. → `RN-INV-045`…`050` |
| `PA-INV-005` | **Lote y vencimiento opcionales por insumo.** Los perecederos de cocina sí, el licor no. Salida sugerida por vencimiento más próximo (FEFO). |
| `PA-INV-006` | **Sí, con orden de producción.** Documento formal que consume insumos y produce otro insumo con costo real derivado. Soporta subproductos (despiece). → `RN-INV-058`…`066` |
| `PA-INV-010` | **Configurable por insumo:** `unidad`, `nivel`, `peso` o `apertura`. La venta siempre descuenta por receta; el método define cómo se verifica el remanente. → `RN-INV-051`…`057` |
| `PA-INV-004` | **Inventario no tiene modo sin conexión.** Sin red no registra, no encola y no muestra existencias. → `RN-INV-067`…`069`. *Queda una consecuencia por confirmar: ver `PA-INV-011`.* |

### Abiertas

- `PA-INV-007` — ¿Se necesita inventario de envases retornables y su control de devolución? Relevante en Colombia para cerveza y gaseosa en vidrio.
- `PA-INV-008` — ¿Quién define las recetas en la práctica: el administrador o el chef/bartender? Si es el segundo, hace falta un flujo de propuesta y aprobación.
- `PA-INV-009` — ¿El conteo físico se hace con un dispositivo móvil en la bodega? Cambia la experiencia, no las reglas. Con método `peso` implica báscula conectada o digitación manual.
- `PA-INV-011` — **Consecuencia de `PA-INV-004`.** Si el POS vende durante un corte de red, esas ventas llegan tarde. ¿Se aplican al inventario cuando vuelve la conexión, o se descartan y la diferencia se corrige con un conteo? `RN-INV-068` asume lo primero; falta confirmarlo.
- `PA-INV-012` — ¿Qué umbral de diferencia se considera aceptable con método `nivel` antes de generar alerta? Necesario para que `RN-INV-055` sea verificable.

## 17. Checklist de cierre

- [x] Las 16 secciones anteriores están completas
- [x] El contrato de autonomía declara los cuatro puntos
- [x] El módulo hace algo útil si es lo único instalado
- [x] Todo dato que consume tiene alternativa manual
- [x] Todos los datos están atados a tenant y a sede
- [x] No hay tasas, impuestos, formatos legales ni festivos dentro del módulo
- [x] Todos los importes llevan moneda
- [x] Todo término nuevo está en el glosario
- [x] Toda historia tiene al menos un criterio de rechazo
- [ ] **No quedan preguntas abiertas** ← 5 pendientes (eran 10; 7 cerradas, 2 nuevas derivadas)
- [x] La ficha se entiende sin abrir la ficha de otro módulo
- [ ] **Revisada y aprobada por Jorge**
