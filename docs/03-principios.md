# Principios de arquitectura funcional

> **Estado:** 🟡 Borrador · **Dueño:** Jorge Hoyos · **Actualizado:** 2026-09-14

Tres reglas gobiernan todo el diseño de Barscode. Cualquier definición de módulo que las
contradiga está mal y se corrige. Cada una tiene su ADR:
[`ADR-0001`](decisiones/ADR-0001-modulos-autonomos.md),
[`ADR-0002`](decisiones/ADR-0002-saas-multi-tenant.md),
[`ADR-0003`](decisiones/ADR-0003-localizacion-como-artefacto.md).

---

## Principio 1 — Módulos autónomos

### El enunciado

> **Todo módulo debe funcionar, instalarse, venderse y operar por sí solo, sin que ningún
> otro módulo de Barscode exista.**

Ejemplo canónico: **Inventario** controla productos, cantidades, unidades de medida,
movimientos y costos. Un negocio debe poder contratar únicamente Inventario y usarlo
completo — con carga manual de salidas si quiere — exista o no el módulo POS. Cuando POS sí
existe, Inventario *aprovecha* sus ventas para descontar stock; pero eso es una mejora, no
un requisito.

### Por qué

- **Comercial:** permite vender por partes y entrar a negocios que ya tienen POS.
- **De producto:** un módulo que solo funciona acompañado es un módulo mal delimitado.
- **De proyecto:** es lo que evita la bola de nieve. Módulos autónomos se definen, aprueban
  y construyen de a uno, sin que definir Inventario obligue a definir POS primero.

### Qué significa en la práctica

Cada ficha de módulo debe declarar un **contrato de autonomía** con cuatro puntos:

1. **Funciona solo.** Qué hace el módulo cuando es lo único instalado.
2. **Degradación.** Qué funcionalidad se pierde o pasa a manual si un módulo vecino no está.
3. **Consume.** Qué información le pide a otros módulos *cuando están presentes*, y qué hace
   si no lo están.
4. **Expone.** Qué información y qué operaciones ofrece a quien quiera usarlas.

Ejemplo, para Inventario:

| | |
|---|---|
| **Funciona solo** | Catálogo de insumos, unidades, stock por bodega, entradas, salidas, ajustes, mermas, conteos, costeo, alertas de mínimo. |
| **Degradación** | Sin POS: las salidas por venta se registran manualmente o por importación. Sin Compras: las entradas se registran manualmente. |
| **Consume** | De POS: ventas confirmadas (para descontar). De Compras: recepciones (para ingresar). Ambos opcionales. |
| **Expone** | Existencia y costo de un insumo; capacidad de descontar/ingresar cantidades; alertas de stock. |

### Reglas derivadas

- `RN-ARQ-001` — Un módulo no puede bloquear una operación propia por la ausencia de otro módulo.
- `RN-ARQ-002` — Todo dato que un módulo consuma de otro debe tener una vía de captura manual equivalente.
- `RN-ARQ-003` — La comunicación entre módulos ocurre siempre a través del contrato declarado en la ficha, nunca por conocimiento implícito de los datos del otro.
- `RN-ARQ-004` — Un módulo no impone su vocabulario a otro. Si dos módulos nombran distinto la misma cosa, el [glosario](04-glosario.md) decide.
- `RN-ARQ-005` — Los módulos de la capa **Plataforma** (tenancy, IAM, configuración) son la única excepción: son transversales y obligatorios para cualquier instalación.

### Lo que este principio **no** significa

- No significa que no compartan datos. Comparten, a través de contratos explícitos.
- No significa duplicar información. Significa que cada módulo es dueño de sus propios datos
  y los demás los consultan, no los poseen.
- No dice nada todavía sobre **cómo** se comunican (API, eventos, monolito modular). Eso es
  decisión técnica y se toma después. El principio es funcional: *la definición de un
  módulo debe poder leerse sin leer la de otro*.

---

## Principio 2 — SaaS multi-tenant

### El enunciado

> **Barscode es una sola plataforma que atiende a muchos negocios independientes. Ningún
> dato cruza de un negocio a otro, jamás.**

### Jerarquía

```
Tenant (negocio / organización)
  └── Sede (establecimiento físico)
        └── Zona / Salón
              └── Mesa  ──► QR
```

- **Tenant** — la empresa cliente de Barscode. Unidad de aislamiento de datos, de facturación
  y de configuración global (carta base, marca, políticas).
- **Sede** — un local físico. Unidad de operación: inventario, caja, turnos y Gamecenter
  viven aquí. Un tenant puede tener una o muchas.
- **Zona / Mesa** — la granularidad del lado cliente. El QR identifica una mesa de una sede.

### Reglas

- `RN-ARQ-010` — Todo dato de negocio pertenece a exactamente un tenant. No existe dato compartido entre tenants.
- `RN-ARQ-011` — Todo dato operativo (stock, caja, turnos, pedidos) pertenece además a exactamente una sede.
- `RN-ARQ-012` — Un usuario del staff pertenece a un tenant y tiene roles asignados por sede. Un mesero de la sede A no opera la sede B salvo asignación explícita.
- `RN-ARQ-013` — Un cliente final **no** pertenece a un tenant: es un usuario de la plataforma que establece sesiones con sedes. Su cuenta (si la crea) es de Barscode, no del bar.
- `RN-ARQ-014` — El plan contratado determina qué módulos están habilitados por tenant y, si aplica, por sede.
- `RN-ARQ-015` — Toda pantalla y todo reporte debe dejar explícito a qué sede corresponde lo que muestra.

### Preguntas abiertas

- `PA-ARQ-001` — ¿Los módulos se contratan por tenant o por sede? (Una cadena podría querer inventario solo en una sede.)
- `PA-ARQ-002` — ¿La carta se define a nivel tenant con sobreescritura por sede, o cada sede define la suya? Afecta a `catalogo` y a `pos`.
- `PA-ARQ-003` — ¿Existe un rol de "franquiciado" con visibilidad parcial entre sedes del mismo tenant?

---

## Principio 3 — La localización es un artefacto sustituible

### El enunciado

> **Nada específico de un país vive dentro de la lógica de un módulo. Los módulos declaran
> interfaces; el país las implementa.**

Barscode arranca en Colombia. Pero el POS no sabe qué es el IVA, ni qué es la DIAN, ni qué
es el impuesto al consumo. El POS sabe que existe un **cálculo de impuestos** al que le
entrega los ítems de una cuenta y que le devuelve los tributos que aplican, cómo se
presentan y cuánto suman. Quién responde esa pregunta es el **artefacto de país**.

### El patrón

```
        ┌──────────────────────────────┐
        │  Módulo POS (agnóstico)      │
        │  "calcular tributos de esta  │
        │   cuenta"                    │
        └───────────────┬──────────────┘
                        │  interfaz declarada
          ┌─────────────┴─────────────┐
          ▼                           ▼
   Artefacto Colombia          Artefacto México (futuro)
   IVA 19% / 5% / 0%           IVA 16%
   INC 8% consumo              IEPS
   Factura electrónica DIAN    CFDI SAT
   Propina voluntaria          Propina
```

### Interfaces de localización identificadas

Estas son las preguntas que ningún módulo responde por sí mismo. Cada una es una interfaz
que el artefacto de país implementa:

| Interfaz | Pregunta que resuelve | Implementación Colombia |
|---|---|---|
| **Cálculo de tributos** | Dados unos ítems, ¿qué impuestos aplican, sobre qué base y cuánto suman? | IVA (19/5/0), INC 8% para consumo en sitio, régimen simple |
| **Documento fiscal** | ¿Qué documento se emite, con qué numeración, con qué contenido y ante quién se reporta? | Factura electrónica DIAN, POS electrónico, resolución y numeración autorizada |
| **Identificación fiscal** | ¿Cómo se identifica fiscalmente un cliente o proveedor? | NIT / CC, dígito de verificación |
| **Medios de pago** | ¿Qué formas de pago existen y cómo se liquidan? | Efectivo, tarjeta, PSE, Nequi, Daviplata, Bold, Wompi |
| **Moneda y formato** | ¿Qué moneda, cuántos decimales, cómo se redondea? | COP, sin decimales, redondeo a la unidad |
| **Propina** | ¿Es legal, es voluntaria, cómo se presenta y cómo se distribuye? | Voluntaria, debe ofrecerse explícitamente, no puede darse por incluida |
| **Reglas laborales** | ¿Qué restricciones tiene un turno: jornada máxima, recargos, descansos? | Jornada legal, recargo nocturno/dominical/festivo |
| **Calendario** | ¿Qué días son festivos? | Festivos colombianos |
| **Datos personales** | ¿Qué consentimiento se requiere y cuánto se retiene? | Ley 1581 de 2012, habeas data |
| **Edad / licor** | ¿Desde qué edad se puede consumir y qué debe verificarse? | 18 años |

### Reglas

- `RN-ARQ-020` — Ningún módulo contiene una tasa, un porcentaje, un formato de documento o una regla legal específica de un país.
- `RN-ARQ-021` — Todo importe lleva moneda explícita. No existe "el importe" sin moneda.
- `RN-ARQ-022` — Cada sede tiene exactamente un artefacto de país activo, determinado por su ubicación.
- `RN-ARQ-023` — Un cambio normativo (una tasa que sube, una resolución nueva) se resuelve modificando el artefacto de país, nunca un módulo funcional.
- `RN-ARQ-024` — Todo documento fiscal emitido conserva la versión del artefacto con la que se calculó. Un cambio de tasa no reescribe el pasado.
- `RN-ARQ-025` — Si un módulo necesita una regla de país que no está en la lista de interfaces, se agrega una interfaz nueva a esta tabla. No se resuelve dentro del módulo.

### Preguntas abiertas

- `PA-ARQ-010` — ¿Barscode emite la factura electrónica directamente ante la DIAN o se integra con un proveedor tecnológico autorizado? Decisión de alto impacto en costo, riesgo y tiempos.
- `PA-ARQ-011` — ¿Se incluye facturación electrónica o basta con comprobante de venta / tirilla? Muchos bares pequeños operan con documento equivalente.
- `PA-ARQ-012` — ¿Régimen tributario del tenant (común, simple, no responsable de IVA) es configuración de tenant o de sede?
- `PA-ARQ-013` — ¿El artefacto de país incluye también las reglas laborales del módulo `schedule`, o eso se aísla aparte?

---

## Cómo se aplican estos principios al revisar una ficha

Checklist obligatorio antes de marcar un módulo 🟢:

- [ ] ¿La ficha declara los cuatro puntos del contrato de autonomía?
- [ ] ¿El módulo hace algo útil si es lo único instalado?
- [ ] ¿Todo dato que consume de otro módulo tiene alternativa manual?
- [ ] ¿Todo dato del módulo está atado a un tenant y, si es operativo, a una sede?
- [ ] ¿Hay alguna tasa, impuesto, formato legal o festivo escrito dentro del módulo? (debe ser *no*)
- [ ] ¿Todo importe lleva moneda?
- [ ] ¿La ficha se puede leer y entender sin abrir la ficha de otro módulo?
