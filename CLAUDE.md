# Turno Fijo

App de planes de suscripción para barberías. El barbero arma su propio **esquema de trabajo** — agenda, servicios, duraciones, precios, planes con sus ventanas, catálogo de beneficios — y sus clientes fijos compran un plan mensual que les reserva el horario.

**Antes de escribir código de negocio, leé [`docs/propuesta-barbero.md`](docs/propuesta-barbero.md).** Es la especificación del dominio: ciclos, ventanas, splits, créditos, beneficios. Las decisiones técnicas y su razón están en [`docs/adr/`](docs/adr/README.md), y qué falta construir en [`docs/backlog.md`](docs/backlog.md).

---

## Reglas no negociables

### 1. La regla de dependencias

```
src/
  domain/          ← puro. Sin IO, sin base de datos, sin features
  features/        ← agenda · booking · plans · entitlements · benefits · shops · clients
  app/             ← rutas finas
```

- **`domain/` no importa nada de `features/`, ni la base de datos, ni IO.** Sí puede importar los módulos de datos puros de Effect.
- **Ninguna feature importa los internals de otra.** Solo su interfaz pública.
- **`app/` es un adaptador:** corre el caso de uso y mapea el resultado. Nada de lógica.

### 2. El tenant es la barbería

- **`shop_id` en toda la data de negocio.**
- **`users` es global, no scopeado.** Una persona puede tener plan en dos barberías.
- Las membresías y suscripciones sí son scopeadas.
- **Nunca accedas a una tabla de negocio directo.** Todo pasa por la capa de datos scopeada por tenant. El `TenantContext` vive en el canal `R` de Effect, así una query sin tenant no compila.
- **El barbero es un recurso dentro de la barbería**, no el tenant. Un local puede tener varios sillones.

### 3. Effect: dónde sí y dónde no

| Capa | Qué usar |
|---|---|
| `domain/` | TypeScript plano + módulos de datos **puros** de Effect (errores etiquetados, tipo resultado, `DateTime`). **Nunca `Effect<>`, nunca servicios** |
| Casos de uso | `Effect<A, E, R>`. Acá van IO, tenant y errores de negocio |
| Edges de Next.js | Un adaptador fino que corre el effect y mapea errores tipados a respuesta o estado de UI |

- **Versión RC pineada exacta.** Sin rangos de semver.
- **Effect v4, no v3.** Si un idiom te suena de v3, verificalo contra los docs de v4 antes de escribirlo. Este es el error más probable en este repo.
- Superficie de Effect chica a propósito: si el dominio es puro y grande, lo que hay que revisar es poco.

### 4. Fechas

- **Guardar en UTC. Siempre.**
- **Calcular en la zona de la barbería.** La zona es parte del esquema de cada barbería, no una constante.
- **Nunca date math con strings.** Nunca comparar una fecha de calendario con un instante sin conversión explícita.
- Librería: **`DateTime` de Effect, zoned.**

### 5. La reserva

- **La garantía de no-doble-booking vive en la base**, con un exclusion constraint. No se chequea-y-después-se-inserta en código: se intenta el insert y se maneja la violación **por código de error de Postgres, nunca por el texto del mensaje**.
- **Driver: `node-postgres` sobre el endpoint pooleado de Neon**, con `max` chico (1 o 2). No `neon-http`: la transacción interactiva es innegociable.
- **Rangos semiabiertos `[inicio, fin)`**, para que dos turnos pegados no colisionen.
- El constraint es **por `barber_id`**, no por `shop_id`.
- **Consumir el crédito e insertar el turno van en una sola transacción.** Si el insert falla, el crédito no se puede haber quemado.
- Las **retenciones vencidas se limpian de forma perezosa dentro de esa transacción**, porque un `EXCLUDE` no puede tener un predicado con `now()`.

### 6. Estado temporal: derivado, no agendado

- Un turno cuya ventana cerró **está vencido cuando lo leés**. No hay job que lo marque.
- La renovación del ciclo y la pausa por semana cerrada las **dispara el barbero**. Son eventos, no relojes.
- "Cobrar hoy" **es una query**.
- Al arrancar cada ciclo se **congela un entitlement**: copia de turnos, ventanas, beneficios y reglas vigentes en ese momento. La reserva lee el entitlement, nunca el plan vivo. Eso implementa "los cambios aplican desde el próximo ciclo".

### 7. Tipos que atrapan errores

El código lo escribe la IA y lo revisa una persona sola. **Diseñá para que el código incorrecto no compile.**

- **IDs branded:** `ShopId`, `BarberId`, `SubscriptionId`. Que no se pueda pasar uno donde va otro.
- Tipos distintos para instante y fecha de calendario.
- Errores de negocio como uniones etiquetadas, con exhaustividad verificada por el compilador.

### 8. Estilos

- **StyleX.** CSS propio, sin frameworks de utilidades.
- **Primitivas headless** para comportamiento y accesibilidad.
- Antes de inventar un patrón nuevo de estilos, buscá si ya existe uno en el repo y seguilo.

### 9. Tests

| Nivel | Qué |
|---|---|
| Unitario (Vitest, sin base) | El dominio puro. **Como tablas de casos sacadas de `docs/propuesta-barbero.md`** |
| Integración (Vitest + Postgres real) | Reservas concurrentes solapadas · aislamiento de tenant · la transacción crédito+turno. Chico: 3 o 4 tests |
| E2E (Playwright) | 2 o 3 flujos completos |

No se busca cobertura total. Los tests del dominio son la especificación ejecutable del negocio.

### 10. Logs

- **pino, JSON a stdout.** `pino-pretty` solo en dev.
- **Un solo sistema de logging:** un `Layer` reemplaza el logger de Effect con pino, para que logs y spans queden correlacionados.
- pino necesita runtime Node. No Edge.

---

## Cosas a verificar antes de construir sobre ellas

Estas están abiertas a propósito. Si tocás alguna, verificala contra la fuente y actualizá el ADR correspondiente.

| Qué | ADR |
|---|---|
| Que `db.transaction()` de Drizzle sobre `node-postgres` dé transacciones interactivas reales contra el endpoint pooleado | 0001, 0003 |
| Si el pooling en modo transaction se lleva bien con los prepared statements de Drizzle | 0001 |
| Que Neon exponga `btree_gist` | 0001 |
| Si Drizzle Kit puede expresar `EXCLUDE`, o va como SQL crudo | 0003 |
| Los nombres y el comportamiento de los módulos de datos puros en Effect v4 | 0004 |
| Si el tipo resultado del dominio se puede *yield*ear dentro de un `Effect.gen` en v4 | 0004 |
| El soporte de fecha de calendario de `DateTime` en v4 | 0006 |
| Cuál librería de primitivas trae headless de calendario y campo de fecha | 0008 |
| El estado del plugin de StyleX con App Router y Turbopack | 0008 |

---

## Al trabajar en este repo

- **Cambios chicos y revisables.** El cuello de botella es la revisión, no la escritura.
- **Si una decisión de arquitectura cambia, escribí un ADR nuevo** que reemplace al viejo. No edites el viejo.
- **Si el dominio cambia, actualizá `docs/propuesta-barbero.md` en el mismo cambio.** Ese documento y los tests del dominio tienen que decir lo mismo.
