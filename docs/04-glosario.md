# Glosario

> **Estado:** 🟡 Borrador · **Dueño:** Jorge Hoyos · **Actualizado:** 2026-09-14

Vocabulario único del proyecto. **Si un término está aquí, se usa así en todo el KB, en la
interfaz y en el código.** Si aparece un sinónimo, se corrige.

---

## Plataforma

**Tenant (negocio)** — La empresa cliente de Barscode. Unidad de aislamiento de datos y de
facturación de la suscripción. *Sinónimos prohibidos: cliente, empresa, cuenta.* En este KB
"cliente" siempre significa la persona que consume en el local.

**Sede** — Establecimiento físico de un tenant. Unidad de operación diaria: caja, inventario,
turnos y Gamecenter viven en una sede. *Sinónimos prohibidos: local, sucursal, punto de venta.*

**Plan** — Conjunto de módulos habilitados y límites contratados por un tenant.

**Módulo** — Unidad funcional autónoma del producto (Inventario, POS, Gamecenter…). Ver
[Principio 1](03-principios.md).

**Artefacto de país** — Implementación de las reglas legales, fiscales y culturales de un país
(tributos, documento fiscal, moneda, festivos, medios de pago). Sustituible sin tocar módulos.
Ver [Principio 3](03-principios.md).

---

## Espacio físico

**Zona** — Agrupación de mesas dentro de una sede: terraza, barra, salón principal, VIP.
Puede tener carta, horario o servicio distintos.

**Mesa** — Punto físico identificable donde se sienta un cliente. Tiene un QR asociado.
La barra puede modelarse como mesas o como una zona sin mesas (`PA-GLO-001`).

**QR de mesa** — Código impreso y fijo en una mesa. Identifica sede + mesa. **No cambia con
cada visita.** Al escanearse, abre una sesión de mesa.

---

## Lado cliente

**Cliente** — Persona que consume en el local. *Nunca* significa el negocio.

**Cliente anónimo** — Cliente que escaneó el QR y opera sin crear cuenta. Es el caso por
defecto.

**Cliente registrado** — Cliente con cuenta de Barscode. Conserva historial, puntos e
identidad de juego entre visitas.

**Sesión de mesa** — Vínculo temporal entre uno o varios clientes y una mesa de una sede,
abierto al escanear el QR. Es el contexto de todo lo que hace el cliente: pedir, pagar,
jugar, interactuar. Termina cuando la mesa se cierra o por inactividad.

**Alias** — Nombre visible de un cliente dentro de una sesión (para la cuenta compartida y
para el Gamecenter). Puede ser autogenerado. No revela identidad real.

**Gamecenter** — Espacio de juego de una sede, accesible desde la sesión de mesa. Juegos,
rankings y torneos entre quienes están en el local.

---

## Carta y producto

**Insumo** — Cosa que se compra, se almacena y se consume. Vive en Inventario. Tiene unidad
de medida y costo. Ejemplo: *botella de ron 750 ml*.

**Producto** — Cosa que se vende al cliente. Vive en Catálogo. Tiene precio. Ejemplo:
*Cuba Libre*. Un producto puede corresponder 1:1 a un insumo (una cerveza en botella) o
consumir varios (un coctel).

> Distinción clave: **insumo ≠ producto.** Es la separación que permite que Inventario y
> Catálogo sean módulos autónomos. Inventario no sabe de precios de venta; Catálogo no sabe
> de existencias.

**Receta (ficha técnica)** — Relación entre un producto vendible y los insumos que consume,
con cantidades. Es lo que permite descontar stock al vender y calcular costo.

**Modificador** — Variación de un producto que el cliente elige: *sin hielo*, *doble*,
*término medio*. Puede alterar precio y consumo de insumos.

**Carta** — Conjunto de productos ofrecidos, organizados y vigentes en un momento y una zona
dados. Un mismo producto puede estar en varias cartas con distinto precio.

---

## Operación

**Comanda** — Instrucción de preparación enviada a una estación (cocina, barra). Agrupa los
ítems de un pedido que le corresponden a esa estación.

**Estación** — Punto de preparación dentro de una sede: cocina, barra, parrilla. Cada
producto se prepara en una estación.

**Pedido** — Conjunto de ítems solicitados en un momento. Un cliente puede hacer varios
pedidos durante una visita.

**Cuenta** — Acumulado de todo lo consumido en una mesa durante una sesión, con sus tributos
y descuentos. Es lo que se cobra. *Sinónimos prohibidos: ticket, factura, orden.*

**Factura / documento fiscal** — Documento legal emitido al cobrar. Lo define el artefacto de
país. **No es lo mismo que la cuenta**: la cuenta es operativa, la factura es legal.

**Turno de caja** — Periodo entre la apertura y el cierre de una caja, con un responsable,
una base inicial y un arqueo final.

**Arqueo** — Conteo del dinero físico al cerrar un turno de caja, contrastado con lo que el
sistema dice que debería haber.

**Cierre de sede (cierre Z)** — Consolidación del día de operación de una sede.

---

## Inventario

**Bodega** — Lugar de almacenamiento dentro de una sede: bodega principal, barra, nevera.
El stock siempre es *de un insumo en una bodega*.

**Ámbito de un insumo** — `global` (definido en el tenant, disponible en todas las sedes) o
`local` (propio de una sede). Compartir la ficha del insumo **no** significa compartir el
stock: las existencias y los costos son siempre por sede y bodega.

**Dimensión** — Volumen, masa o conteo. Cada insumo vive en una sola. **No existe conversión
entre dimensiones**: no hay densidad ni factor ml↔g.

**Empaque** — Forma concreta en que se compra o se maneja un insumo, que declara cuánto
contiene *expresado en la unidad base del insumo*. Un tetrapak de crema declara 1030 g, no
1 litro. Es lo que permite comprar en volumen y consumir en masa sin convertir nunca entre
dimensiones.

**Envase en servicio** — Envase ya abierto del que se está sirviendo. Solo existe para
insumos cuyo método de control lo requiere.

**Método de control de envase abierto** — Cómo se verifica lo que queda en un envase
empezado: `unidad` (indivisible), `nivel` (estimación visual), `peso` (báscula, con tara) o
`apertura` (se descuenta completo al abrir). Se configura por insumo.

**Orden de producción** — Documento que transforma unos insumos en otro insumo distinto:
almíbar, salsa madre, infusión, despiece. El insumo producido entra con costo real derivado,
no estimado.

**Existencia (stock)** — Cantidad de un insumo en una bodega en un momento dado.

**Movimiento de inventario** — Todo hecho que cambia la existencia: entrada, salida, ajuste,
traslado, merma. Es la única forma en que el stock cambia.

**Merma** — Pérdida de insumo sin venta: rotura, vencimiento, derrame, cortesía, robo.
Siempre lleva motivo y responsable.

**Conteo físico** — Verificación manual de existencias contra lo que dice el sistema.
Genera ajustes.

**Costo promedio ponderado** — Método de valoración del inventario. Ver
[`modulos/inventario.md`](modulos/inventario.md).

---

## Personas del negocio

**Staff** — Cualquier persona que trabaja para el tenant y usa Barscode: dueño,
administrador, mesero, cajero, cocina, bodeguero.

**Rol** — Conjunto de permisos asignable a un miembro del staff **en una sede**.

---

## Dinero

**Importe** — Cantidad de dinero. **Siempre lleva moneda explícita.** No existe un importe
sin moneda (`RN-ARQ-021`).

**Tributo** — Impuesto aplicable a una cuenta. Su cálculo lo resuelve el artefacto de país,
nunca un módulo.

**Propina** — Valor adicional voluntario. Su legalidad, presentación y distribución las
define el artefacto de país.

**Medio de pago** — Forma en que se paga: efectivo, tarjeta, transferencia, billetera
digital. Su catálogo lo define el artefacto de país.

---

## Preguntas abiertas

- `PA-GLO-001` — ¿La barra es una zona con mesas, una zona sin mesas, o un concepto aparte? Afecta `salon-mesas` y `pedidos-qr`.
- `PA-GLO-002` — ¿Se usará "cuenta" también cuando un cliente pide para llevar sin mesa? ¿Existe ese caso?
- ~~`PA-GLO-003`~~ — **Resuelta** (2026-09-14) junto con `PA-INV-001`: maestro común por tenant más insumos locales por sede. Ver *Ámbito de un insumo*.
- `PA-GLO-004` — ¿"Producto" incluye cosas no consumibles que el bar venda (mercancía, cover, entrada)?
