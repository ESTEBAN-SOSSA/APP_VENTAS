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
- **Cálculo de Unidades Base:** Las recetas se definen usando la unidad base del insumo (ej. 60 ml de Ron, 20 ml de jarabe, 0.5 unidades de limón).
- **Validación de Stock Agrupada:** Antes de guardar o confirmar un pedido (comanda), el sistema valida que el stock disponible de cada insumo cubra la demanda total requerida por todos los productos de la comanda (tanto directos como preparados).
  - Si un insumo no cumple con la cantidad necesaria, la interfaz bloquea el envío de la comanda o advierte visualmente al mesero en tiempo real.
- **Márgenes de Costo:** El costo estimado del producto se autocalcula como `Costo = Σ (cantidad_receta × costo_unitario del insumo base)`. Esto se recalcula automáticamente cuando se registra una compra que altera el `costo_unitario` del insumo.

## Criterios de aceptación
- [ ] Un producto preparado descuenta todos sus insumos al venderse (ej. 1 Cuba Libre descuenta Ron y Gaseosa).
- [ ] Se bloquea/avisa la comanda si falta stock de algún insumo base o insumos compartidos en el pedido.
- [ ] El costo total de la receta se actualiza si cambian los costos unitarios de los insumos.
