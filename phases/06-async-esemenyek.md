[← Előző fázis: 05. Tesztelés](05-teszteles.md) · [Főoldal](../README.md) · Következő fázis: [07. Megfigyelhetőség és production-readiness →](07-observability.md)

# 6. fázis — Aszinkron munka, események, háttérfeladatok

Időtartam: ~1-2 hét

Cél: a Laravel Queue/Job/Event rendszer Spring megfelelőinek megismerése.

Ez a fázis alfejezetekre van bontva, mindegyik konkrét PHP/Laravel összehasonlításokkal és
kódpéldákkal. Haladj sorban — a záró gyakorlat a korábbi fázisokban felépített Todo API
regisztrációs folyamatához ad aszinkron eseménykezelést, majd üzenetsort:

1. [Async alapok](06-async-esemenyek/01-async-alapok.md) —
   `@Async` + `@EnableAsync`, `CompletableFuture<T>` — a Laravel `dispatch()` in-memory,
   nem perzisztens megfelelője
2. [Alkalmazás események](06-async-esemenyek/02-alkalmazas-esemenyek.md) —
   `ApplicationEventPublisher`, `@EventListener`, `@TransactionalEventListener` — a Laravel
   Event/Listener rendszer megfelelője, tranzakció-tudatos listenerekkel
3. [Üzenetsorok](06-async-esemenyek/03-uzenetsorok.md) —
   RabbitMQ bevezetése (`spring-boot-starter-amqp`), `@RabbitListener` — a Laravel Redis
   queue/Horizon megfelelője
4. [Ütemezett feladatok](06-async-esemenyek/04-utemezett-feladatok.md) —
   `@Scheduled` + `@EnableScheduling` — a Laravel Task Scheduling (`schedule()`) párja, központi
   Kernel osztály nélkül
5. [Záró gyakorlat: esemény, majd üzenetsor](06-async-esemenyek/05-gyakorlat-esemeny-es-queue.md) —
   "email küldés regisztrációkor" előbb async listenerrel, majd RabbitMQ-alapú queue-ra cserélve

---

[← Előző fázis: 05. Tesztelés](05-teszteles.md) · [Főoldal](../README.md) · Következő fázis: [07. Megfigyelhetőség és production-readiness →](07-observability.md)
