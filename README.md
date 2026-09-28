# Turno Fijo

Planes de suscripción para negocios que trabajan por turno. El dueño arma su propio **esquema de trabajo** — agenda, servicios, duraciones, precios, planes y beneficios — y sus clientes fijos compran un plan mensual que les reserva el horario.

Primer rubro: barberías.

---

## Por dónde empezar

| Documento | Para qué |
|---|---|
| [`CLAUDE.md`](CLAUDE.md) | Las reglas del repo. Léelo antes de escribir código |
| [`docs/propuesta-barbero.md`](docs/propuesta-barbero.md) | La especificación del dominio: ciclos, ventanas, splits, créditos, beneficios |
| [`docs/backlog.md`](docs/backlog.md) | Qué construir, en qué orden, y qué bloquea qué |
| [`docs/adr/`](docs/adr/README.md) | Las decisiones técnicas y su razón |
| [`docs/analisis-interno.html`](docs/analisis-interno.html) | El análisis de viabilidad y la economía del modelo |

## Estado

Fase de diseño. Todavía no hay código.

El próximo paso del backlog es **F01 — Bootstrap**, y en paralelo el dominio puro (F02, F03), que no depende de infraestructura.

## Stack

Next.js · Postgres en Neon con Drizzle · Effect v4 · better-auth · StyleX · Vitest y Playwright.

El detalle y el porqué de cada elección está en los ADR.
