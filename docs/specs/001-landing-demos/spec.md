# 001 — Landing con demos en vivo de presets-kits

- **Estado:** borrador
- **Fecha:** 2026-10-11
- **Fase:** SPEC (decisiones del propietario y correcciones QA incorporadas; pendiente de aprobación antes de plan.md y tasks.md)
- **Constitución:** `constitution.md` (raíz) prevalece sobre esta spec en caso de conflicto.

## Overview

Landing única en la ruta `/` que presenta las librerías `gsap-presets-kit` y `motion-presets-kit`. El visitante ve un hero animado, una galería de demos agrupada por librería, filtrable y con **vista previa animada al pasar sobre cada tarjeta**, y al activar una demo se abre un modal con la **demo en vivo**, **controles básicos** (replay + ajustes simples), **descripción del preset** y **snippet de código copiable**. El encabezado mantiene visibles los enlaces a las futuras secciones `/docs` y `/playground`, marcados como «próximamente» y sin navegación (RF-1). Recorrido deseado: ver → probar → copiar, sin instalar nada. No existen URLs por demo: el modal se abre únicamente por interacción directa en la galería.

## Goals

1. Presentar el valor de ambas librerías: presets de animación tipados, validados y listos para usar.
2. Permitir probar cada preset en vivo con controles básicos y copiar el código ya configurado.
3. Estética brutalista conforme a `DESIGN.md`: tipografía enorme, elementos irregulares, estructura no uniforme, paleta primary/dark/light.
4. Accesibilidad: respeto de `prefers-reduced-motion` del sistema en hero, vistas previas y demos, modal accesible, navegación por teclado.
5. Responsive de móvil a escritorio.
6. Calidad: lógica pura cubierta por tests unitarios con Vitest, única herramienta de pruebas del proyecto.

## Non-Goals

- **Sitio de documentación completo** (páginas individuales por preset, referencia API exhaustiva, guías MDX).
- **Deep-links a demos:** no existe URL por demo (`/?demo=<id>`); el modal se abre solo por interacción y no se sincroniza con la URL.
- **Suite E2E con Playwright** (explícitamente fuera de alcance).
- **Tests de componentes/UI** (React Testing Library, jsdom): la estrategia de pruebas es solo Vitest sobre lógica pura.
- i18n: el contenido es solo en español.
- Playground exhaustivo con todos los parámetros de cada preset (solo controles básicos).
- Blog, changelog, analítica, toggle de tema/dark mode.
- **Secciones `/docs` y `/playground`:** no existen como rutas; sus enlaces del encabezado quedan «próximamente» sin navegación (RF-1).
- **Toggle manual de reduced-motion:** no hay conmutador ni preferencia persistida; solo se respeta la preferencia del sistema (RF-9).

## User Stories

- Como visitante, quiero ver de un vistazo qué hacen las dos librerías, para decidir si me interesan.
- Como visitante que recorre la galería, quiero ver una vista previa animada del preset al pasar sobre su tarjeta, para saber qué hace sin necesidad de abrirla.
- Como desarrollador front-end, quiero abrir una demo en vivo y trastear con duración/dirección/velocidad, para entender el preset sin instalar nada.
- Como desarrollador, quiero copiar el snippet de código tal como lo configuré, para pegarlo en mi proyecto.
- Como usuario sensible al movimiento, quiero que el sitio respete la preferencia `prefers-reduced-motion` de mi sistema, para navegar sin molestias (tarjetas incluidas).
- Como usuario móvil, quiero la misma experiencia usable (galería, filtros, modal), para explorar desde el teléfono.

## Functional Requirements

### RF-1 — Landing única en `/`

**Mientras** el sitio esté desplegado, toda la experiencia se servirá en la ruta `/` (una sola página, sin rutas adicionales). **Cuando** el usuario visite `/`, el sistema deberá mostrar: encabezado con los enlaces a las futuras secciones `/docs` y `/playground` visibles pero marcados como **«próximamente»** (deshabilitados o con indicador visual de no disponible, sin navegar), hero (RF-3), presentación de ambas librerías, galería de demos con filtros (RF-4/5) y pie de página. La página deberá tener título y meta descripción en español. La URL no variará según la demo abierta: no existen deep-links a demos.

- *Caso límite:* los enlaces «próximamente» no conducen a ninguna ruta (no generan un 404): quedan no operables y comunican su estado tanto visual como semánticamente (p. ej. `aria-disabled`), manteniéndose visibles en el encabezado.
- *Caso límite:* cualquier URL inexistente usa el 404 por defecto de Astro; no se construyen rutas nuevas.
- *Caso límite:* parámetros de query desconocidos u obsoletos (p. ej. `?demo=<id>`) se ignoran sin error y sin abrir modal; ninguna interacción del usuario modifica la URL ni el historial del navegador.

### RF-2 — Cobertura del catálogo de demos

El sistema deberá incluir **una demo por cada preset animable** de ambas librerías:

- `gsap-presets-kit`: `animateFrom`, `animateTo`, `animateFromTo`, `splitTextReveal`, `typewriter`, `marquee`, `counter`, `parallax`, `clipReveal`, `scrollTimeline`.
- `motion-presets-kit`: `fade`, `slide`, `scale`, `clipReveal`, `parentVariants`, `TextAnimate`, `Marquee`, `Draggable`, `useCounter`.

`presetContext` es utilidad de limpieza (no preset animable): no tendrá demo propia y se mencionará como nota en las demos GSAP que la usan.

- *Caso límite:* presets homónimos entre librerías (`clipReveal`, `marquee`/`Marquee`) se distinguirán siempre por su librería (agrupación y etiqueta).

### RF-3 — Hero animado con preset configurable

**Cuando** la página cargue, el hero deberá animarse con un preset protagonista y seguir la estética brutalista (tipografía muy grande, composición irregular). El preset protagonista **no se fija en esta spec**: será una referencia configurable declarada aparte del registro de demos (p. ej. una constante `HERO_PRESET_ID` en el mismo módulo del catálogo) que apunta al `id` de una demo existente; cambiarlo requerirá actualizar únicamente esa referencia, sin editar la página ni otras demos. **Si** el usuario tiene `prefers-reduced-motion: reduce`, el hero deberá mostrarse directamente en su estado final sin animación.

- *Caso límite:* si la referencia configurada no resuelve a un preset válido del registro, el hero deberá degradar a un estado estático correcto sin romper el render.

### RF-4 — Galería agrupada por librería con vista previa animada

El sistema deberá mostrar la galería agrupada por librería (`gsap-presets-kit` / `motion-presets-kit`). Cada tarjeta deberá mostrar: nombre del preset, categoría, descripción de una línea, etiqueta de librería y un área de **vista previa del preset**. **Cuando** el usuario pase el puntero sobre una tarjeta o la enfoque con teclado, el sistema deberá activar en esa área una vista previa animada y ligera del preset (versión simplificada, no la demo completa). **Cuando** el usuario active una tarjeta (clic, Enter o Espacio), el sistema deberá abrir el modal de esa demo (RF-6).

- *Caso límite:* **si** el reduced-motion está activo (sistema, RF-9), la vista previa no se anima: la tarjeta se muestra estática.
- *Caso límite:* en dispositivos de solo táctil / sin hover, el área de vista previa renderiza el **primer fotograma estático** del preset (sin animación); la animación en vivo vive en el modal (RF-6), al que se llega activando la tarjeta.
- *Caso límite:* los presets dirigidos por scroll (p. ej. `parallax`, `scrollTimeline`) ofrecerán una vista previa aproximada del efecto sin requerir scroll real.
- *Caso límite:* con el filtro de librería activo, solo se muestra el grupo filtrado.

### RF-5 — Filtros combinables por librería y categoría

**Cuando** el usuario aplique filtros, el sistema deberá actualizar la galería sin recarga de página combinando las dimensiones entre sí por **intersección** (librería **Y** categoría). Dentro de cada dimensión la selección será **única**: exactamente una opción activa en cada momento, con la opción **«Todos»** (valor por defecto) que anula el filtro de esa dimensión; seleccionar una opción sustituye a la activa previa de esa misma dimensión, nunca la acumula. Las categorías serán: `entrada`, `salida`, `texto`, `scroll`, `loop`, `interacción` (cada demo tiene exactamente una categoría principal). Los filtros activos deberán ser visibles, operables por teclado y desactivables individualmente (volviendo a «Todos» esa dimensión).

- *Caso límite:* si la intersección de dimensiones no produce resultados, el sistema deberá mostrar un estado vacío con mensaje y una acción para restablecer los filtros («Todos» en ambas dimensiones).
- *Caso límite:* «Todos» en ambas dimensiones equivale a la galería completa (sin filtro), que es el estado por defecto.

### RF-6 — Modal de demo accesible, abierto solo por interacción

**Cuando** el usuario abra una demo desde la galería, el sistema deberá mostrar un modal accesible con `role="dialog"` y `aria-modal="true"`, etiquetado por el título de la demo mediante `aria-labelledby` (constitución §1), que contenga: demo en vivo (RF-7), título, descripción de qué hace el preset y sus opciones relevantes, snippet copiable (RF-8) y controles (RF-7). **Al abrirse**, el foco deberá situarse dentro del modal. **Mientras** el modal esté abierto, el sistema deberá atrapar el foco dentro del modal (el Tab no sale de él), bloquear el scroll del fondo y permitir el cierre con `Escape` y con un botón visible; **al cerrar**, el foco deberá devolverse a la tarjeta de origen.

- *Caso límite:* solo puede haber un modal abierto a la vez; abrir otra demo sustituye al actual.
- *Caso límite:* el modal no es direccionable por URL (no existe `/?demo=<id>`) ni se sincroniza con el historial; una recarga con el modal abierto devuelve la página a su estado inicial.

### RF-7 — Demo en vivo con controles básicos

**Cuando** el modal se abra, la demo deberá ejecutarse con los valores por defecto del preset. El sistema deberá ofrecer un control de **replay** y, cuando aplique, ajustes simples por demo (p. ej. duración, delay, dirección, distancia, velocidad) limitados a un subconjunto pequeño de opciones. **Cuando** el usuario cambie un control o pulse replay, la demo deberá reiniciarse reflejando la nueva configuración. **Si** un valor queda fuera del rango permitido, el sistema deberá acotarlo (clamp) antes de aplicarlo.

- *Caso límite:* demos sin opciones relevantes muestran solo replay; las opciones disponibles dependen de la API real de cada preset (no se inventan parámetros).

### RF-8 — Snippet copiable sincronizado con los controles

El sistema deberá mostrar el snippet de código del ejemplo **reflejando la configuración actual de los controles** y un botón "Copiar" con confirmación visual de copiado. **Si** la API de portapapeles falla o no está disponible, el sistema deberá seleccionar el texto del snippet y avisar al usuario.

### RF-9 — Reduced motion: solo preferencia del sistema

**Si** el usuario tiene `prefers-reduced-motion: reduce`, el sistema deberá desactivar las animaciones (estado final o alternativa estática) en hero, vistas previas de tarjetas y demos, apoyándose en el soporte `reducedMotion` de ambos kits (nunca sobrescribirlo a `false` desde la landing). La detección se realiza **exclusivamente** contra la preferencia del sistema: no existe conmutador manual ni preferencia persistida entre visitas. El mismo criterio se aplicará de forma consistente al hero y a la galería/isla de React.

- *Caso límite:* la detección en SSR no debe romper el render (ambos kits son SSR-safe; la landing no debe asumir `window` en el servidor).
- *Caso límite:* **si** el sistema reporta `no-preference` o no reporta nada, las animaciones se ejecutan con normalidad; no existe ningún otro mecanismo que las desactive.

### RF-10 — Responsive

El sistema deberá ser usable en móvil, tablet y escritorio: rejilla con reflow, tipografía fluida, modal a ancho completo en móvil y controles operables por táctil.

## Technical Approach

*(Resumen de decisiones; el detalle de archivos va en `plan.md`.)*

- **Rutas:** solo `/` (`src/pages/index.astro`). Sin content collections, sin middleware, sin i18n y sin deep-links: el modal no modifica la URL ni el historial.
- **Astro vs React:** el shell estático en Astro (layout, encabezado, encabezados de sección, pie). Hero animado con `gsap-presets-kit` desde un `<script>` de Astro leyendo `prefers-reduced-motion` del sistema (alternativa descartada: isla React para el hero — añade JS sin beneficio). Una única isla React `DemoExplorer` (filtros + galería + modal) hidratada con `client:visible` (sección below-the-fold tras el hero).
- **Isla única vs constitución §2:** la isla `DemoExplorer` es **ligera**: contiene solo la UI de tarjetas, filtros y modal. El **catálogo de metadatos (datos tipados) se importa estáticamente** — es el índice que alimenta esa UI —, pero **ningún componente de demo se hidrata junto con la galería**: cada demo concreta se carga como módulo lazy dinámico (`import()`) bajo demanda al abrir el modal. Así no se hidrata ni se descarga toda la galería de demos de golpe, cumpliendo la constitución §2 (alternativas descartadas: una isla por tarjeta — N hidrataciones y estado compartido más complejo; una isla que empaquete las demos — hidrataría todas las demos de golpe, violando §2).
- **Vistas previas de tarjetas:** animaciones ligeras disparadas por hover/focus desde la propia galería, sin cargar el componente de demo completo; bajo reduced-motion del sistema la tarjeta es estática (alternativa descartada: tarjetas estáticas — decidido por el propietario: preview animada); en dispositivos sin hover, el área muestra el primer fotograma estático del preset (RF-4). La demo completa solo se carga al abrir el modal.
- **Hero configurable:** el preset protagonista se declara como una constante/entrada aparte del registro, en el mismo módulo del catálogo (p. ej. `HERO_PRESET_ID`), que apunta al `id` de una demo existente (única fuente de verdad); si no resuelve, fallback estático (RF-3). Cambiarlo no requiere editar la página ni otras demos (constitución §5–6).
- **Demos:** componentes React cargados perezosamente al abrir el modal (importación dinámica `import()`), cada demo como módulo independiente (nunca toda la galería de golpe). Limpieza de demos GSAP con `presetContext()` en `useEffect` (alternativa descartada: `@gsap/react` — evitar dependencia nueva). Las demos motion usan los presets/variantes directamente.
- **Catálogo de demos:** registro tipado en TS como única fuente de verdad: `{ id, library: 'gsap' | 'motion', presetName, category, title, description, controls: [{ key, type: 'range' | 'select' | 'toggle', min?, max?, step?, options? }], snippet, preview (ligera), demo (lazy) }`. El registro (datos) se importa estáticamente; la entrada `demo` es la única vía de carga del componente, diferida al modal. Alimenta galería, filtros, modal, controles, generación de snippets (función pura a partir de los valores actuales) y la elección del preset del hero (`HERO_PRESET_ID`). TypeScript estricto, sin `any`.
- **Estilos:** Tailwind v4 CSS-first; paleta y tipografía como tokens en `src/styles/global.css` (`@theme`), conforme a `DESIGN.md`; nada ad-hoc.
- **Pruebas (solo Vitest):** nueva devDependency + config en entorno Node — sin jsdom ni React Testing Library. Tests unitarios de lógica pura: integridad del registro (ids únicos, categorías válidas, cobertura RF-2, snippet presente, `HERO_PRESET_ID` resuelta a un `id` de demo existente), filtrado por intersección de dimensiones con selección única por dimensión (incluye caso vacío), generación de snippet desde valores de controles y clamp de controles. Sin E2E.

## Out of Scope

- Playwright/Cypress o cualquier test E2E.
- Rutas adicionales a `/` y deep-links a demos (`/?demo=<id>`): decidido NO por el propietario; el modal se abre solo por interacción en la galería.
- Tests de renderizado de componentes (React Testing Library, jsdom, happy-dom): solo Vitest sobre lógica pura.
- Páginas de documentación por preset, MDX, referencia API completa.
- Búsqueda textual en el catálogo (solo filtros RF-5).
- Toggle de tema/dark mode, blog, changelog, analítica.
- Toggle manual de reduced-motion y persistencia de preferencia de movimiento: decidido NO por el propietario; solo `prefers-reduced-motion` del sistema (RF-9).
- Rutas `/docs` y `/playground`: no existen; sus enlaces del encabezado quedan «próximamente» sin navegación (RF-1).

## Open Questions

Ninguna pendiente. Las cuatro preguntas previas y los hallazgos de la revisión QA quedaron resueltos por decisión del propietario (2026-10-11):

1. **Deep-links:** NO. Sin URL por demo; el modal se abre solo por interacción en la galería (RF-1, RF-6).
2. **Vista previa en tarjetas:** SÍ, animada al hover/focus, ligera y estática bajo reduced-motion; primer fotograma estático en dispositivos sin hover (RF-4, RF-9).
3. **Preset del hero:** no se fija en la spec; configurable mediante `HERO_PRESET_ID` en el mismo módulo del catálogo, con fallback estático si no resuelve (RF-3).
4. **Tests:** solo Vitest sobre lógica pura; sin React Testing Library ni jsdom (Technical Approach).
5. **Enlaces `/docs` y `/playground`:** visibles en el encabezado como «próximamente», deshabilitados o con indicador de no disponible, sin navegar a 404 (RF-1).
6. **Reduced-motion:** sin toggle manual ni persistencia de preferencia; solo la detección de `prefers-reduced-motion` del sistema, consistente entre hero y galería (RF-9).
7. **Filtros:** selección única por dimensión (librería, categoría) con opción «Todos», combinadas entre sí por intersección (RF-5).
