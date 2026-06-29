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

Correspondencia usada en el [mockup](../../index.html). Las cantidades están en la unidad base del insumo.

| Producto | Tipo | Insumos descontados por unidad vendida |
|---|---|---|
| Cerveza Nacional | directo | 1 und cerveza nacional |
| Cerveza Importada | directo | 1 und cerveza importada |
| Michelada | preparado | 1 und cerveza nacional + 1 limón |
| Cuba Libre | preparado | 60 ml ron + 200 ml gaseosa + 1 limón |
| Mojito | preparado | 50 ml ron + 2 limón + 0.05 kg hielo |
| Margarita | preparado | 45 ml tequila + 20 ml triple sec + 1 limón |
| Gin Tonic | preparado | 50 ml ginebra + 1 agua tónica + 1 limón |
| Aguardiente (trago) | directo | 30 ml aguardiente |
| Ron (trago) | directo | 45 ml ron |
| Whisky (trago) | directo | 45 ml whisky |
| Picada personal | directo | 0.25 kg papa a la francesa |
| Picada para 2 | directo | 0.5 kg papa a la francesa |
| Alitas x6 | directo | 0.2 kg papa a la francesa |
| Gaseosa | directo | 1 lata gaseosa 350 ml |
| Agua | directo | 1 agua embotellada |
| Jugo natural | preparado | 1 fruta + 0.05 kg hielo |

> En el mockup, al **confirmar el pago** se descuentan estos insumos del inventario; si el stock no alcanza, la comanda muestra un aviso de **stock insuficiente**; y si un insumo cruza su tope, se dispara la **alerta** ([M05](M05-alertas-stock.md)) por correo/WhatsApp.

## Criterios de aceptación
- [ ] Un producto preparado descuenta todos sus insumos al venderse (ej. 1 Cuba Libre descuenta Ron y Gaseosa).
- [ ] Se bloquea/avisa la comanda si falta stock de algún insumo base o insumos compartidos en el pedido.
- [ ] El costo total de la receta se actualiza si cambian los costos unitarios de los insumos.
