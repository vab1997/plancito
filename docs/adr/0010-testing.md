# ADR 0010 — Estrategia de testing

**Estado:** aceptada · 2026-09-25

## Contexto

El código lo escribe Claude Code y lo revisa una persona. Los tests dejan de ser red de seguridad y pasan a ser **lo que atrapa lo que la revisión no ve** — y el criterio objetivo de "terminado" para la IA.

No se busca cobertura total.

## Decisión

Tres niveles, cada uno con un trabajo distinto.

| Nivel | Herramienta | Qué cubre |
|---|---|---|
| **Unitario** | Vitest, sin base de datos | El dominio puro: ventanas, splits, ciclos, vencimientos, el arranque adelantado |
| **Integración** | Vitest + Postgres real (branch de Neon o Docker) | **Reservas concurrentes solapadas** · aislamiento de tenant · la transacción crédito+turno |
| **E2E** | Playwright | Dos o tres flujos completos: activar plan, reservar con crédito, reservar sin cuenta |

**El nivel de integración existe porque el bug más difícil del proyecto no entra en los otros dos:** dos reservas concurrentes con rangos solapados necesitan base de datos (no es unitario) y concurrencia, no un navegador (no es e2e). Mantenerlo chico: tres o cuatro tests. Cubren garantías, no comportamiento.

**Los tests del dominio se escriben como tablas de casos sacadas de `docs/propuesta-barbero.md`:**

- Los splits de 2, 3 y 4 turnos con sus ventanas.
- Los escenarios de "arranco hoy", incluido el arranque en el último día de la semana.
- Consumir el turno que vence antes cuando hay dos disponibles.
- El tope de turnos por semana con ventanas montadas.
- Pausa del ciclo cuando se cierra una semana entera.

Esos tests **son** la especificación ejecutable del negocio.
