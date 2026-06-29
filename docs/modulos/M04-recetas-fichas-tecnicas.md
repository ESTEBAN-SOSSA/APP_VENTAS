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

## Mapa de recetas del demo (producto → insumos)

Correspondencia usada en el [mockup](../../index.html). Todo en **cantidades** (unidades/porciones enteras), sin conversión de ml/kg.

| Producto | Tipo | Insumos descontados por unidad vendida |
|---|---|---|
| Cerveza Nacional | directo | 1 cerveza nacional |
| Cerveza Importada | directo | 1 cerveza importada |
| Michelada | preparado | 1 cerveza nacional + 1 limón |
| Cuba Libre | preparado | 1 ron + 1 gaseosa + 1 limón |
| Mojito | preparado | 1 ron + 2 limón + 1 hielo |
| Margarita | preparado | 1 tequila + 1 triple sec + 1 limón |
| Gin Tonic | preparado | 1 ginebra + 1 agua tónica + 1 limón |
| Aguardiente (trago) | directo | 1 aguardiente |
| Ron (trago) | directo | 1 ron |
| Whisky (trago) | directo | 1 whisky |
| Picada personal | directo | 1 papa a la francesa |
| Picada para 2 | directo | 2 papa a la francesa |
| Alitas x6 | directo | 1 papa a la francesa |
| Gaseosa | directo | 1 gaseosa lata 350 ml |
| Agua | directo | 1 agua embotellada |
| Jugo natural | preparado | 1 fruta + 1 hielo |

> En el mockup, al **confirmar el pago** se descuentan estos insumos del inventario; si el stock no alcanza, la comanda muestra un aviso de **stock insuficiente**; y si un insumo cruza su tope, se dispara la **alerta** ([M05](M05-alertas-stock.md)) por correo/WhatsApp.

## Criterios de aceptación
- [ ] Un producto preparado descuenta todos sus insumos al venderse (ej. 1 Cuba Libre descuenta Ron y Gaseosa).
- [ ] Se bloquea/avisa la comanda si falta stock de algún insumo base o insumos compartidos en el pedido.
- [ ] El costo total de la receta se actualiza si cambian los costos unitarios de los insumos.
