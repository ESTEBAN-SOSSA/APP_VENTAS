# M12 — Respaldos y auditoría

**Fase:** 4 · **Backend:** `auditoria/` · **Frontend:** `modules/` (admin)

## Objetivo
Proteger los datos (todo es local) y mantener trazabilidad de las acciones sensibles.

## Requerimientos funcionales
- **Backup** de la base de datos (manual y programado) — clave porque todo es local.
- **Restauración** de respaldo.
- **Bitácora de auditoría:** quién hizo qué y cuándo en acciones sensibles (anulaciones, cambios de precio, ajustes de inventario).

## Entidades
- `Auditoria(id, usuario_id, accion, entidad, detalle, fecha)`
  - `accion`: Código corto de la acción (ej. `LOGIN`, `ANULACION_VENTA`, `CAMBIO_PRECIO`, `AJUSTE_INVENTARIO`, `ELIMINACION_ITEM_COMANDA`).
  - `entidad`: Nombre de la tabla afectada (ej. `Venta`, `Insumo`, `ItemComanda`).
  - `detalle`: Campo de texto de formato JSON que almacena el valor anterior, el nuevo valor y el motivo ingresado por el usuario (trazabilidad completa).

## Reglas de negocio
- **Automatización de Backups:** El backend programa una tarea interna (cron) diaria a una hora de bajo consumo (ej. 3:00 AM) para realizar una copia en caliente del archivo SQLite (`.db`) hacia un directorio de copias de seguridad de volumen persistente.
- **Acciones Auditadas Obligatorias:**
  - Anulación de ventas (debe registrar el id de la venta, el motivo ingresado y el total cancelado).
  - Ajuste de inventario manual (debe registrar la diferencia física y el motivo).
  - Eliminación de ítems en comandas ya enviadas a preparación (KDS).
  - Cambios de precio en el menú o cambios en la configuración tributaria.
- **Integridad Local:** Durante el arranque de la app, el servidor verifica la existencia y permisos de escritura del archivo `.db` y asegura la activación del modo WAL (`PRAGMA journal_mode=WAL;`) para prevenir la corrupción de datos ante cortes eléctricos inesperados.

## Criterios de aceptación
- [ ] El administrador puede generar y descargar copias de seguridad en caliente desde la interfaz web (en formato `.sql` o `.db`).
- [ ] Restauración del sistema a partir de un archivo de backup cargado desde la UI.
- [ ] Bitácora de auditoría inmutable consultable por el administrador, que muestra el detalle JSON formateado de cada acción sensible realizada.
- [ ] Las anulaciones, cambios de precio, eliminaciones en comanda y ajustes quedan en la bitácora vinculadas al usuario activo.
