# CLAUDE.md — Reglas para FiscalOS

Este archivo define las reglas que Claude Code debe seguir siempre en este proyecto. Leerlo antes de cualquier acción.

---

## Contexto del Proyecto

**FiscalOS** es un portal de colaboración fiscal para despachos contables. Conecta a contadores con sus clientes para gestionar documentos, declaraciones y solicitudes.

- Arquitectura: Next.js (App Router) + FastAPI + Supabase + PostgreSQL
- Multi-tenant: cada despacho es una Organization; RLS aísla los datos
- Dos repositorios: `fiscal-portal-web` (Next.js) y `fiscal-portal-api` (FastAPI)
- Documentación completa: VISION.md, PRODUCT.md, ARCHITECTURE.md, TASKS.md

---

## Stack y Versiones

- **Next.js 14+** con App Router — no Pages Router
- **TypeScript** en modo estricto — sin `any`, sin `as unknown`
- **Tailwind CSS** — sin CSS modules, sin styled-components
- **shadcn/ui** — usar los componentes existentes antes de crear nuevos
- **Supabase** — Auth, Database (PostgreSQL), Storage, Realtime
- **FastAPI** — solo para operaciones con side effects, no CRUD simple
- **Zod** — validación en formularios y respuestas de API, sin excepciones
- **react-hook-form** — todos los formularios, siempre con zodResolver

---

## Reglas de Código

### General
- TypeScript estricto siempre. Tipar explícitamente las respuestas de Supabase y FastAPI.
- No usar `any`. Si el tipo no se conoce, definirlo en `src/types/domain.ts`.
- Preferir editar archivos existentes a crear nuevos.
- No agregar features, abstracciones ni refactors que el task no requiera.
- No agregar comentarios que expliquen qué hace el código. Solo comentar el WHY cuando no es obvio.
- Sin emojis en el código ni en los archivos generados.

### Componentes React
- Server Components por default. Agregar `"use client"` solo cuando sea necesario (formularios, hooks, realtime).
- No usar `useState` para filtros, modals o tabs — esos van en la URL como search params.
- Props tipadas siempre con interfaces o types, no con `any`.
- Nombres de componentes en PascalCase. Nombres de archivos en kebab-case.

### Supabase
- Nunca usar la `service_role_key` en el cliente del browser.
- El cliente del browser usa `NEXT_PUBLIC_SUPABASE_ANON_KEY`.
- El cliente del servidor (RSC, API routes) puede usar `SUPABASE_SERVICE_ROLE_KEY`.
- Nunca exponer rutas permanentes de Storage — siempre generar presigned URLs.
- RLS habilitado en todas las tablas nuevas, sin excepciones.

### FastAPI
- Solo para operaciones con side effects: notificaciones, audit log, cambios de estado en cascada.
- No duplicar en FastAPI lo que ya hace Supabase con RLS.
- Verificar el JWT de Supabase en cada endpoint como dependencia global.
- Usar Pydantic para todos los schemas de request y response.

### Seguridad
- Nunca hardcodear secrets, URLs de Supabase ni API keys en el código.
- Usar variables de entorno para toda configuración sensible.
- No confiar en filtros del frontend como única capa de seguridad — RLS es la fuente de verdad.
- Validar siempre el `organization_id` del usuario en operaciones del servidor.

---

## Estructura de Rutas (App Router)

```
(auth)/     → Login, invitaciones, onboarding
(client)/   → Portal del cliente: dashboard, documentos, solicitudes, declaraciones
(internal)/ → Workspace del contador: clientes, cola de tareas, calendario
(admin)/    → Consola del despacho: equipo, configuración, billing
```

La redirección por rol ocurre en `middleware.ts`, no en los layouts ni en los componentes.

---

## Organización de Componentes

```
src/components/ui/       → shadcn/ui base. No modificar directamente.
src/components/domain/   → Componentes de negocio con lógica acoplada.
src/components/layout/   → AppShell, sidebars, topbar.
```

Antes de crear un componente nuevo en `domain/`, verificar si shadcn/ui ya tiene uno.
Antes de crear un componente en `layout/`, verificar si el AppShell ya lo contempla.

---

## Formularios y Validación

- Todos los formularios usan `react-hook-form` con `zodResolver`.
- El schema de Zod vive en `src/types/domain.ts` o junto al componente si es específico.
- No usar `e.preventDefault()` manual ni manejar el submit sin react-hook-form.
- Los errores de validación se muestran con el componente `FormMessage` de shadcn/ui.

---

## Estado

- Estado del servidor: RSC + SWR o React Query para datos con mutaciones.
- Estado de UI (modals, sidebars abiertos): Zustand en `src/stores/ui.store.ts`.
- Estado de URL (filtros, tabs, items seleccionados): `useSearchParams` + `useRouter`.
- No usar Context API para datos — solo para configuración estática (tema, locale).

---

## Nomenclatura

| Elemento | Convención | Ejemplo |
|----------|-----------|---------|
| Archivos de componentes | kebab-case | `document-card.tsx` |
| Componentes React | PascalCase | `DocumentCard` |
| Hooks | camelCase con prefijo `use` | `useCurrentUser` |
| Stores | camelCase con sufijo `.store` | `ui.store.ts` |
| Tipos e interfaces | PascalCase | `Filing`, `DocumentRequest` |
| Variables y funciones | camelCase | `getSignedUrl` |
| Constantes globales | UPPER_SNAKE_CASE | `MAX_FILE_SIZE_MB` |
| Rutas de API (FastAPI) | kebab-case | `/api/filings/update-stage` |
| Tablas de base de datos | snake_case | `activity_log`, `filing_stages` |

---

## Lo que NO hacer

- No crear archivos de documentación (`.md`) a menos que se pidan explícitamente.
- No agregar manejo de errores para escenarios imposibles.
- No crear abstracciones "por si acaso se necesitan en el futuro".
- No modificar `src/components/ui/` — los componentes de shadcn/ui se extienden en `domain/`.
- No usar `console.log` en código de producción.
- No agregar comentarios que describan qué hace el código (los nombres deben ser autoexplicativos).
- No instalar librerías nuevas sin consultar si hay una alternativa ya en el stack.
- No hacer commits con `--no-verify`.
- No crear variables de entorno sin agregarlas al `.env.example`.

---

## Flujo de Trabajo

1. Revisar TASKS.md para entender qué está en progreso antes de empezar.
2. Marcar la tarea como completada en TASKS.md tan pronto como esté hecha.
3. Cada feature nueva empieza por el tipo en `domain.ts`, luego la UI, luego la conexión con Supabase/FastAPI.
4. Al agregar una tabla nueva: habilitar RLS, definir políticas, agregar el tipo en `supabase.ts`.
5. Al crear un endpoint nuevo en FastAPI: documentar el schema Pydantic y verificar el JWT.

---

## Referencias

- Diseño y arquitectura completa: `ARCHITECTURE.md`
- Funcionalidades y roadmap: `PRODUCT.md`
- Visión y principios: `VISION.md`
- Tareas activas: `TASKS.md`
