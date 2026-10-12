# constitution.md — Reglas duras del proyecto

Principios inmutables de `presets-kits`. Ninguna spec, plan, tarea o implementación puede violarlos; en caso de conflicto, prevalece este documento. Modificar esta constitución exige una decisión explícita del propietario del proyecto, nunca un efecto secundario de una tarea.

## 1. Accesibilidad primero

- Toda animación respeta `prefers-reduced-motion` y ofrece una alternativa sin movimiento (o estática).
- Modales con ARIA correcto: `role="dialog"`, `aria-modal`, foco atrapado dentro y restaurado al cerrar.
- Navegación por teclado completa en todo control interactivo (Tab, Enter, Espacio, Esc).

## 2. Rendimiento

- Las islas React se hidratan solo cuando son necesarias.
- No hidratar toda la galería de demos de golpe: cada demo carga como isla independiente.
- `client:visible` por defecto para demos que no están above-the-fold; `client:load` solo para islas críticas al cargar.

## 3. Calidad

- TypeScript estricto, sin `any`.
- Toda lógica pura (registro del catálogo, utilidades, transformaciones de datos) tiene tests unitarios con Vitest.
- Sin E2E de Playwright en este proyecto por ahora; añadirlo requiere decisión explícita.

## 4. Consistencia visual

- `DESIGN.md` es la fuente de verdad de tokens y estilos; la paleta y tipografía viven en `src/styles/global.css`.
- No introducir colores, fuentes ni espaciados ad-hoc fuera de esos tokens.

## 5. Separación de concerns

- Los datos del catálogo de presets viven en un registro tipado, separado de la UI que los renderiza.
- La UI consume el registro; no define datos de catálogo inline en componentes.

## 6. Mantenibilidad

- Cada demo de preset es autocontenido y registra sus metadatos: librería, categoría y props editables.
- Añadir un preset nuevo no debe requerir modificar demos existentes, solo el registro.
