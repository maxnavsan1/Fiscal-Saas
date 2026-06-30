# ARCHITECTURE — FiscalOS

## Stack Tecnológico

| Capa | Tecnología | Justificación |
|------|-----------|---------------|
| Frontend | Next.js 14+ (App Router) | RSC, file-based routing, middleware en edge |
| Lenguaje | TypeScript estricto | Seguridad de tipos end-to-end |
| Estilos | Tailwind CSS | Utility-first, consistente con el design system |
| Componentes | shadcn/ui | Accesible, sin lock-in, personalizable |
| Auth | Supabase Auth | Magic links, JWT, RLS integrado |
| Base de datos | PostgreSQL vía Supabase | RLS nativo, realtime, SQL estándar |
| Storage | Supabase Storage | Presigned URLs, integrado con RLS |
| Realtime | Supabase Realtime | Notificaciones y activity feed en vivo |
| Backend | FastAPI (Python) | Business logic con side effects, workers |
| Containerización | Docker | FastAPI deployable en cualquier ambiente |
| Deploy frontend | Vercel o Render | CI/CD automático desde Git |
| Deploy backend | Render | Docker support, managed deploys |
| Estado global UI | Zustand | Ligero, sin boilerplate innecesario |
| Formularios | react-hook-form + Zod | Validación declarativa, un schema como fuente de verdad |

---

## Topología del Sistema

```
Browser
  │
  ▼ HTTPS
Next.js App (Vercel/Render)
  │              │
  ▼              ▼
Supabase       FastAPI (Render + Docker)
  │              │
  ▼              ▼
PostgreSQL    PostgreSQL (mismo DB, distinto cliente)
  │
  ├── Supabase Auth
  ├── Supabase Storage
  └── Supabase Realtime
```

---

## Multi-tenancy

Cada despacho es un **Organization**. Es el tenant raíz del sistema.

Cada tabla del dominio tiene `organization_id`. Las políticas de Row-Level Security en PostgreSQL garantizan el aislamiento de datos — el filtrado no vive solo en el frontend.

```
Organization (Despacho)
  ├── Users (Staff + Clients)
  ├── Clients
  │     ├── Documents
  │     ├── Filings
  │     ├── Requests
  │     └── Activity Log
  └── Settings
```

**Regla absoluta:** Ninguna query llega a producción sin que RLS esté habilitado en la tabla correspondiente.

---

## Modelo de Datos (Entidades Principales)

```sql
organizations          -- El despacho fiscal (tenant)
users                  -- Todos los usuarios (con role: client | staff | admin)
clients                -- Clientes del despacho (vinculados a un user)
documents              -- Archivos subidos
document_folders       -- Organización por período fiscal
filing_types           -- Catálogo de tipos de declaración
filings                -- Declaraciones por cliente
filing_stages          -- Etapas por filing (timeline)
requests               -- Solicitudes de documentos del contador al cliente
request_comments       -- Hilo de comentarios por request
activity_log           -- Audit trail inmutable
notifications          -- Notificaciones in-app por usuario
fiscal_deadlines       -- Catálogo de fechas límite fiscales
```

Cada tabla incluye: `id` (UUID), `organization_id`, `created_at`, `updated_at`.

---

## Decisión de Split: Supabase vs. FastAPI

La regla es simple: **lecturas van a Supabase directo; escrituras con side effects van a FastAPI**.

| Operación | Supabase directo | FastAPI |
|-----------|-----------------|---------|
| Listar documentos de un cliente | ✓ | |
| Listar declaraciones activas | ✓ | |
| Subir un documento (Storage) | ✓ | |
| Leer activity feed | ✓ | |
| Crear una declaración nueva | | ✓ genera audit log + notificación |
| Aprobar un documento | | ✓ cambia estado + notifica + cierra request |
| Crear una solicitud de documento | | ✓ notifica al cliente + registra en audit log |
| Cambiar etapa de un filing | | ✓ dispara notificación + escribe en timeline |
| Enviar recordatorios de deadline | | ✓ worker programado |

FastAPI **no es un proxy de CRUD**. Existe para operaciones con consecuencias en múltiples entidades.

---

## Seguridad

### Row-Level Security (RLS)
- Habilitado en todas las tablas del dominio sin excepción
- Las políticas verifican `organization_id` y `user_id` contra el JWT de Supabase
- Un contador solo ve los clientes de su organización
- Un cliente solo ve sus propios documentos y declaraciones

### Acceso a archivos
- Supabase Storage con buckets privados
- Todo acceso a documentos genera una **presigned URL con expiración** (máximo 1 hora)
- Nunca se expone la ruta permanente del bucket en el frontend
- La generación de presigned URLs verifica permisos antes de firmar

### Autenticación
- Magic links como método principal (sin contraseñas)
- JWT de Supabase Auth verificado en el middleware de Next.js y en FastAPI
- El middleware de Next.js resuelve el rol del usuario y protege las rutas antes de que cargue cualquier componente
- FastAPI verifica el JWT en cada request como dependencia global

### Invitaciones
- Los clientes son invitados por el contador — no hay auto-registro
- El token de invitación tiene expiración (72 horas)
- Al aceptar la invitación, se crea el user, se establece el rol y se vincula al organization_id en una sola transacción

### Audit Log
- La tabla `activity_log` es inmutable: no hay UPDATE ni DELETE habilitados vía RLS
- Solo INSERT, realizado desde FastAPI con privilegios de service role
- Registra: `actor_id`, `action`, `resource_type`, `resource_id`, `metadata` (JSONB), `ip_address`, `created_at`

---

## Patrones de Frontend

### App Router + RSC como default
- Las páginas de listado y detalle se construyen como React Server Components
- Los componentes interactivos (modals, formularios, realtime) se marcan `"use client"` quirúrgicamente
- Los datos se cargan en el servidor — no hay loading spinners en la carga inicial de página

### URL como fuente de verdad del estado de UI
- Filtros, modals abiertos, tabs seleccionados y items expandidos viven en la URL como search params
- Las vistas son bookmarkeables y shareables
- Compatible con el botón atrás del browser

### Optimistic UI para mutaciones
- Las acciones del usuario se reflejan en la UI inmediatamente
- Si la operación falla, se revierte con un mensaje de error
- Implementado con SWR o React Query

### Validación con Zod en todas las capas
- Un schema de Zod por entidad de dominio
- El mismo schema valida el formulario en el frontend y la respuesta de la API
- react-hook-form + zodResolver sin excepciones

---

## Estructura del Proyecto

### `fiscal-portal-web/` (Next.js)

```
src/
├── app/
│   ├── (auth)/
│   │   ├── login/
│   │   ├── invite/[token]/
│   │   └── onboarding/
│   ├── (client)/                   # Portal del cliente
│   │   ├── layout.tsx
│   │   ├── dashboard/
│   │   ├── documents/
│   │   │   └── [folderId]/
│   │   ├── requests/
│   │   │   └── [requestId]/
│   │   ├── filings/
│   │   │   └── [filingId]/
│   │   └── activity/
│   ├── (internal)/                 # Workspace del contador
│   │   ├── layout.tsx
│   │   ├── clients/
│   │   │   └── [clientId]/
│   │   ├── queue/
│   │   ├── filings/
│   │   └── calendar/
│   ├── (admin)/                    # Consola de admin
│   │   ├── layout.tsx
│   │   ├── settings/
│   │   ├── team/
│   │   └── clients/
│   └── api/
│       └── webhooks/
├── components/
│   ├── ui/                         # shadcn/ui (no modificar directamente)
│   ├── domain/                     # Componentes de negocio
│   │   ├── documents/
│   │   ├── filings/
│   │   ├── requests/
│   │   └── activity/
│   └── layout/
│       ├── AppShell.tsx
│       ├── ClientSidebar.tsx
│       ├── InternalSidebar.tsx
│       └── TopBar.tsx
├── lib/
│   ├── supabase/
│   │   ├── client.ts
│   │   ├── server.ts
│   │   └── middleware.ts
│   ├── api/
│   │   └── client.ts               # FastAPI typed client
│   └── utils/
│       ├── dates.ts
│       └── formatters.ts
├── hooks/
│   ├── useRealtime.ts
│   ├── useCurrentUser.ts
│   └── useOrganization.ts
├── stores/
│   └── ui.store.ts                 # Solo estado de UI (Zustand)
└── types/
    ├── domain.ts
    └── supabase.ts                 # Generados por Supabase CLI
```

### `fiscal-portal-api/` (FastAPI)

```
app/
├── routers/
│   ├── filings.py
│   ├── documents.py
│   ├── requests.py
│   ├── clients.py
│   └── notifications.py
├── services/
│   ├── filing_service.py
│   ├── document_service.py
│   └── notification_service.py
├── models/                         # SQLAlchemy ORM models
├── schemas/                        # Pydantic schemas
├── db/
│   ├── session.py
│   └── migrations/                 # Alembic
├── workers/
│   └── deadline_reminders.py       # APScheduler jobs
└── core/
    ├── config.py
    ├── auth.py                     # Verificación JWT de Supabase
    └── dependencies.py

Dockerfile
docker-compose.yml
```

---

## Variables de Entorno

### Next.js
```
NEXT_PUBLIC_SUPABASE_URL
NEXT_PUBLIC_SUPABASE_ANON_KEY
SUPABASE_SERVICE_ROLE_KEY          # Solo en servidor, nunca en cliente
FASTAPI_BASE_URL
NEXT_PUBLIC_APP_URL
```

### FastAPI
```
SUPABASE_URL
SUPABASE_SERVICE_ROLE_KEY
DATABASE_URL
JWT_SECRET                         # Mismo que Supabase JWT secret
ENVIRONMENT                        # development | staging | production
```

---

## Deploy

| Servicio | Plataforma | Configuración |
|----------|-----------|---------------|
| Next.js | Vercel o Render | Deploy automático desde rama `main` |
| FastAPI | Render | Docker container, deploy desde `main` |
| PostgreSQL | Supabase | Managed, backups automáticos |
| Storage | Supabase | Buckets privados por organización |

**Entornos:**
- `development` — Supabase local (Docker), FastAPI local, Next.js local
- `staging` — Supabase proyecto separado, Render preview, datos de prueba
- `production` — Supabase producción, Render production services

---

## Decisiones que No Cambian en el MVP

1. RLS habilitado en todas las tablas — sin excepciones
2. Archivos siempre vía presigned URLs — nunca rutas directas
3. Clientes invitados por el despacho — sin auto-registro
4. Audit log inmutable — sin UPDATE ni DELETE
5. FastAPI solo para operaciones con side effects — no como proxy CRUD
6. Roles resueltos en middleware — no en componentes individuales
7. URL como estado de UI — no useState para filtros o modals
