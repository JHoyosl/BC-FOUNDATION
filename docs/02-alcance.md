# Alcance y fases

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

Esas cosas se abren **después** de que la Fase 1 funcional esté 🟢 aprobada.

## 3. Fases del producto

Las fases ordenan **qué se define y se construye primero**. Un módulo en Fase 2 puede
definirse antes si conviene, pero no se construye antes.

### Fase 0 — Fundación de plataforma

Sin esto no existe nada más. Es la única capa que no es opcional.

- `tenancy` — tenants, sedes, planes, habilitación de módulos
- `iam` — usuarios del staff, roles, permisos
- `localizacion-co` — artefacto Colombia (moneda, tributos, identificación fiscal)

### Fase 1 — Operación mínima vendible

El objetivo de la Fase 1 es que **un bar pueda operar un día completo con Barscode**, y que
el cliente pueda pedir desde la mesa.

- `catalogo` — productos, carta, modificadores, precios
- `salon-mesas` — zonas, mesas, generación de QR
- `pos` — venta, cuenta, cobro
- `caja` — apertura, arqueo, cierre
- `pedidos-qr` — sesión de mesa por QR, carta y pedido desde el teléfono
- `kds` — comandas en cocina y barra
- `inventario` — stock, movimientos, costos
- `reportes` (básico) — ventas del día, cierre, productos más vendidos

### Fase 2 — Diferenciación y profundidad

- `identidad-cliente` — cuenta del cliente, perfil, anonimato persistente
- `gamecenter` — juegos, rankings, torneos por sede
- `social` — interacción anónima entre asistentes del mismo local
- `pagos` — pago en línea, división de cuenta, propina
- `compras` — proveedores, órdenes, recepción
- `schedule` — turnos y asistencia del personal

### Fase 3 — Escala

- `fidelizacion` — puntos, cupones, promociones
- `reservas` — reserva de mesa
- `reportes` (avanzado) — BI, márgenes, comparativos entre sedes
- `multi-sede` avanzado — consolidación, transferencias entre sedes
- Segundo artefacto de país

> `PA-ALC-001` — El Gamecenter es el diferenciador comercial del producto y hoy está en Fase 2.
> ¿Tiene sentido vender Barscode en Fase 1 sin él? Posible que una versión mínima del
> Gamecenter deba subir a Fase 1. Decisión pendiente (ver `PA-VIS-004`).

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
- `RES-004` — Existe un MVP previo con posibles usuarios. No condiciona el diseño, pero puede condicionar la transición.
- `RES-005` — Venta de licor: el producto opera en un contexto de consumo de alcohol; hay implicaciones de edad y de responsabilidad que deben quedar definidas en `pedidos-qr` y `identidad-cliente`.

## 6. Supuestos

- `SUP-001` — Los clientes del local tienen smartphone con cámara y datos o WiFi del sitio.
- `SUP-002` — El negocio tiene al menos un dispositivo para el POS y uno para cocina/barra.
- `SUP-003` — El negocio está dispuesto a llevar inventario con disciplina; sin eso, el módulo no entrega valor.

## 7. Preguntas abiertas

- `PA-ALC-001` — ¿El Gamecenter sube a Fase 1? (ver arriba)
- `PA-ALC-002` — ¿Se exige operación sin conexión (offline) en Fase 1, o se asume conectividad? Define buena parte de la complejidad de POS y KDS.
- `PA-ALC-003` — ¿La Fase 1 incluye facturación electrónica DIAN o se limita a comprobante de venta?
- `PA-ALC-004` — ¿App móvil nativa es Fase 1 o Fase 2? La web por QR cubre al cliente; la app nativa es lo que habilita bien el Gamecenter.
- `PA-ALC-005` — ¿Hay una fecha objetivo o un compromiso externo que condicione el orden de las fases?
