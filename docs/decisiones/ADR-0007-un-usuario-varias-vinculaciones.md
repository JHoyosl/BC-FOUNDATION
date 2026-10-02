# ADR-0007 — Un usuario, varias vinculaciones

> **Estado:** Aceptada · **Fecha:** 2026-10-02 · **Decide:** Jorge Hoyos

## Contexto

`RN-ARQ-012` decía que "un usuario del staff pertenece a un tenant". No aguanta un caso común en
el sector: la misma persona trabaja en dos negocios.

> Un mesero trabaja entre semana por las mañanas en un sitio y los fines de semana por la noche
> en otro. En uno es manager y en el otro es jefe de meseros.

Los dos negocios son tenants distintos, sin relación entre sí, y la persona tiene permisos
diferentes en cada uno.

## Decisión

**Una persona tiene un solo usuario de Barscode. Trabaja para cada negocio mediante una
vinculación, y de cada vinculación cuelgan sus permisos.**

```
Usuario (la persona)
  ├── Vinculación con el tenant A ──► sede Chapinero · perfil Manager
  └── Vinculación con el tenant B ──► sede Zona T    · perfil Jefe de meseros
```

- **El usuario es de la persona.** No pertenece a ningún tenant, y ningún tenant puede
  eliminarlo.
- **La vinculación es del tenant.** El negocio la crea por invitación, la suspende o la termina.
  Al terminarla conserva su historial de auditoría.
- **Los perfiles son de cada tenant** y se asignan a la vinculación, por sede
  ([`ADR-0006`](ADR-0006-permisos-por-modulo-y-perfiles.md)). El perfil "Manager" de un negocio
  no tiene nada que ver con los perfiles del otro.
- **Un tenant activo a la vez.** En una terminal del negocio, el tenant y la sede los da la
  terminal: la persona solo se identifica. En un dispositivo propio, elige el negocio al entrar y
  puede cambiar sin volver a identificarse. Los permisos de dos tenants **nunca se suman**.
- **Ningún tenant ve las otras vinculaciones** de un usuario ni dato alguno suyo en otro tenant.
- **El PIN de terminal compartida es de la vinculación:** es distinto en cada negocio.
- **El horario no cuenta.** Que trabaje mañanas en uno y noches en el otro no se configura en los
  permisos (`ADR-0006`). Los turnos son asunto del `schedule` de cada tenant, y ninguno ve los
  del otro.
- **El Propietario lo es de un tenant.** La misma persona puede ser Propietario de un negocio y
  mesero en otro.

## Alternativas consideradas

| Alternativa | Por qué se descartó |
|---|---|
| Una cuenta distinta en cada negocio | La persona carga dos contraseñas y no puede usar el mismo correo o celular en los dos. Es lo más simple de construir y lo más incómodo de usar. |
| Un usuario con los permisos de todos sus negocios sumados | Rompe el aislamiento entre tenants (`RN-ARQ-010`): lo que alguien puede en un negocio no dice nada de lo que puede en otro. |

## Consecuencias

**A favor**

- La persona tiene una sola identidad, como ya la tiene el cliente final (`RN-ARQ-013`).
- El aislamiento entre negocios queda intacto: lo único compartido es la persona, y el negocio
  no lo ve.
- Cambiar de trabajo no obliga a crear un usuario nuevo: se termina una vinculación y se crea
  otra.

**En contra**

- Aparece un concepto más, la vinculación, entre el usuario y sus permisos.
- El dato personal del usuario es de la persona y no del negocio; hay que definir qué ve el
  negocio de él (Ley 1581 de 2012).
- `iam` debe resolver el cambio de negocio en un dispositivo propio y el contexto fijo de una
  terminal.

## Reglas que cambian

- `RN-ARQ-012` queda derogada por `RN-ARQ-016`…`019`
  ([`03-principios.md`](../03-principios.md), Principio 2).
- `RN-ROL-011` y `RN-ROL-027` ([`05-actores-y-roles.md`](../05-actores-y-roles.md)).

## Preguntas que esta decisión deja abiertas

- `PA-ARQ-004` — ¿El usuario del staff y el cliente registrado son la misma cuenta de Barscode?
  El mesero de un bar es cliente de otro en su noche libre. Se resuelve al definir `iam` e
  `identidad-cliente`.
- `PA-ACT-005` — Cómo se identifica el staff (PIN, tarjeta, usuario y contraseña) sigue abierta.

## Referencias

- [`ADR-0002`](ADR-0002-saas-multi-tenant.md) — SaaS multi-tenant
- [`ADR-0006`](ADR-0006-permisos-por-modulo-y-perfiles.md) — permisos y perfiles
- [`docs/04-glosario.md`](../04-glosario.md) — Usuario, Vinculación
