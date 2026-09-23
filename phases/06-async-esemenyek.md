[← Előző: 5. Tesztelés](05-teszteles.md) · [Főoldal](../README.md) · Következő: [07. Observability, production-readiness →](07-observability.md)

# 6. fázis — Aszinkron munka, események, háttérfeladatok

Időtartam: ~1-2 hét

Cél: a Laravel Queue/Job/Event rendszer Spring megfelelőinek megismerése.

- `@Async` + `@EnableAsync` — egyszerű háttérfeladatok, mint a Laravel `dispatch()`
- `ApplicationEvent` + `@EventListener` (és `@TransactionalEventListener`) — a Laravel
  Event/Listener rendszer megfelelője
- Üzenetsor bevezetése: RabbitMQ (`spring-boot-starter-amqp`) vagy Kafka — ez a Laravel Redis
  queue/Horizon megfelelője, csak itt dedikált broker infrastruktúrával dolgozol
- Ütemezett feladatok: `@Scheduled` — a Laravel Task Scheduling (`schedule()`) párja

## Gyakorlat

Adj egy "email küldés regisztrációkor" eseményt async listener-rel, majd cseréld le
RabbitMQ-alapú queue-ra.

---

[← Előző: 5. Tesztelés](05-teszteles.md) · [Főoldal](../README.md) · Következő: [07. Observability, production-readiness →](07-observability.md)
