# Documentación — Sistema de Gestión de Bar/Taberna (APP_VENTAS / TabernaPOS)

Aplicación de **ventas (POS) + control de inventario** para una tienda/bar, con foco en:

- **Control de inventario** con alertas por nivel mínimo (tope).
- **Control de valores a cobrar por mesa**: saber qué pidió cada mesa, acumular consumos y, al cerrar, calcular el total con el detalle de lo pedido.

Esta carpeta organiza el [levantamiento de requerimientos](../levantamiento_sistema_bar.md) en **módulos** y **flujos** independientes, pensados para generarse/desarrollarse por separado.

---

## Índice

### Módulos funcionales (`docs/modulos/`)

| ID | Módulo | Fase |
|----|--------|------|
| [M01](modulos/M01-autenticacion-usuarios.md) | Autenticación y usuarios | 1 |
| [M02](modulos/M02-catalogo-productos.md) | Catálogo de productos / Menú | 1 |
| [M03](modulos/M03-inventario.md) | Inventario | 1 |
| [M04](modulos/M04-recetas-fichas-tecnicas.md) | Recetas / Fichas técnicas | 1 |
| [M05](modulos/M05-alertas-stock.md) | Alertas de stock | 2 |
| [M06](modulos/M06-gestion-mesas.md) | Gestión de mesas | 2 |
| [M07](modulos/M07-comandas-pedidos.md) | Comandas / Pedidos | 2 |
| [M08](modulos/M08-ventas-cobro-caja.md) | Ventas, cobro y caja | 3 |
| [M09](modulos/M09-informes.md) | Informes | 4 |
| [M10](modulos/M10-compras-proveedores.md) | Compras / Proveedores | 4 |
| [M11](modulos/M11-configuracion.md) | Configuración del sistema | 4 |
| [M12](modulos/M12-respaldos-auditoria.md) | Respaldos y auditoría | 4 |

### Flujos principales (`docs/flujos/`)

| ID | Flujo |
|----|-------|
| [F1](flujos/F1-atencion-de-mesa.md) | Atención de una mesa (pedido → cobro) |
| [F2](flujos/F2-reabastecimiento-inventario.md) | Reabastecimiento de inventario |
| [F3](flujos/F3-apertura-cierre-caja.md) | Apertura y cierre de caja |
| [F4](flujos/F4-gestion-mesas.md) | Gestión de mesas (agregar/quitar) |

---

## Resumen del proyecto

| Campo | Valor |
|---|---|
| **Tipo** | Aplicación web auto-alojada (self-hosted), red local (LAN/Wi-Fi) |
| **Usuarios concurrentes** | 3–15 (meseros, cajero, administrador) |
| **Conectividad** | 100% red local, **sin internet** |
| **Idioma / moneda** | Español / Pesos colombianos (COP) |
| **Stack recomendado** | NestJS + Prisma + SQLite (backend) · React + Vite + TypeScript + Tailwind PWA (frontend) · Socket.IO (tiempo real) · Docker |

## Plan de desarrollo por fases

- **Fase 0** — Base del proyecto (monorepo, Prisma + SQLite, Docker, esquema de datos).
- **Fase 1** — Núcleo de catálogo e inventario (M01, M02, M03, M04).
- **Fase 2** — Operación de salón (M06, M07, M05 con WebSockets).
- **Fase 3** — Cobro y caja (M08).
- **Fase 4** — Soporte y análisis (M09, M10, M11, M12).
- **Fase 5** — Pulido (PWA, impresión térmica ESC/POS, pruebas E2E, guía de instalación).

Ver detalle en [el modelo de datos](modelo-de-datos.md).
