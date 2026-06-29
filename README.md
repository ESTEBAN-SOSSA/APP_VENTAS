# APP_VENTAS — Sistema de Gestión de Bar/Taberna (TabernaPOS)

Aplicación de **ventas (POS) + control de inventario** para una tienda/bar. Permite llevar el **control de inventario** y los **valores a cobrar por mesa**: registrar lo que pide cada mesa, acumular consumos y, al cerrar, calcular el total con el detalle de lo pedido.

## ¿Qué hay aquí?

- [`levantamiento_sistema_bar.md`](levantamiento_sistema_bar.md) — levantamiento de requerimientos completo.
- [`docs/`](docs/README.md) — documentación organizada en **módulos** y **flujos**:
  - [`docs/modulos/`](docs/README.md#módulos-funcionales-docsmodulos) — los 12 módulos funcionales (M01–M12).
  - [`docs/flujos/`](docs/README.md#flujos-principales-docsflujos) — los 4 flujos principales.
  - [`docs/modelo-de-datos.md`](docs/modelo-de-datos.md) — entidades y relaciones.

## Características principales

- Control de inventario con **alertas por nivel mínimo (tope)**.
- Gestión de **mesas** (agregar/quitar dinámicamente) con estado y total en tiempo real.
- **Comandas por mesa** y cálculo del valor a cobrar (subtotal + INC + propina).
- **Recetas/fichas técnicas**: un cóctel descuenta varios insumos al venderse.
- **Caja**: apertura, cierre y arqueo.
- **Informes** de ventas e inventario.
- Roles y autenticación (Administrador, Cajero, Mesero).

## Stack recomendado

NestJS + Prisma + SQLite (backend) · React + Vite + TypeScript + Tailwind PWA (frontend) · Socket.IO (tiempo real) · Docker para despliegue local.

> Operación 100% en **red local**, sin necesidad de internet. Moneda: COP. Idioma: español.
