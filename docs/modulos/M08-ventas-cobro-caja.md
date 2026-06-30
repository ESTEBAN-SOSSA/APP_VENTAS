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
- `Venta(id, comanda_id, subtotal, impuesto_inc, descuento, propina, total, metodo_pago, recibido?, cambio?, usuario_id, caja_id, fecha, tipo_division, fraccion_pago?)`
  - `tipo_division`: Completo (sin dividir), Partes (por partes iguales), Items (por selección de ítems).
  - `fraccion_pago`: ej. `0.5` si es la mitad en una división por valor.
- `Caja(id, usuario_apertura_id, base_inicial, fecha_apertura, fecha_cierre?, total_esperado?, total_contado?, diferencia?, estado[abierta|cerrada])`
- `MovimientoCaja(id, caja_id, tipo[ingreso|egreso], monto, motivo, fecha)`

## Reglas de negocio
- **Relaciones:** `Comanda 1—N Venta`; `Caja 1—N Venta`.
- **Descuento de Inventario Diferido:** El descuento físico de insumos del inventario ([M03](M03-inventario.md)/[M04](M04-recetas-fichas-tecnicas.md)) se ejecuta en tiempo real para cada ítem cobrado (si la cuenta es dividida por ítems, el stock se reduce a medida que se liquidan los productos correspondientes; si es dividida por partes, se descuenta de forma prorrateada o al completarse el cobro final).
- **Cálculo de Impuestos Individualizado:** El Impuesto Nacional al Consumo (INC) se calcula a nivel de producto (si `Producto.aplica_inc` es verdadero, se calcula el `%` sobre el precio base y se suma al subtotal).
- **Fórmulas de Totales:**
  - `subtotal_items = Σ (precio_unitario × cantidad)` de los ítems a cobrar en esta venta.
  - `impuesto_inc = Σ (precio_unitario × cantidad × inc_porcentaje)` de los ítems gravados.
  - `propina = subtotal_items × propina_sugerida_%` (sugerida en UI, editable por el usuario).
  - `total = subtotal_items − descuento + impuesto_inc + propina`.
- **Informes de Caja (Arqueo):**
  - **Reporte X (Cierre de Turno/Parcial):** Conciliación que realiza el cajero a mitad de la jornada sin cerrar el sistema fiscalmente. Muestra ventas del turno, egresos y arqueo temporal.
  - **Reporte Z (Cierre Diario):** Cierre definitivo del día fiscal en el bar. Bloquea la caja abierta actual y consolida todas las ventas del día, reseteando la acumulada diaria para la nueva jornada.
- **Anulaciones:** Requieren pin de Administrador. Devuelven el inventario y registran un egreso en la caja si el pago ya se había recibido, guardando rastro en M12.

## Implementado en el mockup
- **Apertura de caja** con base inicial; estado Abierta/Cerrada visible.
- Cada **cobro confirmado** registra la venta en la caja del turno con su **método de pago**.
- **Movimientos de caja**: ingresos y egresos con motivo y hora.
- **Esperado en efectivo** = `base + ventas en efectivo + ingresos − egresos` (tarjeta/transferencia no afectan el efectivo).
- **Cierre con arqueo**: se ingresa el efectivo contado, se calcula la **diferencia** (cuadre / sobrante / faltante) y se guarda el **Reporte Z** del turno.

## Criterios de aceptación
- [ ] Al cobrar, se muestra el **detalle de lo pedido** y el **total a cobrar** con desglose de INC y propina.
- [ ] Soporte completo de división de cuenta (ver [Flujo 5](../flujos/F5-division-de-cuentas.md)).
- [ ] Se registran ventas y se puede hacer apertura/cierre de caja con arqueo generando reportes tipo X y Z.
- [ ] La anulación de ventas requiere pin de Administrador y revierte el stock del inventario.
