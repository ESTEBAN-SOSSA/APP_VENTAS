# M07 — Comandas / Pedidos

**Fase:** 2 · **Backend:** `comandas/`, `realtime/` · **Frontend:** `modules/comandas/`

## Objetivo
Registrar lo que pide cada mesa y mantener el subtotal en vivo, base del **valor a cobrar por mesa**.

## Requerimientos funcionales
- Abrir comanda asociada a una mesa (o venta directa en barra/para llevar).
- Agregar productos del menú a la comanda con cantidad y **notas** (ej. "sin hielo").
- Editar/eliminar ítems mientras la comanda esté abierta (con control de permisos).
- Estado de ítems: Pendiente → En preparación → Servido (opcional, para barra/cocina).
- **Cálculo automático del subtotal de la mesa en vivo.**
- Envío de comanda a barra/cocina (e impresión opcional del ticket de preparación).
- Identificación de quién tomó/modificó la comanda.

## Entidades
- `Comanda(id, mesa_id?, tipo[mesa|barra|llevar], estado[abierta|cerrada|anulada], usuario_id, fecha_apertura, fecha_cierre?)`
- `ItemComanda(id, comanda_id, producto_id, cantidad, precio_unitario, notas, estado[pendiente|preparacion|servido])`

## Reglas de negocio
- `Comanda 1—N ItemComanda`; el `precio_unitario` se fija al agregar el ítem.
- El subtotal de la mesa = Σ (cantidad × precio_unitario) de sus ítems.
- Los cambios se reflejan en tiempo real en caja y demás dispositivos.

## Criterios de aceptación
- [ ] Se puede abrir una mesa, agregar lo pedido y ver el total en vivo.
- [ ] Cada modificación queda asociada al usuario que la hizo.
