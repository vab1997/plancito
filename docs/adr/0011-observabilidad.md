# ADR 0011 — Observabilidad

**Estado:** aceptada · 2026-09-25

## Contexto

En este dominio un fallo silencioso no es una métrica fea: es un cliente que pagó y perdió un corte. Eso no se ve en un dashboard de latencia.

Escala chica y presupuesto chico.

## Decisión

| Qué | Para qué |
|---|---|
| **pino** | Logs estructurados en JSON |
| **Tracing de Effect** | Spans, casi gratis por usar Effect |
| **Logs de Vercel** | Retención y consulta en v1. Sin plataforma de logs aparte |
| **Vista "cosas que salieron mal"** | Reservas fallidas, activaciones con error, créditos vencidos con excepción en el medio |

**Dos reglas de integración:**

1. **Un solo sistema de logging.** Un `Layer` reemplaza el logger por defecto de Effect con pino, así los logs salen correlacionados con los spans en vez de por dos caminos paralelos.
2. **pino escribe JSON a stdout**, que Vercel captura solo. `pino-pretty` solo en dev. Los transports de pino (worker threads, archivos) no funcionan bien en serverless, y pino requiere runtime Node — no Edge.

## Consecuencias

- Esto da **registro, no aviso**. Nadie lee líneas de log por su cuenta. La vista de "cosas que salieron mal" hay que mirarla.
- Cuando haga falta que algo avise sin que nadie mire, entra Sentry o un digest diario por mail. Queda diferido a propósito.
