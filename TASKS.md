# TASKS — FiscalOS Backlog

Cada sprint construye una **capacidad de negocio**, no una pantalla.
El orden de los sprints refleja dependencias del dominio: no se construye la UI de un Filing hasta que el dominio de Filing esté definido en la base de datos.

Estado: `[ ]` pendiente · `[~]` en progreso · `[x]` completado · `[-]` cancelado

---

## Sprint 0 — Foundation

**Objetivo:** El proyecto existe, corre localmente y tiene su base de datos versionada.

### Repositorios
- [ ] Crear repositorio `fiscal-portal-web` (Next.js)
- [ ] Crear repositorio `fiscal-portal-api` (FastAPI)
- [ ] Configurar `.gitignore` y `.env.example` en ambos repos
- [ ] Inicializar Next.js 14 con App Router y TypeScript estricto
- [ ] Configurar Tailwind CSS
- [ ] Instalar y configurar shadcn/ui
- [ ] Configurar ESLint + Prettier con las reglas del proyecto
- [ ] Inicializar FastAPI con la estructura de `ARCHITECTURE.md`
- [ ] Crear `Dockerfile` y `docker-compose.yml` para FastAPI
- [ ] Configurar Supabase CLI y proyecto local con Docker

### Base de datos — Esquema inicial
- [ ] Crear migración: `organizations`, `organization_settings`
- [ ] Crear migración: `users`, `user_invitations`
- [ ] Crear migración: `clients`, `client_contacts`
- [ ] Crear migración: `fiscal_periods`, `fiscal_deadlines`
- [ ] Crear migración: `documents`, `document_folders`, `document_versions`
- [ ] Crear migración: `document_requests`, `request_comments`
- [ ] Crear migración: `filing_types`, `filings`, `filing_stages`, `filing_documents`
- [ ] Crear migración: `activity_log` (solo INSERT en RLS)
- [ ] Crear migración: `notifications`
- [ ] Habilitar RLS en todas las tablas
- [ ] Definir políticas RLS base por rol (client / staff / admin)
- [ ] Generar tipos TypeScript: `supabase gen types typescript`
- [ ] Configurar Alembic para migraciones en FastAPI

### Verificación
- [ ] `docker-compose up` levanta FastAPI y Supabase local sin errores
- [ ] `next dev` corre sin errores de TypeScript
- [ ] Los tipos de Supabase se generan correctamente desde el esquema
- [ ] Un test de RLS verifica que un `client` no puede ver datos de otro cliente

**Definición de done:** El stack completo corre localmente. La base de datos tiene todas las tablas con RLS habilitado.

---

## Sprint 1 — Dominio Auth

**Objetivo:** Un usuario puede recibir una invitación, aceptarla y entrar al sistema con el rol correcto.

### Backend (FastAPI)
- [ ] Endpoint `POST /auth/invite` — crear invitación con token y rol
- [ ] Endpoint `POST /auth/accept-invite` — aceptar invitación: crear user + vincularlo a org
- [ ] Lógica de expiración de tokens (72 horas)
- [ ] Activity log: registrar `user.invited` y `user.joined`

### Frontend (Next.js)
- [ ] Página `/login` — formulario de email para magic link
- [ ] Página `/invite/[token]` — formulario de aceptación de invitación
- [ ] `middleware.ts` — verificar autenticación en todas las rutas protegidas
- [ ] `middleware.ts` — redirección por rol: `client` → `/inicio`, `staff` → `/cartera`, `admin` → `/cartera`
- [ ] `src/lib/supabase/client.ts` — cliente browser (anon key)
- [ ] `src/lib/supabase/server.ts` — cliente servidor (service role)
- [ ] `src/domains/user/hooks/use-current-user.ts`

### Onboarding del despacho
- [ ] Página `/onboarding` — wizard de configuración inicial (solo para el primer admin)
- [ ] Paso 1: Nombre del despacho, RFC, logo
- [ ] Paso 2: Invitar al primer cliente
- [ ] Guardar en `organizations` y `organization_settings`

**Definición de done:** El primer admin puede configurar el despacho. Un cliente puede recibir un email de invitación, hacer click y acceder al portal con su rol. Un staff puede entrar al workspace.

---

## Sprint 2 — Dominio Organization

**Objetivo:** El despacho tiene identidad, el workspace tiene estructura y la navegación está en pie.

### Frontend (Next.js)
- [ ] Componente `AppShell` — layout base con sidebar y topbar
- [ ] Componente `PortalSidebar` — navegación del cliente: Inicio, Documentos, Declaraciones, Perfil
- [ ] Componente `WorkspaceSidebar` — navegación del staff: Cartera, Cola, Calendario
- [ ] Componente `TopBar` — logo del despacho, usuario activo, notificaciones, logout
- [ ] Layouts de App Router: `(portal)/layout.tsx`, `(workspace)/layout.tsx`, `(admin)/layout.tsx`
- [ ] Componente `PageHeader` — título consistente con breadcrumb y acciones
- [ ] Hook `use-organization.ts` — acceso a datos de la Organization del usuario activo
- [ ] Consola admin — página de configuración del despacho (nombre, RFC, logo, colores)
- [ ] Subida del logo del despacho a Supabase Storage

### Componentes UI base
- [ ] `StatusBadge` — pill de colores para estados del sistema (ver `DESIGN_SYSTEM.md`)
- [ ] `EmptyState` — estado vacío con título, descripción y CTA
- [ ] `ConfirmDialog` — diálogo para acciones destructivas
- [ ] `SectionCard` — card container con header/content/footer
- [ ] `DeadlineCountdown` — fecha con urgencia visual verde/amarillo/rojo

**Definición de done:** El shell completo de la aplicación está en pie. Cada rol aterrriza en su contexto correcto. La navegación corresponde exactamente a las 4 secciones del cliente y el workspace del staff.

---

## Sprint 3 — Dominio User & Permission

**Objetivo:** El despacho puede gestionar su equipo y sus clientes con los roles correctos.

### Backend (FastAPI)
- [ ] Endpoint `POST /users/invite-staff` — invitar nuevo contador o admin
- [ ] Endpoint `PATCH /users/{id}/role` — cambiar rol de un usuario
- [ ] Endpoint `PATCH /users/{id}/deactivate` — desactivar acceso
- [ ] Validación: un admin no puede degradar su propio rol
- [ ] Activity log para todos los cambios de usuario

### Frontend (Next.js)
- [ ] Consola admin — página de gestión de equipo
- [ ] Lista de usuarios activos del despacho con su rol
- [ ] Formulario de invitación de nuevo staff
- [ ] Cambio de rol con confirmación
- [ ] Desactivar usuario con confirmación y advertencia de consecuencias
- [ ] Vista del perfil propio del usuario (en todas las rutas: `/perfil`, `/workspace/perfil`)

### Permisos y acceso
- [ ] Verificar que las políticas RLS cubren la matriz de permisos de `DOMAIN.md`
- [ ] Tests de aislamiento: staff de org A no puede ver datos de org B
- [ ] Tests de rol: client no puede acceder a rutas de workspace

**Definición de done:** El admin puede invitar staff, cambiar roles y desactivar usuarios. Los permisos de RLS cubren todos los casos de la matriz definida en `DOMAIN.md`.

---

## Sprint 4 — Dominio Document

**Objetivo:** Los clientes pueden subir documentos. El staff puede revisarlos. Los archivos son accesibles de forma segura.

### Backend (FastAPI)
- [ ] Endpoint `POST /documents/approve` — aprobar documento + activity log + notificación
- [ ] Endpoint `POST /documents/reject` — rechazar documento + activity log + notificación
- [ ] Endpoint `GET /documents/{id}/signed-url` — generar presigned URL (verificar permisos antes de firmar)
- [ ] Validación de tipos MIME permitidos y tamaño máximo (25MB)

### Frontend (Next.js — portal del cliente)
- [ ] Página `(portal)/documentos` — vault organizado por período fiscal
- [ ] Página `(portal)/documentos/[periodoId]` — documentos de un período
- [ ] Componente `DocumentVault` — árbol de períodos + listado de documentos
- [ ] Componente `DocumentCard` — archivo con nombre, tipo, tamaño, estado, fecha
- [ ] Componente `FileUploadZone` — drag & drop con validación de tipo y tamaño
- [ ] Subida de archivos a Supabase Storage
- [ ] Acceso a documentos vía presigned URL (descarga y preview)
- [ ] Estado vacío para períodos sin documentos

### Frontend (Next.js — workspace del staff)
- [ ] Tab Documentos dentro del perfil del cliente
- [ ] Aprobar y rechazar documentos con comentario
- [ ] Organización de documentos por período en la vista del staff

**Definición de done:** Un cliente puede subir un documento con drag & drop. Un contador puede verlo, aprobarlo o rechazarlo. Los archivos son accesibles solo vía presigned URLs con expiración.

---

## Sprint 5 — Dominio DocumentRequest

**Objetivo:** El contador puede solicitar documentos específicos. El cliente recibe una tarea clara y puede responderla. Nada se pide por WhatsApp.

### Backend (FastAPI)
- [ ] Endpoint `POST /document-requests` — crear solicitud + activity log + notificación al cliente
- [ ] Endpoint `POST /document-requests/{id}/comments` — agregar comentario
- [ ] Endpoint `POST /document-requests/{id}/submit` — cliente responde con documento
- [ ] Endpoint `POST /document-requests/{id}/approve` — staff aprueba
- [ ] Endpoint `POST /document-requests/{id}/reject` — staff rechaza + notificación

### Frontend (Next.js — portal del cliente)
- [ ] Solicitudes pendientes integradas al inicio de `(portal)/documentos`
- [ ] Indicador de urgencia cuando hay solicitudes vencidas o próximas a vencer
- [ ] Componente `RequestThread` — solicitud con contexto, hilo de comentarios y upload
- [ ] Cambio de estado al responder: `pending` → `submitted`
- [ ] Estado vacío: "No tienes documentos pendientes de entrega"

### Frontend (Next.js — workspace del staff)
- [ ] Formulario de creación de solicitud: tipo, descripción, fecha límite, cliente
- [ ] Tab Solicitudes dentro del perfil del cliente
- [ ] Lista de solicitudes con filtro por estado
- [ ] Aprobar o rechazar respuesta del cliente con comentario

**Definición de done:** El flujo completo solicitud → respuesta → aprobación funciona. Las solicitudes aparecen integradas en la vista de Documentos del cliente, no en una página separada.

---

## Sprint 6 — Dominio Filing

**Objetivo:** El contador puede gestionar el ciclo de vida completo de una declaración. El cliente puede ver su estado en todo momento. El Peace of Mind Status refleja el estado real de los filings.

### Backend (FastAPI)
- [ ] Poblar catálogo `filing_types` (IVA mensual, ISR, IMSS, DIOT, anual, etc.)
- [ ] Endpoint `POST /filings` — crear filing + etapa inicial + activity log
- [ ] Endpoint `PATCH /filings/{id}/stage` — avanzar etapa + activity log + notificación al cliente
- [ ] Endpoint `POST /filings/{id}/documents` — adjuntar documento a etapa

### Frontend (Next.js — portal del cliente)
- [ ] Componente `PeaceOfMindStatus` — el indicador hero del cliente (ver `PRODUCT.md`)
- [ ] Página `(portal)/inicio` — estado de salud + pendientes + próximas fechas
- [ ] Página `(portal)/declaraciones` — lista de filings del cliente
- [ ] Página `(portal)/declaraciones/[filingId]` — timeline completo del filing
- [ ] Componente `FilingTimeline` — progreso visual a través de etapas
- [ ] Componente `FilingRow` — fila de declaración con estado y fecha límite

### Frontend (Next.js — workspace del staff)
- [ ] Formulario de creación de filing: tipo, período, cliente, fecha límite
- [ ] Selector de avance de etapa con confirmación
- [ ] Adjuntar documentos a etapas (acuse, comprobante de pago)
- [ ] Tab Declaraciones dentro del perfil del cliente

### Lógica del Peace of Mind Status
- [ ] Calcular el estado de salud del cliente basado en sus filings y requests activos
- [ ] Al corriente: sin filings en `collecting_documents` vencidos, sin requests `pending` vencidos
- [ ] Pendiente: hay items activos pero ninguno vencido
- [ ] Urgente: hay items vencidos o que vencen en ≤ 48 horas

**Definición de done:** El contador puede crear y actualizar declaraciones. El cliente ve el estado en tiempo real. El Peace of Mind Status en la página de Inicio refleja el estado correcto.

---

## Sprint 7 — Dominio Notification

**Objetivo:** Los usuarios son informados de cambios relevantes sin necesidad de revisar el sistema constantemente.

### Backend (FastAPI)
- [ ] Refactorizar todos los endpoints anteriores para usar un `NotificationService` centralizado
- [ ] Worker APScheduler: recordatorio 7 días antes de fecha límite fiscal
- [ ] Worker APScheduler: recordatorio 3 días antes de fecha límite fiscal
- [ ] Worker APScheduler: alerta el día del vencimiento
- [ ] Poblar `fiscal_deadlines` con fechas del SAT e IMSS para el año en curso
- [ ] Endpoint para configurar preferencias de notificación del usuario

### Frontend (Next.js — in-app)
- [ ] Badge de notificaciones en TopBar con contador de no leídas
- [ ] Panel de notificaciones (dropdown con lista)
- [ ] Marcar como leída individual y "marcar todas como leídas"
- [ ] Suscripción a nuevas notificaciones en tiempo real (Supabase Realtime)
- [ ] Actualización del Peace of Mind Status en tiempo real

### Email
- [ ] Configurar servicio de email (Resend)
- [ ] Template: email de invitación al portal
- [ ] Template: nueva solicitud de documento
- [ ] Template: declaración actualizada (cambio de etapa)
- [ ] Template: recordatorio de fecha límite

### Calendario fiscal (workspace)
- [ ] Componente `FiscalCalendar` — calendario del mes con obligaciones marcadas
- [ ] Página `(workspace)/calendario` — vista del staff con fechas de todos sus clientes

**Definición de done:** Los eventos críticos generan notificaciones in-app y por email. Los recordatorios de fechas límite se envían automáticamente. El Peace of Mind Status se actualiza en tiempo real.

---

## Sprint 8 — Deployment & Quality

**Objetivo:** El sistema está en producción, es estable y el equipo puede iterar con confianza.

### Tests
- [ ] Tests de integración FastAPI: filings, document requests, invitaciones
- [ ] Tests de RLS: aislamiento entre organizaciones (cliente A no ve datos de cliente B)
- [ ] Tests de roles: client no puede acceder a endpoints de staff
- [ ] Tests E2E con Playwright: flujo de invitación → primer upload
- [ ] Tests E2E: flujo de solicitud → respuesta → aprobación
- [ ] Tests E2E: flujo de creación de filing → avance de etapa → notificación

### CI/CD
- [ ] Pipeline GitHub Actions: lint + typecheck en cada PR
- [ ] Pipeline GitHub Actions: tests en cada PR
- [ ] Pipeline GitHub Actions: build de Docker image en cada PR
- [ ] Deploy automático a staging desde rama `main`
- [ ] Deploy manual a producción con aprobación explícita

### Deploy
- [ ] Configurar proyecto en Vercel (o Render) para Next.js
- [ ] Configurar servicio en Render para FastAPI con Docker
- [ ] Configurar proyectos Supabase separados: staging y production
- [ ] Configurar variables de entorno en producción
- [ ] Health check endpoint en FastAPI (`GET /health`)
- [ ] Configurar dominio de email personalizado para magic links (evitar spam)
- [ ] Verificar que backups automáticos de Supabase están activos

### Calidad y performance
- [ ] Lighthouse score > 90 en páginas principales del portal del cliente
- [ ] Verificar estados de carga (skeleton) en todas las listas
- [ ] Verificar estados vacíos en todas las listas
- [ ] Verificar estados de error con recuperación
- [ ] Responsive: verificar portal del cliente en mobile (375px)
- [ ] Responsive: verificar workspace en tablet (768px)
- [ ] Accesibilidad: navegación por teclado en flujos principales
- [ ] Headers de seguridad HTTP configurados en Next.js

**Definición de done:** El sistema está en producción. Los tests críticos pasan. El CI/CD bloquea deploys con errores. El despacho inicial puede operar con clientes reales.

---

## Backlog — Fuera del MVP

Estas capacidades están identificadas pero no entran en la v1:

- [ ] Calendario fiscal con vistas mensuales detalladas por cliente
- [ ] Búsqueda global cross-dominios
- [ ] Exportar historial de actividad a PDF
- [ ] Versionado de documentos con diff visual
- [ ] Integración SAT web services
- [ ] Firma electrónica con e.firma
- [ ] Módulo de billing con Stripe
- [ ] White-labeling por despacho (dominio personalizado, colores)
- [ ] Self-service onboarding para nuevos despachos (SaaS)
- [ ] API pública con documentación OpenAPI
- [ ] Extracción de datos de documentos con IA
- [ ] Clasificación automática de documentos
- [ ] SSO / SAML para clientes enterprise
- [ ] App móvil nativa
- [ ] Passkeys como método de autenticación alternativo

---

## Notas de desarrollo

- El orden de implementación dentro de cada sprint es: esquema de DB → RLS → tipos → backend → frontend
- Una tarea no está completa hasta que tiene estados de carga, vacío y error
- Las variables de entorno nuevas van al `.env.example` antes del merge
- Al agregar una nueva entidad: consultar `DOMAIN.md` primero
- Al implementar seguridad: consultar `SECURITY.md` primero
- Al crear componentes: consultar `DESIGN_SYSTEM.md` primero
