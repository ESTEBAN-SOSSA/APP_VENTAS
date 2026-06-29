# M08 — Ventas, cobro y caja

**Fase:** 3 · **Backend:** `ventas/`, `caja/` · **Frontend:** `modules/caja/`

## Objetivo
Cerrar la cuenta de una mesa mostrando **el detalle de lo pedido** y **cuánto hay que cobrar**, registrar el pago y manejar la caja del turno.

## Requerimientos funcionales

### Cobro y cierre de cuenta
- Cierre de cuenta de una mesa: muestra el detalle de **lo que se pidió** y **cuánto hay que cobrar**.
- Cálculo de totales:
  - Subtotal.
  - **Impuesto al Consumo (INC 8%)** configurable (activable/desactivable y editable).
  - Descuentos (por valor o porcentaje, con permiso).
  - **Propina** sugerida/editable (configurable, ej. 10%).
- **Dividir la cuenta** (por partes iguales o por ítems) — deseable.
- **Métodos de pago:** Efectivo, Tarjeta, Transferencia (Nequi/Daviplata/Bancolombia QR), **Mixto**.
- Cálculo de **cambio** en pago en efectivo.
- Generación de comprobante/recibo (imprimible o PDF).
- **Anulación** de ventas con motivo y permiso de Administrador (devuelve insumos al inventario).

### Manejo de caja
- Apertura de caja con **base inicial**.
- Registro de ingresos/egresos del turno.
- Cierre de caja con **arqueo** (esperado vs. contado, diferencias).

## Entidades
- `Venta(id, comanda_id, subtotal, impuesto_inc, descuento, propina, total, metodo_pago, recibido?, cambio?, usuario_id, caja_id, fecha)`
- `Caja(id, usuario_apertura_id, base_inicial, fecha_apertura, fecha_cierre?, total_esperado?, total_contado?, diferencia?, estado[abierta|cerrada])`
- `MovimientoCaja(id, caja_id, tipo[ingreso|egreso], monto, motivo, fecha)`

## Reglas de negocio
- `Comanda 1—1 Venta`; `Caja 1—N Venta`.
- Al confirmar la venta: descuenta inventario ([M03](M03-inventario.md)/[M04](M04-recetas-fichas-tecnicas.md)), registra en la caja abierta, genera recibo y libera la mesa.
- `total = subtotal − descuento + impuesto_inc + propina`.
- La anulación requiere permiso de Administrador, queda en auditoría ([M12](M12-respaldos-auditoria.md)) y revierte los movimientos de inventario.

## Criterios de aceptación
- [ ] Al cobrar, se muestra el **detalle de lo pedido** y el **total a cobrar**.
- [ ] Se registran ventas y se puede hacer apertura/cierre de caja con arqueo.
