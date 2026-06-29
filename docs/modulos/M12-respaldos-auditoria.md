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

## Reglas de negocio
- El archivo SQLite se respalda de forma manual y programada (cron).
- Toda acción sensible escribe un registro de auditoría con el usuario responsable.

## Criterios de aceptación
- [ ] Existe respaldo (y restauración) de la base de datos.
- [ ] Las anulaciones, cambios de precio y ajustes quedan en la bitácora.
