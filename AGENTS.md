# AGENTS.md

Guía operativa de `presets-kits` — sitio Astro 7 + React 19 + Tailwind v4 + TypeScript que presenta y documenta las librerías `gsap-presets-kit` y `motion-presets-kit`.

**`constitution.md` (raíz) es la fuente de verdad de las reglas duras del proyecto. Si algo aquí la contradice, gana la constitución.**

## Entorno

- Node >= 22.12 y pnpm.
- Alias de importación: `@/*` → `src/*`.
- Librerías de animación ya instaladas: `gsap-presets-kit` y `motion-presets-kit` (usar sus skills del agente antes de escribir animaciones).

## Comandos clave

- `pnpm dev` — desarrollo en background: lanzar con `astro dev --background`; gestionar con `astro dev stop`, `astro dev status`, `astro dev logs`.
- `pnpm build` — build de producción.
- `pnpm preview` — servir `dist/`.
- `pnpm test` / `pnpm test:watch` — tests unitarios con Vitest.
- `pnpm lint` / `pnpm format` — lint y formato con Biome.

> Nota: los scripts de test y de Biome aún no están definidos en `package.json`; son los comandos canónicos del proyecto y se añaden en la primera tarea de implementación.

## Convenciones

- **Astro** para páginas y layout estático (`src/pages`, `src/layouts`); nada de React donde basta HTML estático.
- **React** solo como islas interactivas: `client:load` si la isla es crítica al cargar, `client:visible` para el resto.
- **Tailwind v4** CSS-first: tokens en `@theme` (`src/styles/global.css`), sin `tailwind.config.js`.
- **TypeScript estricto** (preset `astro/tsconfigs/strict`).

## Reglas de código

- Prohibido `any`: usa tipos explícitos o `unknown` con narrowing.
- Componentes pequeños, con una sola responsabilidad.
- La lógica pura (testeable con Vitest) vive separada de los componentes React.
- Estilos según `DESIGN.md` y los tokens de `global.css`; nada de valores ad-hoc.

## Specs (flujo SDD)

Flujo: **spec → clarificación → plan/tareas → implementación por tareas → validación**.

Estructura: `docs/specs/NNN-nombre/` (NNN = siguiente número secuencial libre):

- `spec.md` — requisitos funcionales en sintaxis EARS (QUÉ y POR QUÉ).
- `plan.md` — plan técnico: archivos, componentes, pruebas y RF cubiertos.
- `tasks.md` — máximo 10 tareas ordenadas por dependencia, con RF asociados y criterio "Hecho cuando:".

## Documentación

- Astro: <https://docs.astro.build> (consultar guías de routing, componentes e islas antes de tareas relacionadas).
