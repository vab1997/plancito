# ADR 0001 — Stack base

**Estado:** aceptada · 2026-09-25

## Contexto

MVP para validar el modelo de negocio con una barbería. Escala irrelevante: una barbería son ~150 turnos/mes; cien barberías, 15.000. El código lo escribe Claude Code y lo revisa una persona sola.

Lo difícil del dominio no es volumen: es aritmética de fechas, reserva sin doble-booking y aislamiento entre barberías.

## Decisión

| Pieza | Elección |
|---|---|
| Framework | Next.js (App Router) |
| Hosting | Vercel |
| Base de datos | Postgres en Neon |
| Driver | **`node-postgres` sobre el endpoint pooleado.** No `neon-http` |
| ORM | Drizzle |
| Backend | Effect v4 (ver ADR 0004) |
| Estilos | StyleX (ver ADR 0008) |
| Auth | better-auth (ver ADR 0007) |
| Plataforma | Web. React Native + Expo se evalúa después de validar |

Postgres es la decisión central: la garantía de no-doble-booking con duraciones variables se resuelve en la base, no en el código (ADR 0003).

### Por qué `node-postgres` y no `neon-http`

La reserva necesita **transacciones interactivas reales**: consumir el crédito e insertar el turno son atómicos, y si el insert viola el exclusion constraint el crédito no se puede haber quemado (ADR 0003). El driver HTTP no tiene el soporte necesario para eso.

## Consecuencias

- Un solo repo cubre las tres superficies: panel del barbero, app del suscriptor y formulario anónimo.
- **El `max` del pool tiene que ser chico — 1 o 2, no el default.** Cada instancia serverless de Vercel levanta su propio pool; contra el endpoint pooleado de Neon eso se tolera, pero no con pools grandes.
- pino y `node-postgres` necesitan **runtime Node**, no Edge. Las rutas que tocan base o loguean lo declaran explícito.
- Neon hace scale-to-zero. Un cold start de cientos de milisegundos es tolerable para este uso, pero conviene saberlo.
- La app cliente se construye como PWA instalable. Cubre el 90% de la sensación nativa y posterga React Native sin costo.

## A verificar

- Que **`db.transaction()` de Drizzle sobre `node-postgres`** dé transacciones interactivas reales contra el endpoint pooleado, y que el rollback deshaga de verdad (spike S01).
- Si el pooling en modo transaction se lleva bien con los **prepared statements** de Drizzle, o hay que desactivarlos.
- Que Neon exponga la extensión `btree_gist`.
- Límites de Vercel Cron por plan, **si** llegara a hacer falta un job (ADR 0005 dice que probablemente no).
