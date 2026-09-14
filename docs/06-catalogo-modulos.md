# Catálogo de módulos

> **Estado:** 🟡 Borrador · **Dueño:** Jorge Hoyos · **Actualizado:** 2026-09-14

Todos los módulos de Barscode. Cada uno cumple el [Principio 1](03-principios.md): funciona
solo, se vende solo, se define solo.

**Leyenda de estado:** ⚪ no iniciado · 🔴 incompleto · 🟡 borrador · 🟢 aprobado · 🔵 congelado

---

## Capa Plataforma (transversal, obligatoria)

Excepción al principio de autonomía (`RN-ARQ-005`): estos módulos son la base de cualquier
instalación.

| Id | Módulo | Qué hace | Fase | Estado | Ficha |
|---|---|---|---|---|---|
| `tenancy` | Tenants y sedes | Alta de negocios, sedes, plan contratado, habilitación de módulos, suscripción. | 0 | ⚪ | — |
| `iam` | Identidad y permisos | Usuarios del staff, autenticación, roles por sede, registro de auditoría. | 0 | ⚪ | — |
| `localizacion-co` | Artefacto Colombia | Implementa las interfaces de país: tributos, documento fiscal, moneda, medios de pago, festivos, reglas laborales. | 0 | ⚪ | — |
| `notificaciones` | Avisos | Envío de avisos a staff y clientes (pedido listo, stock bajo, turno asignado). | 2 | ⚪ | — |

---

## Dominio Operación

| Id | Módulo | Qué hace | Fase | Estado | Ficha |
|---|---|---|---|---|---|
| `catalogo` | Catálogo y carta | Productos vendibles, categorías, modificadores, precios, cartas por zona y horario. | 1 | ⚪ | — |
| `salon-mesas` | Salón y mesas | Zonas, mesas, capacidad, estado de la mesa, generación e impresión de QR. | 1 | ⚪ | — |
| `pos` | Punto de venta | Abrir mesa, tomar pedido, gestionar la cuenta, descuentos, anulaciones, cobro. | 1 | ⚪ | — |
| `caja` | Caja y cierres | Apertura de turno, base, movimientos de efectivo, arqueo, cierre de turno y de día. | 1 | ⚪ | — |
| `kds` | Pantalla de preparación | Comandas por estación, tiempos, estados de preparación. | 1 | ⚪ | — |

---

## Dominio Back office

| Id | Módulo | Qué hace | Fase | Estado | Ficha |
|---|---|---|---|---|---|
| `inventario` | Inventario | Insumos, unidades, bodegas, existencias, movimientos, recetas, costeo, mermas, conteos. | 1 | 🟡 | [ver](modulos/inventario.md) |
| `compras` | Compras y proveedores | Proveedores, órdenes de compra, recepción de mercancía, cuentas por pagar. | 2 | ⚪ | — |
| `schedule` | Turnos y personal | Plantilla de personal, planificación de turnos, asistencia, novedades. | 2 | ⚪ | — |
| `reportes` | Reportes y BI | Ventas, costos, márgenes, productos, comparativos entre sedes, exportación. | 1 (básico) / 3 (avanzado) | ⚪ | — |

---

## Dominio Cliente final

| Id | Módulo | Qué hace | Fase | Estado | Ficha |
|---|---|---|---|---|---|
| `pedidos-qr` | Pedido desde la mesa | Escaneo del QR, sesión de mesa, carta en el teléfono, pedido, seguimiento. | 1 | ⚪ | — |
| `identidad-cliente` | Identidad del cliente | Cliente anónimo, alias, cuenta opcional, perfil, historial, consentimientos. | 2 | ⚪ | — |
| `pagos` | Pagos del cliente | Pago en línea, división de cuenta, propina, comprobante. | 2 | ⚪ | — |
| `gamecenter` | Gamecenter | Catálogo de juegos por sede, partidas, rankings, torneos, premios. | 2 | ⚪ | — |
| `social` | Capa social | Interacción anónima entre asistentes de una misma sede: mensajes, retos, reacciones. | 2 | ⚪ | — |
| `fidelizacion` | Fidelización | Puntos, niveles, cupones, promociones, campañas. | 3 | ⚪ | — |
| `reservas` | Reservas | Reserva de mesa, confirmación, no-show. | 3 | ⚪ | — |

**Total: 21 módulos.**

---

## Mapa de relaciones

Las flechas indican **qué información fluye cuando ambos módulos están presentes**. Ninguna
flecha es obligatoria: si el módulo origen no existe, el destino tiene una vía manual
(`RN-ARQ-002`).

```
                       ┌─────────────────────────────────┐
                       │  PLATAFORMA                     │
                       │  tenancy · iam · localizacion   │
                       └────────────────┬────────────────┘
                                        │ (transversal a todo)
   ┌────────────────────────────────────┴────────────────────────────────────┐
   │                                                                          │
   ▼  CLIENTE                                    OPERACIÓN                     ▼
┌──────────────┐  pedido    ┌──────────┐  cuenta     ┌────────┐  efectivo  ┌──────┐
│ pedidos-qr   │──────────► │   pos    │───────────► │ pagos  │──────────► │ caja │
└──────┬───────┘            └────┬─────┘             └────────┘            └──────┘
       │ sesión                  │ comanda                  ▲
       ▼                         ▼                          │ precios
┌──────────────┐            ┌──────────┐             ┌──────┴────┐
│identidad-cli │            │   kds    │             │ catalogo  │
└──────┬───────┘            └──────────┘             └──────┬────┘
       │                         ▲                          │ receta
       ▼                         │ mesa                      ▼
┌──────────────┐          ┌──────┴──────┐             ┌────────────┐
│ gamecenter   │          │ salon-mesas │             │ inventario │◄──── compras
│ social       │          └─────────────┘             └─────┬──────┘
│ fidelizacion │                                            │ costo
└──────────────┘                                            ▼
                                                      ┌───────────┐
                              schedule ──────────────►│ reportes  │
                                                      └───────────┘
```

---

## Verificación de autonomía

Prueba que todo módulo debe pasar: **¿qué hace si es lo único instalado?**

| Módulo | Funciona solo como… |
|---|---|
| `inventario` | Sistema de control de stock y costos independiente. Entradas y salidas manuales. |
| `catalogo` | Gestor de carta digital y lista de precios, publicable por QR sin POS. |
| `pos` | Caja registradora con carta interna simple y cobro. |
| `caja` | Control de efectivo, turnos y arqueos, incluso con ventas registradas a mano. |
| `kds` | Tablero de comandas con ingreso manual desde cocina. |
| `salon-mesas` | Mapa del local y estado de ocupación de mesas. |
| `pedidos-qr` | Carta digital por QR con envío de pedido a una pantalla o correo del local. |
| `compras` | Gestión de proveedores y órdenes de compra. |
| `schedule` | Planificador de turnos y control de asistencia. |
| `reportes` | Tablero sobre los datos de los módulos que existan. **Es el único módulo que no tiene sentido solo** → ver `PA-CAT-002`. |
| `gamecenter` | Plataforma de juegos por sede, activable con un QR, sin necesidad de POS. |
| `social` | Muro/chat anónimo por sede, activable con un QR. |
| `fidelizacion` | Programa de puntos con acumulación manual o por importe registrado. |
| `reservas` | Sistema de reservas independiente. |
| `pagos` | Cobro por QR de un importe ingresado a mano. |
| `identidad-cliente` | Identidad de cliente reutilizable por los módulos de cliente que existan. |
| `notificaciones` | Servicio de avisos configurable. |

---

## Orden de definición propuesto

El orden importa: definir un módulo aclara el vocabulario de los siguientes.

1. `inventario` — 🟡 hecho. Es el módulo que más reglas tiene y el que mejor prueba el principio de autonomía.
2. `catalogo` — define producto, receta y precio; desbloquea POS y pedidos-qr.
3. `salon-mesas` — define zona, mesa y QR; desbloquea pedidos-qr.
4. `pos` — el corazón de la operación.
5. `caja` — cierra el ciclo del dinero.
6. `pedidos-qr` — la experiencia que diferencia al producto.
7. `kds`
8. `tenancy` + `iam` + `localizacion-co` — se definen con precisión una vez se sabe qué necesitan los módulos anteriores.
9. `gamecenter` + `social` + `identidad-cliente` + `pagos`
10. `compras`, `schedule`, `reportes`, `fidelizacion`, `reservas`, `notificaciones`

> Los módulos de plataforma van en el puesto 8 a propósito: definirlos antes obligaría a
> adivinar qué permisos y qué configuración necesitan los módulos funcionales.

---

## Preguntas abiertas

- `PA-CAT-001` — ¿`catalogo` e `inventario` son dos módulos o uno? Aquí van separados: catálogo es lo que se vende (precio), inventario lo que se consume (costo). La receta los une. ¿De acuerdo?
- `PA-CAT-002` — `reportes` no pasa la prueba de autonomía. ¿Se acepta como excepción (es un módulo agregador) o se disuelve, dejando que cada módulo traiga sus propios reportes?
- `PA-CAT-003` — ¿`social` es un módulo aparte o una funcionalidad dentro de `gamecenter`? La visión los menciona como etapas distintas.
- `PA-CAT-004` — ¿`pagos` cubre también el cobro con tarjeta en el POS (lado negocio) o solo el pago del cliente desde su teléfono?
- `PA-CAT-005` — ¿Falta un módulo de **integraciones** con sistemas externos del tenant (contabilidad, e-commerce, domicilios)?
- `PA-CAT-006` — ¿`kds` merece ser módulo o es una vista del `pos`? Como módulo autónomo su caso de uso solo es fuerte si un local quiere comandas sin cambiar su POS actual.
