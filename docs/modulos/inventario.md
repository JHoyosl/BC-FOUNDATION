# Módulo: `inventario` — Inventario

> **Estado:** 🟡 Borrador · **Dueño:** Jorge Hoyos · **Orden:** 1 (en curso) · **Actualizado:** 2026-09-15

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
| **Funciona solo** | Catálogo de insumos (global por tenant o local por sede) con unidades, dimensiones y empaques · bodegas · existencias · entradas, salidas, traslados, ajustes y mermas manuales · envases abiertos por estación con cierre y apertura de turno · fichas técnicas (recetas) y costeo · órdenes de producción interna · costo promedio ponderado · lotes y vencimientos · conteos físicos · alertas de mínimo · valorización del inventario · trazabilidad completa de movimientos. Un negocio que ya tiene POS de otro proveedor puede contratar solo Inventario y usarlo entero. |
| **Degradación** | **Sin `pos`:** las salidas por venta se registran manualmente o por importación de un archivo de ventas; la estación se declara *por archivo* y sus diferencias esperan a que llegue el archivo que cubre el turno. **Sin `compras`:** las entradas se registran como entrada manual con costo digitado, sin orden de compra ni proveedor formal (queda un campo libre de proveedor). **Sin `catalogo`:** las recetas se definen contra un "producto vendible" declarado localmente por Inventario, no contra el catálogo comercial, y no se muestra la venta perdida. **Sin `kds` ni `pos`:** cada bodega de venta opera con su estación por defecto. **Sin `reportes`:** Inventario entrega sus propios informes básicos. |
| **Consume** | De `pos`: ventas confirmadas (producto, cantidad, modificadores, sede, estación, fecha y hora) → para descontar insumos vía receta. De `compras`: recepciones de mercancía (insumo, cantidad, costo, proveedor) → para ingresar stock. De `catalogo`: identidad de los productos vendibles y su precio de carta → para asociar recetas y calcular la venta perdida. De `kds` o `pos`: estaciones → para atribuir envases abiertos. De `tenancy`: sedes activas. De `iam`: usuarios y permisos. **Todos con alternativa manual.** |
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
| **Bartender / cocinero de turno** | Abre, finaliza y presta los envases de su estación. Hace el cierre y la apertura de su turno. |

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
- **Tipo de control** — Cómo se lleva la existencia de un insumo: `unidad` (indivisible),
  `granel` (sin envase) o `envase abierto` (se abre y se sirve de él durante un tiempo). Se configura
  por insumo.
- **Verificación** — Para `envase abierto`, cómo se mide lo que queda: `al finalizar`, `nivel` o
  `peso`.
- **Estación** — Punto de preparación que consume de una bodega: cocina, barra, pastelería. Toda
  bodega de venta tiene una estación por defecto. Ver [glosario](../04-glosario.md).
- **Envase en servicio** — Envase abierto de un insumo, a cargo de la **estación** que lo abrió.
  Tiene historia propia: quién lo abrió, cuándo, qué se le vendió y cada medición.
- **Turno de estación** — Periodo entre la apertura y el cierre de una estación, en el que esa
  estación responde por sus envases en servicio.
- **Diferencia de envase** — Lo medido menos lo que las ventas dicen que debería quedar. Faltante
  si es negativa, sobrante si es positiva.
- **Diferencia entre turnos** — Lo que cambió entre el cierre de un turno y la apertura del
  siguiente, cuando la estación no estaba operando. No se atribuye a ninguno de los dos turnos.
- **Nota de venta tardía** — Registro de que unas ventas llegaron después de una medición y
  explican parte de su diferencia.
- **Porción de referencia** — Cantidad que representa una porción de un insumo (un trago de 50 ml).
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
- **Tipo de control** (ver 5.3): `unidad` · `granel` · `envase abierto`
- Solo para `envase abierto`:
  - **Verificación**: `al finalizar` · `nivel` · `peso`
  - **Porción de referencia** (ej. 50 ml), opcionalmente asociada a un producto vendible
  - **Tolerancia de diferencia**, en % del consumo
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

Un empaque de un insumo que se verifica por `peso` declara además su **peso vacío** y su **peso
lleno**:

```
Insumo: Whisky 12 años
  Unidad base: ml  (dimensión: volumen)
  Empaques:
    · Botella 750 = 750 ml · peso vacío 500 g · peso lleno 1.250 g
```

Con un peso bruto de 875 g, el contenido es (875 − 500) ÷ (1.250 − 500) × 750 = **375 ml**.

**Por qué así:** la báscula no convierte gramos en mililitros; solo dice **qué fracción del
envase queda**. No hay densidad (`RN-INV-046`), y el peso vacío vive en el empaque y no en el
insumo: dos botellas del mismo licor pueden pesar distinto vacías.

### 5.3 Tipo de control y verificación

> Resuelve `PA-INV-010` (2026-09-14) · **revisada 2026-09-15**.

Cada insumo declara **cómo se lleva su existencia**. Es configurable por insumo porque no
compensa el mismo rigor para un whisky de 300.000 COP que para una caja de cerveza.

| Tipo | Cómo funciona | Ejemplos |
|---|---|---|
| `unidad` | El empaque es indivisible. La venta descuenta unidades enteras. | Cerveza en botella, gaseosa en lata |
| `granel` | No viene en un envase que se abra. La venta descuenta por receta de la bodega y se verifica en el conteo físico. | Limón, carne, hielo |
| `envase abierto` | Se abre un envase y se sirve de él durante un tiempo. La venta descuenta del **envase en servicio** de la estación que prepara. | Ron, whisky, aceite, crema en tetrapak |

Un insumo `envase abierto` declara además **cómo se verifica** lo que queda:

| Verificación | Cómo se mide | Para qué sirve |
|---|---|---|
| `al finalizar` | No se mide mientras está abierto. Al acabarse se marca finalizado y la medición es 0. | El caso general. Nadie estima a ojo |
| `nivel` | Se estima una fracción del envase (¼, ⅓, ½, ¾). | Control diario sin equipo |
| `peso` | Se pesa el envase; el contenido sale del peso vacío y lleno del empaque (5.2). | Licor premium, insumos caros |

**En todos los tipos, la venta descuenta según la receta.** La verificación solo define **cuándo y
con qué precisión** se compara lo vendido contra lo que queda.

### 5.4 Envase en servicio
Solo para insumos `envase abierto`. Es un envase concreto que ya se abrió.

- Insumo, **estación** a cargo y bodega de la que salió
- Empaque de origen y contenido inicial
- Apertura: responsable, hora de registro y **hora declarada** (si se registró tarde)
- Ventas cargadas y **contenido teórico restante**
- **Porciones teóricas restantes** (contenido ÷ porción de referencia)
- Mediciones: valor, cuándo y en qué (cierre, apertura, conteo, finalización)
- ml pendientes recibidos de ventas sin envase en servicio, con su aviso
- Préstamos a otras estaciones, con o sin medición
- Lote y vencimiento, si el insumo los controla
- Estado: `en servicio` · `finalizado` · `descartado`
- Diferencia al finalizar, y versiones si se reabrió

**Así se ve la existencia** de un insumo `envase abierto` en una bodega:

```
Ron añejo · Bodega Barra
  Cerradas      10 botellas                                          7.500 ml
  En servicio   Barra terraza · abierta 21:10 · quedan 6 porciones     300 ml
                                                              Total   7.800 ml
```

### 5.5 Bodega
Lugar de almacenamiento. Pertenece a una **sede**.

- Nombre, tipo (bodega principal, barra, cocina, nevera)
- ¿Permite venta directa desde ella? (define de dónde descuenta el POS)
- **Estación por defecto** (solo bodegas de venta): la que consume de esta bodega cuando no se indica
  otra. Se crea con la bodega y se puede cambiar (`RN-INV-112`)
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
- Para insumos `envase abierto`: envases **cerrados** + contenido de los **envases en servicio** de
  las estaciones que sacan de esa bodega (`RN-INV-077`)

### 5.7 Movimiento de inventario
El hecho que cambia la existencia. **Es el corazón del módulo.**

- Tipo: `entrada` · `salida` · `traslado` · `ajuste` · `merma`
- Subtipo/motivo (ver 5.8)
- Insumo, bodega origen y/o destino
- Cantidad y unidad en que se registró + cantidad convertida a unidad base
- Costo unitario y costo total (importe con moneda)
- Origen: `manual` · `pos` · `compras` · `conteo` · `importacion` · `estación`
- Referencia de origen (id de venta, de recepción, de conteo, de turno de estación)
- Responsable, **fecha del hecho y fecha de recepción** (iguales salvo llegada tardía), nota
- Estado: `borrador` · `confirmado` · `anulado`
- Lote y vencimiento (si el insumo los controla)

### 5.8 Motivos de movimiento
Catálogo configurable por tenant. Por defecto:

| Tipo | Motivos |
|---|---|
| Entrada | Compra · Devolución de cliente · Producción interna · Traslado entrante · Saldo inicial |
| Salida | Venta · Consumo interno · Cortesía · Traslado saliente · Devolución a proveedor |
| Merma | Rotura · Vencimiento · Derrame · Deterioro · Robo · Error de preparación |
| Ajuste | Diferencia de conteo · Diferencia de envase · Corrección de registro |

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
- **Incluir abiertas** (sí/no): mide también los envases en servicio (auditoría)
- Líneas marcadas **por recontar** (se abrió un envase durante el conteo)
- Estado: `abierto` · `en conteo` · `cerrado` · `anulado` · `reabierto`
- Ajustes generados al cerrar
- Versión anterior, si fue reabierto

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

### 5.13 Estación (configuración en inventario)

> Resuelve la titularidad de la estación (2026-09-15): inventario la reconoce por su
> identificador, venga del módulo que venga.

- Estación, reconocida por su **identificador** (`RN-INV-113`)
- Bodega de la que saca (la bodega de venta de `RN-INV-022`)
- ¿Es la estación por defecto de esa bodega?
- **Origen de ventas**: `en línea` · `por archivo`
- Plazo para alertar si no llegan las ventas de un turno (solo `por archivo`)

### 5.14 Turno de estación

- Sede, estación
- **Apertura**: responsable, hora, modo (`recibido conforme` · `medida`) y mediciones
- **Cierre**: responsable, hora y una línea por envase en servicio con su medición según la
  verificación (peso, nivel o "sigue ahí")
- Diferencia del turno, por envase y total
- Diferencia entre turnos (contra el cierre anterior)
- Estado: `abierto` · `pendiente de ventas` · `cerrado`
- Notas: de venta tardía, de corrección, "recalculado por reapertura"
- Versiones, si el cierre o la apertura se reabrieron

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
│ abierto │──────────►│ en conteo  │──────────►│ cerrado │──► genera ajustes
└────┬────┘           └─────┬──────┘           └────┬────┘
     │                      │                       │ reabrir (motivo + autorización)
     │ anular               │ anular                ▼
     ▼                      ▼                 ┌───────────┐
 ┌─────────┐          ┌─────────┐             │ reabierto │──► versión nueva
 │ anulado │◄─────────│ anulado │             └───────────┘
 └─────────┘          └─────────┘
```

Al pasar a **en conteo** se congela la cantidad teórica de cada línea. Al **cerrar**, cada
diferencia genera un movimiento de ajuste confirmado. Un conteo cerrado **puede reabrirse** con
motivo y autorización (`RN-INV-103`…`107`): sus ajustes se anulan con movimientos inversos y se
crea una versión nueva. Nunca se edita.

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
   abrir (estación · hora real opcional)
        │
        ▼
┌──────────────┐   finalizar (mide 0)   ┌────────────┐
│ en servicio  │───────────────────────►│ finalizado │
└──────┬───────┘                        └─────┬──────┘
       │ merma (rotura, vencimiento)          │ reabrir (motivo + autorización)
       ▼                                      ▼
┌──────────────┐                   vuelve a en servicio; si la estación ya
│ descartado   │                   tiene otro, el contenido corregido se
└──────────────┘                   suma a ese (RN-INV-108)
  genera merma por el
  contenido restante
```

**Prestar** a otra estación (medición opcional, `RN-INV-114`): el envase sigue `en servicio`, a
cargo de la estación de destino.

### 6.5 Turno de estación

```
      apertura (recibido conforme · medida)
             │
             ▼
      ┌────────────┐
      │  abierto   │
      └─────┬──────┘
            │ cierre (mide según verificación)
     ┌──────┴────────────┐
 en línea           por archivo
     │                   ▼
     │       ┌─────────────────────┐   llega el archivo que
     │       │ pendiente de ventas │   cubre el turno
     │       └──────────┬──────────┘
     ▼                  ▼
┌────────────────────────────┐
│          cerrado           │
└─────────────┬──────────────┘
              │ reabrir (hasta que cierre el turno siguiente)
              ▼
   versión nueva · la anterior queda como reabierta
```

- **abierto** → la estación opera; las ventas descuentan de sus envases en servicio.
- **pendiente de ventas** → ya se midió, pero la diferencia espera el archivo. Sin alertas.
- **cerrado** → la diferencia está calculada. Lo medido es el punto de partida del turno siguiente.

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
- `RN-INV-031` — *(derogada por `RN-INV-103`)* ~~Un conteo cerrado no se reabre ni se modifica. Un error se corrige con un nuevo conteo.~~
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

### Envases abiertos *(grupo derogado el 2026-09-15; ver los grupos desde "Tipo de control y verificación")*

- `RN-INV-051` — *(derogada por `RN-INV-070`)* ~~Cada insumo declara su método de control de envase abierto: `unidad`, `nivel`, `peso` o `apertura`. Por defecto, `unidad`.~~
- `RN-INV-052` — *(derogada por `RN-INV-074`)* ~~En todos los métodos, la venta descuenta según la receta. El método solo determina cómo se verifica el remanente en el conteo físico.~~
- `RN-INV-053` — *(derogada por `RN-INV-078`)* ~~Con método `apertura`, abrir un envase genera una salida por su contenido completo y crea un envase en servicio. Las ventas posteriores de ese insumo no vuelven a descontar stock hasta que se abra el siguiente.~~
- `RN-INV-054` — *(derogada por `RN-INV-075`)* ~~Con método `peso`, el insumo declara la tara de su envase. El conteo registra el peso bruto y el sistema deriva el contenido restando la tara.~~
- `RN-INV-055` — *(derogada por `RN-INV-076` y `RN-INV-086`)* ~~Con método `nivel`, el conteo registra una fracción del envase (⅓, ½, ¾) y el sistema la convierte a la unidad base. Se asume imprecisión: la diferencia resultante no dispara alerta salvo que supere el umbral configurado.~~
- `RN-INV-056` — *(derogada por `RN-INV-079`)* ~~Un envase en servicio pertenece a una bodega concreta. Trasladarlo entre bodegas es un movimiento de traslado por su contenido restante.~~
- `RN-INV-057` — *(derogada por `RN-INV-080`)* ~~Puede haber varios envases en servicio del mismo insumo en la misma bodega. El conteo los verifica uno por uno.~~

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
- `RN-INV-068` — *(derogada por `RN-INV-100` y `RN-INV-101`)* ~~Un movimiento que llega tarde porque su módulo de origen estuvo sin conexión se registra con dos fechas: la del hecho original y la de su recepción. El descuento se aplica al recibirse.~~
- `RN-INV-069` — *(derogada por `RN-INV-102`)* ~~Un movimiento con fecha de hecho anterior al cierre de un conteo físico ya cerrado no altera ese conteo. Se registra, afecta el stock actual y genera una alerta de llegada tardía para que alguien lo revise.~~

### Tipo de control y verificación

- `RN-INV-070` — Cada insumo declara su tipo de control: `unidad`, `granel` o `envase abierto`. Por defecto, `unidad`.
- `RN-INV-071` — Con tipo `unidad`, el empaque es indivisible: la venta descuenta unidades enteras y no existe envase en servicio.
- `RN-INV-072` — Con tipo `granel`, la venta descuenta por receta de la existencia de la bodega y el remanente se verifica solo en el conteo físico.
- `RN-INV-073` — Un insumo `envase abierto` declara su verificación (`al finalizar`, `nivel` o `peso`), su porción de referencia y su tolerancia de diferencia.
- `RN-INV-074` — En todos los tipos, la venta descuenta según la receta (`RN-INV-020`). Con tipo `envase abierto`, descuenta del envase en servicio de la estación que prepara el producto.
- `RN-INV-075` — Con verificación `peso`, el empaque declara peso vacío y peso lleno, y el contenido se deriva por proporción lineal entre ambos, en la unidad base. Un peso bruto menor que el vacío o mayor que el lleno se rechaza.
- `RN-INV-076` — Con verificación `nivel`, la medición registra una fracción del envase (¼, ⅓, ½, ¾) y el sistema la convierte a la unidad base con el contenido del empaque.

### Envase en servicio

- `RN-INV-077` — La existencia de un insumo `envase abierto` en una bodega es la suma de sus envases cerrados y del contenido de los envases en servicio de las estaciones que sacan de esa bodega. Ambos cuentan en la valorización.
- `RN-INV-078` — Abrir un envase no cambia la existencia ni el costo: pasa un envase cerrado a envase en servicio de la estación y registra estación, responsable y hora. Queda visible en el kardex.
- `RN-INV-079` — Un envase en servicio pertenece a la estación que lo abrió y sale de la bodega de esa estación (`RN-INV-022`). Varias estaciones pueden sacar de la misma bodega.
- `RN-INV-080` — Una estación tiene como máximo un envase en servicio por insumo. Para abrir otro, debe finalizar el anterior.
- `RN-INV-081` — Un envase en servicio muestra su contenido teórico restante y las porciones que le quedan según la porción de referencia del insumo.
- `RN-INV-082` — Descartar un envase en servicio (rotura, vencimiento) genera una merma por su contenido teórico restante.
- `RN-INV-083` — Una venta de un insumo `envase abierto` sin envase en servicio en su estación descuenta la existencia igual y genera una alerta de trazabilidad. El sistema no abre envases por su cuenta. Los ml quedan pendientes y se cargan al próximo envase que se abra en esa estación, con un aviso de lo realizado.
- `RN-INV-114` — Un envase en servicio puede prestarse a otra estación que saque de la misma bodega; medirlo al prestarlo es opcional. Con medición, se calcula la diferencia en la estación de origen y el envase arranca en la de destino con lo medido. Sin medición, pasa con su contenido teórico y la siguiente medición calcula una diferencia marcada *compartida entre estaciones*. Si la estación de destino ya tiene un envase en servicio de ese insumo, el contenido se suma a ese.

### Diferencia de envase

- `RN-INV-084` — Toda medición de un envase en servicio (cierre, apertura, conteo o finalización) calcula su diferencia: contenido medido menos contenido teórico. Finalizar es una medición de 0.
- `RN-INV-085` — La diferencia se registra como ajuste con motivo *diferencia de envase*, valorizado al CPP vigente en el momento de la medición (`RN-INV-015`). No es merma: su causa es desconocida.
- `RN-INV-086` — Una diferencia cuyo valor absoluto no supera la tolerancia del insumo, calculada sobre el consumo por ventas del periodo medido, se registra sin generar alerta.
- `RN-INV-087` — En una estación `en línea`, cuando lo vendido de un envase en servicio supera su contenido, se genera en ese momento una alerta de *envase excedido*.
- `RN-INV-088` — Si `catalogo` está instalado y la porción de referencia está asociada a un producto vendible, la diferencia se muestra además como venta perdida: diferencia ÷ porción × precio de carta de ese producto.

### Estaciones y turno de estación

- `RN-INV-112` — Toda bodega de venta tiene una estación por defecto que consume de ella. Se crea con la bodega y se puede cambiar por otra estación.
- `RN-INV-113` — Inventario relaciona bodegas y estaciones por el identificador de la estación y no depende del módulo que la haya creado. Sin ningún módulo que provea estaciones, cada bodega de venta opera con su estación por defecto.
- `RN-INV-089` — Un turno de estación va de una apertura a su cierre. Puede haber varios por día. Es un registro propio de inventario: no depende de `caja` ni de `schedule`.
- `RN-INV-090` — En el cierre se registra cada envase en servicio de la estación según su verificación: con `peso` se pesa, con `nivel` se estima y con `al finalizar` solo se confirma que el envase sigue ahí.
- `RN-INV-091` — El contenido medido en un cierre pasa a ser el contenido de partida del turno siguiente.
- `RN-INV-092` — El cierre de estación no cuenta envases cerrados, salvo que la bodega la use una sola estación; en ese caso puede incluirlos.
- `RN-INV-093` — En la apertura, quien recibe marca *recibido conforme* o mide. La diferencia entre lo medido en la apertura y lo registrado en el cierre anterior es *diferencia entre turnos* y no se atribuye a ninguno de los dos.
- `RN-INV-094` — Si una estación no hizo cierre, no se bloquea ninguna operación: se genera la alerta *estación sin cierre*, la apertura siguiente no admite *recibido conforme* y la diferencia de los dos turnos queda como una sola, marcada así.
- `RN-INV-095` — Cada estación declara su origen de ventas: `en línea` o `por archivo`.
- `RN-INV-096` — En una estación `por archivo`, las mediciones ajustan la existencia igual, pero la diferencia queda *pendiente de ventas*: no genera alertas ni entra en reportes hasta importar un archivo cuyo periodo cubra el turno completo. Esas ventas no generan nota de venta tardía.
- `RN-INV-097` — Si el archivo que cubre un turno pendiente no llega en el plazo configurado, se genera la alerta *turno sin ventas cargadas*.
- `RN-INV-098` — Un archivo de ventas sin hora por venta se concilia por día de operación y estación; la diferencia por envase y por turno queda marcada *sin detalle*. Si el archivo no trae estación, las ventas van a la estación por defecto de la bodega.
- `RN-INV-099` — Al registrar la apertura o la finalización de un envase se puede declarar la hora real. Se guardan la hora declarada y la de registro. La hora declarada debe estar dentro del turno abierto de la estación y no puede ser anterior al evento previo de ese insumo en la estación.

### Llegadas tardías

- `RN-INV-100` — Un movimiento que llega tarde se registra con dos fechas: la del hecho y la de recepción.
- `RN-INV-101` — Si después de la fecha del hecho no hubo ninguna medición del insumo en esa bodega o estación, el movimiento afecta la existencia al recibirse.
- `RN-INV-102` — Si después de la fecha del hecho hubo una medición (conteo, cierre, apertura o finalización), el movimiento **no cambia la existencia**: se registra con su fecha del hecho junto con un ajuste inverso que referencia el ajuste de esa medición, se recalcula la diferencia del periodo medido y queda una *nota de venta tardía* (*nota de movimiento tardío* si no es una venta). El documento de la medición no se modifica.

### Reapertura de mediciones

- `RN-INV-103` — Un cierre, una apertura, una finalización o un conteo físico pueden reabrirse. Reabrir no edita: anula con movimientos inversos los ajustes que generó (`RN-INV-006`), deja la versión original visible como *reabierta* y crea una versión nueva que la referencia.
- `RN-INV-104` — Mientras no haya habido operación sobre esos envases o esa bodega después de la medición, la versión nueva puede volver a medir. Después, solo corrige el valor registrado.
- `RN-INV-105` — Una medición puede reabrirse hasta que se cierre la siguiente medición del mismo alcance: el turno siguiente de la estación o, para un conteo, el siguiente conteo o cierre que mida esos insumos. Las diferencias que cambian quedan marcadas *recalculado por reapertura*.
- `RN-INV-106` — Pasado ese límite no se reabre ni se registran movimientos: una *nota de corrección* liga las diferencias afectadas como un mismo error, y los reportes las muestran compensadas.
- `RN-INV-107` — Reabrir exige motivo y un rol con permiso (Propietario, Administrador de sede o Supervisor). Quien hizo la medición no puede autorizar su propia reapertura, salvo el Propietario. Aplica `RN-ROL-004`.
- `RN-INV-108` — Si se reabre la finalización de un envase y la estación ya tiene otro en servicio del mismo insumo, el contenido corregido se suma al envase en servicio, con nota.

### Conteo físico y envases abiertos

- `RN-INV-109` — Por defecto, el conteo físico cuenta envases cerrados. Los envases en servicio de las estaciones que sacan de esa bodega aparecen con su última medición, o con su contenido teórico si nunca se midieron, y no generan diferencia de conteo.
- `RN-INV-110` — Un conteo marcado *incluir abiertas* mide los envases en servicio con `nivel` o `peso`, aunque su verificación sea `al finalizar`. La diferencia es *diferencia de envase* con origen `conteo`: entre turnos si la estación está cerrada, del turno en curso si está abierta. Si no coincide con el último cierre, queda marcada así, y lo medido es el punto de partida de la apertura siguiente.
- `RN-INV-111` — Si se abre un envase de un insumo mientras su bodega está en conteo, la línea de ese insumo queda *por recontar*. El conteo no se cierra sin recontarla o sin confirmar que se contó antes de la apertura.

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

- `CA-1` — Dado un producto *Cuba Libre* con receta (50 ml ron, 200 ml gaseosa), ron `envase abierto` con una botella en servicio en la Barra y gaseosa `granel`, cuando el POS confirma la venta de 2 unidades, entonces se descuentan 100 ml del envase en servicio de ron de la Barra y 400 ml de gaseosa de la bodega de venta, en un movimiento con origen `pos` y referencia a la venta.
- `CA-2` — Dado el mismo producto con modificador *doble*, cuando se vende 1 con ese modificador, entonces se descuentan 100 ml de ron y 200 ml de gaseosa.
- `CA-3` — Dado un producto sin receta, cuando se vende, entonces no se genera ningún movimiento de inventario y no se reporta error.
- `CA-4` *(rechazo/alerta)* — Dado un insumo `granel` con stock 30 ml, cuando se confirma una venta que consume 50 ml, entonces la venta **no se bloquea**, el stock queda en −20 ml y se genera una alerta de inconsistencia.
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
- `CA-4` *(rechazo)* — Dado un conteo cerrado, cuando intento editarlo, entonces el sistema lo impide y ofrece reabrirlo con motivo y autorización (`RN-INV-103`).
- `CA-5` — Dado un conteo de la bodega Barra sin *incluir abiertas*, entonces se cuentan las botellas cerradas de ron y la botella en servicio de Barra terraza aparece con los 440 ml de su último cierre, sin línea de diferencia.
- `CA-6` — Dado un conteo con *incluir abiertas*, la Barra terraza cerrada y su último cierre en 440 ml de whisky, cuando la auditoría mide 300 ml, entonces se registra una diferencia entre turnos de −140 ml marcada "no coincide con el cierre de las 03:00", y 300 ml es el punto de partida de la apertura siguiente.
- `CA-7` *(rechazo)* — Dado que la cocina abrió una botella de ron a las 10:15 durante el conteo, cuando intento cerrar el conteo, entonces el sistema lo impide hasta que recuente la línea del ron o confirme que la conté antes de las 10:15.

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

### `HU-INV-010` — Controlar una botella abierta

> Como **jefe de barra** quiero saber cuánto se sirvió realmente de cada botella abierta para
> detectar si se está sirviendo de más o se está perdiendo licor.

- `CA-1` — Dado un ron `envase abierto` con 10 botellas cerradas en la bodega Barra, cuando la Barra terraza abre una a las 21:10, entonces quedan 9 cerradas y un envase en servicio de 750 ml a cargo de Barra terraza, la existencia total no cambia y el kardex registra la apertura con responsable y hora.
- `CA-2` — Dada esa botella con porción de referencia de 50 ml y tolerancia del 3 %, cuando se venden 13 Cuba Libre (650 ml) y la finalizo, entonces se registra un ajuste de −100 ml con motivo *diferencia de envase* al CPP vigente y se genera una alerta, porque 100 ml superan el 3 % de 650 ml.
- `CA-3` — Dado un whisky con verificación `peso` y empaque de 750 ml con peso vacío 500 g y lleno 1.250 g, cuando registro un peso bruto de 875 g, entonces el sistema deriva 375 ml sin usar densidad.
- `CA-4` — Dada una botella en servicio de 750 ml en una estación `en línea`, cuando las ventas llegan a 800 ml sin que se haya finalizado, entonces se genera en ese momento una alerta de *envase excedido*.
- `CA-5` — Dado que no hay botella de ron en servicio en la Barra, cuando se venden 3 Cuba Libre (150 ml), entonces la venta no se bloquea, se genera una alerta de trazabilidad y, al abrir una botella a las 21:40, se le cargan los 150 ml con el aviso "se cargaron 150 ml de 3 ventas sin botella abierta".
- `CA-6` — Dadas la cocina caliente y pastelería sacando aceite de la misma despensa, cuando cada una abre su botella, entonces hay dos envases en servicio, uno por estación, y cada uno responde por sus ventas.
- `CA-7` — Dada una cerveza `unidad`, cuando se vende una, entonces se descuenta una unidad y no existe envase en servicio.
- `CA-8` *(rechazo)* — Dada una botella de ron en servicio en la Barra terraza, cuando intento abrir otra en la misma estación, entonces el sistema me pide finalizar la anterior.
- `CA-9` *(rechazo)* — Cuando registro un peso bruto de 450 g para un empaque con peso vacío de 500 g, entonces el sistema lo rechaza indicando que el envase no puede pesar menos que vacío.
- `CA-10` — Dada la botella de aceite en servicio de la cocina caliente, cuando se la presta a pastelería sin medirla, entonces pasa a cargo de pastelería con su contenido teórico, y la siguiente medición calcula una diferencia marcada *compartida entre estaciones*.

### `HU-INV-011` — Compartir insumos entre sedes

> Como **dueño de dos sedes** quiero definir los insumos una sola vez y aun así ver el costo
> real de cada local.

- `CA-1` — Dado un insumo `global`, cuando lo creo en el tenant, entonces está disponible en todas las sedes sin volver a crearlo.
- `CA-2` — Dado ese insumo con entradas a distinto precio en dos sedes, entonces cada sede mantiene su propio costo promedio, y eso no es un error.
- `CA-3` — Dado un insumo que solo usa una sede, cuando lo creo como `local`, entonces no aparece en las demás sedes.
- `CA-4` *(rechazo)* — Cuando intento usar un insumo `local` en una receta `global`, entonces el sistema lo rechaza y ofrece promover el insumo a `global`.
- `CA-5` *(rechazo)* — Dado que soy Administrador de una sede, cuando intento crear un insumo `global`, entonces el sistema me lo niega por falta de alcance.

### `HU-INV-012` — Cerrar y abrir el turno de una estación

> Como **bartender** quiero entregar y recibir las botellas abiertas al cambiar de turno para que
> cada turno responda solo por lo suyo.

- `CA-1` — Dada la Barra terraza con un whisky `peso`, un triple sec `nivel` y un ron `al finalizar` en servicio, cuando hago el cierre, entonces peso el whisky, estimo el triple sec y solo confirmo que el ron sigue ahí.
- `CA-2` — Dado el whisky con 400 ml teóricos, cuando en el cierre lo peso en 375 ml, entonces se registra un ajuste de −25 ml con motivo *diferencia de envase* y 375 ml es el contenido de partida del turno siguiente.
- `CA-3` — Dado ese cierre, cuando el turno siguiente abre con *recibido conforme*, entonces arranca con 375 ml y no hay diferencia entre turnos.
- `CA-4` — Dado ese cierre, cuando el turno siguiente mide el whisky en 250 ml, entonces se registra una diferencia entre turnos de −125 ml que no se atribuye a ninguno de los dos turnos.
- `CA-5` *(rechazo)* — Dado que la Barra terraza no hizo cierre, cuando el turno siguiente intenta abrir con *recibido conforme*, entonces el sistema lo rechaza y exige medir; además existe la alerta *estación sin cierre*.

### `HU-INV-013` — Recibir ventas que llegan tarde

> Como **administrador de sede** quiero que las ventas que llegan tarde no dañen la existencia
> ni culpen al turno equivocado.

- `CA-1` — Dado un whisky de 750 ml abierto a las 21:00, 6 ventas (300 ml) retenidas por un corte de red entre 22:00 y 23:00 y un cierre a las 03:00 que lo pesa en 440 ml con diferencia de −310 ml, cuando las ventas llegan a las 03:20, entonces la existencia sigue en 440 ml, la diferencia del turno pasa a −10 ml, la alerta se cierra y queda una nota de venta tardía.
- `CA-2` — Dado un ron `al finalizar` en servicio sin mediciones desde las 22:00, cuando llegan tarde ventas de las 22:30, entonces descuentan del envase normalmente.
- `CA-3` — Dada una estación `por archivo`, cuando a la 01:00 se finaliza una botella sin ventas cargadas, entonces la diferencia queda *pendiente de ventas* y no hay alerta; cuando a las 04:00 se importa el archivo del periodo 18:00–03:00, entonces se calcula la diferencia y se generan las alertas que correspondan, sin notas de venta tardía.
- `CA-4` — Dado un archivo que solo trae totales por día, cuando se importa, entonces la diferencia se calcula por día de operación y estación, y la de cada envase y turno queda *sin detalle*.
- `CA-5` — Dado un corte de red durante el cual una botella de ron se acabó a las 22:10 y se abrió otra, cuando al volver la red registro la finalización y la apertura declarando las 22:10, entonces las ventas de antes de las 22:10 van a la botella vieja y las de después a la nueva.
- `CA-6` *(rechazo)* — Dada una botella finalizada a las 22:10, cuando intento registrar la apertura de la siguiente declarando las 22:05, entonces el sistema lo rechaza por ser anterior a la finalización.

### `HU-INV-014` — Reabrir un cierre mal hecho

> Como **administrador de sede** quiero corregir un cierre equivocado sin perder el rastro de lo
> que se registró primero.

- `CA-1` — Dado un cierre de las 03:00 que registró el whisky en 240 ml (diferencia −210 ml) y la barra aún sin abrir, cuando lo reabro a las 11:20 con motivo "se pesó la botella equivocada" y lo peso en 440 ml, entonces el ajuste de −210 ml se anula con un movimiento inverso, la versión nueva registra −10 ml y la versión original queda visible como *reabierta*.
- `CA-2` — Dado que el turno siguiente ya abrió con *recibido conforme*, cuando reabro el cierre anterior, entonces solo puedo corregir el valor registrado, y las diferencias entre turnos y del turno en curso quedan marcadas *recalculado por reapertura*.
- `CA-3` — Dada una botella marcada finalizada con 200 ml y otra ya en servicio en la estación, cuando reabro la finalización, entonces los 200 ml se suman al envase en servicio, con nota.
- `CA-4` *(rechazo)* — Dado que el turno siguiente ya se cerró, cuando intento reabrir el cierre, entonces el sistema no lo permite y ofrece una nota de corrección que liga las dos diferencias.
- `CA-5` *(rechazo)* — Dado que soy el Supervisor que hizo el cierre, cuando intento autorizar su reapertura, entonces el sistema lo niega y pide la autorización de un Administrador o del Propietario.

---

## 9. Casos límite y errores

| Situación | Comportamiento definido |
|---|---|
| Venta que deja stock negativo | Se registra igual + alerta (`RN-INV-012`). |
| Venta de un producto sin receta | No genera movimiento, no es error (`RN-INV-019`). |
| Insumo de la receta inactivo | La venta descuenta igual; se genera alerta de configuración. |
| Dos movimientos simultáneos sobre la misma existencia | Se procesan en serie; el CPP se recalcula en el orden de confirmación. Nunca se pierde un movimiento. |
| Sin conexión | Inventario no opera: no registra, no encola, no muestra existencias (`RN-INV-067`). Lo que otro módulo haya encolado se aplica al reconectar, con fecha de hecho y fecha de recepción (`RN-INV-100`…`102`). Aperturas y finalizaciones se registran al volver, con hora declarada (`RN-INV-099`). |
| Conversión entre dimensiones (ml → g) | **Siempre rechazada** (`RN-INV-046`). El caso real se resuelve declarando el contenido del empaque en la unidad base: un tetrapak = 1030 g. |
| Entrada con costo cero | Permitida con motivo (`RN-INV-017`). |
| Insumo con stock en una bodega que se desactiva | No se permite desactivar una bodega con stock ≠ 0. Debe trasladarse o ajustarse antes. |
| Receta con ciclo (A contiene B que contiene A) | Rechazada al guardar (`RN-INV-025`). |
| Cambio de unidad base de un insumo con movimientos | Prohibido. Debe crearse un insumo nuevo. |
| Movimiento que llega después de cerrado un conteo | No cambia la existencia: reclasifica la diferencia del conteo y deja nota; el documento del conteo no se modifica (`RN-INV-102`). |
| Orden de producción sin existencia suficiente | Rechazada entera (`RN-INV-066`). A diferencia de la venta, producir sí puede esperar. |
| Diferencia de envase dentro de la tolerancia | Se registra sin alerta (`RN-INV-086`). |
| Venta de un insumo `envase abierto` sin envase en servicio | No se bloquea; alerta y ml pendientes para el próximo envase de esa estación (`RN-INV-083`). |
| Abrir un segundo envase del mismo insumo en la misma estación | Exige finalizar el anterior (`RN-INV-080`). |
| Peso bruto fuera del rango vacío–lleno | Rechazado (`RN-INV-075`). |
| Envase prestado a otra estación sin medir | Pasa con su contenido teórico; la siguiente diferencia queda *compartida entre estaciones* (`RN-INV-114`). |
| Venta sin estación (archivo o registro manual) | Consume de la estación por defecto de la bodega (`RN-INV-112`). |
| Estación que no hizo cierre | No se bloquea; alerta y apertura siguiente con medición obligatoria (`RN-INV-094`). |
| Venta que llega después de una medición | No cambia la existencia; reclasifica la diferencia y deja nota (`RN-INV-102`). |
| Archivo de ventas sin hora | Conciliación por día y estación, sin detalle por envase ni turno (`RN-INV-098`). |
| Archivo de ventas que no llega | Turno *pendiente de ventas* y alerta al vencer el plazo (`RN-INV-097`). |
| Hora declarada anterior al evento previo | Rechazada (`RN-INV-099`). |
| Error en un cierre descubierto con el turno siguiente ya cerrado | No se reabre; nota de corrección (`RN-INV-106`). |
| Envase abierto durante un conteo de su bodega | Línea *por recontar* (`RN-INV-111`). |
| Insumo `local` usado en una receta `global` | Rechazado al guardar. Hay que promover el insumo a `global` (`RN-INV-042`). |
| Despiece que rinde menos de lo esperado | La diferencia se registra como merma de producción con su costo (`RN-INV-063`). |
| Importación masiva con filas inválidas | Se importan las válidas, se reporta línea por línea lo rechazado. No se importa parcialmente un movimiento. |

---

## 10. Interfaces funcionales

| Interfaz | Dirección | Con quién | Qué información | Si el otro no existe |
|---|---|---|---|---|
| Venta confirmada | ◄ entra | `pos` | Producto, cantidad, modificadores, sede, estación, fecha y hora, referencia | Salida manual con motivo *venta*, o importación de un archivo de ventas con periodo (desde–hasta) y, si los tiene, estación y hora por venta |
| Precio de carta | ◄ entra | `catalogo` | Precio del producto asociado a la porción de referencia | No se muestra la venta perdida |
| Estaciones | ◄ entra | `kds`, `pos` | Identificador y nombre de las estaciones activas de la sede | Cada bodega de venta opera con su estación por defecto (`RN-INV-113`) |
| Recepción de mercancía | ◄ entra | `compras` | Insumo, cantidad, unidad, costo, proveedor, lote, referencia | Entrada manual con costo digitado |
| Identidad de productos | ◄ entra | `catalogo` | Producto vendible y sus modificadores | Inventario declara localmente los productos a los que asocia recetas |
| Sedes y bodegas activas | ◄ entra | `tenancy` | Sedes del tenant | No aplica: módulo de plataforma obligatorio |
| Usuarios y permisos | ◄ entra | `iam` | Identidad y rol del responsable | No aplica: módulo de plataforma obligatorio |
| Existencia de un insumo | ► sale | cualquiera | Cantidad y costo en una bodega y fecha | — |
| Disponibilidad de un producto | ► sale | `pos`, `pedidos-qr`, `catalogo` | Cuántas unidades alcanzan los insumos de su receta | — |
| Costo de receta | ► sale | `catalogo`, `reportes` | Costo unitario derivado | — |
| Alertas de stock | ► sale | `notificaciones`, `compras` | Insumo, bodega, tipo de alerta | Se consultan desde el propio módulo |
| Alertas y avisos de envases | ► sale | `notificaciones` | Envase excedido, diferencia sobre tolerancia, estación sin cierre, turno sin ventas cargadas, notas de venta tardía | Se consultan desde el propio módulo |
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
| Abrir / finalizar / prestar envase | ✔ | ✔ | ✔ | — | — | ✔ | — |
| Hacer cierre y apertura de estación | ✔ | ✔ | ✔ | — | — | ✔ | — |
| Reabrir cierre, apertura, finalización o conteo | ✔ | ✔ | ✔ | — | — | — | — |
| Conteo con *incluir abiertas* (auditoría) | ✔ | ✔ | — | — | — | — | — |
| Configurar mínimos y umbrales | ✔ | ✔ | — | — | — | — | — |
| Configurar tipo de control, verificación, tolerancia y estaciones | ✔ | ✔ | — | — | — | — | — |

> **Provisional:** quién abre, finaliza y hace cierre en la estación ("Jefe estación") se alinea
> con el rol *Estación* de [`05-actores-y-roles`](../05-actores-y-roles.md), que hoy no tiene
> permisos de inventario. Pendiente de alinear ambas matrices.

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
- **Envases en servicio**: por estación, desde cuándo, contenido teórico y porciones restantes.
- **Diferencias de envase**: por insumo, envase, estación y turno, con faltante y sobrante, costo y,
  si hay `catalogo`, venta perdida. Separa las **diferencias entre turnos**.
- **Turnos de estación**: cierres y aperturas, estaciones sin cierre y turnos pendientes de ventas.
- **Auditoría de mediciones**: reaperturas con versiones, motivos y autorizaciones; notas de venta
  tardía y de corrección.
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
| Tipo de control por defecto | Tenant | `unidad` |
| Verificación por defecto (para `envase abierto`) | Tenant | `al finalizar` |
| Tolerancia de diferencia por defecto | Tenant | 3 % del consumo |
| Origen de ventas de una estación | Estación | `en línea` si `pos` está instalado; si no, `por archivo` |
| Plazo para alertar turno sin ventas cargadas | Sede | 24 h |
| Estación por defecto de una bodega de venta | Bodega | La que se crea con la bodega |
| Ámbito por defecto al crear un insumo | Tenant | `global` |
| Exigir autorización para confirmar orden de producción | Sede | No |

---

## 15. Fuera de alcance de este módulo

| No hace | Lo hace |
|---|---|
| Precios de venta | `catalogo` |
| Órdenes de compra y proveedores formales | `compras` |
| Cuentas por pagar | Fuera del producto (software contable) |
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
| `PA-INV-010` | **Configurable por insumo** — *revisada 2026-09-15:* tipo de control `unidad`, `granel` o `envase abierto`; verificación `al finalizar`, `nivel` o `peso`. La venta siempre descuenta por receta; el envase abierto pertenece a una estación, que lo entrega y recibe con cierre y apertura de turno. → `RN-INV-070`…`099`, `112`…`114` (las originales `051`…`057` quedan derogadas) |
| `PA-INV-004` | **Inventario no tiene modo sin conexión.** Sin red no registra, no encola y no muestra existencias. → `RN-INV-067`, `RN-INV-100`…`102`. *Su consecuencia, `PA-INV-011`, se cerró el 2026-09-15.* |

### Cerradas en la sesión del 2026-09-15

| Id | Resolución |
|---|---|
| `PA-INV-011` | **Las ventas tardías sí se aplican.** Si hubo una medición posterior no cambian la existencia: reclasifican su diferencia y dejan nota de venta tardía. → `RN-INV-100`…`102` |
| `PA-INV-012` | **Reemplazada por la tolerancia por insumo**, calculada sobre el consumo del periodo medido. → `RN-INV-086` |

### Abiertas

- `PA-INV-007` — ¿Se necesita inventario de envases retornables y su control de devolución? Relevante en Colombia para cerveza y gaseosa en vidrio.
- `PA-INV-008` — ¿Quién define las recetas en la práctica: el administrador o el chef/bartender? Si es el segundo, hace falta un flujo de propuesta y aprobación.
- `PA-INV-009` — ¿El conteo físico se hace con un dispositivo móvil en la bodega? Cambia la experiencia, no las reglas. Con verificación `peso` implica báscula conectada o digitación manual.

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
- [ ] **No quedan preguntas abiertas** ← 3 pendientes (`PA-INV-007`, `008`, `009`)
- [x] La ficha se entiende sin abrir la ficha de otro módulo
- [ ] **Revisada y aprobada por Jorge**
