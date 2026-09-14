# ADR-0002 — Barscode es un SaaS multi-tenant

> **Estado:** Aceptada · **Fecha:** 2026-09-14 · **Decide:** Jorge Hoyos

## Contexto

Barscode puede concebirse de dos formas: como un sistema que se instala para un negocio
concreto, o como una plataforma única que atiende a muchos negocios independientes. La
decisión condiciona el modelo de datos, el de negocio, la seguridad y prácticamente cada
regla del producto.

## Decisión

**Barscode es una plataforma SaaS multi-tenant.** Muchos bares y restaurantes independientes
se suscriben a la misma plataforma. El aislamiento entre ellos es absoluto.

Jerarquía:

```
Tenant (negocio)  →  Sede (local)  →  Zona  →  Mesa  →  QR
```

- **Tenant**: unidad de aislamiento de datos, de facturación de la suscripción y de
  configuración global.
- **Sede**: unidad de operación. Inventario, caja, turnos y Gamecenter viven aquí.
- **Mesa**: granularidad del lado cliente; el QR identifica sede + mesa.

Consecuencia particular: **el cliente final no pertenece a un tenant.** Es un usuario de la
plataforma Barscode que establece sesiones con sedes. Su cuenta, su historial y su identidad
de juego son suyos y de Barscode, no del bar (`RN-ARQ-013`).

## Alternativas consideradas

| Alternativa | Por qué se descartó |
|---|---|
| Instalación dedicada por negocio | Costo operativo y de soporte inviable para el segmento objetivo. Imposibilita el Gamecenter como red. |
| Multi-sede para una sola cadena | Limita el producto a un solo cliente. No es un negocio de software. |
| Multi-tenant con clientes finales por tenant | Obligaría al cliente a crear una identidad distinta en cada bar. Destruye la propuesta del Gamecenter y la fidelización. |

## Consecuencias

**A favor**

- Un solo producto que escala a muchos clientes.
- El cliente final tiene una sola identidad en Barscode, reutilizable en cualquier local
  — condición necesaria para que el Gamecenter y la fidelización tengan sentido.
- Actualizaciones y correcciones llegan a todos a la vez.

**En contra**

- El aislamiento de datos pasa a ser un requisito crítico de seguridad, no un detalle.
- Obliga a definir planes, límites y habilitación de módulos desde el principio (módulo
  `tenancy`).
- La identidad del cliente final fuera del tenant plantea preguntas de propiedad del dato y
  de protección de datos personales que hay que resolver (Ley 1581 de 2012).
- Un negocio podría querer "sus" clientes y no aceptar que la relación sea con Barscode.
  Riesgo comercial a manejar.

## Preguntas que esta decisión deja abiertas

- `PA-ARQ-001` — ¿Los módulos se contratan por tenant o por sede?
- `PA-ARQ-002` — ¿La carta se define por tenant con sobreescritura por sede, o por sede?
- `PA-VIS-001` — ¿Cuál es exactamente la unidad de cobro de la suscripción?

## Referencias

- [`docs/03-principios.md`](../03-principios.md) — Principio 2
- [`docs/04-glosario.md`](../04-glosario.md) — Tenant, Sede, Zona, Mesa
