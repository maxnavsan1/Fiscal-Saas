# VISION — FiscalOS

## Qué estamos construyendo

Un **workspace de colaboración fiscal** que convierte la relación caótica entre un despacho contable y sus clientes en un proceso estructurado, transparente y auditable.

No es un sistema de contabilidad. No es un ERP. Es la capa de operaciones que vive entre el despacho y sus clientes: el lugar donde los documentos se entregan con contexto, los trámites tienen un estado visible, y nada se pierde en un hilo de WhatsApp.

## Por qué existe este producto

Hoy, en cualquier despacho fiscal de tamaño pequeño o mediano, la operación cotidiana se parece a esto:

- El contador pide documentos por WhatsApp o email
- El cliente no sabe en qué etapa está su declaración
- Los archivos viven dispersos entre correos, Google Drive y carpetas locales sin estructura
- Las fechas límite del SAT e IMSS llegan como sorpresa
- No hay registro de quién entregó qué y cuándo
- Escalar la cartera de clientes significa contratar más personal administrativo, no mejores herramientas

Este caos operativo no es un problema de actitud. Es un problema de infraestructura. No existe una herramienta accesible, moderna y específica para este mercado.

## La propuesta de valor

**Para el cliente:** Saber en todo momento qué necesita entregar, para cuándo y por qué — sin llamadas ni mensajes de seguimiento.

**Para el contador:** Una vista unificada de toda su cartera con lo que está pendiente, lo que está atrasado y lo que está resuelto.

**Para el despacho:** Escalar clientes sin escalar caos administrativo.

## La referencia de producto

La experiencia de usuario no se inspira en QuickBooks ni en ContaWeb. Se inspira en **Stripe Dashboard**: cualquier usuario entiende el estado de sus asuntos en 30 segundos, sin necesitar una llamada de explicación.

La velocidad y opinionatedness de **Linear**. La claridad estructural de **Stripe**. La flexibilidad de **Notion**. La confianza que genera **Vercel** con su transparencia de estado.

## El camino a SaaS

La primera versión sirve a un solo despacho. La arquitectura está diseñada desde el inicio para multi-tenancy, de forma que cuando el producto esté validado, cualquier despacho pueda registrarse, configurar su workspace y invitar a sus clientes sin intervención técnica.

El despacho inicial es el primer cliente, el primer caso de uso real y el primer validador del producto.

## Principios de diseño

1. **Claridad sobre completitud** — Mejor una sola cosa que funcione perfectamente que diez que confundan.
2. **Trazabilidad como confianza** — Cada acción queda registrada. El audit log no es un log técnico, es una promesa al cliente de que nada desaparece.
3. **El tiempo del usuario es valioso** — Cero pasos innecesarios. Cero clics de confirmación sin propósito.
4. **Estado visible siempre** — El usuario nunca debería tener que preguntar "¿en qué va mi declaración?".
5. **Diseñado para escalar sin fricción** — Las decisiones de arquitectura de hoy no deben bloquearnos mañana.

## Norte

> "El cliente entra al portal y en 30 segundos sabe exactamente qué hacer. El contador abre su dashboard y en 30 segundos sabe en qué está atrasado. El socio del despacho no necesita preguntar el estatus de ningún cliente."
