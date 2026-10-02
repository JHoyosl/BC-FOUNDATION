# Glosario

> **Estado:** 🟡 Borrador · **Dueño:** Jorge Hoyos · **Actualizado:** 2026-10-02

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

**Módulo habilitado** — Módulo que el plan de un tenant incluye y que ese tenant puede usar.
Barscode es un SaaS: nada se instala en el negocio. Donde el KB dice que un módulo está
"instalado" o "presente", significa habilitado.

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
de medida y costo. Ejemplo: *ron añejo*, que se lleva en ml y se compra en botellas de 750 ml
(la botella es un **empaque**, no el insumo).

**Producto (producto vendible)** — Cosa que se vende al cliente. Vive en Catálogo. Tiene precio. Ejemplo:
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
producto se prepara en una estación. Cada bodega de venta tiene una estación por defecto que
consume de ella.

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

**Bodega de venta** — Bodega de la que descuentan las ventas. Tiene una estación por defecto
que consume de ella.

**Ámbito de un insumo** — `global` (definido en el tenant, disponible en todas las sedes) o
`local` (propio de una sede). Compartir la ficha del insumo **no** significa compartir el
stock: las existencias y los costos son siempre por sede y bodega.

**Unidad base** — Unidad en la que se lleva la existencia de un insumo: ml, g o unidad.
Determina su dimensión. Todo insumo tiene exactamente una.

**Dimensión** — Volumen, masa o conteo. Cada insumo vive en una sola. **No existe conversión
entre dimensiones**: no hay densidad ni factor ml↔g.

**Empaque** — Forma concreta en que se compra o se maneja un insumo, que declara cuánto
contiene *expresado en la unidad base del insumo*. Un tetrapak de crema declara 1030 g, no
1 litro. Es lo que permite comprar en volumen y consumir en masa sin convertir nunca entre
dimensiones.

**Tipo de control** — Cómo se lleva la existencia de un insumo: `unidad` (indivisible, se
descuenta entero), `granel` (sin envase; se verifica en el conteo) o `envase abierto` (se abre y se
sirve de él durante un tiempo). Se configura por insumo.

**Verificación** — Para un insumo `envase abierto`, cómo se mide lo que queda: `al finalizar`
(solo cuando se acaba), `nivel` (estimación a ojo) o `peso` (báscula, con peso vacío y lleno del
empaque).

**Envase en servicio** — Envase ya abierto del que se está sirviendo, a cargo de la estación que
lo abrió. Cuenta en la existencia de la bodega de la que salió.

**Turno de estación** — Periodo entre la apertura y el cierre de una estación, en el que esa
estación responde por sus envases en servicio. Es de inventario; no es un turno de caja ni de
personal.

**Diferencia de envase** — Contenido medido menos contenido teórico de un envase en servicio.
Faltante si es negativa, sobrante si es positiva. Se registra como ajuste, no como merma, porque
su causa es desconocida.

**Diferencia entre turnos** — Lo que cambió un envase entre el cierre de un turno y la apertura
del siguiente. No se atribuye a ninguno de los dos.

**Nota de venta tardía** — Registro de que unas ventas llegaron después de una medición y
explican parte de su diferencia. La existencia no cambia.

**Medición** — Registro de cuánto contiene realmente un envase en servicio: en un cierre, una
apertura, un conteo o al finalizarlo. Toda medición calcula una diferencia de envase.

**Nota de corrección** — Registro que liga como un mismo error dos diferencias de mediciones que
ya no pueden reabrirse. No genera movimientos; los reportes las muestran compensadas.

**Porción de referencia** — Cantidad que representa una porción de un insumo (un trago de 50 ml).
Sirve para mostrar porciones restantes y la venta perdida.

**Venta perdida** — Diferencia de envase expresada en dinero de venta: diferencia ÷ porción de
referencia × precio de carta. Solo se muestra si Catálogo está habilitado.

**Orden de producción** — Documento que transforma unos insumos en otro insumo distinto:
almíbar, salsa madre, infusión, despiece. El insumo producido entra con costo real derivado,
no estimado.

**Existencia (stock)** — Cantidad de un insumo en una bodega en un momento dado.

**Movimiento de inventario** — Todo hecho que cambia la existencia: entrada, salida, ajuste,
traslado, merma. Es la única forma en que el stock cambia.

**Ajuste** — Movimiento que corrige la existencia cuando la causa se desconoce o hubo un error
de registro: diferencia de conteo, diferencia de envase, corrección de registro. No es merma.

**Traslado** — Paso de insumo de una bodega a otra de la misma sede, en un solo movimiento.
Entre sedes no hay traslado: es salida en una y entrada en otra.

**Merma** — Pérdida de insumo sin venta: rotura, vencimiento, derrame, cortesía, robo.
Siempre lleva motivo y responsable.

**Conteo físico** — Verificación manual de existencias contra lo que dice el sistema.
Genera ajustes.

**Lote** — Identificación de una entrada de un insumo, con su fecha de vencimiento. Solo lo
llevan los insumos que lo exigen.

**Costo promedio ponderado** — Método de valoración del inventario. Ver
[`modulos/inventario.md`](modulos/inventario.md).

**Kardex** — Historial cronológico de los movimientos de un insumo en una bodega, con saldo y
costo después de cada uno. Muestra también los eventos de envase (abrir, prestar), que no
cambian la existencia.

**Valorización** — Valor del inventario a una fecha: las existencias por su costo promedio, por
sede y bodega.

---

## Personas del negocio

**Staff** — Cualquier persona que trabaja para el tenant y usa Barscode: dueño,
administrador, mesero, cajero, cocina, bodeguero.

**Usuario** — Identidad con la que una persona del staff entra a Barscode. Es de la persona:
no pertenece a ningún tenant. Una persona tiene un solo usuario aunque trabaje en varios
negocios. Ver [`ADR-0007`](decisiones/ADR-0007-un-usuario-varias-vinculaciones.md).

**Vinculación** — Relación entre un usuario y un tenant para el que trabaja. La crea, la
suspende y la termina el tenant. De ella cuelgan los perfiles de la persona en ese negocio. Un
usuario puede tener varias, y ningún tenant ve las de los demás.

**Rol** — *Término retirado.* Lo reemplaza **perfil** ([`ADR-0006`](decisiones/ADR-0006-permisos-por-modulo-y-perfiles.md)).

**Permiso** — Acceso a una función concreta de un módulo, con código propio (`PRM-INV-001`).
Los permisos de un módulo forman un árbol: módulo, grupo y permiso. El catálogo lo define
Barscode; un negocio no crea permisos.

**Permiso sensible** — Permiso que expone importes o costos, autoriza, anula, reabre, configura
o administra usuarios y perfiles. Nunca llega a un perfil sin que alguien del negocio lo acepte.

**Perfil** — Conjunto de permisos que arma el negocio. Puede ser `global` (del tenant) o `local`
(de una sede). Una persona puede tener varios, y sus permisos se suman.

**Perfil sugerido** — Perfil que Barscode entrega como punto de partida. Se usa *vinculado* (se
actualiza con Barscode y no se edita) o se duplica como *propio*.

**Asignación** — Lo que le da permisos a una persona: su vinculación, un perfil, una sede y,
si aplica, estaciones.

**Propietario** — Quien tiene todos los permisos de un tenant. No es un perfil y no se edita.

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
