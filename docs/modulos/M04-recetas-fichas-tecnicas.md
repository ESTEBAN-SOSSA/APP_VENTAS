# M04 — Recetas / Fichas técnicas (descuento de insumos)

**Fase:** 1 · **Backend:** `recetas/` · **Frontend:** `modules/inventario/`

> Pieza clave para un **bar**: vender un cóctel debe descontar varios insumos.

## Objetivo
Definir la composición de los productos preparados para descontar automáticamente sus insumos al vender.

## Requerimientos funcionales
- Definir la composición de un producto preparado: lista de insumos + cantidad por unidad vendida.
  - Ej. *Cuba Libre* = 60 ml ron + 200 ml gaseosa + 1 lima.
- Al registrar una venta, el sistema **descuenta automáticamente** del inventario los insumos según la receta y la cantidad vendida.
- **Validación de stock** antes de confirmar la venta (avisar si no alcanza el insumo).
- Cálculo del **costo** del producto a partir de sus insumos (útil para márgenes).

## Entidades
- `Receta(id, producto_id, insumo_id, cantidad)` — un producto preparado tiene varias filas de Receta.

## Reglas de negocio
- `Producto (preparado) 1—N Receta N—1 Insumo`.
- Antes de confirmar una venta se valida que haya stock suficiente de cada insumo de la receta × cantidad vendida.
- El costo del producto = Σ (cantidad de insumo × costo_unitario del insumo).

## Criterios de aceptación
- [ ] Un producto preparado descuenta todos sus insumos al venderse.
- [ ] Se bloquea/avisa la venta si falta stock de un insumo.
