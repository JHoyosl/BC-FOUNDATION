# Cómo usar y mantener este KB

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
no a un endpoint HTTP. La capa técnica se abrirá en `docs/tecnico/` cuando la Fase 1
funcional esté 🟢.

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
| 🔵 Congelado | Aprobado y cerrado para la fase en curso. Cambiarlo exige un ADR. |

Un documento con **Preguntas abiertas** no puede estar 🟢. Sin excepción.

### Identificadores

| Tipo | Formato | Ejemplo |
|---|---|---|
| Módulo | `kebab-case` | `inventario`, `pedidos-qr` |
| Requisito / regla | `RN-<MOD>-<nnn>` | `RN-INV-014` |
| Historia de usuario | `HU-<MOD>-<nnn>` | `HU-INV-003` |
| Criterio de aceptación | `CA-<HU>-<n>` | `CA-HU-INV-003-2` |
| Decisión | `ADR-<nnnn>` | `ADR-0001` |
| Pregunta abierta | `PA-<MOD>-<nnn>` | `PA-INV-005` |

Los identificadores **no se reciclan**. Si una regla se elimina, se marca `(derogada por
RN-INV-021)` y se deja en el documento.

### Redacción de reglas de negocio

Una regla de negocio es una afirmación verificable, en presente, sin condicionales vagos.

- ✅ `RN-INV-007` — Un movimiento de inventario confirmado no se edita; se corrige con un movimiento de ajuste que lo referencia.
- ❌ "El inventario debería poder corregirse de alguna manera si el usuario se equivoca."

### Historias de usuario

```
HU-INV-003 — Registrar merma
Como jefe de barra
quiero registrar una botella rota como merma
para que el stock refleje la realidad y el costo quede atribuido.

CA-HU-INV-003-1 — Dado un producto con stock 12, cuando registro merma de 1 con motivo
                  "rotura", entonces el stock queda en 11 y el movimiento queda con motivo
                  y responsable.
CA-HU-INV-003-2 — Dado un producto con stock 0, cuando intento registrar merma, entonces
                  el sistema lo rechaza indicando stock insuficiente.
```

Criterios en formato Dado / Cuando / Entonces, siempre incluyendo al menos un caso de
rechazo. La mitad de la bola de nieve vive en los caminos infelices.

## Flujo de trabajo

1. **Se abre** la ficha del módulo desde `docs/modulos/_plantilla.md`. Estado ⚪ → 🔴.
2. **Se define** en sesión: objetivo, actores, reglas, entidades, estados, historias.
3. **Se cierran preguntas.** Cada duda se escribe como `PA-…`; la sesión termina cuando no
   queda ninguna o las que quedan se escalan a ADR.
4. **Revisión de Jorge.** 🟡 → 🟢.
5. **Congelado** al entrar la fase a desarrollo. 🟢 → 🔵.

Un módulo nunca se define aislado del [catálogo](06-catalogo-modulos.md): al cerrarlo hay que
revisar si cambió algún contrato con otro módulo.

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

- `PA-GUIA-001` — ¿Se versiona el KB con tags (`kb-v1.0`) al congelar cada fase, o basta el historial de Git?
- `PA-GUIA-002` — ¿El KB se mantiene solo en español, o las fichas de módulo necesitan versión en inglés para terceros?
