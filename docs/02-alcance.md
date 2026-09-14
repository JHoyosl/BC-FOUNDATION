# Alcance y cola de construcción

> **Estado:** 🟡 Borrador · **Dueño:** Jorge Hoyos · **Actualizado:** 2026-09-14

## 1. Alcance del producto (visión completa)

Barscode, en su forma completa, cubre cuatro dominios:

| Dominio | Qué abarca |
|---|---|
| **Operación** | Lo que pasa mientras el local está abierto: vender, pedir, preparar, cobrar, cerrar caja. |
| **Back office** | Lo que sostiene la operación: inventario, compras, costos, personal, reportes. |
| **Cliente final** | Lo que vive el cliente en la mesa: carta, pedido, pago, Gamecenter, capa social. |
| **Plataforma** | Lo transversal: tenants, sedes, usuarios, permisos, planes, localización. |

El detalle módulo por módulo está en [`06-catalogo-modulos.md`](06-catalogo-modulos.md).

## 2. Alcance de esta etapa del proyecto

**Esta etapa produce documentación, no software.**

Está dentro:

- Definición funcional completa de todos los módulos del catálogo.
- Reglas de negocio, entidades, estados, historias de usuario y criterios de aceptación.
- Contratos funcionales entre módulos.
- Interfaces de localización y su implementación para Colombia.
- Decisiones estructurales registradas como ADR.

Está fuera, deliberadamente:

- Elección de lenguajes, frameworks, base de datos, nube o proveedores.
- Modelo de datos físico, esquema de tablas, diseño de API.
- Diseño visual, wireframes y sistema de diseño.
- Estimaciones de esfuerzo y cronograma de desarrollo.
- Migración del MVP existente.

Esas cosas se abren **después**, módulo por módulo, una vez su definición funcional esté 🟢 aprobada.

## 3. Cola de construcción

> **No hay fases.** Se define y se construye **un módulo a la vez, hasta terminarlo**, y se
> pasa al siguiente. Ver [`ADR-0005`](decisiones/ADR-0005-construccion-modulo-a-modulo.md).

Un módulo está terminado cuando pasa su checklist de cierre, incluyendo **cero preguntas
abiertas** (`RN-ARQ-006`).

### En curso

| # | Módulo | Estado | Qué falta |
|---|---|---|---|
| 1 | [`inventario`](modulos/inventario.md) | 🟡 Borrador | Cerrar 5 preguntas abiertas y aprobar |

### Cola

El orden es **reordenable**. Al terminar cada módulo se revisa cuál sigue, con dos
criterios: *(a)* ¿vale por sí solo? *(b)* ¿su vocabulario desbloquea a los siguientes?

```
inventario  ←  en curso
    │
    ├─ catalogo          productos, recetas, cartas, precios
    ├─ salon-mesas       zonas, mesas, QR
    ├─ pos               venta, cuenta, cobro
    ├─ caja              apertura, arqueo, cierre
    ├─ pedidos-qr        sesión de mesa, pedido desde el teléfono
    ├─ kds               comandas en cocina y barra
    │
    ├─ tenancy · iam · localizacion-co        plataforma
    │
    ├─ gamecenter · social · identidad-cliente · pagos
    │
    └─ compras · schedule · reportes · fidelizacion · reservas · notificaciones
```

### Por qué la plataforma va en medio y no al principio

`tenancy`, `iam` y `localizacion-co` son transversales y obligatorios, pero definirlos
primero obligaría a **adivinar** qué permisos, qué configuración y qué reglas de país
necesitan los módulos funcionales. Así que cada módulo documenta lo que **necesita** de la
plataforma, y la plataforma se define después con esa lista en mano.

Mientras tanto, los módulos asumen lo mínimo: que existe un tenant, que existe una sede,
que el usuario está identificado y que hay un artefacto de país al que preguntarle.

## 4. Fuera de alcance del producto

Cosas que Barscode **no** hace, escritas para que nadie las asuma:

| No hace | Por qué |
|---|---|
| Domicilios y delivery propio | Otro negocio, otra logística. Posible integración con terceros, no módulo propio. |
| Marketplace de restaurantes para descubrimiento | Barscode empieza cuando el cliente ya está en el local. |
| Contabilidad completa | Barscode entrega información para el contador; no reemplaza el software contable. |
| Nómina y liquidación laboral | `schedule` gestiona turnos y asistencia, no paga sueldos. |
| Reservas de eventos y catering | Fuera de la operación diaria del local. |
| Hardware propio (terminales, impresoras) | Se integra con hardware estándar; no se fabrica ni se vende. |
| Red social entre locales | Lo social es **dentro** de una sede y durante una visita. |
| Chat entre clientes fuera del local | La sesión de mesa termina; la interacción también. |

## 5. Restricciones conocidas

- `RES-001` — Mercado inicial Colombia. Moneda COP. Todo lo tributario detrás del artefacto de país.
- `RES-002` — El cliente final debe poder usar el flujo de pedido **sin instalar la app**. El QR abre web.
- `RES-003` — El local puede tener conectividad inestable. El POS y el KDS deben tener un comportamiento definido sin red (ver `PA-ALC-002`).
- `RES-004` — ~~Existe un MVP previo con posibles usuarios.~~ **Levantada** (2026-09-14): el MVP solo tuvo pruebas internas. Sin usuarios de terceros, sin restricción de migración.
- `RES-005` — Venta de licor: el producto opera en un contexto de consumo de alcohol; hay implicaciones de edad y de responsabilidad que deben quedar definidas en `pedidos-qr` y `identidad-cliente`.
- `RES-006` — Segmento inicial **mixto bar-restaurante**: cocina y barra conviven. Ninguna definición puede asumir solo bar ni solo restaurante.

## 6. Supuestos

- `SUP-001` — Los clientes del local tienen smartphone con cámara y datos o WiFi del sitio.
- `SUP-002` — El negocio tiene al menos un dispositivo para el POS y uno para cocina/barra.
- `SUP-003` — El negocio está dispuesto a llevar inventario con disciplina; sin eso, el módulo no entrega valor.

## 7. Preguntas abiertas

- ~~`PA-ALC-001`~~ — **Disuelta** (2026-09-14): no hay fases. Ver [`ADR-0005`](decisiones/ADR-0005-construccion-modulo-a-modulo.md).
- `PA-ALC-002` — ¿Se exige operación sin conexión (offline)? **Resuelta para `inventario`** (2026-09-14): no tiene modo sin conexión (`RN-INV-067`). **Sigue abierta para `pos` y `kds`**, y de ella depende `PA-INV-011`: si el POS vende sin red, hay que decidir qué pasa con esas ventas cuando vuelve la conexión.
- `PA-ALC-003` — ¿Se incluye facturación electrónica DIAN o basta el comprobante de venta? Se resuelve al definir `localizacion-co`.
- `PA-ALC-004` — ¿App móvil nativa o basta la web por QR? La web cubre el pedido; la app nativa es lo que habilita bien el Gamecenter. Se resuelve al definir `pedidos-qr`.
- `PA-ALC-005` — ¿Hay una fecha objetivo o un compromiso externo que condicione el orden de la cola?
