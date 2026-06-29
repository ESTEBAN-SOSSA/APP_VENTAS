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
- `Configuracion(clave, valor)` — parámetros del sistema (INC, propina, etc.).

## Reglas de negocio
- Solo **Administrador** modifica la configuración.
- Los valores (INC, propina) son consumidos por [M08](M08-ventas-cobro-caja.md) en cada cobro.

## Criterios de aceptación
- [ ] El INC y la propina son configurables y se aplican en el cobro.
- [ ] Se configuran datos del establecimiento, zonas y categorías.
