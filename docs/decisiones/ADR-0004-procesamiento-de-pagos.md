# ADR-0004 — Procesamiento de pagos: decisión abierta

> **Estado:** 🟠 **Pendiente** · **Fecha de apertura:** 2026-09-14 · **Decide:** Jorge Hoyos
> **Debe resolverse antes de:** definir el módulo `pagos`

Este ADR está deliberadamente sin resolver. Se registra ahora para que la pregunta no se
pierda ni se responda por accidente, escribiendo código que asuma una de las dos opciones.

## Contexto

Barscode permite que el cliente pague desde su teléfono. Eso abre una pregunta con
consecuencias legales, financieras y de modelo de negocio muy distintas según la respuesta:

**¿El dinero pasa por Barscode, o va directo del cliente al bar?**

No es un detalle técnico. Determina si Barscode custodia fondos de terceros, y eso cambia
la naturaleza de la empresa.

## Las dos opciones

### Opción A — Barscode solo integra pasarelas

El cliente paga a través de Wompi, Bold, Nequi, PSE o similar. El dinero va **directo a la
cuenta del establecimiento**. Barscode orquesta el cobro, recibe la confirmación y la
registra, pero nunca toca los fondos.

**A favor**

- Carga regulatoria mínima. Barscode no es entidad de pago ni custodia dinero ajeno.
- No hay riesgo de tesorería ni de flotante.
- No hay que responder por conciliaciones ni por dinero retenido.
- Implementación notablemente más simple y más rápida.

**En contra**

- No se puede cobrar comisión sobre transacciones — se pierde una línea de ingreso.
- La relación económica del cobro es del bar con la pasarela; Barscode queda en medio sin
  palanca.
- La conciliación depende de lo que reporte la pasarela.

### Opción B — Barscode procesa y liquida

El cliente paga a Barscode. Barscode liquida al establecimiento según un calendario
acordado, reteniendo su comisión.

**A favor**

- Habilita comisión por transacción: una línea de ingreso que escala con el uso, no con el
  número de clientes.
- Control total de la experiencia de pago y de la conciliación.
- Facilita productos futuros: adelantos de liquidación, split automático de propinas.

**En contra**

- **Custodia de fondos de terceros.** En Colombia esto implica marco regulatorio propio y,
  con alta probabilidad, figura habilitada o alianza con una entidad vigilada.
- Riesgo de tesorería, de fraude y de contracargos que Barscode asume.
- Obligaciones de prevención de lavado de activos.
- Aumenta mucho el costo y el tiempo de salir al mercado.

## Qué hay que saber antes de decidir

1. ¿Qué exige la regulación colombiana para recaudar a nombre de terceros? ¿Basta un
   convenio con una pasarela habilitada o hace falta figura propia?
2. ¿Cuánto vale realmente la comisión por transacción frente al ingreso por suscripción?
   Si es marginal, no justifica el riesgo.
3. ¿Aceptaría un bar que su dinero pase por Barscode y llegue con retraso? En hostelería el
   flujo de caja diario es crítico.
4. ¿Existe una opción intermedia? Algunas pasarelas ofrecen *split payments* o marketplace,
   donde la plataforma cobra su comisión sin custodiar el resto.

## Mientras tanto

- El módulo `pagos` **no se define** hasta resolver esto.
- Ningún otro módulo asume que Barscode recibe dinero. `pos` y `caja` registran el cobro y
  el medio de pago; qué pasa con el dinero después no es asunto suyo.
- El [modelo de negocio](../01-vision.md#7-modelo-de-negocio) mantiene la comisión sobre
  pagos como línea **posible**, nunca comprometida.

## Referencias

- [`docs/01-vision.md`](../01-vision.md) — sección 7, modelo de negocio
- [`docs/06-catalogo-modulos.md`](../06-catalogo-modulos.md) — módulo `pagos`
