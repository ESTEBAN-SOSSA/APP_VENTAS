# M06 — Gestión de mesas

**Fase:** 2 · **Backend:** `mesas/`, `realtime/` · **Frontend:** `modules/mesas/`

## Objetivo
Administrar las mesas del establecimiento y mostrar su estado y total acumulado en tiempo real. Permite **agregar y quitar mesas** dinámicamente.

## Requerimientos funcionales
- CRUD de mesas: **agregar y quitar mesas** dinámicamente.
- Atributos de mesa: número/nombre, zona (ej. Terraza, Interior, Barra), capacidad opcional.
- **Estados de mesa:** Libre, Ocupada, Por cobrar, Reservada (opcional).
- Vista tipo **"mapa/tablero"** con el estado y el **total acumulado** de cada mesa, actualizada en tiempo real.
- Acciones: abrir mesa, ver/editar comanda, **transferir consumo** a otra mesa, **unir/dividir** mesas (deseable).

## Entidades
- `Mesa(id, nombre, zona, capacidad?, estado[libre|ocupada|por_cobrar|reservada])`

## Reglas de negocio
- `Mesa 1—N Comanda`.
- Solo se pueden **quitar/editar** mesas que no estén ocupadas.
- Los cambios de estado y total se difunden por WebSocket a todos los dispositivos.

## Criterios de aceptación
- [ ] Se pueden crear, editar y **eliminar mesas**.
- [ ] El tablero muestra estado y total en vivo para todos los dispositivos.
