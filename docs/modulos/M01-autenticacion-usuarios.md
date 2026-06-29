# M01 — Autenticación y usuarios

**Fase:** 1 · **Backend:** `auth/`, `usuarios/` · **Frontend:** `modules/auth/`

## Objetivo
Controlar el acceso al sistema, gestionar usuarios y roles, y garantizar la trazabilidad de cada acción relevante.

## Requerimientos funcionales
- Login con usuario y contraseña (hash seguro con **bcrypt**).
- Gestión de usuarios: crear, editar, desactivar — **solo Administrador**.
- Asignación de roles.
- Cierre de sesión y expiración de sesión por inactividad.
- **PIN rápido** opcional para meseros (login ágil en tablet compartida).

## Roles y permisos

| Rol | Permisos principales |
|---|---|
| **Administrador** | Acceso total: configuración, usuarios, productos, inventario, reportes, precios. |
| **Cajero** | Cobro y cierre de cuentas, apertura/cierre de caja, arqueo, ver reportes del turno. |
| **Mesero** | Crear/editar comandas, gestionar mesas, enviar pedidos. **No** cobra ni ve reportes financieros. |
| **(Opcional) Barra/Cocina** | Vista de comandas pendientes para preparar (KDS). |

## Entidades
- `Usuario(id, nombre, usuario, password_hash, rol, pin, activo)`

## Reglas de negocio
- Cada acción relevante queda registrada con el usuario que la realizó (ver [M12](M12-respaldos-auditoria.md)).
- Las contraseñas nunca se almacenan en texto plano.
- El control de acceso se aplica por rol en cada endpoint.

## Criterios de aceptación
- [ ] Cada usuario tiene su rol y sus permisos.
- [ ] Login con contraseña hasheada y, opcionalmente, con PIN.
- [ ] Solo el Administrador gestiona usuarios.
