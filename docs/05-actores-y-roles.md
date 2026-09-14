# Actores y roles

> **Estado:** 🟡 Borrador · **Dueño:** Jorge Hoyos · **Actualizado:** 2026-09-14

Los **actores** son quienes interactúan con Barscode. Los **roles** son los conjuntos de
permisos que se asignan al staff. Un actor puede tener varios roles; un rol se asigna
siempre **en una sede** (`RN-ARQ-012`).

---

## 1. Actores

### 1.1 Staff del tenant

| Actor | Qué busca | Módulos que toca | Dispositivo típico |
|---|---|---|---|
| **Dueño** | Saber si el negocio gana. Decidir carta, precios, expansión. | Reportes, Catálogo, Tenancy, todos en lectura | Web, móvil |
| **Administrador de sede** | Que el local funcione hoy. Cuadrar caja, cubrir turnos, que no falte producto. | Todos los de su sede | Web |
| **Cajero** | Cobrar rápido y que cuadre. | POS, Caja, Pagos | Terminal / tablet |
| **Mesero** | Tomar pedidos y atender mesas sin perder tiempo. | POS, Salón y mesas | Móvil / tablet |
| **Cocina / Barra** | Ver qué preparar y en qué orden. | KDS | Pantalla fija |
| **Bodeguero / Compras** | Que el stock del sistema sea el stock real. | Inventario, Compras | Web / móvil |
| **Jefe de personal** | Cubrir turnos y controlar asistencia. | Schedule | Web |

### 1.2 Clientes del local

| Actor | Qué busca | Cómo entra |
|---|---|---|
| **Cliente anónimo** | Pedir sin esperar, pagar sin esperar, entretenerse. | Escanea el QR de la mesa. No crea cuenta. |
| **Cliente registrado** | Lo anterior, más historial, puntos e identidad de juego persistente. | Cuenta de Barscode. |
| **Anfitrión de mesa** | Coordinar el pedido y la cuenta del grupo. | Es el primero en abrir la sesión de mesa, o quien el grupo designe. |

> El **cliente anónimo es el caso por defecto**, no la excepción. Todo flujo del lado cliente
> debe funcionar completo sin cuenta, salvo lo que explícitamente requiera identidad
> persistente (puntos, ranking histórico).

### 1.3 Actores de la plataforma

| Actor | Qué hace |
|---|---|
| **Operador Barscode** | Personal de Barscode: alta de tenants, soporte, planes. No accede a datos de negocio salvo autorización explícita (`PA-ACT-003`). |
| **Sistema externo** | Pasarela de pago, proveedor de facturación electrónica, sistema contable del tenant. |

---

## 2. Roles del staff

Roles propuestos. Cada uno es un conjunto de permisos, asignado por sede.

| Rol | Alcance | Puede | No puede |
|---|---|---|---|
| **Propietario** | Tenant completo | Todo, en todas las sedes. Gestionar plan y facturación. | — |
| **Administrador de sede** | Una sede | Operar y configurar su sede, ver todos sus reportes, anular ventas, autorizar descuentos, cerrar caja. | Cambiar el plan, ver otras sedes. |
| **Supervisor de turno** | Una sede, durante su turno | Autorizar anulaciones y descuentos, reabrir mesas, ver reportes del turno. | Modificar carta, precios ni configuración. |
| **Cajero** | Una sede | Abrir/cerrar su caja, cobrar, emitir documento fiscal, registrar medios de pago. | Anular después del cobro sin autorización, ver reportes de otros turnos. |
| **Mesero** | Una sede | Abrir mesas, tomar pedidos, enviar comandas, trasladar mesas, pedir la cuenta. | Cobrar, anular, aplicar descuentos. |
| **Estación (cocina/barra)** | Una sede, una estación | Ver comandas de su estación, marcarlas en preparación y listas. | Ver precios, ver la cuenta, tomar pedidos. |
| **Bodega** | Una sede | Registrar entradas, salidas, traslados, mermas y conteos. Ver costos. | Ver ventas ni cuentas. |
| **Compras** | Una o varias sedes | Gestionar proveedores, crear órdenes de compra, recibir mercancía. | Operar POS. |
| **Recursos / Personal** | Una o varias sedes | Crear turnos, asignar personal, controlar asistencia. | Ver información financiera. |
| **Solo lectura / Contador** | Tenant o sede | Ver reportes y exportar. | Modificar cualquier cosa. |

### Reglas de roles

- `RN-ROL-001` — Todo rol se asigna a un miembro del staff **en una sede concreta**. No existen roles sin sede, salvo *Propietario* y *Solo lectura* a nivel tenant.
- `RN-ROL-002` — Un mismo miembro del staff puede tener roles distintos en sedes distintas.
- `RN-ROL-003` — Toda acción sensible (anulación, descuento, ajuste de inventario, reapertura de mesa, apertura de cajón) queda registrada con usuario, fecha y motivo.
- `RN-ROL-004` — Una acción que un rol no puede hacer por sí mismo puede ejecutarse con **autorización de un rol superior presente** (código o confirmación). El registro guarda a los dos: quien ejecuta y quien autoriza.
- `RN-ROL-005` — El rol *Estación* nunca ve importes. La cocina no necesita saber precios.
- `RN-ROL-006` — Los permisos disponibles dependen de los módulos habilitados en el plan. Un tenant sin módulo Compras no tiene rol Compras.
- `RN-ROL-007` — El staff se autentica siempre. No existe operación anónima del lado negocio. Se admite autenticación rápida (PIN) para operación en terminal compartida, pero la identidad queda registrada.

---

## 3. Matriz actor × módulo

Lectura: **C** crea · **L** lee · **M** modifica · **—** sin acceso

| Módulo | Propietario | Admin sede | Supervisor | Cajero | Mesero | Estación | Bodega | Compras | Personal |
|---|---|---|---|---|---|---|---|---|---|
| Catálogo | CLM | CLM | L | L | L | — | L | L | — |
| Salón y mesas | CLM | CLM | LM | L | LM | — | — | — | — |
| POS | CLM | CLM | CLM | CLM | CL | — | — | — | — |
| Caja | L | CLM | LM | CLM | — | — | — | — | — |
| KDS | L | L | L | — | L | LM | — | — | — |
| Pedidos QR | L | LM | L | L | L | — | — | — | — |
| Inventario | L | CLM | L | — | — | L | CLM | L | — |
| Compras | L | CLM | — | — | — | — | L | CLM | — |
| Schedule | L | CLM | L | — | L | L | — | — | CLM |
| Reportes | L | L | L (turno) | L (turno) | — | — | L (stock) | L | L (personal) |
| Gamecenter | CLM | CLM | L | — | — | — | — | — | — |
| Tenancy / Plan | CLM | — | — | — | — | — | — | — | — |
| IAM | CLM | CLM (su sede) | — | — | — | — | — | — | — |

> Esta matriz es una propuesta inicial. Se valida módulo por módulo al redactar cada ficha.

---

## 4. Preguntas abiertas

- `PA-ACT-001` — ¿Existe la figura de "mesero" separada del "cajero" en el segmento objetivo, o en bares pequeños la misma persona hace todo? Si es lo segundo, el diseño de roles debe favorecer un rol combinado.
- `PA-ACT-002` — ¿El mesero puede cobrar? En Colombia es común que el mesero cobre en la mesa. Si es así, el rol Mesero necesita permiso de cobro condicionado.
- `PA-ACT-003` — ¿Bajo qué condiciones el operador Barscode puede ver datos de un tenant (soporte)? Necesita política explícita y registro.
- `PA-ACT-004` — ¿Los roles son fijos o el tenant puede crear roles propios con permisos a la medida? Cambia mucho el módulo `iam`.
- `PA-ACT-005` — ¿Cómo se autentica el staff en una terminal compartida: PIN, tarjeta, usuario y contraseña?
- `PA-ACT-006` — ¿El "anfitrión de mesa" es un concepto real del producto (con poderes sobre la cuenta del grupo) o solo una forma de hablar? Afecta `pedidos-qr` y `pagos`.
