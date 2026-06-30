# CLAUDE.md — Reglas para FiscalOS

Este archivo define las reglas que Claude Code debe seguir siempre en este proyecto. Leer antes de cualquier acción.

---

## Contexto del proyecto

**FiscalOS** es el Sistema Operativo del Despacho Fiscal Moderno. No es un portal para clientes — es la infraestructura de operaciones que conecta al despacho con sus clientes, organiza el trabajo del equipo y garantiza el cumplimiento de obligaciones fiscales.

El concepto central del producto es **Peace of Mind First**: cuando el cliente entra, la primera respuesta del sistema es si está al corriente.

**Repositorios:**
- `fiscal-portal-web` — Next.js (App Router)
- `fiscal-portal-api` — FastAPI (Python, Docker)

**Documentación completa:**
- `VISION.md` — Posicionamiento, filosofía y principios
- `DOMAIN.md` — El corazón del proyecto. Leer antes de crear cualquier entidad.
- `PRODUCT.md` — Usuarios, dominios, MVP y roadmap
- `ARCHITECTURE.md` — Stack, estructura de carpetas, patrones técnicos
- `SECURITY.md` — Estrategia completa de seguridad
- `DESIGN_SYSTEM.md` — Filosofía visual, tokens, componentes
- `TASKS.md` — Backlog por sprints de dominio
- `ADR/` — Razonamiento detrás de cada decisión de stack

---

## Stack y versiones

- **Next.js 14+** con App Router — no Pages Router
- **TypeScript** en modo estricto — sin `any`, sin `as unknown`
- **Tailwind CSS** — sin CSS modules, sin styled-components
- **shadcn/ui** — usar los componentes existentes antes de crear nuevos
- **Supabase** — Auth, Database (PostgreSQL), Storage, Realtime
- **FastAPI** — solo para escrituras con side effects, no CRUD simple
- **Zod** — validación en formularios y respuestas de API, sin excepciones
- **react-hook-form** — todos los formularios, siempre con `zodResolver`
- **Zustand** — solo para estado de UI, nunca para datos del servidor

---

## Reglas de código

### General
- TypeScript estricto siempre. Tipar explícitamente las respuestas de Supabase y FastAPI.
- Nunca usar `any`. Si el tipo no existe, definirlo en `domains/{dominio}/types.ts`.
- Preferir editar archivos existentes a crear nuevos.
- No agregar features, abstracciones ni refactors que el task no requiera.
- Sin comentarios que expliquen qué hace el código. Solo comentar el WHY cuando no es obvio.
- Sin emojis en el código ni en archivos generados.

### Domain-Driven Design
- Antes de crear cualquier tipo, componente o servicio: consultar `DOMAIN.md`.
- Si un concepto no está en `DOMAIN.md`, no existe en el sistema. Agregarlo primero al documento.
- La lógica de dominio vive en `src/domains/{dominio}/` — no en `components/` ni en `app/`.
- Los componentes en `app/` son de routing únicamente — no tienen lógica de negocio.
- Los componentes de dominio viven en `src/domains/{dominio}/components/`.

### Estructura de dominios
Cada carpeta en `src/domains/` sigue esta estructura:
```
domains/{dominio}/
├── types.ts          # Tipos TypeScript del dominio
├── schemas.ts        # Schemas Zod para validación
├── components/       # Componentes React específicos del dominio
├── hooks/            # Hooks específicos del dominio
└── services.ts       # Funciones de acceso a datos del dominio
```

### Componentes React
- Server Components por default. `"use client"` solo cuando sea necesario.
- No usar `useState` para filtros, modals o tabs — van en la URL como search params.
- Props tipadas siempre con interfaces o types.
- Nombres de componentes en PascalCase. Nombres de archivos en kebab-case.

### Supabase
- Nunca usar `service_role_key` en el cliente del browser.
- El cliente del browser usa `NEXT_PUBLIC_SUPABASE_ANON_KEY`.
- El cliente del servidor (RSC, API routes) puede usar `SUPABASE_SERVICE_ROLE_KEY`.
- Nunca exponer rutas permanentes de Storage — siempre presigned URLs.
- RLS habilitado en todas las tablas nuevas, sin excepciones.

### FastAPI
- Solo para operaciones con side effects: notificaciones, audit log, cambios en cascada.
- No duplicar en FastAPI lo que ya hace Supabase con RLS.
- Verificar el JWT de Supabase en cada endpoint como dependencia global.
- Todo request y response usa schemas Pydantic.
- Toda escritura llama a `activity_service.log()` antes de retornar.

### Seguridad
- Nunca hardcodear secrets, URLs ni API keys en el código.
- Todas las variables sensibles en `.env` — y en `.env.example` sin valores.
- No confiar en filtros del frontend como única capa de seguridad — RLS es la fuente de verdad.
- Validar siempre el `organization_id` del JWT en operaciones del servidor.
- Ver `SECURITY.md` antes de implementar cualquier flujo de autenticación o autorización.

---

## Rutas del App Router

```
(auth)/       → Login, invitaciones, onboarding
(portal)/     → Contexto del cliente: inicio, documentos, declaraciones, perfil
(workspace)/  → Contexto del staff/admin: cartera, [clientId], cola, calendario
(admin)/      → Solo admin: clientes, equipo, configuración
```

**Regla:** La redirección por rol ocurre en `middleware.ts` — no en layouts ni en componentes.

---

## Navegación del cliente: exactamente 4 secciones

El portal del cliente tiene exactamente estas 4 secciones en la navegación principal:

1. **Inicio** — Peace of Mind Status + pendientes
2. **Documentos** — Vault + solicitudes integradas (no hay página separada de solicitudes)
3. **Declaraciones** — Filings con timeline
4. **Perfil** — Datos del usuario + historial de actividad

No crear una quinta sección sin aprobar el cambio en `PRODUCT.md` primero.

---

## Organización de componentes

```
src/domains/{dominio}/components/   → Componentes con lógica de negocio
src/components/ui/                  → shadcn/ui. No modificar directamente.
src/components/layout/              → AppShell, sidebars, topbar.
```

Antes de crear un componente en `domains/`, verificar si shadcn/ui ya lo tiene.
Los componentes de shadcn/ui se extienden en `domains/`, no se modifican en `ui/`.

---

## Estado

- **Datos del servidor:** RSC + SWR o React Query para mutaciones
- **Estado de UI:** Zustand en `src/stores/ui.store.ts` (modals abiertos, sidebars)
- **Estado de URL:** `useSearchParams` + `useRouter` para filtros, tabs, items
- **No usar Context API** para datos — solo para configuración estática (tema, locale)

---

## Nomenclatura

| Elemento | Convención | Ejemplo |
|----------|-----------|---------|
| Archivos de componentes | kebab-case | `filing-timeline.tsx` |
| Componentes React | PascalCase | `FilingTimeline` |
| Hooks | camelCase con prefijo `use` | `useCurrentUser` |
| Stores | camelCase con sufijo `.store` | `ui.store.ts` |
| Tipos e interfaces | PascalCase | `Filing`, `DocumentRequest` |
| Variables y funciones | camelCase | `getSignedUrl` |
| Constantes globales | UPPER_SNAKE_CASE | `MAX_FILE_SIZE_MB` |
| Rutas de API (FastAPI) | kebab-case | `/api/filings/update-stage` |
| Tablas de base de datos | snake_case | `activity_log`, `filing_stages` |
| Carpetas de dominio | kebab-case | `document-request/` |

---

## Lo que NO hacer

- No crear páginas de "Actividad", "Solicitudes" o "Notificaciones" separadas en el portal del cliente — están integradas en las 4 secciones
- No modificar `src/components/ui/` — extender en `domains/`
- No usar `console.log` en código de producción
- No crear abstracciones "por si acaso se necesitan después"
- No instalar librerías sin verificar si hay una alternativa en el stack actual
- No hacer commits con `--no-verify`
- No crear variables de entorno sin agregarlas a `.env.example`
- No escribir en `activity_log` desde el frontend — solo FastAPI con service role
- No usar el mismo proyecto Supabase para development y production

---

## Flujo de trabajo

1. Consultar `TASKS.md` para entender el sprint activo antes de empezar
2. Consultar `DOMAIN.md` antes de crear cualquier entidad de negocio nueva
3. Al crear una entidad nueva: primero el tipo en `domains/{dominio}/types.ts`, luego el schema Zod, luego la UI
4. Al agregar una tabla nueva: RLS primero, policies segundo, tipos generados tercero, UI cuarto
5. Al crear un endpoint nuevo en FastAPI: schema Pydantic primero, lógica de negocio segundo, activity log tercero
6. Marcar la tarea como completada en `TASKS.md` tan pronto como esté hecha
7. Una tarea no está completa hasta que tenga estados de carga, vacío y error implementados
