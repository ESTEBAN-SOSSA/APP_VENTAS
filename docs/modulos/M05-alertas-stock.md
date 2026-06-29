# M05 — Alertas de stock

**Fase:** 2 · **Backend:** `alertas/`, `realtime/` · **Frontend:** `modules/inventario/`

## Objetivo
Avisar en tiempo real cuando un insumo llega o baja de su tope mínimo, para reabastecer a tiempo.

## Requerimientos funcionales
- Marca visual cuando un insumo llega o baja del **tope mínimo**.
- Panel/bandeja de alertas con todos los insumos en estado crítico.
- **Notificación en tiempo real (WebSocket)** al Administrador/Cajero cuando un insumo cruza el umbral.
- **Notificación externa por correo y/o WhatsApp** (ver abajo) cuando un insumo entra en estado Bajo o Crítico.
- Niveles configurables:
  - **OK** — stock por encima del tope.
  - **Bajo** — cerca del tope.
  - **Crítico/Agotado** — en o bajo el tope / sin stock.
- Lista/Reporte de **"productos por reabastecer"** con cantidad sugerida a pedir.

## Canales de notificación (correo / WhatsApp)
- **Configurables por el Administrador:** activar/desactivar cada canal y definir el destinatario.
  - **Correo:** dirección destino (ej. `dev@edemco.co`). Envío vía SMTP del establecimiento o un proveedor (ej. Resend/SendGrid). Requiere conexión a internet para salir; si el sistema opera sin internet, las alertas externas se **encolan** y se envían cuando haya salida a la red.
  - **WhatsApp:** número destino (ej. `+57 300 123 4567`). Vía **WhatsApp Cloud API** (Meta) o un proveedor (ej. Twilio). Mensaje con plantilla: insumo, stock restante, tope y cantidad sugerida.
- **Anti-spam / agrupación:** las alertas se agrupan y se respeta un intervalo mínimo entre envíos por insumo (evitar notificar en cada venta). Botón **"Enviar alertas ahora"** para un disparo manual del consolidado.
- **Parámetros guardados en [M11](M11-configuracion.md):** `alerta_email_on`, `alerta_email_dest`, `alerta_wa_on`, `alerta_wa_dest`.

## Entidades
- Deriva de `Insumo.stock_actual` vs `Insumo.stock_minimo` (no requiere entidad propia; opcionalmente cachea estado).
- Opcional `NotificacionAlerta(id, insumo_id, canal[correo|whatsapp], destino, estado[encolada|enviada|fallida], fecha)` para trazabilidad de envíos.

## Reglas de negocio
- El estado se recalcula tras cada `MovimientoInventario` ([M03](M03-inventario.md)).
- Las alertas se emiten por WebSocket a los roles autorizados **y** por los canales externos activos (correo/WhatsApp).
- El **ingreso de inventario** y el **registro de entradas** ([M03](M03-inventario.md)/[M10](M10-compras-proveedores.md)) que devuelven un insumo a nivel OK **cierran** su alerta.

## Criterios de aceptación
- [ ] Se disparan alertas cuando un insumo llega a su tope.
- [ ] Existe una bandeja de alertas y un reporte de "por reabastecer" con cantidad sugerida.
- [ ] El Administrador puede configurar correo y/o WhatsApp como destino de las alertas y activarlos/desactivarlos.
- [ ] Al registrar una entrada que repone el stock, la alerta correspondiente se cierra automáticamente.
