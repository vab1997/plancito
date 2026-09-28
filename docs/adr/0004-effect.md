# ADR 0004 — Effect: dónde sí y dónde no

**Estado:** aceptada · 2026-09-25

## Contexto

El dominio está lleno de errores de negocio con nombre propio: crédito vencido, ventana cerrada, horario tomado, tope excedido, beneficio sin stock. Y el riesgo más grave de la arquitectura es una query sin filtro de tenant.

Además el código lo escribe Claude Code y lo revisa una persona. El criterio de calidad no es velocidad de escritura, es **qué se puede revisar sin haberlo escrito**.

## Decisión

Effect v4, **con la línea en un lugar preciso**.

| Capa | Qué usa | Por qué |
|---|---|---|
| **`domain/`** | TypeScript plano + **los módulos de datos puros de Effect** (errores etiquetados, tipo resultado, `DateTime`). **Sin `Effect<>`** | Motor de fechas, ventanas, ciclos y reglas. Tests como llamadas de función, sin runtime ni layers. Portable a React Native |
| **Casos de uso** | **`Effect<A, E, R>`** | Reservar, activar plan, canjear beneficio. Acá hay IO, tenant y errores de negocio |
| **Edges de Next.js** | Un adaptador fino que corre el effect y mapea errores tipados a respuestas o estado de UI | Que Server Actions y route handlers no sepan de Effect |

**Los dos beneficios concretos por los que se adopta:**

1. **`TenantContext` en el canal `R`.** Una query sin tenant no compila. Es una garantía estructural en el lugar exacto donde el error más duele (ADR 0002).
2. **Errores de negocio en el canal `E`.** Exhaustividad verificada por el compilador en vez de por revisión humana.

Usar los módulos de datos puros de Effect dentro del dominio **no viola** la regla de dependencias (ADR 0009): son valores inmutables, no IO.

## Consecuencias

- **Versión RC pineada exacta.** Sin rangos de semver.
- **Los datos de entrenamiento de la IA están cargados hacia v3.** Claude va a tender a escribir idioms viejos. Mitigación: los idioms elegidos van documentados en `CLAUDE.md` con ejemplos del propio repo.
- **Superficie de Effect chica a propósito.** Si el 80% del código es dominio puro y TypeScript plano, el 20% de Effect es revisable.

## A verificar el día uno, contra los docs de la v4 RC

- Cómo se llaman y comportan los módulos de datos puros en v4. La v4 reorganizó esa parte.
- Si el tipo resultado del dominio se puede *yield*ear directo dentro de un `Effect.gen`. Si sí, el borde dominio ↔ casos de uso sale gratis.
- La integración Drizzle ↔ Effect para la versión elegida.
