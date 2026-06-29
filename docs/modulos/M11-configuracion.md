# M11 — Configuración del sistema

**Fase:** 4 · **Backend:** `configuracion/` · **Frontend:** `modules/` (ajustes)

## Objetivo
Centralizar los parámetros del establecimiento y del sistema.

## Requerimientos funcionales
- Datos del establecimiento (nombre, NIT, dirección, **logo para el recibo**).
- Parámetros tributarios (**INC %**, si aplica IVA).
- Configuración de **propina por defecto**.
- Configuración de **impresora**.
- Zonas/áreas de mesas.
- Categorías de productos e insumos.

## Entidades
- `Configuracion(clave, valor)` — Parámetros dinámicos en formato Clave-Valor.
  - Ejemplos de claves configurables:
    - `establecimiento_nombre`, `establecimiento_nit`, `establecimiento_direccion`, `establecimiento_telefono`
    - `impuesto_inc_defecto` (ej. `8`), `propina_sugerida_defecto` (ej. `10`)
    - `impresora_tipo` (`network` o `usb`), `impresora_ip` (ej. `192.168.1.100`), `impresora_puerto` (ej. `9100`), `impresora_ruta_usb` (ej. `/dev/usb/lp0`)
    - `impresora_codificacion` (ej. `CP860` o `latin1` para caracteres en español: á, é, í, ó, ú, ñ)
    - `recibo_cabecera` (texto personalizado de bienvenida), `recibo_pie` (texto de agradecimiento)

## Reglas de negocio
- Solo el **Administrador** tiene permisos para modificar la configuración general del sistema.
- Los valores de impuestos y propinas configurados aquí se usan como valores predeterminados al abrir mesas y facturar, pero pueden ser ajustados o desactivados en caliente durante la venta si el usuario cuenta con los permisos correspondientes.
- La configuración de impresora se valida mediante una prueba de conexión directa ("Test Print") enviando un comando ESC/POS básico de autodiagnóstico.

## Criterios de aceptación
- [ ] El INC, la propina y los datos de la factura son plenamente parametrizables.
- [ ] Se permite configurar los parámetros de conexión física de la impresora (IP de red o puerto USB).
- [ ] Existe un botón de prueba en la configuración para enviar un ticket de prueba a la impresora térmica configurada.
- [ ] Se configuran zonas de salón, categorías de menú y tipos de insumos de manera dinámica.
