# Visión de producto

> **Estado:** 🟡 Borrador · **Dueño:** Jorge Hoyos · **Actualizado:** 2026-09-14

## 1. Qué es Barscode

Barscode es una **plataforma SaaS para la gestión de bares y restaurantes**, compuesta por
una aplicación web para el negocio y una aplicación móvil para sus clientes.

Del lado del negocio cubre la operación completa del establecimiento: punto de venta,
inventario, compras, carta, mesas, turnos del personal y reportes.

Del lado del cliente, Barscode no se detiene en el pedido. El cliente escanea un **código QR
en su mesa** y con eso queda asociado al sitio y a esa mesa: desde ahí pide, paga y —lo que
diferencia al producto— **entra al Gamecenter del local**, un espacio de juego e interacción
con las otras personas que están en ese mismo sitio en ese mismo momento.

## 2. La tesis del producto

La mayoría del software para restaurantes optimiza al negocio y trata al cliente como un
insumo de la transacción. Barscode parte de una idea distinta:

> **El QR de la mesa no es un menú digital. Es la puerta de entrada a la experiencia del
> local.**

De esa idea se desprenden las tres apuestas del producto:

1. **Reducir la fricción del pedido.** Pedir sin esperar al mesero, sin descargar nada
   obligatoriamente, sin crear una cuenta para empezar.
2. **Hacer del local un espacio social conectado.** Quien está en la mesa 7 puede jugar,
   competir o interactuar de forma anónima con quien está en la barra. El local deja de ser
   un conjunto de mesas aisladas.
3. **Dar al negocio una operación que no se caiga a pedazos.** Inventario real, costos
   reales, turnos reales — no solo una caja registradora bonita.

La primera apuesta trae al cliente. La segunda lo retiene más tiempo en el local (y el
tiempo en el local es consumo). La tercera es lo que hace que el negocio pague la
suscripción todos los meses.

## 3. Problema que resuelve

### Para el negocio

| Dolor | Cómo se ve hoy | Qué hace Barscode |
|---|---|---|
| Operación fragmentada | POS de un proveedor, inventario en Excel, turnos en WhatsApp | Una sola plataforma, módulos que comparten datos sin depender entre sí |
| No se sabe cuánto cuesta un trago | El costo se estima "a ojo" | Fichas técnicas y costeo por receta desde el inventario |
| Fugas de inventario invisibles | El faltante aparece en el conteo mensual, sin explicación | Movimientos trazables: quién, cuándo, por qué |
| Meseros saturados en hora pico | Pedidos que tardan, mesas que se van sin consumir más | Pedido directo desde la mesa vía QR |
| Software caro y rígido | Licencias por terminal, contratos anuales | SaaS por suscripción, modular: se paga lo que se usa |

### Para el cliente del local

| Dolor | Qué hace Barscode |
|---|---|
| Esperar al mesero para pedir y para pagar | Pide y paga desde su teléfono |
| Dividir la cuenta es incómodo | División de cuenta en la app |
| Momentos muertos en la mesa | Gamecenter del local |
| Ir a un sitio social y no interactuar con nadie | Capa social anónima entre asistentes del mismo local |

## 4. Usuarios

### 4.1 Segmento inicial: mixto bar-restaurante

> Resuelve `PA-VIS-003` (2026-09-14).

Barscode apunta a establecimientos donde **cocina y barra conviven**: el gastrobar, el
restaurante que se vuelve bar de noche, el sitio de comida y tragos. No al bar de tragos
puro ni al restaurante familiar puro, sino al que hace las dos cosas.

Es el segmento más exigente, y eso condiciona el diseño:

| Exigencia del segmento | Qué obliga |
|---|---|
| Se vende licor **y** comida | El catálogo maneja con igual solvencia botella, trago, coctel y plato con receta |
| Dos naturalezas de inventario | Licor se mide en volumen y se sirve por trago desde botella abierta; comida se mide en masa y se porciona con merma |
| Dos estaciones reales | Cocina y barra preparan en paralelo, con tiempos distintos. El KDS no es opcional |
| Conversión entre dimensiones | Recetas de cocina mezclan ml y g. Hace falta densidad (`PA-INV-003`) |
| Perecederos de verdad | La comida vence; el licor casi no. Lote y vencimiento dejan de ser opcionales (`PA-INV-005`) |
| Producción interna | Almíbares, infusiones, salsas, despiece. Un insumo se transforma en otro (`PA-INV-006`) |
| Franja horaria amplia | Almuerzo y noche son operaciones distintas: carta, personal y ritmo cambian |
| Gamecenter con sentido | Funciona de noche, en la franja bar. Debe poder apagarse en la franja restaurante |

**Consecuencia para el KB:** ninguna ficha puede asumir "esto es un bar" ni "esto es un
restaurante". Donde las dos naturalezas difieran, la ficha debe cubrir ambas
explícitamente.

### 4.2 Usuarios primarios del negocio

- **Dueño / administrador.** Quiere saber si está ganando plata. Mira reportes, márgenes,
  costos. Decide la carta y los precios.
- **Administrador de sede.** Opera el día a día de un local: turnos, compras, cierres.
- **Cajero / mesero.** Toma pedidos, cobra, cierra mesas. Necesita velocidad, no funciones.
- **Cocina / barra.** Recibe comandas y las marca listas.
- **Bodeguero / jefe de compras.** Recibe mercancía, controla stock, registra mermas.

### 4.3 Usuarios del lado cliente

- **Cliente anónimo.** Escanea el QR, no crea cuenta. Puede ver carta, pedir y jugar.
  Es el caso más frecuente y el que define el diseño por defecto.
- **Cliente registrado.** Crea cuenta para conservar historial, puntos, identidad de juego y
  reputación en el Gamecenter.

Ver detalle en [`05-actores-y-roles.md`](05-actores-y-roles.md).

## 5. Propuesta de valor

**Para el negocio:** *toda la operación de tu bar en una plataforma, y un motivo para que tus
clientes se queden más tiempo.*

**Para el cliente:** *escanea el QR de tu mesa: pide, paga y juega con la gente que está ahí
contigo.*

## 6. Diferenciadores

1. **Gamecenter y capa social por local.** Es la funcionalidad que no tienen los POS
   tradicionales ni las apps de domicilios. Convierte el software de un costo operativo en
   una herramienta de permanencia y consumo.
2. **Módulos verdaderamente autónomos.** Un negocio puede contratar solo Inventario y usarlo
   sin POS. Esto abre un mercado que los sistemas monolíticos no alcanzan y reduce
   drásticamente la fricción de venta. Ver [`03-principios.md`](03-principios.md).
3. **Anonimato por defecto en el lado cliente.** No se le pide cuenta a nadie para empezar a
   pedir. Es una decisión de producto, no una limitación.
4. **Localización desacoplada.** Colombia es el primer mercado, pero lo tributario y lo
   legal vive detrás de una interfaz sustituible; expandir a otro país no reescribe el POS.

## 7. Modelo de negocio

> ⚠️ Sección con preguntas abiertas importantes. No está definida, está planteada.

**Forma:** SaaS multi-tenant por suscripción, cobrada al establecimiento.

**Unidad de cobro propuesta:** por **sede** (no por terminal), con planes que habilitan
conjuntos de módulos. La autonomía de módulos hace natural un esquema de **módulos
contratables por separado**, con un núcleo mínimo obligatorio.

**Posibles líneas adicionales de ingreso** (por evaluar, no comprometidas):

- Comisión o tarifa fija sobre pagos procesados dentro de la app.
- Módulos premium (BI avanzado, Gamecenter con torneos patrocinados).
- Espacios patrocinados dentro del Gamecenter (marcas de licores, promociones del local).

## 8. Métricas de éxito

**Del negocio (Barscode como empresa):**

- Sedes activas y retención mensual.
- Ingreso recurrente mensual por sede.
- Módulos contratados por sede (mide si la estrategia modular funciona).

**Del producto en un local:**

- % de pedidos hechos desde la mesa vs. tomados por mesero.
- Ticket promedio de mesas con sesión Barscode activa vs. mesas sin ella.
- Tiempo de permanencia en mesas con Gamecenter activo vs. sin él.
- Diferencia entre inventario teórico y conteo físico (mide si el módulo de inventario
  realmente sirve).

## 9. Principios de producto

1. **El cliente no descarga nada para empezar.** La app móvil aporta valor extra; el QR
   debe funcionar en el navegador.
2. **Anónimo primero, registrado si vale la pena.** Pedir cuenta es pedir permiso.
3. **Cada módulo se vale por sí mismo.** Si un módulo solo tiene sentido acompañado, está
   mal delimitado.
4. **El local manda.** La experiencia del cliente ocurre siempre en el contexto de una sede
   concreta y una sesión de mesa concreta.
5. **Nada de país quemado en el código.** Impuestos, moneda y facturación son configuración,
   no supuestos.

## 10. Preguntas abiertas

- `PA-VIS-001` — ¿El cobro es por sede, por módulo, por volumen de transacciones, o una mezcla? Define el módulo `tenancy` completo.
- ~~`PA-VIS-002`~~ — **Escalada a [`ADR-0004`](decisiones/ADR-0004-procesamiento-de-pagos.md)** (2026-09-14). Decisión pendiente, documentada como tal.
- ~~`PA-VIS-003`~~ — **Resuelta** (2026-09-14): segmento inicial **mixto bar-restaurante**. Ver sección 4.1.
- ~~`PA-VIS-004`~~ — **Disuelta** (2026-09-14): no se construye por fases sino por módulos independientes. Ver [`ADR-0005`](decisiones/ADR-0005-construccion-modulo-a-modulo.md).
- `PA-VIS-005` — ¿Hay competidores directos en Colombia contra los que haya que posicionarse explícitamente?
- ~~`PA-VIS-006`~~ — **Resuelta** (2026-09-14): el MVP solo tuvo pruebas del propio Jorge. Sin usuarios de terceros, sin restricción de migración.

## 11. Nota sobre el MVP existente

Existe un MVP previo, pero **solo tuvo pruebas internas de Jorge: ningún local ni usuario
de terceros depende de él** (`PA-VIS-006`, resuelta 2026-09-14).

Consecuencia: **no hay restricción de migración, de continuidad ni de datos heredados.**
El KB se redacta con libertad total. El MVP queda como fuente de aprendizajes, nunca como
especificación implícita. Si alguna definición de este KB contradice al MVP, manda el KB.
