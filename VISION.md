# VISION — FiscalOS

## El Sistema Operativo del Despacho Fiscal Moderno

FiscalOS no es un portal para clientes. No es un sistema de contabilidad. No es un CRM.

FiscalOS es **el sistema operativo del despacho fiscal moderno**: la infraestructura de operaciones que conecta al despacho con sus clientes, organiza el trabajo del equipo, garantiza el cumplimiento de obligaciones y crea un registro permanente de todo lo que ocurre.

El portal del cliente es uno de sus módulos. El workspace del contador es otro. La consola de administración, otro más. Juntos, forman un sistema coherente que permite operar un despacho fiscal como una empresa de tecnología opera un producto.

---

## El problema que resuelve

Hoy, en cualquier despacho fiscal de tamaño pequeño o mediano, la operación cotidiana se parece a esto:

- El contador pide documentos por WhatsApp o email, sin trazabilidad
- El cliente no sabe en qué etapa está su declaración hasta que llama
- Los archivos viven dispersos entre correos, Google Drive y carpetas locales
- Las fechas límite del SAT e IMSS llegan como sorpresa, no como plan
- No existe registro de quién entregó qué y cuándo
- Escalar la cartera significa contratar más personal, no mejores herramientas

Este caos no es un problema de actitud. Es un problema de infraestructura. No existe una herramienta accesible, moderna y específica para este mercado.

---

## Peace of Mind First

El concepto central del producto es uno: **tranquilidad**.

El cliente no compra módulos. No compra dashboards. Compra la certeza de saber, en cualquier momento, en qué estado están sus obligaciones fiscales.

Cuando el cliente entra al sistema, debe poder responder una sola pregunta en menos de 10 segundos:

> **¿Estoy al corriente?**

La respuesta puede ser una de tres:

```
✓  ESTÁS AL CORRIENTE
   No tienes pendientes este mes. Próxima obligación: 17 de julio.

⚠  TIENES 2 PENDIENTES
   Documentos por entregar antes del 17 de julio.

🚨 ACCIÓN URGENTE
   Una declaración vence mañana. Requiere tu atención ahora.
```

Todo lo demás en la experiencia del cliente construye sobre este estado. Cada notificación, cada pantalla, cada interacción responde a la misma pregunta: **¿estoy al corriente?**

---

## La propuesta de valor por rol

**Para el cliente:**
Saber siempre si está al corriente, qué necesita entregar y cuándo — sin llamadas, sin mensajes de seguimiento, sin sorpresas.

**Para el contador:**
Una vista unificada de toda su cartera: qué clientes tienen pendientes, qué declaraciones están atrasadas, qué documentos faltan. El trabajo organizado por urgencia, no por llegada.

**Para el despacho:**
Escalar la cartera de clientes sin escalar el caos administrativo. El crecimiento no depende de contratar más personal — depende de tener mejor infraestructura.

---

## La referencia de producto

FiscalOS no se inspira en QuickBooks, ContaWeb ni en ningún software contable tradicional.

Se inspira en:

- **Stripe** — Cualquier usuario entiende el estado de sus asuntos en 30 segundos. La información está ahí, organizada, sin necesitar capacitación.
- **Linear** — Velocidad, opinionatedness, foco en el trabajo. Sin ruido. La interfaz no estorba.
- **Vercel** — Estado siempre visible. Las operaciones tienen feedback inmediato. Nada ocurre en silencio.
- **Notion** — Flexibilidad dentro de una estructura. El contenido se adapta al contexto.

---

## El camino a SaaS

La primera versión sirve a un solo despacho. La arquitectura está diseñada desde el inicio para multi-tenancy real: cada despacho es un `Organization` con sus propios datos completamente aislados.

Cuando el producto esté validado, cualquier despacho podrá registrarse, configurar su workspace e invitar a sus clientes sin intervención técnica. El despacho inicial es el primer cliente, el primer caso de uso real y el primer validador de que esto funciona.

---

## Principios que nunca cambian

1. **Peace of Mind First** — La experiencia siempre responde a "¿estoy al corriente?". Ese estado es visible antes que cualquier otra información.

2. **Trazabilidad como confianza** — Cada acción queda registrada. El audit log no es un log técnico, es una promesa: nada desaparece, nada se puede negar.

3. **El cliente no es un contador** — La interfaz del cliente debe funcionar sin capacitación. Si necesita un manual, fallamos.

4. **Cero sorpresas** — Las fechas límite, los documentos faltantes y los cambios de estado nunca deben llegar sin aviso. El sistema anticipa, no reacciona.

5. **Claridad sobre completitud** — Mejor una cosa que funcione perfectamente que diez que confundan. El MVP no es el producto incompleto, es el producto con el foco correcto.

6. **Diseñado para escalar** — Las decisiones de arquitectura de hoy no deben bloquear el crecimiento de mañana. Multi-tenancy, seguridad por diseño y dominios bien definidos desde el día uno.

---

## El norte

> El cliente entra a FiscalOS y en 10 segundos sabe si está al corriente. El contador abre su workspace y en 30 segundos sabe en qué está atrasado. El socio del despacho nunca necesita preguntar el estatus de ningún cliente.
>
> El despacho opera como una empresa tecnológica. Predecible, auditable, escalable.
