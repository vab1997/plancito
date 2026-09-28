# Decisiones de arquitectura

Una decisión por archivo, con su razón y sus consecuencias. La idea es no volver a discutir lo mismo en la sesión 20.

| # | Decisión |
|---|---|
| [0001](0001-stack-base.md) | Stack base |
| [0002](0002-multi-tenancy.md) | Multi-tenancy: shared schema, tenant = barbería |
| [0003](0003-concurrencia-reservas.md) | La base garantiza que no haya doble-booking |
| [0004](0004-effect.md) | Effect: dónde sí y dónde no |
| [0005](0005-estado-derivado.md) | Estado derivado en vez de jobs programados |
| [0006](0006-fechas.md) | Fechas y zonas horarias |
| [0007](0007-auth.md) | Autenticación e identidad |
| [0008](0008-estilos.md) | Estilos y componentes |
| [0009](0009-arquitectura.md) | Arquitectura del código |
| [0010](0010-testing.md) | Estrategia de testing |
| [0011](0011-observabilidad.md) | Observabilidad |

## Qué construir

- [`../backlog.md`](../backlog.md) — features ordenadas por dependencia, con el hilo mínimo del MVP y los spikes que van primero.

## Contexto de negocio

- [`../propuesta-barbero.md`](../propuesta-barbero.md) — el modelo completo: esquema de trabajo, ciclos, ventanas, catálogo de beneficios.
- [`../analisis-interno.html`](../analisis-interno.html) — el análisis de viabilidad y la economía.

## Si una decisión cambia

No se edita el ADR: se escribe uno nuevo que la reemplace y se marca el viejo como **superada por ADR NNNN**.
