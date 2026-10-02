# Revisión del foundation y plan de mejora

> **Estado:** 🟡 Borrador · **Dueño:** Jorge Hoyos · **Actualizado:** 2026-10-02

Revisión de los 17 archivos de `docs/`, el README y las notas de la Sesión 4. Los hallazgos
describen el KB **en el commit `589f497`**; los números de línea se refieren a ese commit.
La columna *Estado* dice qué pasó con cada uno después.

Leyenda: ✅ corregido · 🟡 corregido en parte · ⏳ pendiente de decisión

## 1. Veredicto

La base es sólida y el método funciona: los tres principios son claros, las decisiones tienen
motivo escrito, y la ficha de `inventario` tiene un nivel de detalle con el que sí se puede
construir. No hay enlaces rotos ni identificadores duplicados.

Los problemas no están en lo que ya se decidió, sino en tres frentes:

1. **Decisiones de rumbo que el KB todavía no ha tomado** y que afectan a todos los módulos (§2).
2. **Deriva entre documentos**: el mismo dato vivía en varios sitios y ya se había desalineado (§3).
3. **Huecos de fondo en `inventario`** que impedirían marcarlo 🟢 aunque se cerraran sus
   preguntas abiertas (§4).

---

## 2. Hallazgos estructurales

Todos requieren decisión de Jorge.

### E-1 · El primer módulo no es entregable sin plataforma ⏳
`ADR-0005` afirma que "cada módulo terminado es entregable: se puede vender, probar y cobrar".
Pero `tenancy`, `iam` y `localizacion-co` son obligatorios (`RN-ARQ-005`) y están en los puestos
8–10 de la cola. `inventario` necesita tenant, sede, usuario, permisos y moneda para existir.
**Propuesta:** un documento "plataforma mínima" con lo que los módulos ya exigen, alimentado por
una sección nueva en la plantilla ("Lo que este módulo necesita de la plataforma"). La definición
completa de la plataforma sigue en su puesto de la cola.

### E-2 · No está escrito cuándo empieza el desarrollo ⏳
`02-alcance` §2: esta etapa produce documentación de "todos los módulos del catálogo".
`ADR-0005`: "se define **y se construye** un módulo a la vez". Son dos planes distintos: 20 fichas
y luego código, o ficha–código–ficha–código. **Propuesta:** decidirlo y escribirlo en `ADR-0005`.

### E-3 · El modelo de permisos aprobado no estaba en el KB 🟡
El 2026-09-15 se aprobó el modelo base de permisos por módulo y perfiles; vivía solo en `_work/`.
**Hecho:** registrado como [`ADR-0006`](../decisiones/ADR-0006-permisos-por-modulo-y-perfiles.md).
**Pendiente:** responder `PA-ACT-007`…`014`, definir el catálogo de Inventario (`PA-INV-016`) y
reescribir `05-actores-y-roles` §2–3, `RN-ARQ-012`, `RN-ROL-004`, `RN-ROL-005` y la plantilla §12.

### E-4 · El límite de la capa Plataforma dice tres cosas distintas ⏳
- `RN-ARQ-005`: tenancy, IAM, "configuración" (no es un módulo).
- `ADR-0001`: `tenancy`, `iam`, `localizacion-*`.
- Catálogo: esos tres más `notificaciones`, marcada obligatoria pero en el puesto 20 y tratada
  como opcional por `inventario`.

Además, `gamecenter`, `social` y `pagos` se declaran "activables con un QR" sin `salon-mesas` ni
`pedidos-qr`, pero el QR lo genera `salon-mesas` y la sesión de mesa la abre `pedidos-qr`. Nadie
es dueño del QR y la sesión cuando esos dos no están. `identidad-cliente` tampoco funciona sola.
**Propuesta:** una sola lista de plataforma; decidir si QR + sesión + identidad del cliente son
plataforma del lado cliente.

### E-5 · La fundación no puede aprobarse nunca con la regla actual ⏳
Los documentos fundacionales tienen decenas de preguntas abiertas y la regla dice que un
documento con preguntas abiertas no puede estar 🟢. Muchas dicen "se resuelve al definir `pos`" o
`localizacion-co`: la fundación quedaría 🟡 hasta el módulo 10. No existe el estado "diferida".
**Propuesta:** triage (§6) y un registro único de preguntas, con la regla de que una pregunta
diferida a un módulo deja de bloquear al documento donde nació.

### E-6 · Patrones nacidos en `inventario` que no están en los principios ⏳
`inventario` descubrió reglas que `catalogo`, `pos` y `caja` van a necesitar iguales:
- Lo confirmado no se edita: se anula con inverso o se reabre como versión nueva.
- La venta nunca se bloquea por el back office (`RN-INV-012`, `083`, `094`).
- Todo hecho lleva dos fechas: la del hecho y la de recepción (`RN-INV-100`).
- Ámbito `global` / `local` (ya reutilizado para perfiles; es la misma `PA-ARQ-002` de la carta).
- Nadie autoriza su propia acción; umbrales configurables por sede.
- "Día de operación" (la noche cruza medianoche): se usa en `RN-INV-098` y no está definido.

Si no suben a `03-principios` antes de abrir `catalogo`, cada ficha los reinventará distinto.

### E-7 · El contrato de autonomía no cubre la transición ⏳
Define qué pasa sin el vecino, no qué pasa cuando el vecino **llega después**: productos
declarados localmente frente a los de `catalogo`, estaciones por defecto frente a las de `pos`,
proveedor en texto libre frente a `compras`. **Propuesta:** quinto punto del contrato.

### E-8 · Tamaño y ritmo ⏳
`inventario.md` pasa de 970 líneas; la regla 1 del README dice partir sobre ~400. Lleva 4
sesiones sin cerrar y quedan 19 módulos. Sus 104 reglas vigentes pesan lo mismo: no hay marca de
qué es núcleo y qué es avanzado. **Propuesta:** carpeta por módulo (índice + partes) y una marca
de "primera entrega" por regla e historia. No son fases entre módulos; es un corte dentro del
módulo.

---

## 3. Contradicciones y deriva entre documentos

| Id | Hallazgo | Estado |
|---|---|---|
| D-1 | **Cuentas por pagar**: el catálogo las pone en `compras` (`06` línea 43); `inventario` §15 dice "fuera del producto". | ⏳ |
| D-2 | **Receta**: la ficha la define como entidad de `inventario` (5.9); catálogo y alcance decían que `catalogo` "define producto, receta y precio". Ligado a `PA-CAT-001`. | ⏳ |
| D-3 | **Conteo desactualizado**: el alcance decía 5 preguntas abiertas en `inventario`; eran 3. | ✅ El alcance ya no repite el conteo. |
| D-4 | **"Fase"** sobrevivía tras `ADR-0005` en el README y tres veces en la guía. | ✅ |
| D-5 | **"Local"** es sinónimo prohibido de sede y aparece 35 veces en 9 documentos, 4 en el propio glosario y en el lema ("el local manda"). Además `local` es ahora valor oficial del ámbito. | ⏳ |
| D-6 | **"Empresa"** está prohibida y se usa en la definición de Tenant. **"Cuenta"** tiene tres sentidos: la de la mesa (oficial), la del cliente registrado y la contable. | ⏳ |
| D-7 | **Insumo** en el glosario tenía de ejemplo "botella de ron 750 ml". El insumo es el ron; la botella es un empaque. | ✅ |
| D-8 | **Cortesía**: merma en el glosario, salida en la ficha 5.8. | ⏳ `PA-INV-014` |
| D-9 | **Formato de criterio**: la guía exigía `CA-HU-INV-003-2`; la ficha usa `CA-1`. | ✅ La guía admite la forma abreviada dentro de la historia. |
| D-10 | **Ejemplos de la guía** usaban identificadores reales con otro texto. | ✅ Ahora citan la ficha. |
| D-11 | **Identificadores sin declarar**: `RES-`, `SUP-`, prefijos que no son módulo, estados de ADR. La guía no llevaba cabecera. | ✅ |
| D-12 | **Git**: la guía hablaba de ramas `kb/<modulo>`; la práctica era `_work/` sin versionar. | ✅ La guía describe `_work/` y la fuente de la verdad. |
| D-13 | **Matriz de `05`**: cubre 13 de 20 módulos; se titula "actor × módulo" pero las columnas son roles; Propietario con **L** aunque su rol dice "todo". | ⏳ Se reescribe con `ADR-0006`. |
| D-14 | **Visión**: "el KDS no es opcional" frente a `PA-CAT-006`; "una aplicación móvil para sus clientes" frente a `PA-ALC-004`. `SUP-002` asume POS. | ⏳ |
| D-15 | **Preguntas duplicadas**: `PA-ALC-003` = `PA-ARQ-011`; `PA-VIS-001` ≈ `PA-ARQ-001`; `PA-ACT-004` abierta aunque ya estaba respondida. | ✅ La primera se fusionó, la segunda quedó enlazada (no son la misma pregunta) y la tercera se cerró. |
| D-16 | **"Instalado"** se usa 15 veces para un SaaS donde nada se instala. | 🟡 El glosario define *módulo habilitado*. Reemplazar la palabra en los textos queda para la decisión de vocabulario. |

**Causa de fondo de D-3 y D-4:** el estado de `inventario` estaba en 4 sitios y la cola en 3.
✅ Ahora la cola y el estado viven solo en el catálogo, y las preguntas abiertas solo en la ficha.

---

## 4. Ficha de `inventario`: hallazgos de fondo

### Reglas que faltan o se rompen

- **I-1 · Costo promedio con existencia cero o negativa.** ⏳ `RN-INV-012` garantiza que el stock
  puede quedar negativo; la fórmula de `RN-INV-014` divide entre `stock + cantidad`. Con stock −20
  y entrada de 20 divide por cero; con −20 y 30 da un costo absurdo. Falta la regla. Tampoco está
  definido a qué costo se valora un movimiento inverso (anulación) ni una devolución.
- **I-2 · Estaciones sin `pos` ni `kds`.** ⏳ `RN-INV-113` solo da la estación por defecto de cada
  bodega. El caso que originó el diseño (3 cocinas, 1 despensa) es imposible con solo Inventario,
  y eso incumple `RN-ARQ-002` (vía manual equivalente). Falta poder crear estaciones a mano.
- **I-3 · Importación de ventas sin definir.** ⏳ Es la vía principal del negocio que contrata solo
  Inventario y no tiene historia propia. No está escrito cómo se asocia un producto del POS
  externo con una receta local, qué pasa con un producto desconocido ni con un periodo importado
  dos veces.
- **I-4 · `nivel` contra la tolerancia.** ⏳ `nivel` mide en fracciones (¼, ⅓, ½, ¾): saltos de más
  de 100 ml en una botella de 750. La tolerancia es 3 % del consumo. Casi toda medición por
  `nivel` dará alerta. `PA-INV-012` se cerró, pero el problema de precisión sigue.
- **I-5 · Catálogo de alertas incompleto.** ⏳ La entidad 5.11 tiene 4 tipos; las reglas generan al
  menos 8 más (inconsistencia, trazabilidad, envase excedido, estación sin cierre, turno sin
  ventas, configuración…). `agotado` y `sobre máximo` no tienen regla. "Atendida" no dice quién
  ni cómo.
- **I-6 · Configuración huérfana.** ⏳ "Exigir conteo antes del cierre de mes" (no existe cierre de
  mes); "Exigir autorización para confirmar orden de producción" (sin regla ni permiso); "Método
  de valoración" configurable con un único valor posible.

### Checklist marcado ✔ que no se cumplía

- **I-7 ·** 🟡 `HU-INV-005` y `HU-INV-008` no tienen criterio de rechazo. `HU-INV-002 CA-4` está
  marcado "rechazo" y es una alerta. *La casilla se desmarcó; faltan los criterios.*
- **I-8 ·** 🟡 Faltaban en el glosario 12 términos que la ficha usa. *Agregados 11. Falta "día de
  operación", que no está definido en ningún sitio (E-6); la casilla quedó desmarcada.*

### Incoherencias internas menores

- **I-9 ·** ✅ §2 decía "todos con alternativa manual"; §10 decía que `tenancy` e `iam` no la tienen.
- **I-10 ·** ✅ §10 pedía "sedes y bodegas activas" a `tenancy`; las bodegas son de `inventario`.
- **I-11 ·** ⏳ `RN-INV-043` nombra un "Administrador con alcance de tenant" que no existe en `05`;
  §12 solo deja al Propietario. → `PA-INV-016`
- **I-12 ·** ⏳ El Supervisor puede reabrir un conteo (§12) pero no cerrarlo. → `PA-INV-016`
- **I-13 ·** ✅ El traslado es un solo movimiento (`RN-INV-007`), pero 5.8 tiene motivos "traslado
  entrante / saliente". *Aclarado: son los del paso entre sedes (`RN-INV-008`).*
- **I-14 ·** ✅ Abrir o prestar un envase "queda visible en el kardex" sin ser un movimiento.
  *Aclarado en la definición de kardex, según lo decidido en la Sesión 4.*
- **I-15 ·** 🟡 `HU-INV-001 CA-2` usaba "unidad de compra" *(corregido a "empaque")*.
  `HU-INV-002 CA-1` trata la gaseosa como `granel` y la definición de `granel` es "no viene en un
  envase que se abra": *pendiente, porque exige decidir si `granel` es una propiedad del insumo o
  una elección de cuánto control se quiere.*
- **I-16 ·** ⏳ `RN-INV-059` prohíbe ingresar un insumo producido salvo por orden de producción. No
  dice qué pasa con su saldo inicial ni con un sobrante de conteo.
- **I-17 ·** ✅ `RN-INV-112`…`114` están fuera de orden numérico. *Se dejó una nota: las reglas
  van agrupadas por tema.*

### Cobertura

- **I-18 ·** ⏳ Sin historia: definir una receta (con sub-recetas y modificadores), configurar un
  insumo y sus empaques, lotes y FEFO (un solo criterio), carga inicial, anular un movimiento.
- **I-19 ·** ⏳ La receta no tiene estados (vigencia, versiones). Depende de `PA-INV-008`.
- **I-20 ·** ⏳ Solo 1 de las 104 reglas vigentes está citada en un criterio de aceptación. No hay
  forma de saber qué reglas no tienen prueba.
- **I-21 ·** ⏳ La merma esperada es un solo porcentaje por receta. En cocina la merma es por
  insumo (pelar, limpiar, porcionar). Vale la pena confirmarlo antes de aprobar.

### Pendientes de la Sesión 4

✅ Registrados como preguntas abiertas de la ficha: `PA-INV-013` (merma sobre umbral que nadie ve),
`PA-INV-014` (cortesía), `PA-INV-015` (maduración), `PA-INV-016` (permisos de Inventario).

---

## 5. Preguntas que el KB todavía no se ha hecho

- **Edad y licor con cliente anónimo.** `RES-005` lo menciona; no hay pregunta registrada sobre
  cómo se verifica la mayoría de edad de alguien que pide alcohol sin cuenta.
- **Moderación de la capa social anónima.** Acoso, menores, bloqueo, denuncia. Sin pregunta.
- **Dueño del QR y de la sesión de mesa** cuando faltan `salon-mesas` o `pedidos-qr` (E-4).
- **Carga inicial de un negocio nuevo** como tema transversal (insumos, saldos, carta, personal).

---

## 6. Triage propuesto de las preguntas fundacionales

| Destino | Preguntas |
|---|---|
| **Cerrar ya** (la respuesta existe o solo depende de Jorge) | `PA-GUIA-001`, `PA-GUIA-002`, `PA-ALC-005`, `PA-CAT-001` (la ficha ya asume la separación), `PA-VIS-005` |
| **Con `ADR-0006` (permisos)** | `PA-ACT-001`, `PA-ACT-002`, `PA-ARQ-003`, `PA-ACT-007`…`014` |
| **A `ADR-0004` (pagos)** | `PA-CAT-004` |
| **Diferir a `catalogo`** (bloquean el siguiente módulo) | `PA-ARQ-002`, `PA-GLO-004` |
| **Diferir a `salon-mesas` / `pos` / `pedidos-qr`** | `PA-GLO-001`, `PA-GLO-002`, `PA-ALC-002`, `PA-ALC-004`, `PA-ACT-006`, `PA-CAT-006` |
| **Diferir a `tenancy` / `iam`** | `PA-VIS-001`, `PA-ARQ-001`, `PA-ACT-003`, `PA-ACT-005` |
| **Diferir a `localizacion-co`** | `PA-ARQ-010`, `PA-ARQ-011`, `PA-ARQ-012`, `PA-ARQ-013` |
| **Decidir al revisar el catálogo** (E-4) | `PA-CAT-002`, `PA-CAT-003`, `PA-CAT-005` |

Ya resueltas en la Sesión 5: `PA-ACT-004` (cerrada por `ADR-0006`) y `PA-ALC-003` (fusionada con
`PA-ARQ-011`).

---

## 7. Plan de mejora

### Etapa 0 — Asegurar lo que ya existe ✅
Hecha el 2026-10-02: los commits de la Sesión 4 y los cambios de la Sesión 5, con `.gitignore`,
están en `origin/main`.

### Etapa 1 — Correcciones mecánicas ✅
Aplicadas el 2026-10-02. Ninguna cambia una decisión. Quedaron fuera, porque sí exigen decidir:
la palabra "instalado" en los textos (D-16) y la gaseosa como `granel` (I-15).

### Etapa 2 — Decisiones transversales ⏳
Las decide Jorge. Cada una sale como ADR o como cambio en `03-principios`.

| # | Decisión | Sale como |
|---|---|---|
| 1 | Evolución del catálogo de permisos: `PA-ACT-007`…`014` (E-3) | Ampliación de `ADR-0006` |
| 2 | Cuándo se construye y qué es la plataforma mínima (E-1, E-2) | Revisión de `ADR-0005` |
| 3 | Lista única de plataforma; dueño del QR, la sesión y la identidad del cliente (E-4) | ADR nuevo + catálogo |
| 4 | Estado "diferida" y registro único de preguntas (E-5, §6) | Guía |
| 5 | Patrones comunes y "día de operación" (E-6) | `03-principios` + glosario |
| 6 | Quinto punto del contrato: transición (E-7) | `03-principios` + plantilla |
| 7 | Vocabulario: "local", "cuenta", "instalado" (D-5, D-6, D-16) | Glosario |
| 8 | Partir fichas y marcar primera entrega (E-8) | Guía + plantilla |

### Etapa 3 — Cerrar `inventario` ⏳
1. Reglas faltantes: I-1, I-2, I-3, I-4, I-5, I-6, I-15, I-16, I-21.
2. Con el catálogo: D-1 y D-2.
3. Sus 7 preguntas abiertas: `PA-INV-007`, `008` (con I-19), `009`, `013`, `014`, `015`, `016`.
4. Cobertura: I-7, I-18, I-20 (tabla regla → criterio).
5. Partir la ficha, revisión de Jorge, 🟢.

### Etapa 4 — Antes de abrir `catalogo` ⏳
1. Plantilla actualizada: plataforma, transición, carga inicial, catálogo de alertas, permisos
   con código, trazabilidad regla → criterio.
2. Cerrar `PA-ARQ-002` y `PA-GLO-004`, que definen el arranque de `catalogo`.
3. Registrar las preguntas de §5 en el documento que corresponda.

### Orden y dependencias

```
Etapa 0 ──► Etapa 2 ──► Etapa 3 ──► Etapa 4 ──► catalogo
              │            ▲
              └─ la decisión 1 (permisos) bloquea PA-INV-016
```

La Etapa 3 puede avanzar en paralelo con la 2, salvo los permisos.
