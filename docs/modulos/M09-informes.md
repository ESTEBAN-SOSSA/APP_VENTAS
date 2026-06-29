# M09 — Informes

**Fase:** 4 · **Backend:** `reportes/` · **Frontend:** `modules/reportes/`

## Objetivo
Dar visibilidad de ventas, inventario y márgenes para la toma de decisiones.

## Requerimientos funcionales
- Ventas por periodo (día, semana, mes, rango personalizado).
- Productos más vendidos / menos vendidos.
- Ventas por categoría.
- Ventas por mesero / por método de pago.
- Reporte de cierre de caja / turno.
- Consumo de inventario y valor del inventario actual.
- Margen estimado (precio vs. costo) — si se cargan costos.
- **Exportación a Excel/PDF.**

## Fuentes de datos
- `Venta`, `ItemComanda`, `Producto`, `MovimientoInventario`, `Caja`, `Insumo`.

## Reglas de negocio
- Solo accesible para roles **Cajero** (reportes del turno) y **Administrador** (todos).
- El margen requiere que los productos/insumos tengan costo cargado.

## Criterios de aceptación
- [ ] Se generan informes de ventas y de inventario.
- [ ] Se pueden exportar a Excel/PDF.
