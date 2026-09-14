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
