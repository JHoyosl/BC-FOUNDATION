# Cómo usar y mantener este KB

> **Estado:** 🟡 Borrador · **Dueño:** Jorge Hoyos · **Actualizado:** 2026-10-02

## Para qué existe

Barscode se detuvo la primera vez porque el alcance no estaba escrito: cada decisión abría
tres decisiones nuevas y el trabajo se volvió una bola de nieve. Este KB es la respuesta a
eso. Su función es **cerrar preguntas antes de que cuesten código**.

Regla que lo resume: *ningún módulo entra a desarrollo mientras su ficha tenga preguntas
abiertas.*

## Qué es y qué no es

**Sí es:** la definición funcional del producto — qué hace, para quién, bajo qué reglas,
con qué estados y qué límites.

**No es:** diseño técnico. Aquí no se decide lenguaje, framework, base de datos,
infraestructura ni esquema de tablas. Cuando el texto menciona una "interfaz" entre módulos,
se refiere al **contrato funcional** (qué información pide uno y qué le responde el otro),
no a un endpoint HTTP. La capa técnica de cada módulo se abrirá en `docs/tecnico/` cuando su
definición funcional esté 🟢.

## Convenciones

### Estados de un documento

Todo documento lleva un bloque de cabecera:

```
> **Estado:** 🟡 Borrador · **Dueño:** Jorge Hoyos · **Actualizado:** 2026-09-14
```

| Estado | Significa |
|---|---|
| ⚪ No iniciado | Existe el hueco, no el contenido. |
| 🔴 Incompleto | Hay contenido, faltan secciones obligatorias. |
| 🟡 Borrador | Completo, pendiente de revisión de Jorge. |
| 🟢 Aprobado | Revisado. Sirve como fuente para desarrollo. |
| 🔵 Congelado | Aprobado y cerrado: el módulo entró a desarrollo. Cambiarlo exige un ADR. |

Un documento con **Preguntas abiertas** no puede estar 🟢. Sin excepción.

Los ADR llevan su propio estado: `Aceptada`, `Pendiente` (la pregunta está registrada y aún no
se decide) o `Reemplazada por ADR-nnnn`. El README y la bitácora no llevan cabecera de estado.

### Identificadores

| Tipo | Formato | Ejemplo |
|---|---|---|
| Módulo | `kebab-case` | `inventario`, `pedidos-qr` |
| Requisito / regla | `RN-<MOD>-<nnn>` | `RN-INV-014` |
| Historia de usuario | `HU-<MOD>-<nnn>` | `HU-INV-003` |
| Criterio de aceptación | `CA-<HU>-<n>` | `CA-HU-INV-003-2` |
| Permiso | `PRM-<MOD>-<nnn>` | `PRM-INV-001` |
| Decisión | `ADR-<nnnn>` | `ADR-0001` |
| Pregunta abierta | `PA-<MOD>-<nnn>` | `PA-INV-005` |
| Restricción | `RES-<nnn>` | `RES-003` |
| Supuesto | `SUP-<nnn>` | `SUP-001` |

`<MOD>` es la abreviatura del módulo (`INV`). En los documentos fundacionales es la del
documento o del tema: `GUIA`, `VIS`, `ALC`, `ARQ`, `GLO`, `ACT`, `ROL`, `CAT`.

Dentro de su historia, un criterio se escribe abreviado (`CA-2`). Desde fuera de la historia se
cita con el identificador completo (`CA-HU-INV-003-2`).

Los identificadores **no se reciclan**. Si una regla se elimina, se marca `(derogada por
RN-INV-021)` y se deja en el documento.

### Redacción de reglas de negocio

Una regla de negocio es una afirmación verificable, en presente, sin condicionales vagos.

- ✅ `RN-INV-005` — Una corrección se hace con un movimiento de ajuste que referencia al original y explica el motivo.
- ❌ "El inventario debería poder corregirse de alguna manera si el usuario se equivoca."

### Historias de usuario

```
HU-INV-003 — Registrar una merma
Como jefe de barra
quiero registrar una botella rota
para que el stock refleje la realidad y la pérdida quede atribuida.

CA-1 — Dado un insumo con stock 12 y CPP $40.000 COP, cuando registro merma de 1 con motivo
       "rotura", entonces el stock queda en 11 y el movimiento registra pérdida de
       $40.000 COP, mi usuario y la hora.
CA-4 (rechazo) — Dado stock 0 y stock negativo deshabilitado, cuando intento registrar merma
       de 1, entonces el sistema la rechaza indicando existencia insuficiente.
```

Criterios en formato Dado / Cuando / Entonces, siempre incluyendo al menos un caso de
rechazo. La mitad de la bola de nieve vive en los caminos infelices.

## Flujo de trabajo

1. **Se abre** la ficha del módulo desde `docs/modulos/_plantilla.md`. Estado ⚪ → 🔴.
2. **Se define** en sesión: objetivo, actores, reglas, entidades, estados, historias.
3. **Se cierran preguntas.** Cada duda se escribe como `PA-…`; la sesión termina cuando no
   queda ninguna o las que quedan se escalan a ADR.
4. **Revisión de Jorge.** 🟡 → 🟢.
5. **Congelado** al entrar el módulo a desarrollo. 🟢 → 🔵.

Un módulo nunca se define aislado del [catálogo](06-catalogo-modulos.md): al cerrarlo hay que
revisar si cambió algún contrato con otro módulo.

## Fuente de la verdad y trabajo en curso

Hay una sola fuente de la verdad, con tres niveles que no compiten entre sí:

1. **`docs/` en `main`** — todo lo que ya está decidido. Es lo único que desarrollo lee.
2. **`_work/`** — notas locales de la sesión en curso (`sesion-<AAAA-MM-DD>.md`) y borradores.
   No se versiona (está en `.gitignore`) y no viaja a otra máquina. Vale solo mientras dura el
   trabajo.
3. **El historial de chat** — nunca es fuente. Lo que no quedó escrito en 1 o 2 no está
   decidido.

Reglas:

- Lo que se decide en una sesión se anota en `_work/` en el momento, no al final.
- Al cerrar un bloque de trabajo, lo decidido pasa a `docs/` y la sesión queda resumida en la
  [bitácora](bitacora.md). **Nada decidido vive solo en `_work/`.**
- Lo que queda sin decidir pasa a **Preguntas abiertas** del documento que corresponda, con la
  propuesta que se haya discutido.
- Si `docs/` y `_work/` se contradicen, manda `docs/`, salvo que la nota de sesión diga
  expresamente que está revisando una regla.

Una revisión completa del KB se guarda en `docs/revisiones/<AAAA-MM-DD>-<tema>.md`, con sus
hallazgos y el estado de cada uno.

## Uso de Git

Rama `main` es la verdad. Cambios grandes en rama `kb/<modulo>` y merge cuando el módulo
quede 🟡.

Mensajes de commit:

```
kb(<modulo>): <qué se definió o cambió>
adr(<nnnn>): <decisión tomada>
docs: <cambios de estructura o guía>
```

Ejemplos reales:

```
kb(inventario): definir estados de movimiento y reglas de ajuste
adr(0003): aislar impuestos y facturación como artefacto de país
```

## Preguntas abiertas

- `PA-GUIA-001` — ¿Se versiona el KB con tags (`kb-v1.0`) al congelar cada módulo, o basta el historial de Git?
- `PA-GUIA-002` — ¿El KB se mantiene solo en español, o las fichas de módulo necesitan versión en inglés para terceros?
