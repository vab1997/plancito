# ADR 0003 — La base garantiza que no haya doble-booking

**Estado:** aceptada · 2026-09-25

## Contexto

Dos personas tocando el mismo horario al mismo tiempo es el bug más probable del sistema, y el formulario anónimo lo hace más probable porque no hay sesión que serialice nada.

Con duración fija por turno alcanzaba un `unique (barber_id, fecha, hora)`. Con **duración variable por servicio** — 30, 45, 60 minutos — eso deja de servir: el problema pasa a ser **solapamiento de rangos**.

## Decisión

Un **exclusion constraint** en Postgres. La garantía vive en la base, no en código de aplicación.

```sql
CREATE EXTENSION IF NOT EXISTS btree_gist;

ALTER TABLE bookings ADD CONSTRAINT bookings_no_overlap
  EXCLUDE USING gist (barber_id WITH =, durante WITH &&);
```

Tres reglas que lo acompañan:

1. **Rangos semiabiertos `[inicio, fin)`.** Así dos turnos pegados no se detectan como solapados.
2. **Por `barber_id`**, no por `shop_id`. Varios sillones pueden trabajar en paralelo.
3. **Nunca chequear-y-después-insertar en código.** Se intenta el insert y se maneja la violación del constraint.

## Consecuencias

- Consumir el crédito e insertar el turno van en **una transacción**. Si el insert falla porque alguien se adelantó, el crédito no se puede haber quemado. Eso es lo que obliga a `node-postgres` en vez del driver HTTP (ADR 0001).
- **La violación del constraint se detecta por el código de error de Postgres, no por el texto del mensaje.** Hay que poder distinguir "este horario se ocupó" — que es un caso de negocio esperado y se le muestra al cliente — de cualquier otro fallo de base.
- El `EXCLUDE` no puede tener un `WHERE` con `now()`: los predicados de índice tienen que ser inmutables. Por eso las retenciones vencidas se limpian de forma perezosa dentro de la transacción de reserva (ADR 0005).
- El test de integración de reservas concurrentes es obligatorio (ADR 0010).

## A verificar

- Si Drizzle Kit puede expresar `EXCLUDE` en su DSL. Lo esperable es que no: va como migración SQL cruda.
