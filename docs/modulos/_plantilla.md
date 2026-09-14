# Módulo: `<id-modulo>` — <Nombre>

> **Estado:** ⚪ No iniciado · **Dueño:** Jorge Hoyos · **Fase:** <0|1|2|3> · **Actualizado:** AAAA-MM-DD

Copiar este archivo a `docs/modulos/<id-modulo>.md` y completar. **Ninguna sección se borra.**
Si una sección no aplica, se escribe *"No aplica porque…"*. Los huecos silenciosos son
exactamente lo que este KB existe para evitar.

---

## 1. Objetivo

Una frase: qué problema resuelve este módulo y para quién. Si no cabe en una frase, el
módulo está mal delimitado.

## 2. Contrato de autonomía

Obligatorio. Ver [Principio 1](../03-principios.md).

| | |
|---|---|
| **Funciona solo** | Qué hace este módulo si es lo único instalado. |
| **Degradación** | Qué se pierde o pasa a manual si falta cada módulo vecino. |
| **Consume** | Qué pide a otros módulos cuando están, y la alternativa manual cuando no. |
| **Expone** | Qué información y operaciones ofrece a quien las quiera usar. |

## 3. Actores

Quién lo usa y para qué. Referenciar [`05-actores-y-roles.md`](../05-actores-y-roles.md); no
inventar actores nuevos sin agregarlos allí.

## 4. Conceptos propios

Términos que este módulo introduce. Todo término nuevo debe subir al
[glosario](../04-glosario.md).

## 5. Entidades

Por cada entidad: qué representa, sus atributos relevantes **a nivel funcional** (sin tipos
de datos ni claves), y su relación con otras. No es un modelo de base de datos.

## 6. Estados y transiciones

Por cada entidad con ciclo de vida: los estados, quién puede provocar cada transición, y qué
transiciones están prohibidas. Incluir siempre el diagrama.

## 7. Reglas de negocio

Numeradas `RN-<MOD>-nnn`. Afirmaciones verificables, en presente. Agrupadas por tema.

## 8. Historias de usuario

Numeradas `HU-<MOD>-nnn`, con criterios `CA-…` en formato Dado / Cuando / Entonces.
**Cada historia lleva al menos un criterio de rechazo.**

## 9. Casos límite y errores

Qué pasa cuando algo sale mal: datos inconsistentes, concurrencia, sin conexión, operación
a medio terminar, valores extremos.

## 10. Interfaces funcionales

Qué le pide este módulo a otros y qué les ofrece. Nivel funcional: qué información, no cómo
viaja.

| Interfaz | Dirección | Con quién | Qué información | Si el otro no existe |
|---|---|---|---|---|

## 11. Dependencias de localización

Qué de este módulo depende del artefacto de país (`RN-ARQ-020`). Si nada depende, decirlo
explícitamente.

## 12. Permisos

Qué puede hacer cada rol en este módulo. Debe cuadrar con la matriz de
[`05-actores-y-roles.md`](../05-actores-y-roles.md).

## 13. Reportes e información que entrega

Qué preguntas puede responder este módulo con sus datos.

## 14. Configuración

Qué es parametrizable por tenant o por sede, con su valor por defecto.

## 15. Fuera de alcance de este módulo

Lo que la gente asumiría que hace y no hace, con el motivo y el módulo que sí lo hace.

## 16. Preguntas abiertas

Numeradas `PA-<MOD>-nnn`. **Mientras quede una, el módulo no puede estar 🟢.**

## 17. Checklist de cierre

- [ ] Las 16 secciones anteriores están completas
- [ ] El contrato de autonomía declara los cuatro puntos
- [ ] El módulo hace algo útil si es lo único instalado
- [ ] Todo dato que consume tiene alternativa manual
- [ ] Todos los datos están atados a tenant y, si son operativos, a sede
- [ ] No hay tasas, impuestos, formatos legales ni festivos escritos dentro del módulo
- [ ] Todos los importes llevan moneda
- [ ] Todo término nuevo está en el glosario
- [ ] Toda historia tiene al menos un criterio de rechazo
- [ ] No quedan preguntas abiertas (o las que quedan se escalaron a ADR)
- [ ] La ficha se entiende sin abrir la ficha de otro módulo
- [ ] Revisada y aprobada por Jorge
