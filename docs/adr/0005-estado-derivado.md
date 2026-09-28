# ADR 0005 — Estado derivado en vez de jobs programados

**Estado:** aceptada · 2026-09-25

## Contexto

El dominio tiene muchas reglas temporales: turnos que vencen al cerrar su ventana, ciclos que renuevan, créditos que se otorgan, el reloj que se pausa cuando el barbero cierra una semana.

El reflejo es resolverlo con cron jobs, y eso arrastra: scheduler confiable, idempotencia (si corre dos veces no puede otorgar dos créditos), ledger append-only, observabilidad de jobs.

## Decisión

**Derivar el estado temporal al leerlo. No agendarlo.**

| Regla | Cómo se resuelve |
|---|---|
| El turno vence al cerrar su ventana | **Un turno cuya ventana cerró está vencido cuando lo leés.** Comparación de fecha |
| Otorgar los turnos del ciclo | Al arrancar el ciclo se escriben **los N turnos con sus ventanas**. Existir + comparar fecha |
| Renovar el ciclo | Lo dispara **el barbero al marcar el pago**. Es un evento |
| Pausar el ciclo por semana cerrada | Lo dispara **el barbero al bloquear la semana**. Es un evento |
| Lista de "cobrar hoy" | **Es una query** |

**Al arrancar cada ciclo se congela un *entitlement*:** una copia de lo que el plan otorgaba en ese momento — turnos, ventanas, beneficios enganchados, reglas de canje. La reserva lee el entitlement, nunca el plan vivo.

Eso es la forma en datos de la regla de negocio "los cambios aplican desde el próximo ciclo".

## Consecuencias

- **En v1, probablemente cero cron jobs.** Los recordatorios de WhatsApp están fuera del piloto. La pregunta de Vercel Cron pasa a ser una nota al pie.
- Sin jobs no hay problema de idempotencia que resolver, y el ledger append-only deja de ser necesario para eso.
- **La única excepción: los horarios retenidos.** Un `EXCLUDE` no puede tener un predicado con `now()`, así que una retención vencida sigue bloqueando el horario. Se resuelve con **limpieza perezosa dentro de la transacción de reserva**: antes de insertar se borran las retenciones vencidas que solapan con el rango pedido.

Ese es el único lugar del sistema donde el estado derivado se choca con la garantía de la base. Está identificado a propósito.
