# DESIGN SYSTEM — FiscalOS

Este documento define las decisiones de diseño visual del sistema. No es una guía de estilo aspiracional — es el contrato entre diseño y código.

Toda implementación de UI debe ser consistente con las decisiones aquí definidas. Si algo no está en este documento, no está en el sistema.

---

## Filosofía visual

FiscalOS es un sistema para operar un negocio. Su interfaz debe transmitir exactamente lo que es: **confiable, estructurado, rápido y claro**.

La referencia no es el software contable tradicional (pesado, denso, confuso). La referencia es el software de infraestructura moderno:

- **Stripe** — Información densa pero nunca abrumadora. Cada número tiene contexto. El estado siempre es visible.
- **Linear** — Velocidad. Las acciones ocurren en el momento. Sin confirmaciones innecesarias. Sin pasos de más.
- **Vercel** — Feedback inmediato. El estado del sistema es siempre visible. Los procesos tienen progreso.
- **Notion** — Flexibilidad dentro de una estructura. El contenido respira. Los bloques de información tienen jerarquía clara.

### Los tres principios visuales

**1. Calm Technology**
La interfaz no llama la atención sobre sí misma. Cuando todo está bien, el sistema es casi invisible. Solo cuando se requiere acción, la UI la solicita. Los colores de alerta se usan estrictamente para alertas reales.

**2. Information Density with Clarity**
FiscalOS maneja información estructurada. La interfaz no debe esconder la densidad informativa detrás de espacio vacío innecesario. El reto es hacer que la información densa sea legible — no eliminarla.

**3. Status Always Visible**
El usuario nunca debe buscar su estado. En la vista del cliente, el Peace of Mind Status es el elemento más prominente. En el workspace del contador, el estado de cada cliente es visible sin hacer click.

---

## Tipografía

### Familia tipográfica
**Inter** — Variable font, cargado desde Google Fonts o self-hosted para performance.

Inter es la elección correcta para un sistema B2B porque:
- Excelente legibilidad en tamaños pequeños y medianos
- Números tabulares para datos financieros y fechas
- Variable font — un solo archivo para todos los pesos
- Es el estándar de facto en software moderno (Linear, Vercel, Notion la usan)

### Escala tipográfica

| Token | Tamaño | Peso | Uso |
|-------|--------|------|-----|
| `text-xs` | 12px | 400/500 | Metadatos, labels secundarios, badges |
| `text-sm` | 14px | 400/500 | Texto de interfaz, descripciones, items de lista |
| `text-base` | 16px | 400 | Cuerpo de texto, párrafos |
| `text-lg` | 18px | 500/600 | Subtítulos de sección |
| `text-xl` | 20px | 600 | Títulos de página secundarios |
| `text-2xl` | 24px | 600/700 | Títulos de página principales |
| `text-3xl` | 30px | 700 | Peace of Mind Status, métricas hero |
| `text-4xl` | 36px | 700 | Pantallas de onboarding, estados vacíos |

### Números y datos
- `font-variant-numeric: tabular-nums` en todos los números — especialmente fechas, montos y contadores
- Los montos fiscales usan peso 600 para distinguirse del texto circundante

### Consideraciones de legibilidad
- Altura de línea base: 1.5 para texto de cuerpo
- Altura de línea para títulos: 1.2
- Longitud máxima de línea: 72 caracteres en texto de descripción

---

## Espaciado

### Sistema de 4 puntos
Todo el espaciado usa múltiplos de 4px. Esto garantiza consistencia visual sin necesidad de mediciones manuales.

| Token Tailwind | Valor | Uso típico |
|---------------|-------|-----------|
| `space-1` | 4px | Separación interna mínima (padding de badges, gaps entre iconos) |
| `space-2` | 8px | Padding interno de botones pequeños, gap entre label e input |
| `space-3` | 12px | Padding de items de lista, gaps en navegación |
| `space-4` | 16px | Padding estándar de componentes, gap entre elementos de form |
| `space-5` | 20px | Separación entre grupos de elementos |
| `space-6` | 24px | Padding de cards, separación entre secciones |
| `space-8` | 32px | Separación entre secciones mayores |
| `space-12` | 48px | Separación entre bloques de página |
| `space-16` | 64px | Márgenes de layout, secciones hero |

### Layout
- Sidebar: 240px de ancho
- Contenido principal: max-width de 1200px
- Padding de página: 24px en desktop, 16px en mobile
- Gap entre columnas en layouts de dos columnas: 24px

---

## Colores

### Paleta base

FiscalOS usa una paleta neutral de grises fríos como base, con un único color de acento que representa la marca del despacho.

**Neutros (base de toda la interfaz)**

| Token | Hex | Uso |
|-------|-----|-----|
| `gray-50` | #FAFAFA | Fondo de página |
| `gray-100` | #F4F4F5 | Fondo de hover, fondo de inputs |
| `gray-200` | #E4E4E7 | Bordes, separadores |
| `gray-300` | #D1D1D6 | Bordes con énfasis |
| `gray-400` | #A1A1AA | Texto placeholder, iconos inactivos |
| `gray-500` | #71717A | Texto secundario, metadata |
| `gray-600` | #52525B | Texto de labels |
| `gray-700` | #3F3F46 | Texto de interfaz |
| `gray-800` | #27272A | Texto principal |
| `gray-900` | #18181B | Títulos, texto de mayor énfasis |

**Color de marca (configurable por despacho en v3)**

Para el MVP, el color de marca es un azul índigo oscuro:

| Token | Hex | Uso |
|-------|-----|-----|
| `brand-50` | #EEF2FF | Fondos sutiles de marca |
| `brand-100` | #E0E7FF | Fondos de estados seleccionados |
| `brand-500` | #6366F1 | Color primario de acciones |
| `brand-600` | #4F46E5 | Color primario hover |
| `brand-700` | #4338CA | Color primario active |

### Colores semánticos (no usarlos fuera de su contexto)

**Success — Verde**
Usado para: estado "al corriente", documentos aprobados, filings completados

| Token | Hex | Uso |
|-------|-----|-----|
| `success-50` | #F0FDF4 | Fondo de estado success |
| `success-500` | #22C55E | Icono de success, borders |
| `success-700` | #15803D | Texto de success |

**Warning — Amarillo ámbar**
Usado para: pendientes, documentos por vencer, solicitudes sin respuesta

| Token | Hex | Uso |
|-------|-----|-----|
| `warning-50` | #FFFBEB | Fondo de estado warning |
| `warning-500` | #F59E0B | Icono de warning, borders |
| `warning-700` | #B45309 | Texto de warning |

**Danger — Rojo**
Usado para: vencimientos inminentes, documentos rechazados, errores críticos

| Token | Hex | Uso |
|-------|-----|-----|
| `danger-50` | #FFF1F2 | Fondo de estado danger |
| `danger-500` | #EF4444 | Icono de danger, borders |
| `danger-700` | #B91C1C | Texto de danger |

**Info — Azul**
Usado para: información contextual, estados neutrales en proceso

| Token | Hex | Uso |
|-------|-----|-----|
| `info-50` | #EFF6FF | Fondo de estado info |
| `info-500` | #3B82F6 | Icono de info |
| `info-700` | #1D4ED8 | Texto de info |

### Uso del color: reglas estrictas

- Los colores semánticos (success/warning/danger/info) solo se usan para comunicar estados reales del sistema
- El rojo nunca se usa para decoración — es exclusivamente para alertas y errores
- El verde no se usa en botones primarios — es exclusivamente para estados de éxito
- El fondo de la aplicación siempre es `gray-50`, nunca blanco puro

---

## Componentes

### Peace of Mind Status
El componente más importante del sistema. Aparece en la vista principal del cliente.

```
Estados:
✓ Al corriente      → fondo success-50, icono verde, texto "ESTÁS AL CORRIENTE"
⚠ Pendientes        → fondo warning-50, icono ámbar, texto "TIENES N PENDIENTES"
🚨 Urgente          → fondo danger-50, icono rojo, texto "ACCIÓN URGENTE"
```

El estado ocupa una card hero en la parte superior de la página. Es el elemento de mayor jerarquía visual.

### StatusBadge
Pill de estado con color semántico. Usada en tablas, listas y cards.

- Fondo: versión 50 del color semántico
- Texto: versión 700 del color semántico
- Tamaño: `text-xs`, padding horizontal 8px, border-radius full
- Sin bordes explícitos — el contraste del fondo es suficiente

### Card
Contenedor de información con estructura interna consistente.

- Fondo: blanco
- Borde: `border border-gray-200`
- Radio: `rounded-lg` (8px)
- Sombra: `shadow-sm` — muy sutil, solo para elevar del fondo
- Padding interno: 24px
- Sin sombras dramáticas — la sombra es solo de separación del fondo

### Botones

| Variante | Uso | Estilo |
|----------|-----|--------|
| Primary | Acción principal de la página | `bg-brand-500 text-white hover:bg-brand-600` |
| Secondary | Acciones secundarias | `border border-gray-200 bg-white text-gray-700 hover:bg-gray-50` |
| Ghost | Acciones terciarias, navegación | `text-gray-600 hover:bg-gray-100` |
| Destructive | Acciones irreversibles | `bg-danger-500 text-white hover:bg-danger-600` |

- Solo puede haber un botón Primary por vista
- Los botones Destructive requieren un ConfirmDialog antes de ejecutar
- Altura de botones: 36px (sm), 40px (base), 44px (lg)
- Border-radius: `rounded-md` (6px) — no redondeado completo, no cuadrado

### Input / Form fields
- Altura: 40px
- Borde: `border border-gray-200`
- Fondo: blanco
- Focus: `ring-2 ring-brand-500 border-brand-500`
- Error: `border-danger-500 ring-danger-500` + mensaje debajo en `text-sm text-danger-700`
- Label: siempre visible, nunca solo placeholder

### Table
- Sin bordes en las celdas — solo una línea separadora entre filas (`border-b border-gray-100`)
- Fondo de header: `bg-gray-50`
- Hover de fila: `hover:bg-gray-50`
- Texto de header: `text-xs font-medium text-gray-500 uppercase tracking-wide`
- Las tablas siempre tienen un estado vacío definido

### Sidebar
- Ancho: 240px (fijo, no colapsable en el MVP)
- Fondo: blanco
- Borde derecho: `border-r border-gray-200`
- Items de navegación: 40px de altura, 12px padding horizontal
- Item activo: `bg-brand-50 text-brand-700 font-medium`
- Item hover: `bg-gray-50 text-gray-900`

---

## Estados de UI

Cada componente que carga, falla o puede estar vacío debe tener los tres estados implementados antes de considerarse completo.

### Loading
- Skeleton screens — nunca spinners en el centro de la página
- Los skeletons tienen la misma forma que el contenido real
- La animación es `animate-pulse` — sutil, no distractora
- Duración máxima esperada visible de loading: 1 segundo (si tarda más, algo está mal)

### Empty State
- Ilustración simple (SVG inline) + título + descripción + CTA cuando aplica
- El título describe el estado, no solo "No hay datos"
  - Bien: "Aún no tienes documentos en este período"
  - Mal: "Sin resultados"
- El CTA debe llevar al usuario a la acción correcta
- Cada lista y tabla tiene su propio empty state — no hay un empty state genérico

### Error State
- Mensaje claro de qué salió mal (sin stack traces visibles para el usuario)
- Acción de recuperación: "Intentar de nuevo" cuando aplica
- Para errores críticos: opción de contactar soporte
- Los errores de validación de formulario aparecen inline, no como toasts

### Success State
- Los éxitos se confirman con un cambio de estado visual en el elemento mismo
- Los toasts solo para confirmaciones de acciones que no tienen un elemento visual asociado
- Duración del toast de éxito: 3 segundos, auto-dismiss

---

## Iconografía

### Librería: Lucide React
Lucide es consistente, mantenida y diseñada para interfaces de software. Es la librería estándar de shadcn/ui.

### Reglas de uso
- Tamaño estándar: 16px para iconos en texto/botones, 20px para iconos standalone
- Los iconos siempre tienen `aria-hidden="true"` cuando son decorativos
- Los iconos funcionales (sin label visible) tienen `aria-label`
- No mezclar familias de iconos — solo Lucide

### Iconos del dominio

| Dominio | Icono principal |
|---------|----------------|
| Document | `FileText` |
| DocumentRequest | `ClipboardList` |
| Filing | `Receipt` |
| Organization | `Building2` |
| User / Client | `User` |
| Activity | `Activity` |
| Notification | `Bell` |
| Calendar / Deadline | `Calendar` |
| Upload | `Upload` |
| Status OK | `CheckCircle2` |
| Status Warning | `AlertTriangle` |
| Status Danger | `AlertCircle` |

---

## Accesibilidad

FiscalOS apunta a cumplir **WCAG 2.1 AA** como línea base.

### Contraste de color
- Texto principal sobre fondo: mínimo 4.5:1
- Texto grande (18px+): mínimo 3:1
- Iconos funcionales: mínimo 3:1
- Los StatusBadges deben cumplir el contraste mínimo en todos sus estados

### Navegación por teclado
- Todos los elementos interactivos son alcanzables por Tab
- El orden de Tab sigue el orden visual de la página
- Focus visible siempre — nunca `outline: none` sin un reemplazo
- Los modals atrapan el foco mientras están abiertos

### Lectores de pantalla
- Todos los formularios tienen labels asociados explícitamente
- Los iconos decorativos tienen `aria-hidden="true"`
- Los iconos funcionales tienen `aria-label`
- Las tablas tienen `<caption>` o `aria-label`
- Los estados de carga tienen `aria-busy="true"`

### Semántica HTML
- Se usa HTML semántico: `<nav>`, `<main>`, `<aside>`, `<header>`, `<section>`
- Los botones son `<button>`, no `<div>` con onClick
- Los links son `<a>`, no `<button>` con navigate
- Las listas son `<ul>/<li>`, no `<div>` apilados

---

## Responsive

FiscalOS es una aplicación web diseñada principalmente para desktop. El cliente puede usarla en mobile — el staff, rara vez.

### Breakpoints (Tailwind defaults)

| Breakpoint | Ancho | Uso |
|------------|-------|-----|
| `sm` | 640px | Smartphones en landscape |
| `md` | 768px | Tablets |
| `lg` | 1024px | Laptops |
| `xl` | 1280px | Desktops |
| `2xl` | 1536px | Pantallas grandes |

### Estrategia

**Mobile (< 768px) — Portal del cliente**
- El sidebar se colapsa en un bottom navigation bar de 4 items
- Las tablas se convierten en listas de cards
- Los modals ocupan pantalla completa
- El Peace of Mind Status es el 100% de la pantalla inicial

**Tablet (768px - 1024px)**
- El sidebar se mantiene pero más compacto (iconos + labels cortos)
- Las vistas de dos columnas se colapsan a una columna

**Desktop (> 1024px) — Experiencia primaria**
- Sidebar fijo de 240px
- Layouts de dos columnas para detalle/lista
- Tablas con todas las columnas visibles

**El workspace del contador (staff/admin) en mobile:**
- No está optimizado para mobile en el MVP
- Se muestra un mensaje indicando que la experiencia está diseñada para desktop

---

## Motion y animaciones

### Principio: animaciones funcionales, no decorativas
Las animaciones sirven para comunicar relaciones entre estados, no para entretener. Una transición que no comunica nada debe eliminarse.

### Duración
- Micro-interacciones (hover, focus): 100-150ms
- Transiciones de componentes (modals, drawers): 200ms
- Transiciones de página: 150ms
- No usar animaciones > 300ms — se perciben lentas

### Easing
- Entrada de elementos: `ease-out` — los elementos entran rápido y se frenan
- Salida de elementos: `ease-in` — los elementos salen despacio y se aceleran
- Transiciones de estado: `ease-in-out`

### Qué se anima
- Apertura y cierre de modals y drawers (slide + fade)
- Aparición de toasts (slide desde abajo)
- Hover en botones y cards (escala sutil: 1.0 → 1.01)
- Loading skeletons (pulse)
- Cambios de estado en StatusBadge (fade de color)

### Qué NO se anima
- Cambios de página completos — no hay page transitions complejas
- Listas de datos — no hay animaciones de inserción/eliminación en tablas
- Elementos que ya están en pantalla sin interacción del usuario

### Respeto a `prefers-reduced-motion`
Todas las animaciones deben respetar la preferencia del sistema operativo. Si el usuario ha indicado que prefiere movimiento reducido, las animaciones se desactivan o se reemplazan por transiciones de opacidad instantáneas.
