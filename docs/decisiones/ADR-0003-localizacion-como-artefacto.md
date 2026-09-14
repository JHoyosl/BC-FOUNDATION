# ADR-0003 — La localización es un artefacto sustituible

> **Estado:** Aceptada · **Fecha:** 2026-09-14 · **Decide:** Jorge Hoyos

## Contexto

Barscode arranca en Colombia. Colombia trae consigo IVA con varias tarifas, impuesto nacional
al consumo del 8% para consumo en sitio, facturación electrónica ante la DIAN con numeración
autorizada, NIT con dígito de verificación, peso colombiano sin decimales, propina
obligatoriamente voluntaria y explícita, recargos nocturnos y dominicales, y su propio
calendario de festivos.

La tentación natural es escribir todo eso dentro del POS y del módulo de caja. Es la vía
rápida y es exactamente la que convierte un producto en algo imposible de llevar a otro país
— y difícil incluso de mantener cuando la norma cambia dentro del mismo país.

## Decisión

**Ninguna regla específica de un país vive dentro de un módulo funcional. Los módulos
declaran interfaces; el país las implementa.**

El POS no sabe qué es el IVA. El POS sabe que existe un *cálculo de tributos* al que le
entrega los ítems de una cuenta y que le devuelve qué tributos aplican, sobre qué base,
cuánto suman y cómo deben presentarse. Quién responde esa pregunta es el **artefacto de
país** — para Colombia, el módulo `localizacion-co`.

Interfaces de localización identificadas: cálculo de tributos, documento fiscal,
identificación fiscal, medios de pago, moneda y formato, propina, reglas laborales,
calendario de festivos, datos personales, y edad mínima para consumo de alcohol. La tabla
completa está en [`docs/03-principios.md`](../03-principios.md).

Cada sede tiene exactamente un artefacto de país activo, determinado por su ubicación
(`RN-ARQ-022`).

## Alternativas consideradas

| Alternativa | Por qué se descartó |
|---|---|
| Reglas de Colombia dentro de los módulos | Imposibilita la expansión y convierte cada cambio normativo en una modificación del POS. |
| Parámetros configurables (una tabla de tasas) | Cubre las tasas, no cubre la estructura: la factura electrónica DIAN no es un porcentaje configurable, es un proceso completo. |
| Motor de reglas genérico y configurable por el cliente | Complejidad desproporcionada. Nadie va a configurar la DIAN desde una pantalla de administración. |

## Consecuencias

**A favor**

- Expandir a otro país no reescribe el POS: se escribe un artefacto nuevo.
- Un cambio normativo (una tasa que sube, una resolución nueva) se toca en un solo lugar.
- Los módulos funcionales se pueden definir y probar sin resolver lo tributario, lo que
  desbloquea el trabajo del KB inmediatamente.
- Obliga a que todo importe lleve moneda explícita, lo que evita una clase entera de errores.

**En contra**

- Hay que declarar y mantener las interfaces con disciplina. Cada vez que un módulo necesite
  una regla de país que no está en la lista, hay que agregar una interfaz — no resolverlo
  dentro del módulo (`RN-ARQ-025`).
- Añade una capa de indirección que, con un solo país en producción, puede parecer
  sobre-ingeniería.
- Los documentos fiscales emitidos deben conservar la versión del artefacto con que se
  calcularon (`RN-ARQ-024`): un cambio de tasa no puede reescribir el pasado.

## Preguntas que esta decisión deja abiertas

- `PA-ARQ-010` — ¿Barscode emite la factura electrónica directamente ante la DIAN o se
  integra con un proveedor tecnológico autorizado? Alto impacto en costo, riesgo y tiempo.
- `PA-ARQ-011` — ¿Se incluye facturación electrónica o basta el documento equivalente?
- `PA-ARQ-013` — ¿Las reglas laborales del módulo `schedule` viven en el artefacto de país o
  se aíslan aparte?

## Referencias

- [`docs/03-principios.md`](../03-principios.md) — Principio 3 y tabla de interfaces
- [`docs/06-catalogo-modulos.md`](../06-catalogo-modulos.md) — módulo `localizacion-co`
