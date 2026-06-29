# M05 — Alertas de stock

**Fase:** 2 · **Backend:** `alertas/`, `realtime/` · **Frontend:** `modules/inventario/`

## Objetivo
Avisar en tiempo real cuando un insumo llega o baja de su tope mínimo, para reabastecer a tiempo.

## Requerimientos funcionales
- Marca visual cuando un insumo llega o baja del **tope mínimo**.
- Panel/bandeja de alertas con todos los insumos en estado crítico.
- **Notificación en tiempo real (WebSocket)** al Administrador/Cajero cuando un insumo cruza el umbral.
- Niveles configurables:
  - **OK** — stock por encima del tope.
  - **Bajo** — cerca del tope.
  - **Crítico/Agotado** — en o bajo el tope / sin stock.
- Reporte de "productos por reabastecer".

## Entidades
- Deriva de `Insumo.stock_actual` vs `Insumo.stock_minimo` (no requiere entidad propia; opcionalmente cachea estado).

## Reglas de negocio
- El estado se recalcula tras cada `MovimientoInventario` ([M03](M03-inventario.md)).
- Las alertas se emiten por WebSocket a los roles autorizados.

## Criterios de aceptación
- [ ] Se disparan alertas cuando un insumo llega a su tope.
- [ ] Existe una bandeja de alertas y un reporte de "por reabastecer".
