# TASKS — FiscalOS Backlog

Estado: `[ ]` pendiente · `[~]` en progreso · `[x]` completado · `[-]` cancelado

---

## Fase 0 — Setup e Infraestructura

### Repositorios y entorno
- [ ] Crear repositorio `fiscal-portal-web` (Next.js)
- [ ] Crear repositorio `fiscal-portal-api` (FastAPI)
- [ ] Configurar `.gitignore`, `.env.example` en ambos repos
- [ ] Inicializar Next.js 14 con App Router y TypeScript estricto
- [ ] Configurar Tailwind CSS + shadcn/ui en el proyecto web
- [ ] Inicializar FastAPI con estructura de carpetas definida en ARCHITECTURE.md
- [ ] Crear `Dockerfile` y `docker-compose.yml` para FastAPI
- [ ] Configurar ESLint + Prettier en el proyecto web
- [ ] Configurar proyecto en Supabase (local con Docker para desarrollo)

### Base de datos
- [ ] Crear tablas: `organizations`, `users`, `clients`
- [ ] Crear tablas: `documents`, `document_folders`
- [ ] Crear tablas: `filings`, `filing_types`, `filing_stages`
- [ ] Crear tablas: `requests`, `request_comments`
- [ ] Crear tabla: `activity_log` (sin UPDATE/DELETE en RLS)
- [ ] Crear tablas: `notifications`, `fiscal_deadlines`
- [ ] Habilitar RLS en todas las tablas
- [ ] Definir políticas RLS por rol (client, staff, admin)
- [ ] Generar types de TypeScript con Supabase CLI (`supabase gen types`)
- [ ] Configurar Alembic para migraciones en FastAPI

---

## Fase 1 — Auth & Onboarding

### Magic links y sesión
- [ ] Configurar Supabase Auth con magic links
- [ ] Crear página `/login` con formulario de email
- [ ] Configurar `middleware.ts` para proteger rutas por autenticación
- [ ] Configurar `middleware.ts` para redirección por rol (client/staff/admin)
- [ ] Crear clientes de Supabase: `client.ts` (browser) y `server.ts` (RSC)
- [ ] Hook `useCurrentUser` para acceder al usuario en componentes cliente

### Sistema de invitaciones
- [ ] Endpoint FastAPI: `POST /invitations` — crear invitación con token
- [ ] Endpoint FastAPI: `POST /invitations/accept` — aceptar invitación
- [ ] Página `/invite/[token]` — formulario de aceptación de invitación
- [ ] Email de invitación con link al portal
- [ ] Expiración de tokens de invitación (72 horas)

### Onboarding del despacho
- [ ] Página `/onboarding` — wizard de configuración inicial
- [ ] Paso 1: Datos del despacho (nombre, RFC, logo)
- [ ] Paso 2: Invitar al primer cliente
- [ ] Guardar configuración en tabla `organizations`

---

## Fase 2 — Layout y Navegación

### Shell de la aplicación
- [ ] Componente `AppShell` — layout base con sidebar y topbar
- [ ] Componente `ClientSidebar` — navegación del portal del cliente
- [ ] Componente `InternalSidebar` — navegación del workspace del contador
- [ ] Componente `AdminSidebar` — navegación de la consola admin
- [ ] Componente `TopBar` — logo, usuario activo, notificaciones, logout
- [ ] Componente `PageHeader` — título de página consistente con breadcrumb
- [ ] Layouts de App Router para cada grupo de rutas: `(client)`, `(internal)`, `(admin)`

### Componentes UI base
- [ ] `StatusBadge` — pill de colores para estados del sistema
- [ ] `TimelineItem` — item del activity feed
- [ ] `EmptyState` — estado vacío con ilustración y CTA
- [ ] `ConfirmDialog` — diálogo de confirmación para acciones destructivas
- [ ] `DeadlineCountdown` — fecha con urgencia visual (verde/amarillo/rojo)
- [ ] `AvatarStack` — múltiples avatares apilados con overflow
- [ ] `SectionCard` — card container con header/content/footer slots

---

## Fase 3 — Portal del Cliente

### Dashboard del cliente
- [ ] Página `/dashboard` — vista general del cliente
- [ ] Widget: solicitudes pendientes (con contador y urgencia)
- [ ] Widget: declaraciones activas con su etapa actual
- [ ] Widget: próximas fechas límite fiscales
- [ ] Widget: actividad reciente (últimas 5 acciones)
- [ ] Estado vacío para cliente recién invitado (sin datos aún)

### Document Vault
- [ ] Página `/documents` — listado de carpetas por período fiscal
- [ ] Página `/documents/[folderId]` — contenido de una carpeta
- [ ] Componente `DocumentCard` — card de archivo con metadatos y acciones
- [ ] Componente `FileUploadZone` — drag & drop con validación de tipo y tamaño
- [ ] Componente `DocumentVault` — árbol de carpetas + listado de documentos
- [ ] Subida de archivos a Supabase Storage
- [ ] Generación de presigned URLs para visualización y descarga
- [ ] Organización automática por período fiscal al subir

### Request Center (cliente)
- [ ] Página `/requests` — listado de solicitudes activas e historial
- [ ] Página `/requests/[requestId]` — detalle de una solicitud
- [ ] Componente `RequestThread` — solicitud con hilo de comentarios y upload
- [ ] Vista de solicitud con contexto: qué se pide, por qué, para cuándo
- [ ] Subida de documento en respuesta a una solicitud
- [ ] Cambio de estado al responder: `pending` → `submitted`
- [ ] Notificación al contador cuando el cliente responde

### Filings Tracker (cliente)
- [ ] Página `/filings` — listado de declaraciones del cliente
- [ ] Página `/filings/[filingId]` — detalle y timeline de una declaración
- [ ] Componente `FilingTimeline` — progreso visual a través de etapas
- [ ] Componente `DeclarationRow` — fila de declaración con estado y metadatos
- [ ] Vista de documentos adjuntos por etapa (acuse, comprobante)

### Activity Feed (cliente)
- [ ] Página `/activity` — historial completo del cliente
- [ ] Componente `ActivityFeed` — lista de eventos con filtros
- [ ] Filtros: por tipo de evento, por período de tiempo
- [ ] Actualización en tiempo real vía Supabase Realtime

---

## Fase 4 — Workspace Interno (Contador)

### Vista de cartera
- [ ] Página `/clients` — listado de todos los clientes del despacho
- [ ] Componente `ClientCard` — card de cliente con estado de la relación
- [ ] Filtros: por estado, por contador asignado, por declaraciones pendientes
- [ ] Búsqueda de clientes por nombre o RFC

### Perfil del cliente (vista interna)
- [ ] Página `/clients/[clientId]` — vista completa del cliente
- [ ] Tab: Documentos (vault del cliente)
- [ ] Tab: Declaraciones (listado + crear nueva)
- [ ] Tab: Solicitudes (historial + crear nueva)
- [ ] Tab: Actividad (audit log del cliente)
- [ ] Componente `ClientHeader` — nombre, RFC, régimen fiscal, contador asignado

### Gestión de solicitudes (contador)
- [ ] Formulario de creación de solicitud: tipo de documento, plazo, mensaje
- [ ] Vista de todas las solicitudes pendientes cross-cliente (cola)
- [ ] Aprobar o rechazar la respuesta del cliente con comentario
- [ ] Historial de solicitudes por cliente

### Gestión de declaraciones (contador)
- [ ] Formulario de creación de declaración: tipo, período, cliente
- [ ] Actualización de etapa: selector de estado con confirmación
- [ ] Adjuntar documentos por etapa (acuse, comprobante de pago)
- [ ] Notificación automática al cliente al cambiar etapa

### Cola de trabajo
- [ ] Página `/queue` — tareas pendientes cross-cliente ordenadas por urgencia
- [ ] Vista: solicitudes sin respuesta del cliente
- [ ] Vista: declaraciones próximas a vencer
- [ ] Vista: documentos esperando aprobación del contador

---

## Fase 5 — Notificaciones

### Sistema de notificaciones in-app
- [ ] Tabla `notifications` con lectura/no lectura
- [ ] Badge de notificaciones en el TopBar con contador
- [ ] Panel de notificaciones (dropdown)
- [ ] Marcar como leída individual y masivamente
- [ ] Eventos que generan notificación:
  - Nueva solicitud del contador → notifica al cliente
  - Documento subido por el cliente → notifica al contador
  - Solicitud aprobada/rechazada → notifica al cliente
  - Etapa de declaración actualizada → notifica al cliente
  - Fecha límite fiscal próxima → notifica a ambos

### Realtime
- [ ] Suscripción a nuevas notificaciones en tiempo real (Supabase Realtime)
- [ ] Actualización del activity feed en tiempo real

### Notificaciones por email
- [ ] Configurar servicio de email (Resend o SendGrid)
- [ ] Template: invitación al portal
- [ ] Template: nueva solicitud de documento
- [ ] Template: declaración actualizada
- [ ] Template: recordatorio de fecha límite

---

## Fase 6 — Admin Console

### Gestión de clientes
- [ ] Página `/admin/clients` — listado completo de clientes del despacho
- [ ] Crear nuevo cliente y asignar contador
- [ ] Editar datos del cliente (RFC, régimen, contacto)
- [ ] Archivar cliente (soft delete)
- [ ] Reasignar cliente a otro contador

### Gestión de equipo
- [ ] Página `/admin/team` — listado de staff del despacho
- [ ] Invitar nuevo contador/staff por email
- [ ] Cambiar rol de usuario (staff ↔ admin)
- [ ] Desactivar acceso de un usuario

### Configuración del workspace
- [ ] Página `/admin/settings` — configuración del despacho
- [ ] Subir logo del despacho
- [ ] Editar nombre, RFC y datos de contacto
- [ ] Configurar colores primarios del portal (brand)

---

## Fase 7 — Calendario Fiscal y Workers

### Catálogo de fechas fiscales
- [ ] Poblar tabla `fiscal_deadlines` con fechas SAT/IMSS para el año en curso
- [ ] Componente `FiscalCalendar` — calendario visual con obligaciones marcadas
- [ ] Página `/calendar` — vista del contador con fechas de todos sus clientes

### Workers de recordatorios
- [ ] Worker APScheduler: recordatorio 7 días antes de fecha límite
- [ ] Worker APScheduler: recordatorio 3 días antes de fecha límite
- [ ] Worker APScheduler: alerta el día de vencimiento
- [ ] Configuración de qué recordatorios recibe cada cliente

---

## Fase 8 — Calidad y Deploy

### Testing
- [ ] Tests de integración para endpoints FastAPI críticos (filings, requests)
- [ ] Tests de RLS: verificar aislamiento entre organizaciones
- [ ] Tests E2E con Playwright: flujo de invitación y primer upload
- [ ] Tests E2E: flujo de solicitud → respuesta → aprobación

### Deploy
- [ ] Configurar proyecto en Vercel (o Render) para Next.js
- [ ] Configurar servicio en Render para FastAPI con Docker
- [ ] Configurar variables de entorno en producción
- [ ] Configurar proyecto Supabase de staging y producción por separado
- [ ] Pipeline CI/CD: lint + typecheck + tests en cada PR
- [ ] Health check endpoint en FastAPI (`/health`)

### Performance y UX final
- [ ] Revisar Lighthouse score en páginas principales
- [ ] Optimizar imágenes con `next/image`
- [ ] Verificar que todas las páginas tienen estados de carga y error
- [ ] Verificar estados vacíos en todas las listas
- [ ] Accesibilidad: roles ARIA en componentes interactivos
- [ ] Responsive: verificar en mobile (375px) y tablet (768px)

---

## Backlog (Sin Fase Asignada)

Estas tareas están identificadas pero no entran en el MVP:

- [ ] Búsqueda global cross-módulos
- [ ] Exportar historial de actividad a PDF
- [ ] Versionado de documentos con diff visual
- [ ] Integración SAT web services
- [ ] Firma electrónica con e.firma
- [ ] Módulo de billing con Stripe
- [ ] White-labeling para SaaS (dominio personalizado, colores por tenant)
- [ ] API pública con documentación Swagger
- [ ] Extracción de datos de documentos con IA
- [ ] SSO / SAML para clientes enterprise
- [ ] App móvil (React Native o PWA avanzada)

---

## Notas de Desarrollo

- Empezar siempre por los tipos en `domain.ts` antes de construir la UI
- Al agregar una tabla: RLS primero, tipos generados segundo, UI tercero
- Las variables de entorno nuevas van siempre al `.env.example`
- No marcar una tarea como completa si no tiene los estados vacío y de error implementados
