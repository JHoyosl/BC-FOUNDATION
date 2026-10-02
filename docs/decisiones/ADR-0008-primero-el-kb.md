# ADR-0008 — Primero el KB, después el plan de desarrollo

> **Estado:** Aceptada · **Fecha:** 2026-10-02 · **Decide:** Jorge Hoyos

## Contexto

El KB decía dos cosas distintas sobre cuándo se escribe código:

- [`02-alcance.md`](../02-alcance.md) §2: esta etapa produce documentación, y dentro está la
  definición funcional de **todos** los módulos del catálogo.
- [`ADR-0005`](ADR-0005-construccion-modulo-a-modulo.md): "se define **y se construye** un módulo
  a la vez", y cada módulo terminado "se puede vender, probar y cobrar sin esperar al resto".

Además, el primer módulo de la cola no podía entregarse solo: `inventario` necesita tenant, sede,
usuarios y permisos, y los módulos de plataforma están en los puestos 8 a 10.

## Decisión

> Vamos a definir primero el KB y con ese KB creamos el plan de desarrollo.

- **La cola ordena la definición, no la construcción.** Pasa a llamarse *cola de definición*.
- **No se escribe código mientras el KB no esté definido.**
- **Con el KB definido se crea el plan de desarrollo.** Qué se construye primero, en qué orden y
  con qué plataforma lo decide ese plan. No hereda el orden de la cola de definición.
- **Sigue vigente de `ADR-0005`:** se define un módulo a la vez, sin fases, y ninguno entra a
  definición mientras el anterior tenga preguntas abiertas (`RN-ARQ-006`).
- **Plataforma mínima.** Lo que cada módulo necesita de la plataforma se acumula en
  [`07-plataforma-minima.md`](../07-plataforma-minima.md), para definirla con esa lista en mano.

## Alternativas consideradas

| Alternativa | Por qué se descartó |
|---|---|
| Construir cada módulo apenas se apruebe su ficha | Se propuso porque pone a prueba el método pronto. Jorge decidió tener primero el KB y planear el desarrollo con el producto completo a la vista. |
| Adelantar `tenancy`, `iam` y `localizacion-co` al inicio de la cola | Obliga a adivinar qué necesitan los módulos funcionales (`ADR-0005`). Con el desarrollo después del KB, ya no hace falta. |

## Consecuencias

**A favor**

- El plan de desarrollo se hace conociendo el producto completo, incluida la plataforma.
- Desaparece el problema del primer módulo que no se podía entregar solo: el orden de
  construcción es una decisión del plan, no de la cola.
- El KB deja de decir dos cosas distintas.

**En contra**

- Nada se valida contra la realidad hasta que el KB esté definido. Un error de método se repite
  en todas las fichas antes de descubrirse.
- "Cada módulo terminado es entregable" (`ADR-0005`) deja de ser una consecuencia de esta etapa.
- Exige todavía más disciplina con el tamaño de las fichas: el riesgo de bola de nieve se muda
  del código a la documentación.

## Referencias

- [`ADR-0005`](ADR-0005-construccion-modulo-a-modulo.md) — módulo a módulo, sin fases
- [`docs/06-catalogo-modulos.md`](../06-catalogo-modulos.md) — cola de definición
- [`docs/07-plataforma-minima.md`](../07-plataforma-minima.md) — plataforma mínima
