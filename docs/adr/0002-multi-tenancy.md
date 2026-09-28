# ADR 0002 — Multi-tenancy: shared schema, tenant = barbería

**Estado:** aceptada · 2026-09-25

## Contexto

El modelo tiene que servir a cualquier barbería, no solo a la primera. Se despliega para una, pero sumar la segunda no puede pedir cambios de código.

Primero se había pensado `barber_id` como clave de tenant. Está mal: se rompe en el caso de una barbería con varios sillones, que ya está contemplado como futuro.

## Decisión

**Shared schema con columna discriminadora.** Una base, un esquema, `shop_id` en toda la data de negocio.

| Concepto | Modelado |
|---|---|
| **Tenant** | `shop_id` — la barbería. Va en todas las tablas de negocio |
| **Barbero** | Recurso dentro de la barbería. Tiene su propia agenda |
| **Usuarios** | **Globales, no scopeados.** Una persona puede tener plan en dos barberías |
| **Membresías y suscripciones** | Scopeadas. Vinculan un usuario global con una barbería |

Regla corta: **`shop_id` en toda la data de negocio; `users` global.**

## Consecuencias

- El riesgo principal de shared schema es un `WHERE` olvidado: una query sin filtro de tenant filtra la agenda de otra barbería.
- La defensa es estructural, no de disciplina: el `TenantContext` vive en el canal `R` de Effect, y **una query sin tenant no compila** (ADR 0004).
- Ningún código de feature accede a las tablas directo. Todo pasa por la capa de datos scopeada.
- El constraint de solapamiento es **por barbero**, no por barbería: dos sillones pueden estar ocupados a la misma hora (ADR 0003).

## Diferido

Onboarding self-service de barberías, cobro del SaaS, branding por local, roles dentro de una barbería.
