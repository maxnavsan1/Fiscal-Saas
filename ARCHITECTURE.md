# ARCHITECTURE — FiscalOS

Este documento define las decisiones de arquitectura técnica del sistema. Para el modelo de seguridad completo, ver `SECURITY.md`. Para los conceptos de dominio, ver `DOMAIN.md`. Para el razonamiento detrás de cada decisión de stack, ver los ADRs en `ADR/`.

---

## Stack tecnológico

| Capa | Tecnología | ADR |
|------|-----------|-----|
| Frontend | Next.js 14+ (App Router) | — |
| Lenguaje | TypeScript estricto | — |
| Estilos | Tailwind CSS | — |
| Componentes | shadcn/ui | — |
| Auth | Supabase Auth | ADR-004 |
| Base de datos | PostgreSQL vía Supabase | ADR-002, ADR-003 |
| Storage | Supabase Storage | ADR-003 |
| Realtime | Supabase Realtime | ADR-003 |
| Backend | FastAPI (Python) | ADR-001 |
| Containerización | Docker | ADR-005 |
| Deploy frontend | Vercel o Render | — |
| Deploy backend | Render (Docker) | ADR-005 |
| Estado global UI | Zustand | — |
| Formularios | react-hook-form + Zod | — |

---

## Topología del sistema

```
                         Browser
                            │
                            ▼ HTTPS + TLS 1.3
                    Next.js App Router
                    (Vercel / Render)
                         │        │
              RSC reads  │        │  Side effects
                         ▼        ▼
                    Supabase    FastAPI
                    (Managed)   (Render + Docker)
                         │        │
                         └────────┘
                              │
                         PostgreSQL
                              │
                    ┌─────────┼──────────┐
                    │         │          │
               Supabase    Supabase   Supabase
                 Auth      Storage    Realtime
```

---

## Multi-tenancy

Cada despacho es una **Organization**. Es el tenant raíz del sistema.

Todas las tablas del dominio tienen `organization_id` como columna obligatoria. Las políticas de Row Level Security en PostgreSQL garantizan el aislamiento — el código de aplicación nunca es la única barrera.

```
Organization (Despacho)
  ├── Users (Staff + Clients)
  ├── Clients
  │     ├── Documents
  │     ├── Filings
  │     ├── DocumentRequests
  │     └── ActivityEvents
  └── OrganizationSettings
```

**Invariante:** Ninguna query de dominio llega a producción sin que RLS esté habilitado en su tabla. Ver `SECURITY.md` para las políticas detalladas.

---

## Modelo de datos

```sql
-- Dominio: Organization
organizations
organization_settings

-- Dominio: User / Role
users                          -- email, role, organization_id, status
user_invitations               -- token, email, role, expires_at

-- Dominio: Client
clients                        -- RFC, régimen, nombre, organization_id, assigned_to
client_contacts

-- Dominio: FiscalPeriod
fiscal_periods                 -- year, month, type (monthly/quarterly/annual)
fiscal_deadlines               -- tipo de obligación, fecha, regímenes aplicables

-- Dominio: Document
documents                      -- nombre, tipo, storage_path, estado, client_id, period_id
document_folders               -- período, client_id
document_versions              -- versiones anteriores de un documento

-- Dominio: DocumentRequest
document_requests              -- título, descripción, due_date, estado, client_id
request_comments               -- actor_id, contenido, request_id

-- Dominio: Filing
filing_types                   -- IVA mensual, ISR, IMSS, DIOT, anual, etc.
filings                        -- client_id, type_id, period_id, stage, due_date
filing_stages                  -- stage, actor_id, filing_id, timestamp, notas
filing_documents               -- document_id, filing_id, stage

-- Dominio: Activity
activity_log                   -- actor_id, action, resource_type, resource_id, metadata JSONB

-- Dominio: Notification
notifications                  -- user_id, type, title, body, read, channel
```

Cada tabla incluye: `id` (UUID), `organization_id`, `created_at`, `updated_at`.

---

## Decisión de split: Supabase vs. FastAPI

**Lecturas van a Supabase directo. Escrituras con side effects van a FastAPI.**

FastAPI no es un proxy de CRUD. Existe para operaciones que afectan múltiples entidades o que tienen consecuencias fuera de la base de datos.

| Operación | Supabase directo | FastAPI |
|-----------|:---------------:|:-------:|
| Listar documentos de un cliente | ✓ | |
| Listar declaraciones activas | ✓ | |
| Leer activity feed | ✓ | |
| Leer notificaciones del usuario | ✓ | |
| Marcar notificación como leída | ✓ | |
| Subir un documento (Storage) | ✓ | |
| Crear una declaración nueva | | ✓ |
| Avanzar etapa de una declaración | | ✓ |
| Crear una solicitud de documento | | ✓ |
| Aprobar o rechazar un documento | | ✓ |
| Invitar a un usuario | | ✓ |
| Aceptar una invitación | | ✓ |
| Enviar recordatorios de deadline | | ✓ (worker) |

Toda escritura vía FastAPI sigue el mismo patrón:
1. Verificar JWT y extraer `organization_id` del actor
2. Ejecutar la operación de negocio
3. Escribir en `activity_log` con el contexto completo
4. Emitir notificaciones a los usuarios relevantes
5. Retornar el resultado

---

## Estructura del proyecto

### Principio: Domain-Driven Organization

El código se organiza por dominio de negocio, no por tipo de archivo. Esto significa que la lógica de un dominio (tipos, componentes, hooks, servicios) vive junta, no dispersa por carpetas técnicas.

### `fiscal-portal-web/` (Next.js)

```
src/
│
├── app/                              # Next.js App Router (routing únicamente)
│   ├── (auth)/
│   │   ├── login/
│   │   ├── invite/[token]/
│   │   └── onboarding/
│   │
│   ├── (portal)/                     # Contexto del cliente
│   │   ├── layout.tsx
│   │   ├── inicio/                   # Peace of Mind Status + pendientes
│   │   ├── documentos/
│   │   │   └── [periodoId]/
│   │   ├── declaraciones/
│   │   │   └── [filingId]/
│   │   └── perfil/
│   │
│   ├── (workspace)/                  # Contexto del staff/admin
│   │   ├── layout.tsx
│   │   ├── cartera/                  # Lista de clientes
│   │   │   └── [clientId]/           # Vista completa del cliente
│   │   ├── cola/                     # Tareas pendientes cross-cliente
│   │   └── calendario/
│   │
│   ├── (admin)/                      # Contexto del admin únicamente
│   │   ├── layout.tsx
│   │   ├── clientes/
│   │   ├── equipo/
│   │   └── configuracion/
│   │
│   └── api/
│       └── webhooks/
│
├── domains/                          # Lógica de dominio
│   ├── organization/
│   │   ├── types.ts
│   │   ├── schemas.ts
│   │   └── services.ts
│   ├── client/
│   │   ├── types.ts
│   │   ├── schemas.ts
│   │   ├── components/
│   │   │   ├── client-card.tsx
│   │   │   ├── client-header.tsx
│   │   │   └── peace-of-mind-status.tsx
│   │   └── hooks/
│   │       └── use-client-status.ts
│   ├── document/
│   │   ├── types.ts
│   │   ├── schemas.ts
│   │   ├── components/
│   │   │   ├── document-card.tsx
│   │   │   ├── document-vault.tsx
│   │   │   └── file-upload-zone.tsx
│   │   └── hooks/
│   │       └── use-signed-url.ts
│   ├── document-request/
│   │   ├── types.ts
│   │   ├── schemas.ts
│   │   ├── components/
│   │   │   └── request-thread.tsx
│   │   └── hooks/
│   ├── filing/
│   │   ├── types.ts
│   │   ├── schemas.ts
│   │   ├── components/
│   │   │   ├── filing-timeline.tsx
│   │   │   └── filing-row.tsx
│   │   └── hooks/
│   ├── activity/
│   │   ├── types.ts
│   │   ├── components/
│   │   │   └── activity-feed.tsx
│   │   └── hooks/
│   │       └── use-realtime-activity.ts
│   ├── notification/
│   │   ├── types.ts
│   │   ├── components/
│   │   │   └── notification-panel.tsx
│   │   └── hooks/
│   │       └── use-notifications.ts
│   └── user/
│       ├── types.ts
│       ├── schemas.ts
│       └── hooks/
│           └── use-current-user.ts
│
├── components/
│   ├── ui/                           # shadcn/ui — no modificar directamente
│   └── layout/
│       ├── app-shell.tsx
│       ├── portal-sidebar.tsx
│       ├── workspace-sidebar.tsx
│       └── top-bar.tsx
│
├── lib/
│   ├── supabase/
│   │   ├── client.ts                 # Browser client (anon key)
│   │   ├── server.ts                 # Server client (service role)
│   │   └── middleware.ts             # Auth + redirección por rol
│   ├── api/
│   │   └── client.ts                 # FastAPI typed client (generado desde OpenAPI)
│   └── utils/
│       ├── fiscal-dates.ts           # Helpers para fechas fiscales
│       └── formatters.ts
│
├── stores/
│   └── ui.store.ts                   # Solo estado de UI (Zustand)
│
└── types/
    └── supabase.ts                   # Generados por Supabase CLI (no editar manualmente)
```

### `fiscal-portal-api/` (FastAPI)

```
app/
├── routers/
│   ├── filings.py
│   ├── documents.py
│   ├── document_requests.py
│   ├── clients.py
│   ├── users.py
│   └── notifications.py
│
├── services/                         # Business logic pura, sin dependencia de HTTP
│   ├── filing_service.py
│   ├── document_service.py
│   ├── request_service.py
│   ├── notification_service.py
│   └── activity_service.py           # Siempre escribe en activity_log
│
├── models/                           # SQLAlchemy ORM models
├── schemas/                          # Pydantic schemas (request / response)
│
├── db/
│   ├── session.py
│   └── migrations/                   # Alembic
│
├── workers/                          # APScheduler jobs
│   └── deadline_reminders.py
│
└── core/
    ├── config.py
    ├── auth.py                       # Verifica JWT de Supabase
    └── dependencies.py

Dockerfile
docker-compose.yml
.env.example
```

---

## Patrones de frontend

### React Server Components como default
Las páginas y listados se construyen con RSC — los datos se cargan en el servidor. Los componentes interactivos (formularios, realtime, modals) se marcan `"use client"` de forma quirúrgica. Sin loading spinners en la carga inicial.

### URL como estado de UI
Los filtros, tabs, modals y ítems expandidos viven en la URL como search params. Las vistas son bookmarkeables, shareables y compatibles con el botón atrás.

### Optimistic UI para mutaciones
Las acciones del usuario se reflejan en la UI inmediatamente. Si la operación falla, se revierte. Implementado con SWR o React Query.

### Validación con Zod en todas las capas
Un schema de Zod por entidad de dominio. El mismo schema valida el formulario en el frontend y la respuesta de la API. `react-hook-form + zodResolver` sin excepciones.

---

## Variables de entorno

### Next.js (`fiscal-portal-web/.env.local`)
```
NEXT_PUBLIC_SUPABASE_URL
NEXT_PUBLIC_SUPABASE_ANON_KEY
SUPABASE_SERVICE_ROLE_KEY          # Solo servidor, nunca cliente
FASTAPI_BASE_URL
NEXT_PUBLIC_APP_URL
```

### FastAPI (`fiscal-portal-api/.env`)
```
SUPABASE_URL
SUPABASE_SERVICE_ROLE_KEY
DATABASE_URL
JWT_SECRET                         # Mismo que Supabase JWT secret del proyecto
ENVIRONMENT                        # development | staging | production
EMAIL_SERVICE_API_KEY              # Resend o SendGrid
```

---

## Deploy

| Servicio | Plataforma | Trigger |
|----------|-----------|---------|
| Next.js | Vercel o Render | Push a `main` |
| FastAPI | Render (Docker) | Push a `main` |
| PostgreSQL | Supabase (managed) | — |
| Storage | Supabase (managed) | — |

### Entornos

| Entorno | Supabase | Backend | Frontend |
|---------|----------|---------|----------|
| `development` | Local (Docker CLI) | `uvicorn` local | `next dev` |
| `staging` | Proyecto Supabase staging | Render preview | Vercel preview |
| `production` | Proyecto Supabase producción | Render production | Vercel production |

**Los proyectos de Supabase de staging y producción son completamente separados. Nunca se apunta al proyecto de producción desde el entorno local.**

---

## Invariantes arquitectónicas

Estas decisiones no se renegocian en el MVP:

1. **RLS habilitado en todas las tablas** — sin excepciones, desde la primera migración
2. **Archivos siempre vía presigned URLs** — nunca rutas directas de Storage
3. **Clientes solo pueden ser invitados** — no hay auto-registro
4. **Activity log es inmutable** — solo INSERT, nunca UPDATE ni DELETE
5. **FastAPI solo para escrituras con side effects** — no como proxy de CRUD
6. **Roles resueltos en middleware** — no en componentes individuales de React
7. **URL como estado de UI** — no `useState` para filtros, tabs o modals
8. **JWT del actor en cada request de FastAPI** — nunca se confía en el cuerpo del request para identidad
9. **Los secretos viven en variables de entorno** — nunca en el código o en el repositorio
10. **Dominio primero, página después** — la lógica vive en `domains/`, las rutas en `app/`
