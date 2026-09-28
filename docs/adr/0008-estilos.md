# ADR 0008 — Estilos y componentes

**Estado:** aceptada · 2026-09-25

## Contexto

La app necesita una identidad visual propia y, del otro lado, el panel del barbero es la UI más pesada del proyecto: grilla de agenda, calendario, selectores de fecha, constructor de planes con el split de ventanas, catálogo de beneficios.

## Decisión

**StyleX** para los estilos, partiendo de CSS propio desde cero. **Primitivas headless** (Radix o Base UI) para el comportamiento y la accesibilidad de los componentes.

StyleX y Tailwind producen los dos CSS atómico en build time, así que **la elección no se justifica por performance** — en un formulario de reserva la diferencia es ruido. Se justifica por querer un design system propio sin el acoplamiento de un framework de utilidades.

## Consecuencias

- El costo cae en el panel del barbero: menos componentes listos para usar que con un ecosistema tipo shadcn.
- **El calendario y el campo de fecha son el componente caro.** Todo lo demás se resuelve con primitivas headless y CSS propio; una grilla de agenda con teclado y accesibilidad, no.
- StyleX tiene menos presencia en los datos de entrenamiento de la IA. Los patrones elegidos van en `CLAUDE.md` temprano, con ejemplos del repo, para que no haya cinco estilos distintos en el mismo proyecto.

## A verificar antes de elegir la librería de primitivas

- Cuál de **Radix, Base UI o React Aria** trae headless de **calendario y campo de fecha**. Define si el panel del barbero lleva días o semanas.
- El estado del plugin de StyleX con App Router y Turbopack.
