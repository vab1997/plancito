# ADR 0006 — Fechas y zonas horarias

**Estado:** aceptada · 2026-09-25

## Contexto

La mitad del dominio es aritmética de fechas: ciclos anclados por suscriptor, ventanas de semanas, ancla igual al primer día de atención de la barbería, pausa del reloj, arranque adelantado.

El dominio razona en **fechas de calendario locales de la barbería** ("sábado 22:00", "la semana que arranca el martes"), pero un `Date` de JavaScript es un **instante en UTC**. Cada cruce entre esas dos ideas necesita una conversión explícita, y ahí viven los bugs — los que la revisión humana pasa de largo.

Argentina hoy no tiene horario de verano, lo que ahorra la mitad del dolor. No es razón para no modelarlo bien.

## Decisión

**`DateTime` de Effect, zoned.** Ya entra Effect al proyecto, así que no suma dependencia y está pensado para esto.

Tres reglas:

1. **Guardar en UTC.** Siempre.
2. **Calcular en la zona de la barbería.** La zona es parte del esquema de cada barbería, no una constante global.
3. **Nunca date math con strings**, y nunca comparar una fecha de calendario con un instante sin conversión explícita.

Descartadas: **Temporal con polyfill** (segunda opción válida, tipos distintos para instante y fecha de calendario) y **date-fns + date-fns-tz** (funciona, pero es la opción que menos protege desde el tipo, y en un código escrito por IA el compilador es el revisor principal).

## A verificar antes de cerrar

- El soporte de **fecha de calendario** de `DateTime` en la v4 RC. Si no alcanza para representar "el sábado de la barbería" sin arrastrar un instante, se cae a Temporal.
