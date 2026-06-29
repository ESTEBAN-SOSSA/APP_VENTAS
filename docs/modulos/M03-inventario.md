# M03 — Inventario

**Fase:** 1 · **Backend:** `inventario/` · **Frontend:** `modules/inventario/`

## Objetivo
Llevar el **control de inventario**: insumos, stock y movimientos, base para las alertas y para el descuento automático por venta.

## Requerimientos funcionales
- CRUD de insumos: nombre, unidad de medida, stock actual, **stock mínimo/tope**, costo unitario, proveedor opcional, categoría.
- Unidades de medida flexibles (unidad, ml, onza, botella, gramo, paquete).
- **Movimientos de inventario** con historial:
  - **Entrada** (compra/abastecimiento).
  - **Salida por venta** (automática al vender).
  - **Ajuste manual** (mermas, daños, conteo físico).
- Conversión de unidades cuando aplique (ej. 1 botella = 750 ml).
- Vista de stock actual con buscador y filtros por categoría/estado.

## Entidades
- `Insumo(id, nombre, categoria_id, unidad_medida, stock_actual, stock_minimo, costo_unitario, proveedor_id?, activo, unidad_compra?, factor_conversion?)`
  - `unidad_medida`: Unidad base de almacenamiento y consumo (ej. `ml`, `oz`, `gr`, `unidad`).
  - `unidad_compra`: Unidad en la que se adquiere del proveedor (ej. `Botella 750ml`, `Caja 24 un`, `Bolsa 1kg`).
  - `factor_conversion`: Factor multiplicador para llevar a la unidad base (ej. `750` para pasar de botella a ml).
- `MovimientoInventario(id, insumo_id, tipo[entrada|salida|ajuste], cantidad, costo_unitario?, motivo, usuario_id, fecha)`
  - La `cantidad` siempre se registra en la `unidad_medida` base.

## Reglas de negocio
- Todo cambio de `stock_actual` se respalda en un `MovimientoInventario` (trazabilidad).
- **Conversión en Entradas (Compras):** Al registrar una entrada en M10, el usuario puede digitar la cantidad en `unidad_compra`. El sistema calcula `cantidad_base = cantidad_compra × factor_conversion` y actualiza el `stock_actual` e historial en la unidad base.
- La salida por venta es automática a partir de [M04](M04-recetas-fichas-tecnicas.md) / producto directo.
- Cuando `stock_actual <= stock_minimo` se dispara una alerta ([M05](M05-alertas-stock.md)).
- Las anulaciones de venta **devuelven** insumos al inventario ([M08](M08-ventas-cobro-caja.md)) revirtiendo el descuento de stock.

## Criterios de aceptación
- [ ] La venta descuenta inventario (directo y por receta) correctamente.
- [ ] El registro de compras convierte unidades automáticamente a la unidad base de inventario.
- [ ] Existe historial de movimientos por insumo en su unidad base.
