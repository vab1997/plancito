# ADR 0009 — Arquitectura del código

**Estado:** aceptada · 2026-09-25

## Contexto

El proyecto tiene que escalar en features sin volverse ilegible, y lo revisa una persona que no lo escribió. La legibilidad no es estética: es la principal defensa contra bugs.

## Decisión

**Feature-based**, con los nombres de carpeta gritando el dominio. Pero lo que define la calidad no es el layout, es **la regla de dependencias**.

```
src/
  domain/          ← puro. Fechas, ventanas, ciclos, reglas. Sin IO
  features/
    agenda/        horarios, bloqueos, disponibilidad
    booking/       reserva, concurrencia, cancelación
    plans/         planes, splits, ventanas
    entitlements/  ciclos, créditos, vencimientos
    benefits/      catálogo, canje
    shops/         esquema del negocio, tenant
    clients/
  app/             ← rutas finas
```

**Las reglas, en orden de importancia:**

1. **`domain/` no importa nada de `features/`, ni la base de datos, ni IO.** Puede importar los módulos de datos puros de Effect (ADR 0004).
2. **Ninguna feature importa los internals de otra.** Solo su interfaz pública.
3. **`app/` es fino.** Un adaptador que corre el caso de uso y mapea el resultado.

## Consecuencias

- El motor de fechas y ventanas queda testeable sin base de datos, y es el módulo donde los tests más rinden porque es donde van a estar los bugs.
- Si más adelante entra React Native, `domain/` se porta sin tocar nada.
