# Documento de Requisitos del Producto (PRD) v2.0

## Sistema Web Administrativo & Catálogo Virtual Interactivo

**Versión Auditada (QA, UI/UX, Multitenancy & Data Isolation)**

---

### Información General del Proyecto

* **Proyecto:** Sistema ERP & E-Commerce Ligero
* **SuperAdmin Designado:** Edgar Mercado (`emercadogarcia@outlook.com`)
* **Ubicación:** Santa Cruz de la Sierra, Bolivia
* **Fecha / Versión:** 29 de Julio, 2026 — v2.0 Auditada
* **Estado:** Aprobado, Validado por QA/UIX e Integrado Técnicamente

---

## 1. Auditoría de UI/UX, Accesibilidad y QA Testing

### 1.1 UI/UX: Temas Dinámicos (Modo Claro / Modo Oscuro) y Diseño Minimalista

* **Soporte de Temas:**
* **Modo Claro (Default):** Fondos limpios (`#FAFAFA` / `#FFFFFF`), textos en alto contraste (`#0F172A`), bordes finos de delimitación (`#E2E8F0`) y tonos acento verdes/esmeralda (`#10B981`) para acciones principales.
* **Modo Oscuro (Dark Mode):** Fondos oscuros mate (`#0F172A` / `#1E293B`), textos claros (`#F8FAFC`), bordes sutiles (`#334155`) y acentos cian/esmeralda (`#38BDF8` / `#10B981`) de bajo brillo para evitar la fatiga visual.
* **Persistencia:** Guardado de preferencia de tema mediante local storage y sincronización con las preferencias del sistema operativo (`prefers-color-scheme`).


* **Experiencia de Usuario (UX):**
* **Diseño Minimalista & Mobile-First:** Interfaz limpia sin saturación visual. Uso de micro-interacciones suaves, loaders esqueléticos (Skeleton Loaders) durante la carga de datos para evitar parpadeos y Floating Action Buttons (FAB) en móviles.



### 1.2 Aislamiento Estricto de Datos (Multitenancy / Data Isolation)

* **Aislamiento por Inquilino/Usuario (Tenant Isolation):**
* Cada usuario registrado opera en un entorno **completamente aislado**. Toda consulta (`SELECT`, `INSERT`, `UPDATE`, `DELETE`) incluye de manera obligatoria la restricción por `tenant_id` / `user_id`.
* Ningún usuario ordinario (vendedor, empleado o cliente) puede acceder, visualizar ni modificar los datos de otro usuario.


* **Acceso Global de SuperAdmin:**
* **SuperAdmin Designado:** Edgar Mercado (`emercadogarcia@outlook.com`).
* El perfil SuperAdmin posee privilegios globales para la auditoría, soporte técnico, métricas consolidadas del sistema y gestión del esquema sin sobrepasar las normativas de privacidad.



### 1.3 Criterios de QA Testing y Calidad de Software

* **Manejo de Errores y Edge Cases:**
* Validaciones estricta *Client-Side* (Zod/Yup) y *Server-Side*.
* Bloqueo transaccional de operaciones inconsistentes (ej. evitar ventas sin stock disponible o reservas duplicadas en llamadas concurrentes).


* **Flujos de Error y Resiliencia:**
* **Sin Conexión:** Manejo de alertas emergentes (Toasts) si la llamada API falla o el usuario pierde internet.
* **Integridad Transaccional:** Uso de ACID / transacciones en DB para operaciones compuestas (ej. Registro de Venta + Descuento de Stock en Kardex + Registro de Transacción Financiera).



---

## 2. Arquitectura de Software y Agnosticism de Base de Datos

El sistema implementa el **Patrón Repositorio (Repository Pattern)** en la capa del backend/API para desacoplar la lógica de negocio de la base de datos persistente.

```
+-----------------------------------------------------------------------------------+
|                            ARQUITECTURA DE INTEGRACIÓN                            |
+-----------------------------------------------------------------------------------+
| [Cliente / Usuario Web] ----> Catálogo Virtual / Carrito ----> WhatsApp API Link  |
|                                                                                    |
| [Panel Admin / Vendedor] ---> React Frontend (Modo Claro / Modo Oscuro)            |
|                                     |                                             |
|                                     v                                             |
|                       REST API / Node.js (Express/NestJS)                         |
|                                     |                                             |
|                      [Capa Abstraída DB / ORM Prisma/Drizzle]                     |
|                                     |                                             |
|         +---------------------------+---------------------------+                 |
|         |                           |                           |                 |
|         v                           v                           v                 |
|  [PostgreSQL Local / SGBD]     [Supabase / Managed PG]     [Firebase / NoSQL]     |
|  (Entorno Local / Docker)     (Cloud Production)         (Sincronización Móvil)   |
+-----------------------------------------------------------------------------------+

```

### Flexibilidad de Proveedores de Base de Datos (Database Agnostic)

* **Motor Primario Recomendado:** **PostgreSQL** (soporte nativo para consultas relacionales, Kardex, transacciones complejas y RLS - Row Level Security).
* **Proveedores Compatibles:**
* **Supabase / Neon / Render:** Para despliegue gestionado en la nube con PostgreSQL.
* **PostgreSQL Local:** Instalación tradicional o contenedores Docker en servidores locales para operación sin costo recurrente de nube.
* **Firebase / Firestore:** Adaptable mediante adaptadores de repositorios para entornos offline-first o clientes móviles.



---

## 3. Especificación Auditada de Requisitos (25 Módulos / Reglas)

| # | Módulo / Área | Opción Elegida | Regla de Negocio, QA y Flujo Integrado |
| --- | --- | --- | --- |
| **P01** | Modo de Transacción | **Opción B** | **Modo Híbrido Permisible:** Compra rápida por WhatsApp sin registro obligatorio. Registro opcional para clientes recurrentes que deseen ver su historial. |
| **P02** | Lógica de Precios | **Opción C** | **Lógica Híbrida Combinada:** Precios automáticamente ajustados según la categoría del cliente (Mayorista/Minorista) y escala por volumen de compra. |
| **P03** | Procesamiento de Pedidos | **Opción A** | **Flujo en 3 Estados:** `Pendiente` $\rightarrow$ `En Preparación / Enviado` $\rightarrow$ `Entregado / Finalizado`. Validación QA: No se puede finalizar un pedido cancelado. |
| **P04** | Control de Stock | **Opción B** | **Reserva Preventiva:** Al generar el pedido se reserva el stock por un tiempo (*Timer* de 30 min por defecto). Si no se confirma el pago, el stock vuelve automáticamente al disponible. |
| **P05** | Multialmacén | **Opción B** | **Multialmacén Avanzado:** Existencias aisladas por bodega/almacén. Transferencias entre almacenes mediante documentos de traspaso con auditoría en Kardex. |
| **P06** | Roles y Permisos | **Opción A** | **Roles RBAC:** `SuperAdmin` (Edgar Mercado - Acceso Global), `Administrador` (Acceso completo de su empresa), `Vendedor` (Operación diaria) y `Repartidor` (Vista simplificada de despachos). |
| **P07** | Integración WhatsApp | **Opción C** | **Notificación Dual:** Formato de mensaje con estructura Markdown enviado tanto al número central de atención (**+591 70203103**) como al número del cliente. |
| **P08** | Módulo Presupuestos | **Opción A** | **Presupuesto Mensual por Categoría:** Definición de metas de ingresos y techos de gasto. Notificación visual cuando el gasto alcanza el 80% y 100% del tope. |
| **P09** | Vista de Resultados | **Opción A** | **Balance Financiero Directo:** Ingresos (Ventas al Contado + Abonos Efectivos) - Egresos (Gastos + Compras + Sueldos) = Utilidad Neta Real del periodo. |
| **P10** | Vista de Deudas | **Opción A** | **Tablero Consolidado Doble:** Vistas tabulares independientes para Cuentas por Cobrar (Clientes) y Cuentas por Pagar (Proveedores) con botones de pago/abono directo. |
| **P11** | Vista de Inventarios | **Opción C** | **Catálogo + Inventario Unificado:** Pantalla centralizada para editar precios, imágenes, descripciones y SEO junto con las existencias por almacén y Kardex. |
| **P12** | Dashboard Principal | **Opción C + Filtro** | **Tablero Híbrido:** Resumen del día (ventas, pedidos, stock crítico) + avance mensual vs. presupuesto. Incluye **Selector Dinámico de Fechas** (Hoy, Esta Semana, Este Mes, Personalizado). |
| **P13** | Estrategia SEO | **Opción B** | **SEO Estándar + Landing Page:** Optimización de metaetiquetas OpenGraph para vistas previas atractivas al compartir enlaces por WhatsApp o redes sociales. |
| **P14** | Compras y Proveedores | **Opción A** | **Registro de Compras Simplificado:** Carga de factura/recibo que incrementa automáticamente el stock del almacén seleccionado y registra la deuda o el egreso en caja. |
| **P15** | Descuentos y Ofertas | **Opción C** | **Reglas Flexibles Híbridas:** Descuentos porcentuales, monto fijo con precio tachado, rebajas por volumen y cupones de descuento personalizables. |
| **P16** | Gestión de Clientes | **Opción C** | **Ficha Avanzada con Segmentación:** Perfil de cliente con geolocalización GPS (link a Google Maps), estado de cuenta, historial de ventas y etiquetado (Mayorista, VIP, Moroso). |
| **P17** | Pasarelas de Pago / QR | **Opción C** | **Híbrido Permisible:** Carga de fotos de comprobante o número de transferencia + despliegue de código QR estático/dinámico para pago rápido. |
| **P18** | Almacenamiento Fotos | **Opción A** | **Compresión Automática a WebP:** Al subir imágenes en el panel, la API las procesa y comprime a formato `.webp` optimizado para la web antes de guardarlas en el storage. |
| **P19** | Notificaciones WhatsApp | **Opción A** | **Enlaces Directos con Plantillas:** Botones inteligentes en la interfaz que abren la App de WhatsApp con textos pre-formateados (confirmación de entrega, cobranza de cuota, abonos). |
| **P20** | Asignación de Stock | **Opción A** | **Almacén Principal por Defecto:** Las ventas desde la web descuentan del almacén asignado como "Principal". El vendedor puede cambiar el almacén origen en ventas manuales. |
| **P21** | Categorías y Tags | **Opción C** | **Categorías + Etiquetas Flexibles:** Categorías de un solo nivel combinadas con etiquetas libres (`#OfertaDelDía`, `#Orgánico`, `#Importado`) para filtrado rápido. |
| **P22** | Auditoría y Logs | **Opción B** | **Registro de Inventario y Caja:** Historial completo e inalterable de todos los movimientos de stock (Kardex) y todas las entradas/salidas de fondos de caja. |
| **P23** | Respaldos y Reportes | **Opción B + C** | **Exportación Dual (Excel/CSV + PDF):** Descarga de listados planos en Excel/CSV e impresión formateada de comprobantes, recibos de abono y balances generales en PDF. |
| **P24** | Gastos Operativos | **Opción A** | **Clasificación por Categorías:** Registro de egresos con categorías asignables (Alquiler, Servicios, Impuestos) y adjunto de comprobante en imagen/PDF. |
| **P25** | Sueldos y Personal | **Opción B** | **Gestión Integrada con Anticipos:** Control de nómina de personal, registro de adelantos/anticipos y cálculo del pago neto mensual registrado como egreso operativo. |

---

## 4. Esquema Relacional de Base de Datos (PostgreSQL DDL)

A continuación se presenta el esquema con la inclusión de `tenant_id` para el aislamiento estricto de datos y la estructuración de la persistencia:

```sql
-- Habilitar extensión para UUIDs
CREATE EXTENSION IF NOT EXISTS "uuid-ossp";

-- 1. TABLA DE TENANTS / USUARIOS PRINCIPALES
CREATE TABLE tenants (
    id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
    nombre_empresa VARCHAR(150) NOT NULL,
    email_owner VARCHAR(150) UNIQUE NOT NULL,
    creado_en TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- 2. TABLA DE USUARIOS Y ROLES (RBAC)
CREATE TABLE usuarios (
    id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
    tenant_id UUID REFERENCES tenants(id) ON DELETE CASCADE,
    nombre_completo VARCHAR(150) NOT NULL,
    email VARCHAR(150) NOT NULL,
    password_hash VARCHAR(255) NOT NULL,
    rol VARCHAR(30) CHECK (rol IN ('SUPERADMIN', 'ADMINISTRADOR', 'VENDEDOR', 'REPARTIDOR')),
    creado_en TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    CONSTRAINT uk_usuario_tenant UNIQUE (tenant_id, email)
);

-- 3. TABLA DE ALMACENES
CREATE TABLE almacenes (
    id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
    tenant_id UUID NOT NULL REFERENCES tenants(id) ON DELETE CASCADE,
    nombre VARCHAR(100) NOT NULL,
    es_principal BOOLEAN DEFAULT FALSE,
    direccion TEXT,
    creado_en TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- 4. TABLA DE CLIENTES (Ficha CRM)
CREATE TABLE clientes (
    id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
    tenant_id UUID NOT NULL REFERENCES tenants(id) ON DELETE CASCADE,
    nombre VARCHAR(150) NOT NULL,
    nit_ci VARCHAR(50),
    telefono VARCHAR(30),
    direccion TEXT,
    ubicacion_gps VARCHAR(255),
    categoria VARCHAR(50) DEFAULT 'MINORISTA', -- MAYORISTA, MINORISTA, VIP
    limite_credito DECIMAL(12, 2) DEFAULT 0.00,
    creado_en TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- 5. TABLA DE PRODUCTOS (Catálogo)
CREATE TABLE productos (
    id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
    tenant_id UUID NOT NULL REFERENCES tenants(id) ON DELETE CASCADE,
    sku VARCHAR(50),
    nombre VARCHAR(150) NOT NULL,
    descripcion TEXT,
    precio_minorista DECIMAL(12, 2) NOT NULL,
    precio_mayorista DECIMAL(12, 2),
    precio_oferta DECIMAL(12, 2),
    imagen_url TEXT,
    tags TEXT[], -- Array de tags: ['#Oferta', '#Orgánico']
    creado_en TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- 6. TABLA DE STOCK EN ALMACÉN
CREATE TABLE inventario_stock (
    id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
    tenant_id UUID NOT NULL REFERENCES tenants(id) ON DELETE CASCADE,
    producto_id UUID NOT NULL REFERENCES productos(id) ON DELETE CASCADE,
    almacen_id UUID NOT NULL REFERENCES almacenes(id) ON DELETE CASCADE,
    stock_disponible INT NOT NULL DEFAULT 0,
    stock_reservado INT NOT NULL DEFAULT 0,
    CONSTRAINT uk_stock_almacen UNIQUE (producto_id, almacen_id)
);

-- 7. TABLA DE KARDEX (Auditoría de Movimientos)
CREATE TABLE kardex_movimientos (
    id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
    tenant_id UUID NOT NULL REFERENCES tenants(id) ON DELETE CASCADE,
    producto_id UUID NOT NULL REFERENCES productos(id),
    almacen_id UUID NOT NULL REFERENCES almacenes(id),
    tipo_movimiento VARCHAR(30) CHECK (tipo_movimiento IN ('ENTRADA_COMPRA', 'SALIDA_VENTA', 'TRANSFERENCIA_ENTRADA', 'TRANSFERENCIA_SALIDA', 'AJUSTE')),
    cantidad INT NOT NULL,
    stock_resultante INT NOT NULL,
    referencia VARCHAR(100),
    usuario_id UUID REFERENCES usuarios(id),
    fecha TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- 8. TABLA DE PEDIDOS / VENTAS
CREATE TABLE pedidos (
    id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
    tenant_id UUID NOT NULL REFERENCES tenants(id) ON DELETE CASCADE,
    cliente_id UUID REFERENCES clientes(id),
    almacen_id UUID REFERENCES almacenes(id),
    total DECIMAL(12, 2) NOT NULL,
    estado VARCHAR(30) CHECK (estado IN ('PENDIENTE', 'EN_PREPARACION', 'ENTREGADO', 'CANCELADO')),
    tipo_pago VARCHAR(30) CHECK (tipo_pago IN ('CONTADO', 'CREDITO')),
    metodo_pago VARCHAR(30), -- QR, TRANSFERENCIA, EFECTIVO
    comprobante_url TEXT,
    creado_en TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- POLÍTICA DE SEGURIDAD RLS (Ejemplo para PostgreSQL)
ALTER TABLE clientes ENABLE ROW LEVEL SECURITY;
CREATE POLICY clientes_isolation_policy ON clientes 
    USING (tenant_id = current_setting('app.current_tenant_id')::UUID OR current_setting('app.current_user_email') = 'emercadogarcia@outlook.com');

```

---

## 5. Matriz de QA Testing y Casos de Prueba Críticos

| ID Caso | Módulo | Descripción del Test | Resultado Esperado |
| --- | --- | --- | --- |
| **QA-01** | Multitenancy | Intento de acceso directo vía API a la lista de clientes de otro `tenant_id`. | **Retorno HTTP 403 Forbidden** o resultado vacío (`[]`). |
| **QA-02** | SuperAdmin | Inicio de sesión con el correo `emercadogarcia@outlook.com`. | **Acceso total** habilitado para seleccionar cualquier Tenant en la consola de administración. |
| **QA-03** | Stock | Dos clientes compran simultáneamente la última unidad de un producto. | El sistema procesa un pedido y el segundo recibe una **alerta de stock agotado** gracias al bloqueo transaccional. |
| **QA-04** | Reserva | Se crea un pedido desde la web que reserva 5 unidades. El pedido no se paga tras 30 minutos. | El proceso en segundo plano libera la reserva y el stock vuelve al estado **Disponible**. |
| **QA-05** | UI/UX | Cambio entre Tema Claro y Tema Oscuro. | La interfaz ajusta sus paletas de color en menos de **100ms** sin parpadear ni perder la sesión o filtros activos. |
| **QA-06** | Fechas | Filtrado del Dashboard con rango personalizado. | Las gráficas, ingresos, egresos y avance del presupuesto se recalculan únicamente con los datos del periodo seleccionado. |
| **QA-07** | Exportación | Descarga del informe de ventas en formato PDF y Excel. | El archivo **Excel (.xlsx)** descarga los datos estructurados en celdas y el **PDF** se genera con formato listo para impresión. |

---

## 6. Plan de Entregables y Roadmap Técnico

1. **Fase 1 (Semanas 1-2):**
* Configuración de la base de datos PostgreSQL con políticas RLS y script de aislamiento por Tenant.
* Asignación del rol de SuperAdmin global a Edgar Mercado (`emercadogarcia@outlook.com`).


2. **Fase 2 (Semanas 3-4):**
* Desarrollo del Catálogo Virtual dinámico con soporte de Temas (Claro/Oscuro) y optimización móvil.
* Envío de pedidos estructurados al WhatsApp central (**+591 70203103**).


3. **Fase 3 (Semanas 5-6):**
* Panel de administración de inventarios multialmacén, procesamiento automático de imágenes a formato `.webp` y Kardex.


4. **Fase 4 (Semanas 7-8):**
* Módulo financiero: Cuentas por Cobrar/Pagar, registro de abonos, control de gastos y planilla de sueldos con anticipos.


5. **Fase 5 (Semanas 9-10):**
* Dashboard interactivo con filtro de fechas dinámico, comparación con presupuestos, generación de reportes PDF e integración de exportación Excel/CSV.