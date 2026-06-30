# PRODUCT — FiscalOS

FiscalOS es el Sistema Operativo del Despacho Fiscal Moderno. El product thinking de este documento parte de dominios de negocio, no de páginas o features.

Para el glosario completo de conceptos del dominio, ver `DOMAIN.md`.

---

## Usuarios y roles

### Cliente (role: `client`)
Dueño de PYME, freelancer o persona física con actividad empresarial. Su relación con el sistema es esporádica: entra cuando hay una acción requerida. No quiere aprender un sistema — quiere responder una pregunta: **¿estoy al corriente?**

**Jobs to be done:**
- Saber en todo momento si mis obligaciones fiscales están al corriente
- Saber qué documentos me están pidiendo y para cuándo
- Subir archivos sin fricción ni capacitación
- Ver el estado de mis declaraciones sin llamar al contador
- Tener un historial permanente de todo lo que he entregado

### Contador / Staff (role: `staff`)
El usuario más intensivo del día a día. Gestiona múltiples clientes, crea solicitudes de documentos, actualiza etapas de declaraciones, revisa documentos. Necesita velocidad y claridad sobre qué está pendiente en toda su cartera.

**Jobs to be done:**
- Ver de un vistazo qué clientes tienen pendientes urgentes
- Solicitar documentos con contexto claro (qué, por qué, para cuándo)
- Actualizar el estado de una declaración y que el cliente sea notificado automáticamente
- Tener el historial completo de cada cliente en un solo lugar
- Priorizar trabajo por urgencia, no por orden de llegada

### Admin / Socio (role: `admin`)
Gestiona el despacho como organización. Ve el estado operativo completo, gestiona el equipo, configura el workspace.

**Jobs to be done:**
- Ver el estado operativo del despacho en su conjunto
- Asignar clientes a contadores
- Configurar la identidad del despacho en el sistema
- Gestionar accesos y roles del equipo
- Onboardear nuevos clientes

---

## Dominios del sistema

Ver `DOMAIN.md` para la definición completa de cada dominio. El sistema se estructura alrededor de los siguientes dominios de negocio:

| Dominio | Responsabilidad central |
|---------|------------------------|
| Organization | El despacho fiscal. Tenant raíz. |
| Workspace | El entorno operativo del staff. |
| Client | El cliente del despacho y su identidad fiscal. |
| FiscalPeriod | La organización temporal de obligaciones y documentos. |
| Document | Los archivos del cliente, almacenados y versionados. |
| DocumentRequest | La solicitud formal de un documento al cliente. |
| Filing | La declaración o trámite fiscal y su ciclo de vida. |
| Activity | El registro inmutable de todo lo que ocurre. |
| Notification | Las alertas entregadas a los usuarios. |
| User | El actor humano autenticado con un rol. |
| Role / Permission | El control de acceso basado en roles. |

---

## Navegación por rol

### Portal del cliente — 4 secciones únicamente

El cliente no es un contador. La navegación debe ser comprensible sin capacitación.

```
Inicio        → Peace of Mind Status + acciones pendientes
Documentos    → Vault de archivos + solicitudes pendientes integradas
Declaraciones → Estado de cada filing con su timeline
Perfil        → Datos del usuario, historial de actividad, preferencias
```

**Principios de la navegación del cliente:**
- No existe una página de "Solicitudes" separada — las solicitudes aparecen como tareas dentro de "Documentos"
- No existe una página de "Actividad" separada — el historial aparece como timeline en "Perfil"
- No existe una página de "Notificaciones" separada — las notificaciones son el driver para entrar al sistema
- El máximo de items en la navegación principal es 4

**Inicio — Peace of Mind First**

La vista de inicio no es un dashboard de widgets. Es una respuesta inmediata a "¿estoy al corriente?" seguida de las acciones pendientes ordenadas por urgencia.

```
Estado           → Un solo indicador: Al corriente / Pendiente / Urgente
Pendientes       → Lista de acciones concretas requeridas (si las hay)
Próximas fechas  → Las 3 fechas límite más cercanas
Declaraciones    → Estado resumido de las declaraciones activas
```

**Documentos — Vault + Requests integrados**

Las solicitudes de documentos aparecen como tareas al inicio de la vista de Documentos cuando están pendientes. Una vez respondidas, el documento queda en el vault del período correspondiente.

**Declaraciones — Filings con estado visible**

Lista de declaraciones del cliente con su etapa actual y fecha límite. El click en una declaración abre el timeline completo.

**Perfil — Datos + Actividad**

Información del usuario, historial de actividad reciente y preferencias de notificación.

---

### Workspace del staff — Vista operativa

```
Cartera          → Lista de todos los clientes asignados con su estado
[Cliente]        → Vista completa del cliente:
                   Tab Inicio → Resumen de estado del cliente
                   Tab Documentos → Vault del cliente + crear solicitud
                   Tab Declaraciones → Filings del cliente + crear nuevo
                   Tab Actividad → Historial del cliente
Cola             → Tareas pendientes cross-cliente ordenadas por urgencia
Calendario       → Fechas límite fiscales del mes
```

### Consola del admin — Gestión del despacho

```
Clientes         → CRUD de clientes, asignación de contadores
Equipo           → Gestión de usuarios y roles
Configuración    → Identidad del despacho, branding
```

---

## MVP — Alcance de la primera versión

La pregunta que el MVP debe responder:

> **¿El sistema reduce la fricción operativa del despacho y da al cliente la claridad que necesita?**

### Capacidades incluidas en el MVP

| Dominio | Capacidades del MVP |
|---------|-------------------|
| Auth | Magic links, invitaciones por email, redirección por rol |
| Organization | Configuración del despacho, onboarding inicial |
| User | Invitaciones, gestión de equipo básica, roles fijos |
| Client | CRUD de clientes, asignación de contador, estado de salud |
| Document | Upload, organización por período, presigned URLs, aprobación |
| DocumentRequest | Crear solicitud, responder, aprobar/rechazar, comentarios |
| Filing | Crear declaración, avanzar etapas, adjuntar documentos |
| Activity | Registro automático de eventos, feed por cliente |
| Notification | In-app + email para eventos críticos, realtime |

### Fuera del MVP

| Capacidad | Razón del aplazamiento |
|-----------|----------------------|
| Integración SAT / IMSS | Requiere FIEL y certificados — semanas de trabajo legal |
| Firma electrónica (e.firma) | Mismo problema + validación de identidad |
| App móvil nativa | Web responsive cubre el caso de uso del cliente |
| SSO / SAML | Solo necesario para clientes enterprise |
| Reportes y analytics | Primero hay que tener datos reales de uso |
| Billing con Stripe | Cobrar manualmente genera más aprendizaje |
| White-labeling por despacho | Solo cuando el primer despacho externo lo pida |
| API pública | Estabilizar el modelo de datos antes de publicarlo |
| Extracción de documentos con IA | Feature de v4, no del MVP |
| 2FA adicional | Magic links son suficientemente seguros para el MVP |
| Versionado visual de documentos | Útil pero no crítico para la primera validación |

---

## Roadmap

### v1 — MVP: Validación del sistema operativo
Un despacho, sus clientes, sus declaraciones. La operación completa digitalizada.

- Auth completo: magic links, invitaciones, roles
- Portal del cliente: Inicio (Peace of Mind), Documentos, Declaraciones, Perfil
- Workspace del staff: cartera, cliente individual, cola de trabajo
- Admin console: clientes, equipo, configuración
- Notificaciones in-app y email

### v2 — Solidez operativa
Basada en lo que los usuarios reales pidan en v1.

- Calendario fiscal con alertas automáticas (workers de deadline)
- Versionado de documentos con historial
- Búsqueda global dentro del despacho
- Reportes básicos para el admin
- Mejoras de UX y navegación basadas en feedback

### v3 — Apertura como SaaS
El sistema está listo para crecer más allá del despacho inicial.

- Self-service onboarding para nuevos despachos
- Billing con Stripe (suscripción mensual por organización)
- White-labeling: logo, colores y dominio personalizado por despacho
- Integraciones: SAT web services, IMSS SUA

### v4 — Diferenciación con inteligencia
El sistema aprende de los datos acumulados.

- Extracción automática de datos de XMLs del SAT
- Clasificación y etiquetado automático de documentos por tipo
- Detección de documentos faltantes por tipo de declaración
- Sugerencias de fechas límite basadas en el historial del cliente
- API pública para integraciones de terceros
