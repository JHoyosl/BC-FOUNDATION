# Bitácora

Registro cronológico de lo que se definió, lo que se decidió y lo que sigue.

---

## 2026-09-14 — Sesión 1: fundación del KB

### Punto de partida

Barscode tiene un MVP pequeño e incompleto. El proyecto se detuvo porque el KB nunca quedó
bien definido: el alcance crecía sin control ("bola de nieve"). La decisión de esta sesión es
**reiniciar la definición desde cero**, por módulos, sin heredar las decisiones del MVP.

### Qué se definió

- **Estructura del KB** y sus convenciones: estados de documento, identificadores
  (`RN-`, `HU-`, `CA-`, `PA-`, `ADR-`), formato de reglas e historias, flujo de trabajo,
  convención de commits. → `docs/00-guia.md`
- **Visión de producto**: plataforma SaaS de gestión para bares y restaurantes, con
  Gamecenter y capa social para los clientes del local. Tesis: *el QR de la mesa no es un
  menú digital, es la puerta de entrada a la experiencia del local*. → `docs/01-vision.md`
- **Alcance y fases**: Fase 0 plataforma, Fase 1 operación mínima vendible, Fase 2
  diferenciación (Gamecenter, social, pagos, compras, turnos), Fase 3 escala. Lista explícita
  de lo que queda fuera del producto. → `docs/02-alcance.md`
- **Tres principios de arquitectura funcional** con sus ADR. → `docs/03-principios.md`
- **Glosario** con la distinción clave *insumo ≠ producto*, que es lo que permite que
  Inventario y Catálogo sean módulos separados. → `docs/04-glosario.md`
- **Actores y roles** con matriz de permisos por módulo. → `docs/05-actores-y-roles.md`
- **Catálogo de 21 módulos** en cuatro dominios, con prueba de autonomía de cada uno y orden
  de definición propuesto. → `docs/06-catalogo-modulos.md`
- **Plantilla de ficha de módulo** de 17 secciones con checklist de cierre.
  → `docs/modulos/_plantilla.md`
- **Módulo `inventario` completo**, como referencia del nivel de detalle esperado:
  37 reglas de negocio, 8 historias con criterios de aceptación, entidades, estados,
  interfaces, permisos, configuración y 10 preguntas abiertas.
  → `docs/modulos/inventario.md`

### Decisiones tomadas

| ADR | Decisión |
|---|---|
| [`ADR-0001`](decisiones/ADR-0001-modulos-autonomos.md) | Todo módulo funciona, se vende y se define por sí solo. Cada ficha declara un contrato de autonomía de cuatro puntos. |
| [`ADR-0002`](decisiones/ADR-0002-saas-multi-tenant.md) | SaaS multi-tenant. Jerarquía tenant → sede → zona → mesa. El cliente final pertenece a la plataforma, no al tenant. |
| [`ADR-0003`](decisiones/ADR-0003-localizacion-como-artefacto.md) | Nada de país vive dentro de un módulo. Colombia se implementa como artefacto sustituible detrás de interfaces declaradas. |

### Preguntas abiertas más urgentes

Estas bloquean o condicionan el trabajo siguiente:

| Id | Pregunta | Bloquea |
|---|---|---|
| `PA-VIS-003` | ¿Segmento inicial exacto: coctelería, discotecas, restaurantes casuales, cadenas? | Todo el producto. El Gamecenter no significa lo mismo en un bar que en un restaurante familiar. |
| `PA-ALC-001` / `PA-VIS-004` | ¿El Gamecenter sube a Fase 1? Es el diferenciador comercial y hoy está en Fase 2. | Orden de construcción |
| `PA-ALC-002` | ¿Se exige operación sin conexión en Fase 1? | Complejidad de `pos`, `kds`, `inventario` |
| `PA-ALC-003` / `PA-ARQ-010` | ¿Facturación electrónica DIAN en Fase 1? ¿Directa o vía proveedor autorizado? | `localizacion-co`, `caja`, `pos` |
| `PA-VIS-002` | ¿Barscode procesa pagos o solo integra pasarelas? | `pagos`, modelo de negocio, regulación |
| `PA-CAT-001` | ¿`catalogo` e `inventario` separados? (aquí van separados) | `catalogo`, `pos` |
| `PA-VIS-006` | ¿El MVP existente tiene usuarios reales hoy? | Restricciones de migración |

### Qué sigue

1. Resolver las preguntas urgentes de la tabla anterior.
2. Revisar y aprobar (🟢) los documentos fundacionales.
3. Definir `catalogo` — desbloquea POS y pedidos por QR.
4. Definir `salon-mesas`.
5. Definir `pos`.

### Estado al cierre de la sesión

| Bloque | Estado |
|---|---|
| Guía y convenciones | 🟡 |
| Visión | 🟡 |
| Alcance y fases | 🟡 |
| Principios | 🟡 |
| Glosario | 🟡 |
| Actores y roles | 🟡 |
| Catálogo de módulos | 🟡 |
| `inventario` | 🟡 |
| Otros 20 módulos | ⚪ |
| Capa técnica | ⚪ (no iniciada a propósito) |

---

## 2026-09-14 — Sesión 2: repo en GitHub y cierre de bloqueantes

### Infraestructura

- Repo publicado en **https://github.com/JHoyosl/BC-FOUNDATION**, rama `main`, commit `405f7f7`.
  Contenido verificado byte por byte contra la copia de trabajo.
- Ruta de trabajo en la máquina `pc` (Windows): `E:\dev\BC-FOUNDATION`.
  Al cambiar de máquina hay que confirmar la ruta antes de tocar nada.
- Decidido: las próximas sesiones se abren desde **claude.ai/code seleccionando
  `BC-FOUNDATION`**, para que los commits los haga Claude directamente.

### Decisiones tomadas

| ADR | Decisión |
|---|---|
| [`ADR-0004`](decisiones/ADR-0004-procesamiento-de-pagos.md) | **Abierta a propósito.** ¿Barscode custodia el dinero de los pagos o solo integra pasarelas? Documentada sin resolver; bloquea el módulo `pagos`. |
| [`ADR-0005`](decisiones/ADR-0005-construccion-modulo-a-modulo.md) | **Se elimina el plan de fases.** Se construye un módulo a la vez, hasta terminarlo. Primero `inventario`. |

`ADR-0005` corrige una contradicción del KB: `ADR-0001` decía que cada módulo vale por sí
solo, y a la vez el alcance los agrupaba en fases que debían terminar juntas. Se eliminó el
agrupamiento y se reemplazó por una cola reordenable.

### Preguntas cerradas

| Id | Resolución |
|---|---|
| `PA-VIS-003` | **Segmento: mixto bar-restaurante.** Cocina y barra conviven. Es el caso más exigente: dos naturalezas de inventario (volumen/masa), dos estaciones, conversión entre dimensiones, perecederos reales, producción interna. |
| `PA-VIS-002` | Escalada a `ADR-0004`. |
| `PA-VIS-004` | Disuelta: sin fases, la pregunta de "¿el Gamecenter sube a Fase 1?" no aplica. |
| `PA-VIS-006` | El MVP solo tuvo pruebas de Jorge. **Sin usuarios de terceros, sin restricción de migración.** `RES-004` levantada. |
| `PA-ALC-001` | Disuelta por `ADR-0005`. |

### Correcciones

- El catálogo decía "21 módulos"; el conteo real es **20**. Corregido en catálogo, README y
  resúmenes.
- La columna *Fase* del catálogo se reemplazó por *Cola*, con la posición de cada módulo.
- Se eliminaron las referencias a "Fase 1" repartidas por guía, principios, alcance y ADR-0003.

### Efecto del segmento sobre `inventario`

Elegir mixto bar-restaurante convierte cuatro preguntas abiertas de "quizá" en "sí":

- `PA-INV-003` conversión ml ↔ g por densidad — **necesaria**, las recetas de cocina la usan.
- `PA-INV-005` lote y vencimiento — **necesaria**, la comida vence.
- `PA-INV-006` producción interna (almíbares, salsas, despiece) — **necesaria**.
- `PA-INV-010` botella abierta servida por trago — **necesaria**, es la mayor fuente de descuadre en barra.

### Qué sigue

Cerrar las 10 preguntas abiertas de [`inventario`](modulos/inventario.md) y aprobarlo.
Nada más entra a definición hasta entonces (`RN-ARQ-006`).

`PA-ALC-002` (¿operación sin conexión?) hay que resolverla en el camino: `PA-INV-004`
depende de ella.

---

## 2026-09-14 — Sesión 3: primera tanda de decisiones de `inventario`

Siete preguntas cerradas, dos nuevas derivadas. El módulo pasa de 10 a 5 preguntas abiertas.

### Decisiones

| Id | Decisión | Reglas nuevas |
|---|---|---|
| `PA-INV-001` | **Maestro común + insumos locales.** Un insumo es `global` (tenant) o `local` (sede). Las existencias y los costos son **siempre** por sede y bodega: compartir la ficha no es compartir el stock. | `RN-INV-038`…`044` |
| `PA-INV-002` | **Costo promedio ponderado**, confirmado. | — |
| `PA-INV-003` | **Prohibida la conversión entre dimensiones.** Sin densidad, sin factor ml↔g. El caso real se resuelve con **empaques que declaran su contenido en la unidad base**. | `RN-INV-045`…`050` |
| `PA-INV-004` | **Inventario no tiene modo sin conexión.** Sin red no registra, no encola y no muestra existencias. | `RN-INV-067`…`069` |
| `PA-INV-005` | **Lote y vencimiento opcionales por insumo.** Cocina sí, licor no. | — |
| `PA-INV-006` | **Producción interna con orden de producción.** Consume insumos, produce otro insumo con costo real derivado. Soporta subproductos (despiece). | `RN-INV-058`…`066` |
| `PA-INV-010` | **Método de control de envase abierto configurable por insumo**: `unidad`, `nivel`, `peso`, `apertura`. La venta siempre descuenta por receta; el método define cómo se verifica el remanente. | `RN-INV-051`…`057` |

### El hallazgo de la sesión: empaques en lugar de densidad

Jorge decidió prohibir la conversión entre dimensiones. Eso dejaba sin resolver un caso
real: *la crema se compra en tetrapak de 1 litro y se consume en gramos.*

La salida no fue reabrir la decisión, sino cambiar dónde vive el dato. **Un empaque declara
su contenido en la unidad base del insumo**: el tetrapak declara 1030 g. Nunca se convierte
volumen a masa; se declara una vez cuánto pesa ese envase concreto.

Resulta ser más exacto que una densidad genérica —dos cremas distintas pesan distinto— y no
obliga a nadie a hacer cuentas. La restricción mejoró el diseño.

### Preguntas nuevas, derivadas de las decisiones

- `PA-INV-011` — Si el POS vende durante un corte de red, esas ventas llegan tarde.
  ¿Se aplican al inventario al reconectar, o se descartan y la diferencia se corrige con un
  conteo? `RN-INV-068` asume lo primero; falta confirmarlo.
- `PA-INV-012` — ¿Qué diferencia se considera aceptable con método `nivel` antes de generar
  alerta? Sin ese número, `RN-INV-055` no es verificable.

### Estado de `inventario`

| | Antes | Ahora |
|---|---|---|
| Reglas de negocio | 37 | **69** |
| Historias de usuario | 8 | **11** |
| Entidades | 9 | **12** |
| Preguntas abiertas | 10 | **5** |

También se cerró `PA-GLO-003` del glosario, que era la misma pregunta que `PA-INV-001`.

### Qué sigue

Las 5 preguntas restantes de `inventario`: `PA-INV-007` (envases retornables),
`PA-INV-008` (quién aprueba recetas), `PA-INV-009` (conteo con móvil),
`PA-INV-011` y `PA-INV-012`. Al cerrarlas, revisión y aprobación 🟢.

---

## 2026-09-15 — Sesión 4: envases abiertos por estación y modelo de permisos

> Registrada el 2026-10-02. La sesión quedó en pausa y sus notas vivían solo en `_work/`.

### Punto de partida

Revisión completa del KB (17 archivos). Diez inconsistencias: cuatro de fondo en `inventario`,
dos entre documentos y cuatro mecánicas.

### Correcciones mecánicas — commit `caf7576`

- README: de 10 a 5 preguntas abiertas de `inventario`.
- Visión §4.1: empaques sin densidad, lote por insumo, orden de producción.
- Principios: `RN-ARQ-006` agregada a las reglas del Principio 1.
- `inventario` §15: retirada la fila "Producción y transformación compleja".

### Bloque A: envases abiertos — commit `589f497`

La ficha tenía tres contradicciones. `peso` convertía gramos a mililitros restando la tara, algo
prohibido por `RN-INV-046` y con error real en cada conteo. `RN-INV-052` (la venta siempre
descuenta por receta) chocaba con `RN-INV-053` (`apertura` descuenta al abrir): doble descuento.
Y con `nivel` o `peso` no había regla de cuándo se abre un envase ni de cuál sale cada venta.

| Punto | Decisión |
|---|---|
| A.1 | Dos ajustes por insumo: **tipo de control** (`unidad`, `granel`, `envase abierto`) y **verificación** (`al finalizar`, `nivel`, `peso`). `apertura` desaparece. Con `peso`, el empaque declara peso vacío y lleno, y el contenido sale por proporción, sin densidad. |
| A.2 | La botella abierta no es otro insumo. La existencia tiene envases cerrados (un número) y el **envase en servicio** (un registro con historia). Abrir no cambia existencia ni costo. Porción de referencia y venta perdida. |
| A.3 | El envase abierto es de la **estación** que lo abrió, no de la bodega: uno por insumo y estación. **Turno de estación** con cierre y apertura; la diferencia entre turnos no se carga a ninguno. Sin cierre no se bloquea nada. |
| A.4 | Venta sin envase en servicio: alerta de trazabilidad y ml pendientes para el próximo envase. El sistema no abre envases por su cuenta. |
| A.5 | Venta tardía de un periodo ya medido: no cambia la existencia, reclasifica la diferencia por **fecha del hecho** y deja nota. Estaciones `por archivo` con diferencia pendiente de ventas. Archivo sin hora: conciliación por día y estación. Hora declarada tras un corte de red. |
| A.6 | Se miden faltante y sobrante. Alerta de envase excedido. |
| A.7 | La diferencia es un **ajuste**, no una merma, con motivo *diferencia de envase*. Tolerancia por insumo sobre el consumo del periodo medido. |
| A.8 | El conteo físico cuenta envases cerrados. *Incluir abiertas* para auditoría. Línea por recontar si se abre un envase durante el conteo. |
| A.9 | Reabrir una medición es anular con inversos y crear una versión nueva, hasta que cierre la medición siguiente. Después, nota de corrección. Nadie autoriza su propia reapertura. |

Además: toda bodega de venta tiene una **estación por defecto**, y `inventario` reconoce las
estaciones por su identificador, venga del módulo que venga (`RN-INV-112`, `113`). Un envase en
servicio puede **prestarse** a otra estación, con medición opcional (`RN-INV-114`).

### Preguntas cerradas

| Id | Resolución |
|---|---|
| `PA-INV-010` | Revisada: tipo de control y verificación por insumo; el envase abierto pertenece a una estación. |
| `PA-INV-011` | Las ventas tardías sí se aplican; si hubo una medición posterior, reclasifican su diferencia. |
| `PA-INV-012` | Reemplazada por la tolerancia por insumo. |

### Estado de `inventario`

| | Antes | Ahora |
|---|---|---|
| Reglas de negocio | 69 | **114** (10 derogadas) |
| Historias de usuario | 11 | **14** |
| Entidades | 12 | **14** |
| Preguntas abiertas | 5 | **3** |

### Bloque B: permisos

Al alinear los permisos de la ficha con `05-actores-y-roles`, Jorge replanteó el modelo:
**permisos por módulo y perfiles a la medida**. El modelo base quedó aprobado y está en
[`ADR-0006`](decisiones/ADR-0006-permisos-por-modulo-y-perfiles.md). Cierra `PA-ACT-004`.

Quedó presentado y sin respuesta cómo evoluciona el catálogo de permisos: `PA-ACT-007`…`014`.

Sin empezar: la merma sobre umbral que nadie ve para autorizar (`PA-INV-013`), si la cortesía es
merma o salida (`PA-INV-014`), la maduración (`PA-INV-015`) y el catálogo de permisos de
Inventario (`PA-INV-016`).

### Infraestructura

- La sesión se hizo en la Mac: `/Users/jorgehoyos/dev/BC-FOUNDATION`.
- El trabajo en curso se llevó en `_work/`, fuera de Git.

---

## 2026-10-02 — Sesión 5: fuente de la verdad, revisión del foundation y correcciones

### Fuente de la verdad

Una sola, con tres niveles: `docs/` en `main` es lo decidido; `_work/` son notas locales de la
sesión en curso; el historial de chat nunca es fuente. Nada decidido vive solo en `_work/`.
→ [`00-guia.md`](00-guia.md)

En consecuencia se volcó al KB lo que solo estaba en `_work/`: `ADR-0006`, las preguntas
`PA-ACT-007`…`014` y `PA-INV-013`…`016`, y la entrada de la Sesión 4 de esta bitácora.

### Revisión completa del foundation

→ [`revisiones/2026-10-02-foundation.md`](revisiones/2026-10-02-foundation.md)

45 hallazgos: 8 estructurales, 16 de deriva entre documentos y 21 en la ficha de `inventario`.
Incluye el reparto propuesto de las preguntas fundacionales y un plan de mejora en cinco etapas.

### Correcciones mecánicas aplicadas

Ninguna cambia una decisión.

- **Un solo lugar para la cola y el estado de los módulos:** el catálogo. README, alcance y
  `ADR-0005` enlazan en lugar de repetir. Desaparece el conteo desactualizado del alcance.
- **Guía:** cabecera de estado; identificadores `PRM-`, `RES-` y `SUP-`; forma abreviada de los
  criterios (`CA-2`); estados de los ADR; ejemplos tomados de la ficha real; sin la palabra
  "fase"; sección nueva sobre la fuente de la verdad y `_work/`.
- **Glosario:** 11 términos que la ficha usaba y faltaban; *módulo habilitado*; *permiso* y
  *perfil*; ejemplo de insumo corregido.
- **`inventario`:** `tenancy` e `iam` sin alternativa manual en §2; "sedes activas" en §10; el
  kardex muestra eventos de envase; aclaración de los motivos de traslado; "empaque" en lugar de
  "unidad de compra". En el checklist se desmarcaron dos casillas que no se cumplían.
- **Preguntas:** `PA-ALC-003` fusionada con `PA-ARQ-011`.

### Estado de `inventario`

Las preguntas abiertas pasan de 3 a **7**. No son dudas nuevas: cuatro pendientes de la Sesión 4
que solo estaban en `_work/`.

### Qué sigue

El plan está en la revisión. En orden:

1. Decisiones transversales, empezando por `PA-ACT-007`…`014`, que desbloquean los permisos.
2. Cerrar `inventario`: reglas faltantes, sus 7 preguntas, cobertura de historias, aprobación.
3. Actualizar la plantilla antes de abrir `catalogo`.
