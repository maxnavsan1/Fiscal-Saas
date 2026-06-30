# SECURITY — FiscalOS

Este documento define la estrategia de seguridad completa del sistema. Toda decisión arquitectónica relacionada con seguridad debe ser consistente con lo aquí definido.

La seguridad en FiscalOS no es una capa que se agrega al final. Es una propiedad del diseño desde el día uno.

---

## Principios de seguridad

### 1. Defense in Depth
Ninguna capa de seguridad es suficiente por sí sola. El sistema aplica controles en múltiples capas: red, base de datos, aplicación y UI. Si una capa falla, las demás la contienen.

### 2. Least Privilege
Cada actor (usuario, servicio, proceso) tiene exactamente los permisos que necesita para operar. Nada más. Un `client` no puede ver datos de otro cliente aunque la query no filtre correctamente — RLS lo previene.

### 3. Zero Trust
No existe una "red interna" de confianza. Cada request, sea del browser, del frontend server o de FastAPI, debe autenticarse y autorizarse independientemente. No se confía en la procedencia.

### 4. Security by Default
La configuración por defecto es la más restrictiva. Abrir permisos es una acción explícita y revisada. Cerrarlos es el estado base. Los buckets de Storage son privados por defecto. Las tablas requieren RLS antes de ser accesibles.

### 5. Auditabilidad
Todo evento significativo queda registrado de forma inmutable. La seguridad no solo previene — también detecta y permite reconstruir lo que ocurrió.

### 6. Separación de secretos
Los secretos nunca viajan en código. Nunca en logs. Nunca en URLs. Viven exclusivamente en variables de entorno gestionadas en los sistemas de deployment.

---

## Autenticación

### Magic Links
El método primario de autenticación es el magic link enviado por email, implementado vía Supabase Auth.

**Cómo funciona:**
1. El usuario ingresa su email
2. Supabase genera un token OTP de un solo uso con expiración de 1 hora
3. El token se envía por email como enlace
4. El usuario hace click, Supabase valida el token y emite un JWT
5. El JWT se almacena en una cookie HttpOnly con SameSite=Strict

**Por qué magic links:**
- No existen contraseñas que robar, filtrar o reutilizar
- No requiere gestión de recuperación de contraseñas
- El vector de ataque principal (email comprometido) ya existiría con contraseñas tradicionales
- Ver ADR-004 para la justificación completa

### JWT (JSON Web Tokens)
- Supabase Auth emite JWTs firmados con el JWT secret del proyecto
- Los JWTs contienen: `sub` (user ID), `role`, `organization_id`, `exp`
- Expiración del JWT: 1 hora
- Refresh automático vía Supabase Auth SDK
- El JWT secret nunca se comparte entre proyectos (staging y production tienen secrets distintos)

### Sesión
- Las sesiones viven en cookies HttpOnly, no en localStorage
- Cookies con atributos: `Secure`, `HttpOnly`, `SameSite=Strict`
- La renovación del token es transparente para el usuario
- El logout invalida la sesión en el servidor (Supabase revoca el refresh token)

### Verificación en FastAPI
- Cada request a FastAPI incluye el JWT en el header `Authorization: Bearer <token>`
- FastAPI verifica la firma del JWT con el secret de Supabase antes de procesar cualquier request
- Si el JWT es inválido, expirado o ausente, FastAPI retorna 401
- La verificación ocurre como dependencia global — no se puede olvidar en un endpoint

---

## Autorización

### Dos capas de autorización

**Capa 1 — Middleware de Next.js (edge)**
El middleware intercepta cada request antes de que llegue al componente. Verifica:
- ¿Está autenticado el usuario?
- ¿Tiene el rol requerido para esta ruta?

Si no, redirige al login o a una página de acceso denegado. Esta capa previene que el usuario siquiera cargue HTML de rutas no autorizadas.

**Capa 2 — Row Level Security en PostgreSQL (base de datos)**
RLS es la última línea de defensa. Incluso si el middleware fallara, incluso si un bug en el código omitiera un filtro, RLS garantiza que la base de datos no retorna datos no autorizados.

La autorización a nivel de datos nunca depende de filtros en el código de aplicación.

### Principio de autorización
No se confía en ninguna capa de forma aislada. Ambas deben coincidir: si el middleware permite el acceso y RLS lo deniega, la query retorna vacía. Si RLS permite el acceso pero el middleware no, el usuario no llega a la query.

---

## RBAC — Control de Acceso Basado en Roles

### Roles definidos

| Role | Definición | Alcance máximo |
|------|-----------|----------------|
| `client` | El cliente del despacho | Solo sus propios datos dentro de su Organization |
| `staff` | Contador o empleado | Todos los clientes de su Organization |
| `admin` | Socio o dueño | Todo dentro de su Organization, incluyendo configuración y equipo |

### Asignación de roles
- El role se asigna en el momento de crear la invitación
- El role vive en la tabla `users` del sistema y se incluye en el JWT
- Solo un `admin` puede modificar roles
- Un `admin` no puede modificar su propio role (prevención de lockout accidental)
- No existe un super-admin cross-organization en el MVP

### Escalación de privilegios
No está permitida por diseño. Las operaciones de FastAPI verifican el role del token JWT. No existe un mecanismo de "sudo" ni de tokens de servicio accesibles desde el browser.

---

## Row Level Security (RLS)

### Estrategia
RLS es el mecanismo principal de aislamiento de datos. Se implementa directamente en PostgreSQL, no en código de aplicación.

### Reglas base para todas las tablas

```sql
-- Todo acceso verifica que el organization_id del registro
-- coincida con el organization_id del usuario autenticado
organization_id = (SELECT organization_id FROM users WHERE id = auth.uid())
```

### Políticas por rol

**Para tablas de cliente** (documents, filings, requests):
```
client: solo puede ver registros donde client_id corresponde a su propio client
staff:  puede ver todos los registros de su organization
admin:  puede ver todos los registros de su organization
```

**Para tablas de organización** (users, organization_settings):
```
client: sin acceso a users, solo puede leer su propio perfil
staff:  puede leer users de su organization
admin:  puede leer y escribir en users y settings de su organization
```

**Para activity_log:**
```
Todos: solo INSERT (via service role de FastAPI)
client: puede leer events donde client_id = su propio client
staff:  puede leer todos los events de su organization
admin:  puede leer todos los events de su organization
```

### Invariantes de RLS
- RLS habilitado en todas las tablas del dominio, sin excepciones
- Las políticas se escriben antes de insertar los primeros datos
- Las migraciones incluyen la verificación de que RLS está habilitado
- Los tests de integración verifican el aislamiento entre organizaciones

---

## Almacenamiento de archivos (Storage)

### Buckets privados
Todos los buckets de Supabase Storage son privados. No existe ningún bucket público. Ningún archivo es accesible por URL directa desde el browser o desde internet.

### Presigned URLs
Todo acceso a un documento ocurre a través de una presigned URL generada en el servidor:
- Expiración máxima: 1 hora
- Generada solo después de verificar que el usuario tiene permiso sobre el documento
- La URL firmada no revela la ruta real del archivo en el bucket
- Cada descarga genera una nueva URL — las URLs expiradas no funcionan

### Estructura de paths en Storage
```
{organization_id}/{client_id}/{fiscal_period}/{document_id}/{filename}
```
El path nunca se expone al cliente. Solo se almacena en la columna `storage_path` de la tabla `documents`.

### Validación de archivos
- Tipos permitidos: PDF, XML, PNG, JPG, JPEG, XLSX, CSV
- Tamaño máximo: 25 MB por archivo
- La validación ocurre tanto en el cliente (UX) como en el servidor (seguridad)
- Los archivos se escanean por tipo MIME real, no solo por extensión

---

## Audit Trail

### Principio
El audit trail no es un log técnico. Es el registro de negocio. Cada acción que cambia el estado del sistema queda registrada de forma permanente.

### Tabla `activity_log`
- Solo INSERT habilitado vía RLS — nunca UPDATE ni DELETE
- La escritura la realiza exclusivamente FastAPI con service role key
- El frontend nunca escribe en esta tabla directamente
- Ningún usuario, incluyendo admins, puede modificar o eliminar registros

### Qué se registra
Cada evento incluye:
- `id` — UUID del evento
- `organization_id` — Tenant del evento
- `actor_id` — User que realizó la acción (o `system` para acciones automáticas)
- `action` — Acción en formato `resource.verb` (ej. `document.approved`)
- `resource_type` — Tipo de recurso afectado
- `resource_id` — ID del recurso afectado
- `metadata` — Contexto adicional en JSONB (estado anterior, estado nuevo, etc.)
- `ip_address` — IP del actor cuando aplica
- `user_agent` — Browser/client cuando aplica
- `created_at` — Timestamp inmutable con timezone

### Retención
- Los registros del audit log no tienen fecha de expiración en el MVP
- Se evalúa una política de retención de 7 años en versiones futuras (conforme a requisitos fiscales)

---

## Gestión de secretos

### Variables de entorno
Todos los secretos viven en variables de entorno. Nunca en el código, nunca en archivos de configuración commiteados, nunca en logs.

### Archivos comprometidos por defecto
El `.gitignore` excluye:
- `.env`
- `.env.local`
- `.env.production`
- `.env.staging`
- Cualquier archivo `*.pem`, `*.key`, `*.p12`

### Secretos por entorno
- `development` — Supabase local, JWT secrets locales
- `staging` — Proyecto Supabase de staging con secretos distintos
- `production` — Proyecto Supabase de producción con secretos distintos
- Los secretos no se comparten entre entornos

### Rotación
- El JWT secret de Supabase puede rotarse desde el dashboard — invalida todas las sesiones activas
- Las API keys de servicios externos (email, etc.) se rotan semestralmente o ante cualquier sospecha de compromiso
- El proceso de rotación está documentado en el runbook de operaciones

### Secretos en CI/CD
- Los secretos se inyectan vía variables de entorno del pipeline (Render environment, Vercel environment)
- Ningún secret aparece en los logs del pipeline
- El pipeline nunca imprime variables de entorno

---

## Cifrado

### En tránsito
- Todo el tráfico usa TLS 1.3 obligatorio
- Los certificados son gestionados por los proveedores de hosting (Vercel, Render, Supabase)
- No se aceptan conexiones HTTP en producción — todo se redirige a HTTPS
- HSTS habilitado con preload

### En reposo
- Los datos en PostgreSQL están cifrados en reposo por Supabase (AES-256)
- Los archivos en Supabase Storage están cifrados en reposo
- Los backups están cifrados con las mismas políticas que el storage primario

### Datos sensibles en la aplicación
- Los tokens de invitación se almacenan como hash (bcrypt) en la base de datos, no en texto plano
- No se almacenan credenciales de ningún tipo en la base de datos

---

## Headers de seguridad HTTP

Los siguientes headers deben estar configurados en Next.js (next.config.js):

```
Content-Security-Policy      — Previene XSS y code injection
X-Frame-Options: DENY        — Previene clickjacking
X-Content-Type-Options: nosniff  — Previene MIME sniffing
Referrer-Policy: strict-origin-when-cross-origin
Permissions-Policy           — Deshabilita features del browser no usadas
Strict-Transport-Security    — Fuerza HTTPS
```

---

## OWASP Top 10 — Mitigaciones

| Vulnerabilidad | Mitigación en FiscalOS |
|---------------|----------------------|
| A01 — Broken Access Control | RLS en PostgreSQL + middleware de Next.js + verificación de JWT en FastAPI |
| A02 — Cryptographic Failures | TLS 1.3, cifrado en reposo, tokens hasheados, no HTTP en producción |
| A03 — Injection | Supabase client usa queries parametrizadas; FastAPI usa SQLAlchemy ORM; Zod valida todo input |
| A04 — Insecure Design | RLS por diseño, presigned URLs, invitaciones con expiración, roles inmutables por usuario |
| A05 — Security Misconfiguration | Variables de entorno por ambiente, headers HTTP de seguridad, buckets privados por defecto |
| A06 — Vulnerable Components | Dependencias auditadas con `npm audit` y `pip audit` en CI. Actualizaciones regulares. |
| A07 — Auth/Identification Failures | Magic links OTP de un solo uso, JWTs con expiración corta, cookies HttpOnly |
| A08 — Software/Data Integrity | CI/CD valida tipos y tests antes de deploy. Migraciones revisadas en PR. |
| A09 — Logging/Monitoring Failures | Audit log inmutable, logs de errores en Render, alertas configuradas para errores 5xx |
| A10 — SSRF | FastAPI no realiza requests a URLs proporcionadas por usuarios. Validación de URLs en Pydantic. |

---

## Logs y monitoreo

### Logs de aplicación
- FastAPI: logs estructurados en JSON con nivel, timestamp, request_id, actor, acción
- Next.js: logs de errores de servidor enviados a la plataforma de hosting
- Formato: JSON estructurado, nunca texto libre con datos variables

### Qué NO se loguea
- Tokens JWT completos
- Emails de usuarios
- Contenido de archivos
- Variables de entorno
- Datos personales en texto plano

### Alertas
- Errores 5xx con frecuencia > 5 en 5 minutos → alerta
- Tiempo de respuesta de FastAPI > 2s promedio → alerta
- Intentos de acceso con JWT inválido en volumen > 50/min → alerta (posible ataque)

---

## Backups

### Base de datos
- Supabase realiza backups automáticos diarios con retención de 7 días (plan Pro)
- Point-in-time recovery disponible con retención de 7 días
- Los backups son cifrados con las mismas políticas que el storage primario

### Storage de archivos
- Supabase Storage no incluye backups automáticos en el plan base
- Para producción: replicación periódica a un bucket externo (S3 o R2) con retención de 30 días

### Procedimiento de restauración
- El procedimiento de restauración está documentado y debe ser probado semestralmente
- Un backup que no se ha restaurado con éxito no es un backup confiable

---

## Estrategia de recuperación ante desastres

### RTO (Recovery Time Objective)
Tiempo máximo tolerable de inactividad: **4 horas** para el MVP.

### RPO (Recovery Point Objective)
Pérdida máxima de datos tolerable: **24 horas** (una copia de seguridad diaria).

### Escenarios y respuesta

**Escenario 1: Fallo del servicio de FastAPI**
- Render reinicia el contenedor automáticamente
- Si el fallo persiste, el frontend puede operar en modo degradado (lecturas directas a Supabase)
- RTO estimado: 15 minutos

**Escenario 2: Fallo de Supabase**
- Supabase tiene SLA de 99.9% en plan Pro
- Si el fallo es de Supabase, no hay mitigación inmediata — el sistema está down
- Comunicación proactiva a los usuarios
- RTO dependiente de Supabase: 1-4 horas

**Escenario 3: Datos corruptos o eliminados accidentalmente**
- Restaurar desde el backup diario o point-in-time recovery
- Identificar el alcance del problema antes de restaurar
- RTO estimado: 2-4 horas

**Escenario 4: Compromiso de credenciales**
- Rotar inmediatamente el JWT secret (invalida todas las sesiones)
- Rotar todas las API keys de servicios externos
- Auditar el `activity_log` para identificar acciones no autorizadas
- Notificar a los usuarios afectados
- RTO para acceso: inmediato tras rotación

### Runbook
Cada escenario de desastre tiene un runbook documentado con pasos concretos, responsables y criterios de éxito. El runbook se revisa semestralmente.
