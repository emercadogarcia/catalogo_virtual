# DOCUMENTO DE REQUERIMIENTOS DE PRODUCTO (PRD)
# PROYECTO: "3MG Store" — Plataforma Transaccional Multi-Tenant de Comercio Electrónico, POS & Facturación Electrónica

---

### METADATOS DEL DOCUMENTO
* **Nombre Oficial del Producto:** 3MG Store Enterprise Multi-Tenant Platform
* **Versión del Documento:** 1.0.0 — Edición Producción 2026
* **Año de Publicación:** 2026
* **Organización Desarrolladora:** [3MGLabs.com](https://3mglabs.com)
* **Director del Proyecto:** Edgar Mercado García
* **Línea de Contacto Oficial & WhatsApp:** +591 70203103
* **Correo de Arquitectura & Super-Admin Inicial:** `dev.emercado@gmail.com`
* **Estado:** Aprobado para Ejecución y Replicación Multi-Tenant

---

## 1. RESUMEN EJECUTIVO Y VISIÓN DEL PRODUCTO

### 1.1. Visión General
**3MG Store** es una plataforma de software transaccional multi-inquilino (Multi-Tenant SaaS / Self-Hosted) de alta disponibilidad concebida y desarrollada por **3MGLabs.com** en 2026 bajo la dirección de **Edgar Mercado García**. El sistema unifica en una sola solución industrial la venta digital omnicanal, el punto de venta físico (POS), la prevención de sobreventas mediante control de concurrencia atómico, un motor de facturación electrónica fiscal con timbrado digital (CUFE/CAE), un motor de listas de precios parametrizables por vigencia temporal, integración integral con WhatsApp en todos los procesos de compra y un módulo de administración de APIs para conectividad con ERPs, pasarelas de pago y pagos por QR.

### 1.2. Objetivos Estratégicos
1. **Multi-Tenancy Robusto:** Permitir que miles de empresas, marcas o sucursales coexistan en una misma infraestructura con aislamiento estricto de datos, catálogos propios, dominios personalizados y configuraciones corporativas independientes.
2. **Replicación Exacta del Backoffice:** Incorporar sin omitir ninguna de las capacidades transaccionales, auditoría, multi-moneda con histórico, listas de precios con vigencia estricta, facturación electrónica y tickets ESC/POS ya validados.
3. **Omnicanalidad WhatsApp 360°:** Automatizar el envío de pedidos, estados de entrega, recibos y facturas en PDF hacia el WhatsApp del comprador en cada etapa del ciclo de vida de la transacción.
4. **Gobierno y Jerarquía de Usuarios:** Establecer al superusuario global `dev.emercado@gmail.com` con control absoluto de tenants, y a los administradores de cada tenant con capacidad de estructurar supervisores y dependientes subalternos.
5. **Independencia y Portabilidad de Base de Datos:** Garantizar la migración fluida entre PostgreSQL 16+, Cloud SQL, Neon, Supabase, MySQL o instancias dedicadas on-premise mediante ORM desacoplado y scripts DDL idempotentes.

---

## 2. ARQUITECTURA MULTI-TENANT Y MODELO DE AISLAMIENTO

### 2.1. Estrategia de Aislamiento de Datos
El sistema soporta dos modalidades arquitectónicas según el plan comercial del tenant:
* **Modalidad Pool (Shared Database, Pooled Schema):**
  - Base de datos relacional PostgreSQL 16+ compartida.
  - Todas las tablas de negocio incluyen la columna `tenant_id UUID NOT NULL REFERENCES tenants(id) ON DELETE RESTRICT`.
  - Se implementa **PostgreSQL Row Level Security (RLS)** obligatorio:
    ```sql
    ALTER TABLE products ENABLE ROW LEVEL SECURITY;
    CREATE POLICY tenant_isolation_policy ON products
      USING (tenant_id = NULLIF(current_setting('app.current_tenant_id', true), '')::uuid);
    ```
  - Cada solicitud HTTP inyecta el `tenant_id` resuelto mediante subdominio (`tenant.3mgstore.com`), dominio personalizado (`mitienda.com`) o encabezado `X-Tenant-ID`.
* **Modalidad Silo (Database-per-Tenant):**
  - Para clientes Enterprise o gubernamentales que requieren aislamiento físico o normativo.
  - El Connection Pooler (PgBouncer) enruta dinámicamente según el identificador de conexión almacenado en el catálogo central de tenants.

### 2.2. Aprovisionamiento y Registro de Tenants
* **Flujo de Onboarding:**
  1. Registro de nuevo comercio mediante **Google OAuth 2.0** o **Correo Electrónico + Contraseña**.
  2. El primer usuario que completa el registro del tenant es asignado automáticamente como **TENANT_OWNER** (Administrador del Tenant).
  3. Aprovisionamiento automático de datos semilla:
     - Catálogo base y categorías por defecto.
     - Lista de precios inicial (*Tarifa General Retail*).
     - Configuración de moneda local sugerida según el país seleccionado.
     - Certificado SSL y subdominio asignado.

---

## 3. GESTIÓN DE IDENTIDAD, ACCESOS Y JERARQUÍA DE USUARIOS

### 3.1. Superusuario Global (Root / Platform Administrator)
* **Identidad:** `dev.emercado@gmail.com`
* **Comportamiento en Primer Inicio:**
  - El sistema detecta si es la primera autenticación de dicha cuenta.
  - Bloquea el acceso ordinario y despliega obligatoriamente la pantalla de **Asignación de Contraseña Maestra y Verificación de Doble Factor (2FA)**.
  - Una vez configurada la credencial, el usuario accede al panel de **Super-Administración Global**.
* **Privilegios Exclusivos del Superusuario:**
  - Visualización sin restricciones de todos los tenants registrados en la plataforma.
  - Capacidad de **Tenant Impersonation** (ingresar a cualquier tenant en modo auditoría/soporte con registro en log de auditoría inmutable).
  - Activación, suspensión y baja definitiva de tenants.
  - Gestión de planes, cobros de suscripción, límites de almacenamiento y cuotas de API.

### 3.2. Jerarquía de Usuarios y Dependencias Subalternas (RBAC Jerárquico)
Dentro de cada tenant, el sistema modela una estructura organizacional de árbol jerárquico:
* **TENANT_OWNER:** Administrador principal del comercio. Acceso ilimitado dentro de su inquilino.
* **STORE_MANAGER (Gerente de Tienda/Sucursal):** Supervisa inventarios, listas de precios, cajeros y reportes.
* **CASHIER_POS (Cajero de Punto de Venta):** Operación de ventas, emisión de tickets y cobros. Reporta a un Gerente.
* **ACCOUNTANT (Contador Fiscal):** Acceso a facturas electrónicas, retenciones, impuestos y exportaciones contables.
* **DISPATCHER (Logística y Despacho):** Preparación de pedidos, actualización de estados de envío y tracking.

#### Esquema de Dependencias Subalternas:
* Cada usuario posee un campo opcional `supervisor_user_id UUID REFERENCES users(id)`.
* Un supervisor puede ver, auditar y reasignar las órdenes y ventas generadas exclusivamente por sus dependientes directos o subalternos de su sucursal.
* El módulo de administración permite visualizar el organigrama interactivo de personal.

---

## 4. CONFIGURACIÓN CORPORATIVA Y REGLAS DE NEGOCIO DE LA EMPRESA

### 4.1. Ficha Corporativa del Comercio
Cada tenant administra los siguientes datos oficiales con impacto en storefront, comprobantes y facturas:
* **Nombre Comercial:** Rótulo para vitrina digital, emails y WhatsApp.
* **Razón Social / Nombre Jurídico:** Nombre legal registrado ante la entidad tributaria.
* **Número de Identificación Fiscal (Tax ID):** NIF, NIT, RUC, RUT, RFC o CIF con dígito verificador.
* **Teléfono Fijo y Celular Corporativo:** Canales oficiales de atención.
* **WhatsApp Corporativo Oficial:** Número validado para envío de comprobantes y atención al cliente.
* **Correo Electrónico Institucional:** Para notificaciones fiscales y transaccionales.
* **Responsable del Negocio / Representante Legal:** Nombre y cargo del titular.
* **Dirección Fiscal y Sucursal Física:** Dirección detallada, ciudad y país.

### 4.2. Selector de País Precargado y Sugerencia de Moneda Inteligente
* **Catálogo Precargado de Países:** El campo país no es de texto libre; se selecciona de una base precargada estandarizada ISO 3166-1 (Bolivia, Colombia, México, Perú, Chile, Argentina, España, Estados Unidos, etc.).
* **Sugerencia Automática de Moneda Local:**
  - Al seleccionar el país, el sistema sugiere automáticamente su moneda de curso legal (ej. Bolivia → `BOB Bs`, Colombia → `COP $`, Perú → `PEN S/`, México → `MXN $`, España → `EUR €`).
  - El administrador confirma o modifica la sugerencia como **Moneda Base Contable**.
  - El sistema habilita como **Segundas Monedas de Transacción y Referencia** a `USD` ($) y `EUR` (€).
  - La moneda seleccionada se refleja en tiempo real en la barra de navegación del comercio, en los precios del catálogo y en el motor multi-moneda con historial auditable.

### 4.3. Procedimiento de Baja de la Cuenta Comercial
* Opción en Backoffice para solicitar o ejecutar la **Baja Temporal o Definitiva de la Cuenta**.
* Requiere motivo obligatorio y confirmación de seguridad.
* Al desactivarse, la vitrina pasa a modo solo lectura informativo, se congelan nuevas órdenes y se notifica al superusuario.

---

## 5. MOTOR DE LISTAS DE PRECIOS CON VIGENCIA ESTRICTA

### 5.1. Reglas de Negocio Idénticas al Backoffice Actual
* **Segmentación por Canal y Campaña:**
  - Creación de múltiples listas de precios: *Tarifa General Retail (Público)*, *Campaña Flash Sale / Cyber*, *Mayoristas B2B*, *Clientes VIP / Convenios Corporativos*.
* **Regla de Intervalo Temporal Estricto `[Fecha_vigencia, Fecha_siguiente)`:**
  - Cada lista posee un `validFrom` (inclusivo) y un `validTo` (exclusivo o nulo si es permanente).
  - El sistema resuelve la tarifa activa para cualquier fecha de venta evitando colisiones y permitiendo pre-programar campañas estacionales.
* **Fórmula de Cálculo Dinámico:**
  $$\text{Precio Final} = \text{Precio Base USD} \times \left(1 - \frac{\text{Descuento Global \%}}{100}\right)$$
* **Matriz de Tarifas por Producto ("Ver Tarifas"):**
  - Modal interactivo que renderiza el catálogo íntegro con columnas: SKU, Producto, Precio Base, Descuento Aplicado, Precio Calculado con esa Lista y Ahorro Efectivo para el Cliente.
* **Asignación de Lista Predeterminada:** Posibilidad de marcar con un clic cualquier lista como la predeterminada del tenant (`isDefault`).
* **Sincronización Inmediata con la Vitrina y el POS:** El selector de tarifas en el encabezado de la tienda recalcula en vivo los precios del catálogo y aplica el valor correcto a las líneas del carrito.

---

## 6. TRANSACCIONALIDAD, CONTROL DE INVENTARIO Y POS

### 6.1. Prevención Concurrente de Sobreventas (Zero Overselling)
* Bloqueo a nivel de base de datos relacional durante la liquidación del carrito:
  ```sql
  SELECT id, stock, is_active FROM products 
  WHERE id = ANY(p_product_ids) 
  FOR UPDATE;
  ```
* Si la cantidad solicitada excede el stock disponible en el milisegundo de confirmación, la transacción se aborta de forma atómica y notifica al comprador sin cobrar.

### 6.2. Soporte Estricto de Idempotencia
* Encabezado HTTP `Idempotency-Key` (UUIDv4) obligatorio en `/api/v1/checkout`.
* Si un cliente presiona dos veces el botón de pago o sufre microcortes de red, el backend retorna la orden previamente creada sin duplicar cargos bancarios ni deducciones de stock.

### 6.3. Punto de Venta (POS) y Tickets Térmicos ESC/POS
* Generador de buffer binario ESC/POS nativo para impresoras térmicas de 58mm (32 columnas) y 80mm (48 columnas).
* Impresión de encabezado comercial, desglose de ítems alineados, totales en moneda local y USD, hash fiscal CUFE y código QR para validación electrónica.
* Compatibilidad con descarga de stream binario directo y emulación visual en pantalla.

---

## 7. MOTOR DE FACTURACIÓN ELECTRÓNICA FISCAL

### 7.1. Características del Motor
* Emisión automática de comprobantes vinculados a la orden de compra.
* Desglose impositivo estricto: Subtotal neto, IVA / Impuesto Local parametrizable por país, retenciones y total liquidado.
* Generación de hash digital SHA-256 (CUFE/CAE) y código de control criptográfico.
* Generador de reportes PDF descargables e imprimibles con selector de formato de papel **Carta (Letter)** y **Oficio (Legal)**.
* Generación de archivo estructurado XML/UBL listo para envío a la autoridad tributaria correspondiente.

---

## 8. INTEGRACIÓN OMNICANAL CON WHATSAPP EN TODOS LOS PROCESOS

### 8.1. Puntos de Contacto Automatizados vía WhatsApp
1. **Confirmación de Orden Recibida:** Envío inmediato tras checkout con número `#PED`, resumen de compra y monto total.
2. **Actualización de Estado Logístico:** Notificación automática al cambiar el estado de la orden (En Preparación, Despachado en Ruta, Entregado) con enlace de rastreo en vivo.
3. **Envío de Recibo Térmico Digital:** Enlace directo para visualizar e imprimir el ticket térmico POS.
4. **Envío de Factura Electrónica Fiscal:** Envío del PDF oficial tributario y código CUFE vía mensaje directo al celular del cliente.
5. **Canal de Soporte Directo:** Botón flotante y enlaces directos al WhatsApp corporativo del tenant y al soporte técnico central (+591 70203103).

---

## 9. TÉCNICAS AVANZADAS DE SEO Y POSICIONAMIENTO

### 9.1. Estrategia de Posicionamiento por Rubro y País
* **URLs Semánticas y Amigables:** Estructura limpia `/categoria/subcategoria/slug-del-producto`.
* **Metadatos Dinámicos SSR:**
  - Etiquetas `<title>`, `<meta name="description">`, `canonical` autogeneradas con palabras clave del país y rubro.
  - OpenGraph (`og:title`, `og:image`, `og:price:amount`, `og:price:currency`) y Twitter Cards para alta conversión al compartir en redes sociales.
* **Datos Estructurados Schema.org (JSON-LD):**
  - Esquema `Product` con ofertas, disponibilidad, SKU y valoraciones.
  - Esquema `Organization` y `LocalBusiness` con dirección fiscal, geolocalización, teléfono y moneda base.
  - Esquema `BreadcrumbList` y `WebSite` con acción de búsqueda interna (`SearchAction`).
* **Optimización de Rendimiento (Core Web Vitals):**
  - Carga diferida de imágenes con formato WebP/AVIF y dimensiones explícitas.
  - Generación de `sitemap.xml` dinámico y archivo `robots.txt` segmentado por tenant.

---

## 10. MÓDULO DE ADMINISTRACIÓN DE APIS Y CONECTIVIDAD

### 10.1. Módulos API RESTful & Webhooks Expuestos
Cada tenant puede generar credenciales de API (API Key + Secret) con permisos granulares (Scopes) para sincronización con sistemas externos:
* **API de Pedidos (`/api/v1/orders`):** Consulta, creación, actualización de estados y asignación de despachadores.
* **API de Ventas y POS (`/api/v1/sales`):** Registro de ventas en mostrador físico y sincronización con cajas registradoras.
* **API de Compras e Inventario (`/api/v1/purchases` & `/api/v1/inventory`):** Entradas de almacén, ajustes por merma y transferencias entre depósitos.
* **API de Facturación Electrónica (`/api/v1/invoices`):** Emisión, anulación, consulta de estado fiscal y descarga de XML/PDF.
* **API de Pagos con QR y Pasarelas (`/api/v1/payments/qr`):**
  - Generación de códigos QR interoperables (Simple QR Bolivia, CoDi México, Pix Brasil, PSE Colombia).
  - Conexión con pasarelas de pago (Stripe, PayPal, MercadoPago, pasarelas bancarias locales) y manejo de webhooks con firma HMAC-SHA256.

### 10.2. Panel de Administración de APIs en Backoffice
* Creación y revocación de API Keys en tiempo real.
* Configuración de URLs de Webhook para eventos: `order.created`, `order.paid`, `invoice.issued`, `stock.low`.
* Registro de auditoría de peticiones con código de respuesta HTTP, latencia y payload.
* Documentación interactiva Swagger / OpenAPI accesible desde el panel.

---

## 11. ESTRATEGIA DE PORTABILIDAD Y MIGRACIÓN DE BASE DE DATOS

### 11.1. Capa de Abstracción de Base de Datos
* El sistema desacopla la lógica de negocio de la base de datos mediante contratos de repositorio y ORM (Drizzle / Prisma).
* **Script DDL Base:** PostgreSQL 16+ con UUIDv4/v7, restricciones CHECK e índices compuestos para alta concurrencia.
* **Capacidad de Migración:**
  - Compatibilidad certificada para migrar a PostgreSQL Gestionado (Cloud SQL, Supabase, Neon, AWS Aurora PostgreSQL) o motores relacionales compatibles mediante adaptadores de dialecto.
  - Herramienta integrada en Backoffice para **Exportación Completa del Tenant**:
    * Volcado de datos en formato JSON estructurado o script SQL estándar.
    * Respaldo de catálogos, clientes, órdenes, facturas y logs de auditoría.
    * Herramienta de importación / restore para recuperación ante desastres o cambio de servidor.

---

## 12. CUMPLIMIENTO LEGAL, PRIVACIDAD Y POLÍTICA DE COOKIES

### 12.1. Consentimiento de Cookies GDPR / LOPD
* Banner flotante con gestión granular de preferencias:
  - **Cookies Necesarias (Técnicas):** Sesión, autenticación, prevención de fraude CSRF e idempotencia.
  - **Cookies de Analítica:** Métricas de navegación anonimizadas sin rastreo invasivo.
  - **Cookies de Marketing:** Medición de campañas publicitarias.
* Almacenamiento en `localStorage` y persistencia en backend para auditoría de cumplimiento.

### 12.2. Marco Legal del Tenant
* Módulos administrables para: Términos y Condiciones del Servicio, Política de Privacidad de Datos, Política de Garantías y Devoluciones, y Condiciones de Despacho Logístico.

---

## 13. IDENTIDAD VISUAL Y ESPECIFICACIÓN DEL ICONO / FAVICON

### 13.1. Icono de Navegador (Favicon)
* Icono vectorial SVG y PNG de alta resolución para pestaña de navegador y aplicaciones móviles instalables (PWA).
* **Composición Gráfica:**
  - Carrito de compras estilizado en líneas blancas modernas con ruedas verdes (#10b981).
  - Distintivo central prominente con el texto en negrita extrema **"3MG"**.
  - Fondo redondeado con gradiente azul industrial (#1d4ed8 a #0f172a).
  - Diseñado para máxima legibilidad tanto en favicon (16x16 / 32x32) como en iconos de escritorio y móviles (192x192 / 512x512).

---

## 14. MATRIZ DE REPLICACIÓN TÉCNICA (CHECKLIST PARA DESARROLLO)

| Módulo / Capacidad | Estado en MVP Actual | Requerimiento en "3MG Store" |
|---|---|---|
| **Multi-Tenancy** | Monotenant extensible | Multi-Tenant nativo con aislamiento RLS |
| **Superusuario Inicial** | Mock de usuario | `dev.emercado@gmail.com` con setup inicial de clave |
| **Registro de Usuarios** | Registro local con OTP | Google OAuth 2.0 + Email/Clave + OTP |
| **Jerarquía de Personal** | Lista plana con roles | Estructura de supervisores y dependientes subalternos |
| **Datos de la Empresa** | Configurable en Backoffice | Configurable + Selector de País precargado + Sugerencia de moneda |
| **Listas de Precios** | 100% Funcional con vigencia | 100% Replicado idéntico + Matriz de Tarifas por Producto |
| **Multi-Moneda** | Base + USD + COP + EUR | Base sugerida por país + USD/EUR + Histórico cronológico |
| **Órdenes & POS** | TanStack Table v8 | Replicado idéntico con filtro por sucursal / dependiente |
| **Tickets ESC/POS** | Rollos 58mm y 80mm con QR | Replicado idéntico + enlace directo vía WhatsApp |
| **Facturación Electrónica** | Timbrado CUFE + Carta/Oficio | Replicado idéntico + envío de PDF vía WhatsApp |
| **WhatsApp 360°** | Enlace estático de soporte | Automatización de pedidos, tracking, recibos y facturas |
| **SEO Avanzado** | Schema.org + Meta tags | Meta tags dinámicos por país, rubro y Core Web Vitals |
| **Módulo de APIs** | Endpoint `/api/v1/checkout` | Suite completa: Ventas, Compras, Invoices, QR Payments |
| **Migración de BD** | DDL PostgreSQL 16 | Exportador completo JSON/SQL y compatibilidad multi-cloud |
| **Favicon 3MG** | Genérico previo | Favicon exclusivo Carrito + "3MG" en SVG y PNG |

---

## 15. INFORMACIÓN DE CONTACTO Y SOPORTE DE INGENIERÍA

Para soporte en la replicación, despliegue en servidores dedicados o licenciamiento de **3MG Store**:
* **Desarrollador Principal:** 3MGLabs.com (2026)
* **Director de Tecnología:** Edgar Mercado García
* **WhatsApp / Teléfono Móvil:** +591 70203103
* **Correo Electrónico:** dev.emercado@gmail.com / contacto@3gmlabs.com
* **Sitio Web Oficial:** [https://3gmlabs.com](https://3gmlabs.com)
