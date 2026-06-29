# M02 — Catálogo de productos / Menú

**Fase:** 1 · **Backend:** `productos/`, `categorias/` · **Frontend:** `modules/menu/`

## Objetivo
Administrar los productos de venta (el menú) y sus categorías.

## Requerimientos funcionales
- CRUD de productos de venta: nombre, descripción, precio, categoría, foto opcional, disponible (sí/no).
- Categorías configurables (ej. Cervezas, Cócteles, Licores, Picadas, Gaseosas, Cigarrillos).
- **Tipo de producto:**
  - **Directo**: se descuenta del inventario 1 a 1 (ej. una cerveza en lata).
  - **Preparado**: descuenta insumos según su receta (ver [M04](M04-recetas-fichas-tecnicas.md)).
- Manejo de precios y, opcionalmente, **costo** (para márgenes).
- Activar/desactivar productos del menú sin borrarlos.

## Entidades
- `Producto(id, nombre, descripcion, categoria_id, precio, costo?, tipo[directo|preparado], insumo_id?, imagen?, disponible)`
- `Categoria(id, nombre, tipo[producto|insumo])`

## Reglas de negocio
- Un producto **directo** referencia un `insumo_id` y descuenta 1 unidad por venta.
- Un producto **preparado** se descuenta según sus filas de `Receta` ([M04](M04-recetas-fichas-tecnicas.md)).
- Desactivar un producto lo oculta del menú pero conserva su histórico de ventas.

## Criterios de aceptación
- [ ] Se pueden crear/editar/activar/desactivar productos y categorías.
- [ ] Cada producto define su tipo (directo o preparado).
