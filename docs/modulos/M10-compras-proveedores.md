# M10 — Compras / Proveedores (entradas de inventario)

**Fase:** 4 · **Backend:** `proveedores/` · **Frontend:** `modules/inventario/`

## Objetivo
Registrar las entradas de mercancía que suman al inventario y administrar los proveedores.

## Requerimientos funcionales
- CRUD de proveedores (opcional pero recomendado).
- Registro de compras/entradas de mercancía que **suman al inventario** y actualizan el costo.
- Historial de compras por proveedor.

## Entidades
- `Proveedor(id, nombre, contacto, telefono)`
- Genera `MovimientoInventario(tipo=entrada, ...)` ([M03](M03-inventario.md)).

## Reglas de negocio
- Una compra/entrada incrementa `stock_actual` y puede actualizar `costo_unitario` del insumo.
- Cada insumo puede vincularse a un `proveedor_id`.

## Criterios de aceptación
- [ ] Se registran entradas que suman al inventario y actualizan costo.
- [ ] Existe historial de compras por proveedor.
