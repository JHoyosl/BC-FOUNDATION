# Barscode — Knowledge Base

Base de conocimiento funcional de **Barscode**, una plataforma SaaS (web + móvil) para la
gestión de bares y restaurantes, con un *gamecenter* y capa social para los clientes del
establecimiento.

Este repositorio **no contiene código**. Contiene la definición funcional completa del
producto: qué hace, para quién, con qué reglas y con qué límites. El objetivo declarado es
que **el desarrollo no tenga pendientes en el KB**: si algo hay que construir, aquí está
escrito antes de escribirlo en código.

---

## Cómo navegar

| Documento | Qué responde |
|---|---|
| [`docs/00-guia.md`](docs/00-guia.md) | Cómo se usa y se mantiene este KB. Convenciones y estados. |
| [`docs/01-vision.md`](docs/01-vision.md) | Qué es Barscode, qué problema resuelve, para quién, cómo gana dinero. |
| [`docs/02-alcance.md`](docs/02-alcance.md) | Qué entra, qué no entra, y en qué fase entra cada cosa. |
| [`docs/03-principios.md`](docs/03-principios.md) | Las tres reglas que gobiernan todo el diseño: modularidad, multi-tenant, localización. |
| [`docs/04-glosario.md`](docs/04-glosario.md) | Vocabulario único del proyecto. Si un término no está aquí, no se usa. |
| [`docs/05-actores-y-roles.md`](docs/05-actores-y-roles.md) | Quién usa el sistema y qué puede hacer cada quien. |
| [`docs/06-catalogo-modulos.md`](docs/06-catalogo-modulos.md) | Los 20 módulos del producto, su contrato de autonomía y la cola de construcción. |
| [`docs/modulos/`](docs/modulos/) | Una ficha funcional completa por módulo. |
| [`docs/decisiones/`](docs/decisiones/) | ADR: cada decisión estructural, con fecha y motivo. |
| [`docs/bitacora.md`](docs/bitacora.md) | Qué se definió, cuándo, y qué sigue. |

---

## Estado actual

**Etapa: definición funcional.** No se escribe código ni se toman decisiones técnicas
(lenguajes, frameworks, base de datos, despliegue). Se define **un módulo a la vez, hasta
terminarlo** — ver [`ADR-0005`](docs/decisiones/ADR-0005-construccion-modulo-a-modulo.md).

**Módulo en curso: [`inventario`](docs/modulos/inventario.md)** — 10 preguntas abiertas por cerrar.

| Bloque | Estado |
|---|---|
| Visión y alcance | 🟡 Borrador para revisión |
| Principios de diseño | 🟡 Borrador para revisión |
| Glosario, actores, catálogo | 🟡 Borrador para revisión |
| Fichas de módulo | 🟡 `inventario` en curso (1 de 20) |
| Decisiones técnicas | ⚪ No iniciado (deliberadamente) |

Leyenda: ⚪ no iniciado · 🔴 incompleto · 🟡 borrador · 🟢 aprobado · 🔵 congelado

---

## Antecedente

Existe un MVP previo, pequeño e incompleto, que **solo tuvo pruebas internas: ningún local
ni usuario de terceros depende de él.** No define el alcance de este KB ni impone
restricciones de migración. Queda como fuente de aprendizajes, no como especificación.

---

## Reglas del repo

1. Una unidad de conocimiento por archivo. Si un documento pasa de ~400 líneas, se parte.
2. Nada se marca 🟢 aprobado sin que Jorge lo revise explícitamente.
3. Toda pregunta sin resolver vive en la sección **Preguntas abiertas** del documento
   correspondiente. Un documento con preguntas abiertas nunca está 🟢.
4. Toda decisión que cambie el rumbo se registra como ADR en `docs/decisiones/`.
5. Los commits describen la decisión, no el archivo: `kb(inventario): definir reglas de merma`.
