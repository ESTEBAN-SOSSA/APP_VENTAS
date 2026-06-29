# Flujo 4 — Gestión de mesas (agregar / quitar)

**Módulos:** [M06](../modulos/M06-gestion-mesas.md) · [M01](../modulos/M01-autenticacion-usuarios.md)

## Pasos
1. Administrador/Mesero **agrega** una mesa nueva (nombre, zona).
2. Puede editar o **quitar** mesas que no estén ocupadas.
3. El **tablero de mesas** se actualiza para todos en tiempo real.

## Diagrama

```mermaid
flowchart TD
    A[Admin/Mesero agrega mesa: nombre, zona] --> B[Mesa creada → Libre]
    B --> C{Accion sobre mesa}
    C -- Editar --> D[Actualiza nombre/zona/capacidad]
    C -- Quitar --> E{Mesa ocupada?}
    E -- Si --> F[Bloquea eliminacion]
    E -- No --> G[Elimina mesa]
    D --> H[Tablero se actualiza en tiempo real]
    G --> H
    B --> H
```

## Resultado esperado
- Mesas agregadas/editadas/eliminadas según su estado.
- Tablero sincronizado en tiempo real en todos los dispositivos.
