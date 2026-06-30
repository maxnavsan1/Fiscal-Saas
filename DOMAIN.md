# DOMAIN — FiscalOS

Este documento es el corazón del proyecto. Define los conceptos de negocio que dan forma a toda la arquitectura, el código y la experiencia del producto.

Cualquier decisión técnica, de diseño o de producto debe ser consistente con los dominios aquí definidos. Si un término no aparece en este documento, no existe en el sistema.

---

## Mapa de dominios

```
Organization
  └── Workspace (contexto operativo del staff)
  └── Client
        └── FiscalPeriod
        └── Document
        └── DocumentRequest
        └── Filing
        └── Activity
  └── User
        └── Role
        └── Permission
  └── Notification
```

---

## 1. Organization

### Descripción
La **Organization** es el despacho fiscal. Es el tenant raíz del sistema. Todo dato, usuario, cliente y configuración pertenece a una Organization y nunca puede cruzar sus límites.

### Responsabilidades
- Ser el contenedor de todos los datos del despacho
- Mantener la identidad del despacho (nombre, RFC, logo, branding)
- Gestionar su propio equipo (Users con roles staff/admin)
- Gestionar su cartera de Clients
- Configurar el workspace operativo

### Entidades principales
| Entidad | Descripción |
|---------|-------------|
| `Organization` | El despacho. Nombre, RFC, plan, estado |
| `OrganizationSettings` | Branding, configuraciones del workspace, preferencias |

### Relaciones
- Una Organization tiene muchos `Users` (staff y admins)
- Una Organization tiene muchos `Clients`
- Una Organization tiene una `OrganizationSettings`

### Casos de uso
- Crear y configurar un nuevo despacho (onboarding)
- Actualizar identidad visual (logo, colores)
- Ver métricas operativas agregadas
- Gestionar el plan y la suscripción (fase SaaS)

---

## 2. Workspace

### Descripción
El **Workspace** es el entorno operativo del despacho: la vista con la que el staff y el admin trabajan día a día. No es una entidad de base de datos — es el contexto desde el cual se accede y opera sobre todos los demás dominios.

### Responsabilidades
- Presentar la cartera de clientes al staff
- Centralizar la cola de trabajo pendiente cross-cliente
- Dar visibilidad del calendario fiscal del despacho
- Permitir crear y gestionar Filings y DocumentRequests

### Relaciones
- El Workspace pertenece a una Organization
- El Workspace opera sobre Clients, Filings y DocumentRequests

### Casos de uso
- Ver todos los clientes con pendientes activos
- Acceder a la cola de tareas urgentes
- Ver el calendario fiscal del mes
- Crear una declaración para un cliente
- Solicitar documentos a un cliente

---

## 3. Client

### Descripción
El **Client** es la persona física o moral que el despacho atiende. Tiene identidad fiscal propia (RFC, régimen), y es el sujeto central de todas las obligaciones, documentos y declaraciones del sistema.

Un Client puede tener un User asociado (para acceder al portal) o no tenerlo (clientes gestionados internamente sin acceso al portal).

### Responsabilidades
- Mantener la identidad fiscal del cliente (RFC, régimen fiscal, actividad)
- Ser el contenedor de sus Documents, Filings y DocumentRequests
- Representar la relación cliente-despacho (contador asignado, estado)

### Entidades principales
| Entidad | Descripción |
|---------|-------------|
| `Client` | RFC, régimen fiscal, nombre legal, estado, contador asignado |
| `ClientContact` | Persona de contacto del cliente (email, teléfono, nombre) |

### Relaciones
- Un Client pertenece a una Organization
- Un Client puede tener un User asociado (el acceso al portal)
- Un Client es asignado a un User con rol staff
- Un Client tiene muchos Documents, Filings, DocumentRequests y ActivityEvents

### Estados del Client
- `active` — Relación activa, obligaciones en curso
- `inactive` — Sin obligaciones activas
- `archived` — Relación terminada, datos conservados

### Casos de uso
- Crear un nuevo cliente
- Asignar a un contador
- Ver el resumen de estado del cliente (peace of mind status)
- Archivar un cliente

---

## 4. FiscalPeriod

### Descripción
El **FiscalPeriod** es la unidad de tiempo fiscal que organiza documentos y declaraciones. Puede ser mensual, trimestral o anual dependiendo del tipo de obligación.

### Responsabilidades
- Servir como eje temporal de organización de Documents y Filings
- Contener las fechas límite aplicables al período

### Entidades principales
| Entidad | Descripción |
|---------|-------------|
| `FiscalPeriod` | Año, mes/trimestre, tipo (monthly/quarterly/annual) |
| `FiscalDeadline` | Fecha límite, tipo de obligación, aplica a qué regímenes |

### Relaciones
- Un FiscalPeriod pertenece a una Organization
- Un FiscalPeriod contiene muchos Documents
- Un FiscalPeriod contiene muchos Filings
- Un FiscalPeriod tiene muchos FiscalDeadlines

### Casos de uso
- Organizar documentos por período
- Calcular próximas fechas límite
- Enviar recordatorios basados en deadlines del período

---

## 5. Document

### Descripción
El **Document** es un archivo digital subido al sistema. Puede ser un estado de cuenta, una factura, un comprobante fiscal, un contrato o cualquier archivo relevante para las obligaciones del cliente.

Todo archivo vive en Supabase Storage y nunca es accesible por URL directa. Siempre se accede vía presigned URL con expiración.

### Responsabilidades
- Almacenar archivos de forma segura con metadatos completos
- Mantener historial de versiones
- Vincularse con DocumentRequests (el documento responde a una solicitud)
- Vincularse con FilingStages (el documento es evidencia de una etapa)

### Entidades principales
| Entidad | Descripción |
|---------|-------------|
| `Document` | Nombre, tipo, tamaño, storage_path, metadatos, versión |
| `DocumentFolder` | Carpeta de organización, típicamente por FiscalPeriod |
| `DocumentVersion` | Versión anterior del documento cuando se reemplaza |

### Relaciones
- Un Document pertenece a un Client
- Un Document pertenece a un FiscalPeriod (vía DocumentFolder)
- Un Document puede satisfacer un DocumentRequest
- Un Document puede estar adjunto a un FilingStage

### Estados del Document
- `pending` — Subido pero no revisado por el staff
- `approved` — Revisado y aceptado
- `rejected` — Revisado y rechazado (requiere resubida)

### Casos de uso
- Subir un documento (drag & drop)
- Responder a un DocumentRequest con un archivo
- Ver versiones anteriores
- Descargar vía presigned URL
- Aprobar o rechazar un documento

---

## 6. DocumentRequest

### Descripción
Un **DocumentRequest** es una solicitud formal del staff al cliente para que entregue un documento específico. Es la unidad de comunicación estructurada entre el despacho y el cliente.

Reemplaza el "oye, me mandas el estado de cuenta de marzo" por WhatsApp con un flujo trazable, con contexto, fecha límite y confirmación.

### Responsabilidades
- Crear una tarea clara para el cliente con contexto (qué, por qué, cuándo)
- Rastrear el estado de cumplimiento
- Permitir comunicación contextual (comentarios en el hilo)
- Registrar la entrega del documento y la aprobación del staff

### Entidades principales
| Entidad | Descripción |
|---------|-------------|
| `DocumentRequest` | Título, descripción, fecha límite, estado, prioridad |
| `RequestComment` | Comentario en el hilo (actor, contenido, timestamp) |

### Relaciones
- Un DocumentRequest pertenece a un Client
- Un DocumentRequest pertenece a un FiscalPeriod
- Un DocumentRequest puede resolverse con un Document
- Un DocumentRequest genera ActivityEvents y Notifications

### Flujo de estados
```
pending → submitted → approved
                   → rejected → pending (nueva iteración)
```

| Estado | Significado |
|--------|-------------|
| `pending` | El cliente aún no ha respondido |
| `submitted` | El cliente subió un documento |
| `approved` | El staff aceptó el documento |
| `rejected` | El staff rechazó, el cliente debe reintentar |

### Casos de uso
- Staff crea una solicitud con contexto y fecha límite
- Cliente recibe notificación y ve la solicitud con claridad
- Cliente sube documento en respuesta
- Staff aprueba o rechaza con comentario
- Ambos partes pueden comentar en el hilo

---

## 7. Filing

### Descripción
Un **Filing** es una obligación fiscal de un cliente: una declaración de IVA, ISR, IMSS, DIOT, declaración anual, u otro trámite con el SAT o autoridades fiscales.

El Filing tiene un ciclo de vida bien definido que el staff controla y el cliente puede observar. Es el núcleo del "Peace of Mind": la suma de Filings activos determina si el cliente está al corriente.

### Responsabilidades
- Representar una obligación fiscal específica con su período y tipo
- Rastrear el progreso a través de etapas definidas
- Vincular los documentos necesarios para cada etapa
- Notificar al cliente en cada cambio de estado

### Entidades principales
| Entidad | Descripción |
|---------|-------------|
| `Filing` | Cliente, tipo, período, estado actual, fecha límite, monto |
| `FilingType` | Catálogo de tipos: IVA mensual, ISR, IMSS, DIOT, anual, etc. |
| `FilingStage` | Etapa del filing con timestamp, actor y documentos adjuntos |
| `FilingDocument` | Documento adjunto a una etapa específica (acuse, complemento) |

### Relaciones
- Un Filing pertenece a un Client
- Un Filing pertenece a un FiscalPeriod
- Un Filing tiene un FilingType
- Un Filing tiene muchas FilingStages (su timeline)
- Un Filing puede tener FilingDocuments por etapa

### Flujo de etapas (stage flow)
```
collecting_documents
       ↓
  in_review
       ↓
 ready_to_file
       ↓
    filed
       ↓
 acknowledged   ← estado final
```

| Etapa | Significado |
|-------|-------------|
| `collecting_documents` | Se esperan documentos del cliente |
| `in_review` | El staff está revisando la información |
| `ready_to_file` | Lista para presentar, pendiente de confirmación |
| `filed` | Presentada ante el SAT/IMSS |
| `acknowledged` | Acuse recibido, proceso completo |

### Casos de uso
- Staff crea un Filing para un cliente y período
- Staff avanza la etapa y adjunta documentos (acuse, complemento)
- Cliente recibe notificación en cada cambio de etapa
- Cliente ve el progreso visual en su panel

---

## 8. Activity

### Descripción
La **Activity** es el registro inmutable de todo lo que ocurre en el sistema. Cada acción significativa genera un ActivityEvent con actor, recurso, acción y contexto.

No es un log técnico de aplicación. Es el registro de negocio: el historial permanente que construye confianza entre el despacho y sus clientes.

### Responsabilidades
- Registrar cada acción relevante con su contexto completo
- Ser inmutable: ningún evento puede editarse ni eliminarse
- Servir como base para el Activity Feed del cliente y del staff
- Ser la fuente de verdad en disputas o auditorías

### Entidades principales
| Entidad | Descripción |
|---------|-------------|
| `ActivityEvent` | actor_id, action, resource_type, resource_id, metadata (JSONB), ip_address, created_at |

### Acciones registradas (no exhaustivo)
- `document.uploaded` — Cliente subió un documento
- `document.approved` — Staff aprobó un documento
- `document.rejected` — Staff rechazó un documento
- `request.created` — Staff creó una solicitud
- `request.submitted` — Cliente respondió a una solicitud
- `filing.stage_updated` — Staff actualizó la etapa de una declaración
- `filing.created` — Staff creó una nueva declaración
- `user.invited` — Se envió invitación a un usuario
- `client.created` — Se creó un nuevo cliente

### Relaciones
- Un ActivityEvent pertenece a una Organization
- Un ActivityEvent referencia cualquier recurso por `resource_type` + `resource_id`
- Un ActivityEvent puede generar una o muchas Notifications

### Invariantes
- Solo INSERT está permitido — jamás UPDATE ni DELETE
- La escritura la realiza exclusivamente FastAPI con service role
- El frontend nunca escribe directamente en esta tabla

---

## 9. Notification

### Descripción
Una **Notification** es un mensaje entregado a un usuario específico como consecuencia de un ActivityEvent. Su función es llevar información relevante al usuario correcto, en el momento correcto, sin requerir que el usuario esté activamente monitoreando el sistema.

### Responsabilidades
- Informar al usuario de eventos relevantes para él
- Distinguir entre notificaciones leídas y no leídas
- Entregar el mensaje por múltiples canales (in-app, email)
- No generar ruido: solo notificar lo que importa a cada rol

### Entidades principales
| Entidad | Descripción |
|---------|-------------|
| `Notification` | user_id, type, title, body, read, channel, activity_event_id |

### Canales
- `in_app` — Badge + panel de notificaciones en el sistema
- `email` — Correo electrónico al email del usuario

### Eventos que generan notificaciones
| Evento | Notifica a |
|--------|-----------|
| Nueva solicitud de documento | Cliente |
| Documento subido por cliente | Staff asignado |
| Documento aprobado/rechazado | Cliente |
| Etapa de declaración actualizada | Cliente |
| Declaración nueva creada | Cliente |
| Fecha límite fiscal en 7 días | Cliente + Staff |
| Fecha límite fiscal en 3 días | Cliente + Staff |
| Fecha límite fiscal vence hoy | Cliente + Staff |

### Relaciones
- Una Notification pertenece a un User
- Una Notification referencia un ActivityEvent

---

## 10. User

### Descripción
Un **User** es un actor humano en el sistema con identidad autenticada. Puede ser el dueño del despacho, un contador, un staff de soporte, o el cliente de la empresa.

### Responsabilidades
- Autenticarse en el sistema
- Actuar dentro del alcance de su Role
- Recibir Notifications relevantes a su rol y contexto

### Entidades principales
| Entidad | Descripción |
|---------|-------------|
| `User` | email, nombre, avatar, estado, organization_id, role |
| `UserInvitation` | token, email, role, expiration, estado |

### Relaciones
- Un User pertenece a una Organization
- Un User tiene un Role
- Un User puede estar vinculado a un Client (cuando el rol es `client`)
- Un User tiene muchas Notifications

### Estados del User
- `invited` — Invitación enviada, no aceptada aún
- `active` — Usuario activo
- `inactive` — Acceso suspendido

---

## 11. Role

### Descripción
Un **Role** es un nombre que agrupa un conjunto de permisos y define qué puede ver y hacer un User en el sistema.

FiscalOS define tres roles fijos para el MVP. No son configurables — son decisiones de producto deliberadas que cubren todos los casos de uso del MVP.

### Roles

| Role | Descripción | Acceso |
|------|-------------|--------|
| `client` | El cliente del despacho | Solo sus propios datos. Portal simplificado. |
| `staff` | Contador o empleado del despacho | Todos los clientes asignados. Workspace completo. |
| `admin` | Socio o dueño del despacho | Todo. Gestión de equipo y configuración. |

### Invariantes
- Un User tiene exactamente un Role
- El Role se asigna en el momento de la invitación
- Solo un admin puede cambiar el Role de otro User
- Un admin no puede degradarse a sí mismo (protección contra lockout)

---

## 12. Permission

### Descripción
Un **Permission** es una capacidad discreta sobre un tipo de recurso: leer, crear, actualizar o eliminar.

Los permisos no son configurables por el usuario en el MVP. Están definidos de forma estática por cada Role y se hacen cumplir en dos capas:

1. **RLS en PostgreSQL** — La capa de datos no devuelve filas que el usuario no debería ver
2. **Middleware de Next.js** — Las rutas no se sirven a roles que no tienen acceso

### Matriz de permisos (MVP)

| Recurso | client | staff | admin |
|---------|--------|-------|-------|
| Organization settings | — | read | read/write |
| Users | — | — | read/write |
| Clients | read (own) | read/write | read/write |
| Documents | read/write (own) | read/write | read/write |
| DocumentRequests | read/respond (own) | read/write | read/write |
| Filings | read (own) | read/write | read/write |
| ActivityEvents | read (own) | read | read |
| Notifications | read/update (own) | read/update (own) | read/update (own) |

### Invariantes
- `client` nunca ve datos de otros clientes
- `staff` solo ve clientes de su Organization
- `admin` puede ver y actuar sobre todo dentro de su Organization
- Ningún rol puede acceder a datos de otra Organization

---

## Glosario

| Término | Definición en el dominio de FiscalOS |
|---------|--------------------------------------|
| Despacho | Sinónimo de Organization |
| Contador | User con role `staff` |
| Socio | User con role `admin` |
| Portal | El área de la aplicación accesible para role `client` |
| Workspace | El área de la aplicación accesible para roles `staff` y `admin` |
| Período fiscal | Instancia de FiscalPeriod: "Enero 2025", "Q1 2025" |
| Obligación | Sinónimo de Filing |
| Solicitud | Sinónimo de DocumentRequest |
| Estado de salud | El "Peace of Mind Status" de un cliente |
| Al corriente | Estado en que no hay Filings ni DocumentRequests pendientes urgentes |
