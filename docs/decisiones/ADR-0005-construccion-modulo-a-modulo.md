# ADR-0005 — Se construye módulo a módulo, no por fases

> **Estado:** Aceptada · **Fecha:** 2026-09-14 · **Decide:** Jorge Hoyos

## Contexto

El KB nació con un plan de fases: Fase 0 plataforma, Fase 1 operación mínima vendible,
Fase 2 diferenciación, Fase 3 escala. Cada fase agrupaba varios módulos que se
construirían juntos.

Al revisarlo apareció la contradicción: **[`ADR-0001`](ADR-0001-modulos-autonomos.md) dice
que cada módulo funciona y se vende solo.** Si eso es cierto, agruparlos en fases no aporta
nada y sí introduce un acoplamiento artificial — una fase no termina hasta que terminan
todos sus módulos, y volvemos a la bola de nieve que el KB existe para evitar.

## Decisión

**Se define y se construye un módulo a la vez, hasta terminarlo, y se pasa al siguiente.**
No hay fases. No hay módulos que se construyan juntos.

Un módulo está terminado cuando pasa su checklist de cierre, incluyendo **cero preguntas
abiertas**. Solo entonces empieza el siguiente.

**El primero es [`inventario`](../modulos/inventario.md).**

### Por qué inventario primero

- Es el módulo con más reglas de negocio: probar el principio de autonomía ahí es probarlo
  en el caso difícil.
- Es vendible por sí solo a negocios que ya tienen POS — un mercado al que los sistemas
  monolíticos no llegan.
- Su definición obliga a fijar el vocabulario central (insumo, unidad, bodega, movimiento,
  receta, costo) del que dependerá todo lo demás.
- No necesita que exista ningún otro módulo para entregar valor.

### Qué reemplaza al plan de fases

El orden de construcción deja de ser un compromiso rígido y pasa a ser una **cola
reordenable**. Al terminar cada módulo se revisa cuál sigue, con dos criterios:

1. **Valor por sí solo** — ¿se puede vender y usar sin nada más?
2. **Claridad que aporta** — ¿su vocabulario desbloquea a los siguientes?

Cola actual, sujeta a revisión después de `inventario`:

```
inventario → catalogo → salon-mesas → pos → caja → pedidos-qr → kds
           → plataforma (tenancy, iam, localizacion-co)
           → gamecenter, social, identidad-cliente, pagos
           → compras, schedule, reportes, fidelizacion, reservas, notificaciones
```

## Consecuencias

**A favor**

- Coherencia total con `ADR-0001`. El KB deja de decir dos cosas distintas.
- Cada módulo terminado es entregable: se puede vender, probar y cobrar sin esperar al resto.
- El progreso es medible sin ambigüedad: un módulo está 🟢 o no lo está.
- Se puede reordenar la cola con lo aprendido, sin renegociar un plan de fases completo.

**En contra**

- El producto completo tarda más en existir como conjunto. Hay que aceptar que durante
  buena parte del camino Barscode será "un módulo bueno" y no "una plataforma".
- Los módulos de plataforma (`tenancy`, `iam`, `localizacion-co`) quedan más adelante en la
  cola, así que los primeros módulos deben definirse sin dar por sentado cómo funcionarán.
  Se documenta lo que cada uno **necesita** de la plataforma, y la plataforma se define
  después con esa lista en mano.
- Exige disciplina: la tentación de saltar al siguiente módulo dejando preguntas abiertas
  atrás es exactamente lo que hundió el primer intento.

## Regla que se deriva

> `RN-ARQ-006` — Ningún módulo entra a definición mientras el anterior de la cola tenga
> preguntas abiertas sin resolver o escalar a ADR.

## Referencias

- [`ADR-0001`](ADR-0001-modulos-autonomos.md) — módulos autónomos
- [`docs/02-alcance.md`](../02-alcance.md) — cola de construcción
- [`docs/06-catalogo-modulos.md`](../06-catalogo-modulos.md) — catálogo y orden
