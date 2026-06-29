# Levantamiento de Requerimientos — Sistema de Gestión para Bar / Taberna-Tienda

> **Documento base de especificación.** Su propósito es servir como insumo para que **Claude CLI** genere los módulos, el modelo de datos y los flujos de desarrollo de la solución. Está escrito para ser leído tanto por personas como por una herramienta de generación de código.

---

## 1. Información general

| Campo | Valor |
|---|---|
| **Nombre del proyecto** | Sistema de Gestión de Bar/Taberna (provisional: **TabernaPOS**) |
| **Tipo de aplicación** | Aplicación web auto-alojada (self-hosted), accesible vía red local (LAN/Wi-Fi) |
| **Despliegue** | Instalación local en un equipo servidor; acceso desde cualquier dispositivo de la red (celular, tablet, PC) por navegador |
| **Usuarios concurrentes esperados** | 3–15 (meseros, cajero, administrador) |
| **Conectividad** | Funciona 100% en red local; **no requiere internet** para operar |
| **Idioma de la interfaz** | Español |
| **Moneda / región** | Pesos colombianos (COP), normativa tributaria de Colombia |

---

## 2. Objetivo

Construir una solución de **punto de venta (POS) + control de inventario** para un bar/taberna que permita:

- Llevar control de inventario con alertas por nivel mínimo (tope).
- Registrar y controlar las ventas.
- Generar informes de ventas y consumo.
- Administrar mesas: saber qué pidió cada mesa, agregar consumos a una mesa y, al cerrar, calcular el total a cobrar con el detalle de lo pedido.
- Agregar y quitar mesas dinámicamente.

El sistema debe instalarse de forma sencilla y operarse desde varios dispositivos en simultáneo dentro del mismo establecimiento.

---

## 3. Alcance

### 3.1 Dentro del alcance (MVP)
- Gestión de productos / menú.
- Gestión de inventario y movimientos de stock.
- Alertas de stock bajo / agotamiento.
- Gestión de mesas (crear, editar, eliminar, estado).
- Toma de pedidos por mesa (comandas).
- Cobro y cierre de cuenta por mesa.
- Manejo de caja (apertura, cierre, arqueo).
- Informes básicos de ventas e inventario.
- Roles y autenticación de usuarios.

### 3.2 Fuera del alcance inicial (futuro / opcional)
- Facturación electrónica DIAN.
- Integración con pasarelas de pago en línea.
- App móvil nativa (se usa PWA en su lugar).
- Programa de fidelización de clientes.
- Multi-sucursal (el MVP es de una sola sede).

---

## 4. Glosario

| Término | Definición |
|---|---|
| **Comanda / Pedido** | Conjunto de productos solicitados en una mesa. |
| **Ítem de comanda** | Una línea del pedido (producto + cantidad + notas). |
| **Insumo** | Producto de inventario que se consume (ej. botella de ron, gaseosa, lima). |
| **Producto de venta** | Lo que el cliente pide del menú (puede ser un insumo directo o una preparación). |
| **Ficha técnica / Receta** | Composición de un producto de venta a partir de uno o más insumos. |
| **Tope / Stock mínimo** | Nivel de inventario que dispara una alerta de reabastecimiento. |
| **Arqueo de caja** | Conteo y cuadre del dinero al cerrar un turno. |
| **INC** | Impuesto Nacional al Consumo (8% para bares/restaurantes en Colombia). |

---

## 5. Stack tecnológico recomendado

Pensado para instalación local sencilla, acceso multidispositivo por navegador y desarrollo asistido por Claude CLI.

| Capa | Recomendación | Alternativa |
|---|---|---|
| **Backend** | Node.js + **NestJS** (estructura modular, ideal para dividir por módulos) | FastAPI (Python) |
| **Base de datos** | **SQLite** (archivo único, cero configuración, perfecto para una sede) | PostgreSQL (si se prevé crecimiento) |
| **ORM** | Prisma (con NestJS) | TypeORM / SQLAlchemy |
| **Frontend** | **React + Vite + TypeScript**, configurado como **PWA** | Vue 3 |
| **UI** | Tailwind CSS + componentes accesibles, diseño *mobile-first* | — |
| **Tiempo real** | WebSockets (Socket.IO) para actualizar mesas/comandas entre dispositivos | Polling |
| **Empaquetado / despliegue** | **Docker + docker-compose** (instalación con un comando) | Ejecutable con `pm2` |
| **Impresión** | Soporte para impresora térmica (ESC/POS) por red o USB | Imprimir desde el navegador (PDF) |

> **Justificación del stack:** NestJS organiza el código en módulos independientes, lo que coincide directamente con la división modular de este documento y facilita que Claude CLI genere cada módulo de forma aislada. SQLite elimina la complejidad de administrar un motor de BD aparte, ideal para un bar de una sola sede. La PWA permite "instalar" la app en celulares/tablets sin tienda de aplicaciones.

---

## 6. Arquitectura general

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

- El **equipo servidor** (un PC o mini-PC siempre encendido) hospeda backend + frontend + base de datos.
- Los dispositivos acceden por la IP local del servidor (ej. `http://192.168.1.50:3000`).
- WebSockets mantienen sincronizadas las mesas/comandas en tiempo real entre todos los dispositivos.

---

## 7. Roles de usuario

| Rol | Permisos principales |
|---|---|
| **Administrador** | Acceso total: configuración, usuarios, productos, inventario, reportes, precios. |
| **Cajero** | Cobro y cierre de cuentas, apertura/cierre de caja, arqueo, ver reportes del turno. |
| **Mesero** | Crear/editar comandas, gestionar mesas, enviar pedidos; **no** cobra ni ve reportes financieros. |
| **(Opcional) Barra/Cocina** | Vista de comandas pendientes para preparar (KDS). |

> Cada acción relevante queda registrada con el usuario que la realizó (trazabilidad).

---

## 8. Módulos funcionales

Cada módulo está pensado como una unidad independiente para que Claude CLI lo genere por separado.

### M1 — Autenticación y usuarios
- Login con usuario y contraseña (hash seguro, ej. bcrypt).
- Gestión de usuarios (crear, editar, desactivar) — solo Administrador.
- Asignación de roles.
- Cierre de sesión y expiración de sesión por inactividad.
- PIN rápido opcional para meseros (login ágil en tablet compartida).

### M2 — Catálogo de productos / Menú
- CRUD de productos de venta (nombre, descripción, precio, categoría, foto opcional, disponible sí/no).
- Categorías configurables (ej. Cervezas, Cócteles, Licores, Picadas, Gaseosas, Cigarrillos).
- Tipo de producto:
  - **Directo**: se descuenta del inventario 1 a 1 (ej. una cerveza en lata).
  - **Preparado**: descuenta insumos según su receta (ver M4).
- Manejo de precios y, opcionalmente, costo (para márgenes).
- Activar/desactivar productos del menú sin borrarlos.

### M3 — Inventario
- CRUD de insumos (nombre, unidad de medida, stock actual, stock mínimo/tope, costo unitario, proveedor opcional, categoría).
- Unidades de medida flexibles (unidad, ml, onza, botella, gramo, paquete).
- **Movimientos de inventario** con historial:
  - Entrada (compra/abastecimiento).
  - Salida por venta (automática al vender).
  - Ajuste manual (mermas, daños, conteo físico).
- Conversión de unidades cuando aplique (ej. una botella = 750 ml).
- Vista de stock actual con buscador y filtros por categoría/estado.

### M4 — Recetas / Fichas técnicas (descuento de insumos)
> Pieza clave para un **bar**: vender un cóctel debe descontar varios insumos.

- Definir la composición de un producto preparado: lista de insumos + cantidad por unidad vendida.
  - Ej. *Cuba Libre* = 60 ml ron + 200 ml gaseosa + 1 lima.
- Al registrar una venta, el sistema descuenta automáticamente del inventario los insumos según la receta y la cantidad vendida.
- Validación de stock antes de confirmar venta (avisar si no alcanza el insumo).
- Cálculo de costo del producto a partir de sus insumos (útil para márgenes).

### M5 — Alertas de stock
- Marca visual cuando un insumo llega o baja del **tope mínimo**.
- Panel/bandeja de alertas con todos los insumos en estado crítico.
- Notificación en tiempo real (WebSocket) al Administrador/Cajero cuando un insumo cruza el umbral.
- Niveles configurables: **OK**, **Bajo** (cerca del tope), **Crítico/Agotado**.
- Reporte de "productos por reabastecer".

### M6 — Gestión de mesas
- CRUD de mesas: **agregar y quitar mesas** dinámicamente.
- Atributos de mesa: número/nombre, zona (ej. Terraza, Interior, Barra), capacidad opcional.
- Estados de mesa: **Libre**, **Ocupada**, **Por cobrar**, **Reservada** (opcional).
- Vista tipo "mapa/tablero" con el estado y total acumulado de cada mesa, actualizada en tiempo real.
- Acciones: abrir mesa, ver/editar comanda, transferir consumo a otra mesa, unir/dividir mesas (deseable).

### M7 — Comandas / Pedidos
- Abrir comanda asociada a una mesa (o venta directa en barra/para llevar).
- Agregar productos del menú a la comanda con cantidad y **notas** (ej. "sin hielo").
- Editar/eliminar ítems mientras la comanda esté abierta (con control de permisos).
- Estado de ítems: Pendiente → En preparación → Servido (opcional, para barra/cocina).
- Cálculo automático del subtotal de la mesa en vivo.
- Envío de comanda a barra/cocina (e impresión opcional del ticket de preparación).
- Identificación de quién tomó/modificó la comanda.

### M8 — Ventas, cobro y caja
- Cierre de cuenta de una mesa: muestra el detalle de **lo que se pidió** y **cuánto hay que cobrar**.
- Cálculo de totales:
  - Subtotal.
  - **Impuesto al Consumo (INC 8%)** configurable (activable/desactivable y editable).
  - Descuentos (por valor o porcentaje, con permiso).
  - **Propina** sugerida/editable (configurable, ej. 10%).
- **Dividir la cuenta** (por partes iguales o por ítems) — deseable.
- Métodos de pago: **Efectivo, Tarjeta, Transferencia (Nequi/Daviplata/Bancolombia QR), Mixto**.
- Cálculo de cambio en pago en efectivo.
- Generación de comprobante/recibo (imprimible o PDF).
- **Manejo de caja**:
  - Apertura de caja con base inicial.
  - Registro de ingresos/egresos del turno.
  - Cierre de caja con **arqueo** (esperado vs. contado, diferencias).
- Anulación de ventas con motivo y permiso de Administrador (devuelve insumos al inventario).

### M9 — Informes e informes
- Ventas por periodo (día, semana, mes, rango personalizado).
- Productos más vendidos / menos vendidos.
- Ventas por categoría.
- Ventas por mesero / por método de pago.
- Reporte de cierre de caja / turno.
- Consumo de inventario y valor del inventario actual.
- Margen estimado (precio vs. costo) — si se cargan costos.
- Exportación a Excel/PDF.

### M10 — Compras / Proveedores (entradas de inventario)
- CRUD de proveedores (opcional pero recomendado).
- Registro de compras/entradas de mercancía que **suman al inventario** y actualizan costo.
- Historial de compras por proveedor.

### M11 — Configuración del sistema
- Datos del establecimiento (nombre, NIT, dirección, logo para el recibo).
- Parámetros tributarios (INC %, si aplica IVA).
- Configuración de propina por defecto.
- Configuración de impresora.
- Zonas/áreas de mesas.
- Categorías de productos e insumos.

### M12 — Respaldos y auditoría
- **Backup** de la base de datos (manual y programado) — clave porque todo es local.
- Restauración de respaldo.
- Bitácora de auditoría (quién hizo qué y cuándo en acciones sensibles: anulaciones, cambios de precio, ajustes de inventario).

---

## 9. Modelo de datos (entidades principales)

```
Usuario(id, nombre, usuario, password_hash, rol, pin, activo)

Categoria(id, nombre, tipo[producto|insumo])

Insumo(id, nombre, categoria_id, unidad_medida, stock_actual,
       stock_minimo, costo_unitario, proveedor_id?, activo)

Producto(id, nombre, descripcion, categoria_id, precio, costo?,
         tipo[directo|preparado], insumo_id?, imagen?, disponible)

Receta(id, producto_id, insumo_id, cantidad)        // ficha técnica
       // un producto preparado tiene varias filas de Receta

Proveedor(id, nombre, contacto, telefono)

MovimientoInventario(id, insumo_id, tipo[entrada|salida|ajuste],
                     cantidad, costo_unitario?, motivo, usuario_id, fecha)

Mesa(id, nombre, zona, capacidad?, estado[libre|ocupada|por_cobrar|reservada])

Comanda(id, mesa_id?, tipo[mesa|barra|llevar], estado[abierta|cerrada|anulada],
        usuario_id, fecha_apertura, fecha_cierre?)

ItemComanda(id, comanda_id, producto_id, cantidad, precio_unitario,
            notas, estado[pendiente|preparacion|servido])

Venta(id, comanda_id, subtotal, impuesto_inc, descuento, propina, total,
      metodo_pago, recibido?, cambio?, usuario_id, caja_id, fecha)

Caja(id, usuario_apertura_id, base_inicial, fecha_apertura,
     fecha_cierre?, total_esperado?, total_contado?, diferencia?, estado[abierta|cerrada])

MovimientoCaja(id, caja_id, tipo[ingreso|egreso], monto, motivo, fecha)

Auditoria(id, usuario_id, accion, entidad, detalle, fecha)

Configuracion(clave, valor)   // parámetros del sistema (INC, propina, etc.)
```

**Relaciones clave**
- `Producto (preparado) 1—N Receta N—1 Insumo`
- `Mesa 1—N Comanda 1—N ItemComanda`
- `Comanda 1—1 Venta`
- `Caja 1—N Venta`
- `Insumo 1—N MovimientoInventario`

---

## 10. Flujos principales

### Flujo 1 — Atención de una mesa
1. Mesero abre **Mesa 5** (estado → *Ocupada*).
2. Agrega ítems a la comanda (2 cervezas, 1 cuba libre, 1 picada).
3. La comanda y el total se reflejan en tiempo real en caja y otros dispositivos.
4. (Opcional) La barra ve la comanda y prepara.
5. Cliente pide la cuenta → mesa pasa a *Por cobrar*.
6. Cajero abre la cuenta: ve **el detalle de lo pedido** y el **total** (subtotal + INC + propina).
7. Cajero registra el pago (método, divide cuenta si aplica, calcula cambio).
8. Sistema:
   - Descuenta insumos del inventario (cervezas directas; ron/gaseosa/lima por receta del cuba libre).
   - Registra la venta en la caja del turno.
   - Genera/imprime el recibo.
   - Libera la **Mesa 5** (estado → *Libre*).
9. Si algún insumo cruzó su tope, se dispara la **alerta de stock**.

### Flujo 2 — Reabastecimiento de inventario
1. Llega mercancía → Administrador registra una **entrada** (compra) o ajuste.
2. Se actualiza `stock_actual` y opcionalmente el costo.
3. Las alertas activas se recalculan.

### Flujo 3 — Apertura y cierre de caja
1. Cajero abre caja con base inicial.
2. Durante el turno se registran ventas y movimientos de caja.
3. Al cerrar: el sistema muestra el total esperado; el cajero ingresa el contado; se calcula la diferencia y se guarda el arqueo.

### Flujo 4 — Gestión de mesas
1. Administrador/Mesero agrega una mesa nueva (nombre, zona).
2. Puede editar o **quitar** mesas que no estén ocupadas.
3. El tablero de mesas se actualiza para todos en tiempo real.

---

## 11. Requerimientos no funcionales

| Categoría | Requerimiento |
|---|---|
| **Multidispositivo** | Interfaz responsive *mobile-first*; usable en celular, tablet y PC. |
| **Tiempo real** | Cambios en mesas/comandas/stock visibles en segundos en todos los dispositivos. |
| **Disponibilidad** | Operación 100% en red local sin depender de internet. |
| **Rendimiento** | Respuesta de la API < 300 ms en operaciones comunes. |
| **Seguridad** | Contraseñas con hash; control de acceso por rol; sesiones protegidas. |
| **Confiabilidad** | Respaldo de BD automatizable; sin pérdida de comandas abiertas ante recarga. |
| **Usabilidad** | Toma de pedidos rápida (pocos toques); UI clara para personal no técnico. |
| **Instalabilidad** | Despliegue con un solo comando (docker-compose) o script de instalación. |
| **PWA** | Instalable en el dispositivo y con buen comportamiento en pantalla táctil. |
| **Localización** | Formato de moneda COP; textos en español. |

---

## 12. Instalación y despliegue

**Objetivo:** que el dueño/administrador pueda instalarlo con el mínimo esfuerzo.

Opción recomendada — **Docker**:
1. Instalar Docker en el equipo servidor (PC/mini-PC siempre encendido).
2. Copiar el proyecto y ejecutar `docker-compose up -d`.
3. Averiguar la IP local del servidor (ej. `192.168.1.50`).
4. Desde cualquier dispositivo en la red: abrir `http://192.168.1.50:3000`.
5. (Opcional) "Instalar" la PWA en cada celular/tablet desde el navegador.

Consideraciones:
- Fijar IP estática al servidor en el router para que la dirección no cambie.
- Programar el respaldo automático del archivo SQLite.
- (Opcional) Configurar la impresora térmica por IP de red.

---

## 13. Plan de desarrollo por fases (para Claude CLI)

> Orden sugerido para generar la solución de forma incremental y verificable. Cada fase puede ser un *prompt/flujo* independiente en Claude CLI.

### Fase 0 — Base del proyecto
- Estructura del monorepo (backend NestJS + frontend React/Vite).
- Configuración de Prisma + SQLite y migraciones.
- Configuración de Docker / docker-compose.
- Esquema base del modelo de datos (sección 9).

### Fase 1 — Núcleo de catálogo e inventario
- M1 Autenticación y usuarios (roles).
- M2 Catálogo de productos y categorías.
- M3 Inventario y movimientos.
- M4 Recetas/fichas técnicas.

### Fase 2 — Operación de salón
- M6 Gestión de mesas (con tiempo real / WebSockets).
- M7 Comandas/pedidos.
- M5 Alertas de stock.

### Fase 3 — Cobro y caja
- M8 Ventas, cobro, métodos de pago, INC, propina, división de cuenta.
- Manejo de caja (apertura, cierre, arqueo).
- Descuento automático de inventario por venta/receta.
- Recibo imprimible/PDF.

### Fase 4 — Soporte y análisis
- M9 Informes y exportaciones.
- M10 Compras/proveedores.
- M11 Configuración del sistema.
- M12 Respaldos y auditoría.

### Fase 5 — Pulido
- PWA, optimización móvil, impresión térmica ESC/POS.
- Pruebas de extremo a extremo y guía de instalación final.

---

## 14. Estructura de proyecto sugerida

```
tabernapos/
├── docker-compose.yml
├── README.md                 # guía de instalación
├── backend/                  # NestJS
│   ├── prisma/
│   │   └── schema.prisma
│   └── src/
│       ├── auth/             # M1
│       ├── usuarios/         # M1
│       ├── productos/        # M2
│       ├── categorias/       # M2
│       ├── inventario/       # M3
│       ├── recetas/          # M4
│       ├── alertas/          # M5
│       ├── mesas/            # M6
│       ├── comandas/         # M7
│       ├── ventas/           # M8
│       ├── caja/             # M8
│       ├── reportes/         # M9
│       ├── proveedores/      # M10
│       ├── configuracion/    # M11
│       ├── auditoria/        # M12
│       └── realtime/         # WebSockets
└── frontend/                 # React + Vite + Tailwind (PWA)
    └── src/
        ├── modules/
        │   ├── auth/
        │   ├── menu/
        │   ├── inventario/
        │   ├── mesas/
        │   ├── comandas/
        │   ├── caja/
        │   └── reportes/
        ├── components/
        └── services/         # cliente API + sockets
```

---

## 15. Funcionalidades adicionales sugeridas (no estaban en la idea original)

Incluidas porque un sistema de bar/taberna las necesita en la práctica:

- **Recetas/fichas técnicas** para descontar insumos compuestos (un cóctel = varios insumos).
- **Roles y autenticación** (mesero, cajero, administrador) con trazabilidad.
- **Manejo de caja y arqueo** (apertura, cierre, cuadre del turno).
- **Impuesto al Consumo (INC 8%)** y propina, configurables.
- **Métodos de pago colombianos** (efectivo, tarjeta, Nequi/Daviplata, mixto) y cálculo de cambio.
- **División de cuenta** y transferencia de consumo entre mesas.
- **Compras/proveedores** para registrar entradas de mercancía.
- **Respaldos automáticos** (crítico al ser un sistema local).
- **Bitácora de auditoría** para anulaciones, cambios de precio y ajustes de inventario.
- **Tiempo real (WebSockets)** para sincronizar mesas y stock entre dispositivos.
- **Vista de barra/cocina (KDS)** opcional para ver comandas pendientes.

---

## 16. Criterios de aceptación (MVP)

- [ ] Se puede instalar y acceder desde varios dispositivos en la red local.
- [ ] Se pueden crear, editar y **eliminar mesas**.
- [ ] Se puede abrir una mesa, agregar lo pedido y ver el total en vivo.
- [ ] Al cobrar, se muestra el **detalle de lo pedido** y el **total a cobrar**.
- [ ] La venta **descuenta inventario** (directo y por receta) correctamente.
- [ ] Se disparan **alertas** cuando un insumo llega a su tope.
- [ ] Se registran ventas y se puede hacer apertura/cierre de caja.
- [ ] Se generan **informes** de ventas y de inventario.
- [ ] Cada usuario tiene su rol y sus permisos.
- [ ] Existe respaldo de la base de datos.

---

### Próximo paso
Usar este documento con **Claude CLI** para generar, fase por fase (sección 13), cada módulo descrito en la sección 8, partiendo del modelo de datos de la sección 9.
