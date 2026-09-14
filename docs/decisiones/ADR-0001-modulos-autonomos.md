# ADR-0001 — Los módulos son autónomos

> **Estado:** Aceptada · **Fecha:** 2026-09-14 · **Decide:** Jorge Hoyos

## Contexto

El primer intento de Barscode se detuvo porque el alcance no estaba cerrado: cada módulo
asumía la existencia de los demás, definir uno obligaba a definir tres, y el trabajo creció
como bola de nieve hasta volverse inabordable.

Además, comercialmente, exigir que un negocio adopte la plataforma completa para obtener
valor es una barrera de entrada alta en un mercado donde la mayoría ya tiene algún POS
instalado.

## Decisión

**Todo módulo de Barscode debe funcionar, venderse, instalarse y operar por sí solo, sin que
ningún otro módulo exista.**

Caso canónico: Inventario controla productos, cantidades, unidades y costos. Un negocio puede
contratar únicamente Inventario y usarlo completo, exista o no el módulo POS. Cuando POS
existe, Inventario aprovecha sus ventas para descontar stock — pero eso es una mejora, no un
requisito.

Cada ficha de módulo debe declarar un **contrato de autonomía** de cuatro puntos: qué hace
solo, cómo degrada sin sus vecinos, qué consume de ellos y qué expone.

Única excepción: los módulos de la capa **Plataforma** (`tenancy`, `iam`, `localizacion-*`),
que son transversales y obligatorios en cualquier instalación (`RN-ARQ-005`).

## Alternativas consideradas

| Alternativa | Por qué se descartó |
|---|---|
| Plataforma integrada monolítica | Es lo que hundió el primer intento. Definir cualquier cosa exige definir todo. |
| Núcleo obligatorio (POS) + satélites | Deja fuera al negocio que ya tiene POS, que es la mayoría del mercado. |
| Módulos autónomos con datos duplicados | Resuelve el acoplamiento creando un problema peor: dos versiones de la verdad. |

## Consecuencias

**A favor**

- Se puede definir, aprobar y construir un módulo a la vez, sin bloqueos.
- Se puede vender por partes y entrar a negocios que ya tienen software.
- Un módulo mal delimitado se detecta de inmediato: no pasa la prueba de "¿qué hace solo?".
- Cada ficha del KB se lee sin abrir otra.

**En contra**

- Todo dato que un módulo consuma de otro exige **también** una vía de captura manual
  (`RN-ARQ-002`). Es trabajo adicional en cada módulo.
- Hay que definir y mantener contratos explícitos entre módulos.
- Un módulo no pasa la prueba: `reportes`, que es agregador por naturaleza (`PA-CAT-002`).
- Riesgo de que el usuario perciba funcionalidad duplicada si los contratos no están bien
  presentados en la interfaz.

## Notas

Esta decisión es **funcional, no técnica**. No implica microservicios, ni bases de datos
separadas, ni despliegue independiente. Un monolito modular puede cumplirla perfectamente.
La decisión técnica sobre cómo se comunican los módulos se tomará después, y esta ADR la
restringe únicamente en que debe respetar los contratos declarados.

## Referencias

- [`docs/03-principios.md`](../03-principios.md) — Principio 1
- [`docs/06-catalogo-modulos.md`](../06-catalogo-modulos.md) — verificación de autonomía por módulo
