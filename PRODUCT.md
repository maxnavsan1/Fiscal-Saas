# PRODUCT — FiscalOS

## Usuarios

### Cliente (Portal User)
Dueño de PYME, freelancer o persona física con actividad empresarial. Su relación con el producto es reactiva: entra cuando recibe una solicitud, sube documentos, revisa el estado de sus declaraciones. No quiere aprender un sistema complejo. Quiere claridad en 30 segundos.

**Jobs to be done:**
- Saber qué documentos me están pidiendo y para cuándo
- Subir archivos sin fricciones
- Ver el estado de mis declaraciones sin llamar al contador
- Tener un historial de todo lo que he entregado

### Contador / Staff (Internal User)
El usuario más intensivo del día a día. Gestiona múltiples clientes, solicita documentos, actualiza estados de declaraciones, deja comentarios. Necesita velocidad y una vista de trabajo que le muestre qué está pendiente en toda su cartera.

**Jobs to be done:**
- Ver de un vistazo qué clientes tienen tareas pendientes
- Solicitar documentos específicos con contexto claro
- Actualizar el estado de declaraciones y que el cliente sea notificado automáticamente
- Tener el historial completo de cada cliente en un solo lugar

### Admin / Socio (Firm Admin)
Gestiona el despacho completo. Ve métricas agregadas, gestiona el equipo, configura el workspace.

**Jobs to be done:**
- Asignar clientes a contadores
- Configurar la identidad del despacho en el portal
- Ver métricas de operación (declaraciones activas, documentos pendientes)
- Gestionar accesos y permisos del equipo

---

## Módulos del Sistema

### 1. Auth & Onboarding
- Login por magic link (sin contraseña)
- Invitación de clientes por email — el contador invita, no el cliente se registra solo
- Onboarding guiado para configurar el workspace del despacho (nombre, logo, RFC)
- Middleware de redirección por rol: cliente → portal, contador → workspace interno, admin → consola

### 2. Client Dashboard
- Vista personal del cliente con resumen de estado
- Declaraciones activas y su etapa actual
- Solicitudes pendientes de documentos (con urgencia visual)
- Próximas fechas límite fiscales
- Actividad reciente

### 3. Document Vault
- Repositorio de documentos organizado por período fiscal (ej. 2025-Q1, 2025-Enero)
- Subida por drag & drop con validación de tipo y tamaño
- Versioning: si se sube un archivo con el mismo nombre, se guarda la versión anterior
- Acceso siempre vía presigned URLs (nunca URLs permanentes)
- Metadatos: quién subió, cuándo, en respuesta a qué solicitud

### 4. Request Center
- El contador crea una solicitud: "Necesito tu estado de cuenta de marzo"
- El cliente recibe notificación y ve la solicitud con contexto
- El cliente sube el documento directamente en el hilo de la solicitud
- El contador aprueba o pide corrección
- Estado de la solicitud: `pending` → `submitted` → `approved` / `rejected`
- Comentarios en el hilo para aclaraciones

### 5. Filings Tracker
- Cada declaración tiene su propia página de estado
- Timeline de etapas: `Documentos requeridos` → `En revisión` → `Lista para presentar` → `Presentada` → `Acuse recibido`
- El contador actualiza la etapa; el cliente recibe notificación automática
- Adjuntos por etapa (acuse, comprobante de pago, complemento)
- Tipos de declaración: IVA mensual, ISR, IMSS, declaración anual, DIOT, etc.

### 6. Activity Feed
- Audit log en tiempo real visible para el cliente y el contador
- Registra: subida de documentos, cambios de estado, nuevas solicitudes, comentarios, aprobaciones
- Filtrable por tipo de evento
- No editable ni eliminable — es el registro de verdad del portal

### 7. Notifications & Deadlines
- Calendario fiscal con fechas límite del SAT e IMSS
- Alertas configurables: 7 días antes, 3 días antes, día de vencimiento
- Notificaciones in-app + email
- Badge de notificaciones en el topbar

### 8. Admin Console
- Gestión de clientes: crear, editar, asignar contador, archivar
- Gestión de equipo: invitar staff, asignar roles
- Configuración del workspace: nombre del despacho, logo, colores primarios
- Vista de actividad agregada del despacho

---

## MVP — Alcance de la Primera Versión

El MVP responde una pregunta: **¿El portal reduce las fricciones de comunicación entre el despacho y sus clientes?**

### Incluido en MVP

| Módulo | Funcionalidades incluidas |
|--------|--------------------------|
| Auth | Magic link, invitaciones, redirección por rol |
| Dashboard del cliente | Estado general, pendientes, próximas fechas |
| Document Vault | Upload, organización por período, presigned URLs |
| Request Center | Crear solicitud, responder con documento, aprobar/rechazar |
| Filings Tracker | Crear declaración, actualizar etapa, notificar al cliente |
| Activity Feed | Registro automático de eventos principales |
| Notificaciones | In-app + email para solicitudes nuevas y cambios de estado |
| Admin Console | CRUD de clientes, gestión de equipo básica |

### Fuera del MVP

| Feature | Razón |
|---------|-------|
| Integración SAT / IMSS | Requiere infraestructura legal y FIEL — semanas de trabajo operativo |
| Firma electrónica (e.firma) | Mismo problema + validación de identidad |
| App móvil nativa | Web responsive cubre el caso de uso del cliente |
| SSO / SAML | Solo necesario para clientes enterprise |
| Reportes y analytics | Primero hay que tener datos reales |
| Módulo de billing / Stripe | Cobrar manualmente da más aprendizaje que automatizar |
| White-labeling para SaaS | Solo cuando el primer despacho externo quiera adoptarlo |
| API pública | Estabilizar modelo de datos antes de publicarlo |
| Extracción de documentos con IA | Feature de v3, no del MVP |
| 2FA más allá de magic links | Suficientemente seguro para el MVP |

---

## Roadmap

### v1 — MVP (Portal funcional para un despacho)
- Auth completo con magic links e invitaciones
- Portal del cliente: dashboard, vault, requests, filings
- Workspace interno del contador
- Admin console básica
- Notificaciones in-app y email

### v2 — Solidez operativa
- Calendario fiscal con alertas automáticas (workers)
- Versioning de documentos
- Búsqueda global en documentos y declaraciones
- Reportes básicos para el admin
- Mejoras de UX basadas en feedback real

### v3 — Camino a SaaS
- Self-service onboarding para nuevos despachos
- Módulo de billing (Stripe)
- White-labeling (logo, dominio personalizado)
- Integraciones: SAT web services, IMSS

### v4 — Diferenciación con IA
- Extracción automática de datos de documentos fiscales
- Clasificación de documentos por tipo
- Detección de documentos faltantes por tipo de declaración
- Sugerencias de fechas límite basadas en historial
