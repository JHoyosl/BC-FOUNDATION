# Plataforma mínima

> **Estado:** 🟡 Borrador · **Dueño:** Jorge Hoyos · **Actualizado:** 2026-10-02

## 1. Para qué existe

Los módulos de plataforma (`tenancy`, `iam`, `localizacion-co`) se definen tarde en la cola a
propósito: hacerlo antes obligaría a adivinar qué necesitan los demás. Mientras tanto, cada
módulo se define **dando por sentadas** ciertas cosas de la plataforma.

Este documento escribe esas cosas en un solo sitio y junta lo que cada módulo le va pidiendo.
Con esa lista se definirán las fichas de plataforma cuando les llegue el turno.

**No reemplaza** esas fichas y no decide nada nuevo: todo lo de la sección 2 ya está decidido en
un principio o en un ADR. Creado por [`ADR-0008`](decisiones/ADR-0008-primero-el-kb.md).

## 2. Lo que todo módulo da por sentado

| Qué existe | En qué consiste | Decidido en |
|---|---|---|
| **Tenant** | Todo dato de negocio pertenece a exactamente un tenant. Nada cruza entre tenants. | `RN-ARQ-010` · `ADR-0002` |
| **Sede** | Todo dato operativo pertenece además a exactamente una sede. Toda pantalla y todo reporte dicen de qué sede son. | `RN-ARQ-011`, `015` |
| **Módulos habilitados** | El plan del tenant dice qué módulos tiene. Un módulo puede preguntar si otro está habilitado. | `RN-ARQ-014` |
| **Usuario y vinculación** | Una persona tiene un solo usuario y una vinculación con cada tenant donde trabaja. Opera en un tenant a la vez. | `RN-ARQ-016`…`019` · `ADR-0007` |
| **Identificación** | El staff se identifica siempre; no hay operación anónima del lado negocio. | `RN-ROL-007` |
| **Permisos y perfiles** | Cada módulo trae su árbol de permisos; el negocio arma perfiles y los asigna por sede. El servidor valida toda operación. | `RN-ROL-008`…`020` · `ADR-0006` |
| **Propietario** | Tiene todos los permisos de su tenant. | `RN-ROL-012` |
| **Autorización por otra persona** | Una acción que exige autorización la autoriza otra persona presente con ese permiso. | `RN-ROL-013` |
| **Auditoría** | Toda acción sensible queda registrada con usuario, tenant, sede, fecha, motivo y permiso. | `RN-ROL-016` |
| **Artefacto de país** | Cada sede tiene uno activo. Responde lo que ningún módulo sabe: tributos, documento fiscal, moneda, medios de pago, festivos. | `RN-ARQ-020`…`025` · `ADR-0003` |
| **Moneda** | Todo importe lleva moneda explícita. | `RN-ARQ-021` |
| **Configuración** | Cada módulo declara sus parámetros por tenant o por sede, con valor por defecto. | Plantilla §14 |

## 3. Lo que cada módulo le pide a la plataforma

Se llena al definir cada ficha, desde su sección 10.1.

### `inventario`

| A quién | Qué necesita | De dónde sale |
|---|---|---|
| `tenancy` | Las sedes activas del tenant. | Ficha §2, §10 |
| `tenancy` | Saber si `pos`, `compras`, `catalogo`, `kds`, `reportes` y `notificaciones` están habilitados, para elegir entre la vía automática y la manual. | Ficha §2 · `RN-INV-088` |
| `tenancy` | Guardar configuración por tenant, por sede, por bodega y por estación. | Ficha §14 |
| `iam` | Usuario identificado y sus permisos en la sede y, para envases, en la estación. | Ficha §10 · `ADR-0006` |
| `iam` | Autorización por otra persona para mermas sobre umbral, cierres de conteo y reaperturas. | `RN-INV-028`, `032`, `107` |
| `iam` | Un permiso con alcance de todo el tenant, para los insumos `global`. | `RN-INV-043` |
| `iam` | Auditoría de movimientos, anulaciones y reaperturas. | `RN-INV-003`, `103` |
| Artefacto de país | Moneda y formato de los costos. | Ficha §11 |
| Artefacto de país | Si un impuesto de compra es descontable o va al costo. | `RN-INV-018` |
| Artefacto de país | Cómo se identifica fiscalmente un proveedor. | Ficha §11 |
| Artefacto de país | Si una merma exige soporte documental. | Ficha §11 |
| `notificaciones` | Entregar las alertas de stock y de envases. Sin él, se consultan dentro del módulo. | Ficha §10 |

## 4. Preguntas abiertas

- `PA-PLT-001` — ¿Qué módulos forman la capa Plataforma? Hoy hay tres listas: `RN-ARQ-005` (tenancy, IAM, "configuración"), `ADR-0001` (`tenancy`, `iam`, `localizacion-*`) y el catálogo, que agrega `notificaciones` como obligatoria aunque `inventario` la trata como opcional.
- `PA-PLT-002` — ¿El QR, la sesión de mesa y la identidad del cliente son plataforma del lado cliente? `gamecenter`, `social` y `pagos` se declaran activables con un QR sin `salon-mesas` ni `pedidos-qr`, que son quienes generan el QR y abren la sesión. `identidad-cliente` tampoco funciona sola.
