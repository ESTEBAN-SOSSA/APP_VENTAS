# Arquitectura — APP_VENTAS (TabernaPOS)

Aplicación web **auto-alojada (self-hosted)** que opera 100% en **red local (LAN/Wi-Fi)**, sin depender de internet. Un equipo servidor hospeda backend + frontend + base de datos; los dispositivos (celular, tablet, PC) acceden por navegador a la IP local.

---

## 1. Vista general (despliegue en red local)

```
┌─────────────────────────────────────────────────────────────┐
│                     Red local del bar (Wi-Fi)                │
│                                                              │
│   📱 Mesero        📱 Mesero        💻 Caja      🖥️ Admin     │
│   (navegador)      (navegador)     (navegador)  (navegador)  │
│        │               │               │            │        │
│        └───────────────┴───────┬───────┴────────────┘        │
│                                 │ HTTP / WebSocket            │
│                  ┌──────────────▼──────────────┐             │
│                  │      Equipo Servidor         │             │
│                  │  ┌────────────────────────┐  │             │
│                  │  │   Frontend (PWA build) │  │             │
│                  │  ├────────────────────────┤  │             │
│                  │  │   API NestJS (módulos) │  │             │
│                  │  ├────────────────────────┤  │             │
│                  │  │   SQLite (archivo .db) │  │             │
│                  │  └────────────────────────┘  │             │
│                  │  🖨️ Impresora térmica (opc.) │             │
│                  └──────────────────────────────┘             │
└─────────────────────────────────────────────────────────────┘
```

- El **equipo servidor** (PC o mini-PC siempre encendido) hospeda todo.
- Los dispositivos acceden por la IP local (ej. `http://192.168.1.50:3000`).
- **WebSockets** mantienen sincronizadas mesas, comandas y stock en tiempo real.

---

## 2. Stack tecnológico

| Capa | Recomendación | Alternativa |
|---|---|---|
| **Backend** | Node.js + **NestJS** (modular) | FastAPI (Python) |
| **Base de datos** | **SQLite** (archivo único, cero config) | PostgreSQL |
| **ORM** | Prisma | TypeORM / SQLAlchemy |
| **Frontend** | **React + Vite + TypeScript** (PWA) | Vue 3 |
| **UI** | Tailwind CSS, *mobile-first* | — |
| **Tiempo real** | WebSockets (Socket.IO) | Polling |
| **Despliegue** | **Docker + docker-compose** | `pm2` |
| **Impresión** | Impresora térmica ESC/POS (red/USB) | PDF desde navegador |

**Justificación:** NestJS organiza el código en módulos independientes que coinciden con la división M01–M12. SQLite elimina la administración de un motor de BD aparte (una sola sede). La PWA permite "instalar" la app en celulares/tablets sin tienda de aplicaciones.

---

## 3. Arquitectura del backend (NestJS modular)

Cada módulo funcional es un módulo NestJS aislado, con su `controller` (HTTP), `service` (lógica) y acceso a datos vía Prisma.

```
backend/  (NestJS)
├── prisma/
│   └── schema.prisma        # modelo de datos (ver docs/modelo-de-datos.md)
└── src/
    ├── auth/                # M01 — login, JWT, guards por rol
    ├── usuarios/            # M01
    ├── productos/           # M02
    ├── categorias/          # M02
    ├── inventario/          # M03 — insumos + movimientos
    ├── recetas/             # M04 — fichas técnicas / descuento de insumos
    ├── alertas/             # M05 — estado de stock + emisión por WS
    ├── mesas/               # M06
    ├── comandas/            # M07
    ├── ventas/              # M08
    ├── caja/                # M08 — apertura/cierre/arqueo
    ├── reportes/            # M09
    ├── proveedores/         # M10
    ├── configuracion/       # M11 — INC, propina, datos del local
    ├── auditoria/           # M12 — bitácora + backups
    └── realtime/            # WebSockets (gateway Socket.IO)
```

### Capas por módulo
```
HTTP Request → Controller → Service → Prisma → SQLite
                   │            │
                   │            └── emite eventos → RealtimeGateway (WS)
                   └── Guard (rol) valida permiso (M01)
```

---

## 4. Arquitectura del frontend (React PWA)

```
frontend/  (React + Vite + Tailwind, PWA)
└── src/
    ├── modules/
    │   ├── auth/            # login / PIN
    │   ├── menu/            # catálogo (M02)
    │   ├── inventario/      # stock, alertas, compras (M03/M05/M10)
    │   ├── mesas/           # tablero de mesas (M06)
    │   ├── comandas/        # toma de pedidos (M07)
    │   ├── caja/            # cobro y arqueo (M08)
    │   └── reportes/        # informes (M09)
    ├── components/          # UI reutilizable
    └── services/            # cliente API (HTTP) + cliente de sockets (WS)
```

- **Mobile-first**, instalable como PWA en cada dispositivo.
- `services/` centraliza llamadas REST y la conexión WebSocket.

---

## 5. Comunicación en tiempo real

```mermaid
sequenceDiagram
    participant Mesero
    participant API as API NestJS
    participant WS as RealtimeGateway
    participant Caja
    Mesero->>API: POST /comandas/:id/items (agrega producto)
    API->>API: actualiza comanda + subtotal
    API->>WS: emite "mesa:actualizada"
    WS-->>Caja: total de la mesa en vivo
    WS-->>Mesero: confirma ítem
    API->>WS: si insumo < tope → emite "stock:alerta"
    WS-->>Caja: notificación de stock bajo
```

Eventos clave: `mesa:actualizada`, `comanda:actualizada`, `stock:alerta`, `caja:actualizada`.

---

## 6. Flujo de datos de una venta (descuento de inventario)

```mermaid
flowchart LR
    A[Venta confirmada - M08] --> B{Tipo de producto}
    B -- Directo --> C[Descuenta 1 insumo - M03]
    B -- Preparado --> D[Lee receta - M04]
    D --> E[Descuenta cada insumo x cantidad]
    C --> F[Registra MovimientoInventario salida]
    E --> F
    F --> G{Insumo bajo tope?}
    G -- Si --> H[Alerta de stock - M05]
    A --> I[Registra Venta en Caja - M08]
```

---

## 7. Despliegue (Docker)

```
docker-compose.yml
└── servicio único (o backend + frontend):
    ├── Backend NestJS  → expone API + WS (puerto 3000)
    ├── Frontend build  → servido como estáticos (PWA)
    └── Volumen         → archivo SQLite persistente + carpeta de backups
```

Pasos: `docker-compose up -d` en el servidor → acceder desde la red en `http://<IP-servidor>:3000`. Se recomienda IP estática en el router y backup programado del archivo SQLite.

---

## 8. Estrategia de ramas (Git)

| Rama | Propósito |
|---|---|
| **main** | Código estable / producción (lo que corre en el establecimiento). |
| **staging** | Pre-producción: integración y pruebas antes de pasar a `main`. |
| **dev** | Desarrollo activo del día a día; de aquí salen las features. |

Flujo sugerido: `feature/* → dev → staging → main`.

---

## 9. Requerimientos no funcionales (resumen)

| Categoría | Requerimiento |
|---|---|
| Multidispositivo | Responsive *mobile-first* (celular, tablet, PC). |
| Tiempo real | Cambios visibles en segundos en todos los dispositivos. |
| Disponibilidad | 100% en red local, sin internet. |
| Rendimiento | API < 300 ms en operaciones comunes. |
| Seguridad | Hash de contraseñas, control de acceso por rol, sesiones protegidas. |
| Confiabilidad | Backup automatizable; sin pérdida de comandas abiertas ante recarga. |
| Instalabilidad | Despliegue con un solo comando (docker-compose). |
| Localización | Moneda COP, textos en español. |
