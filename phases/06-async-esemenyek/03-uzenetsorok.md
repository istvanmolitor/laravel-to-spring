[← Előző: 02. Alkalmazás események](02-alkalmazas-esemenyek.md) · [Fázis index](../06-async-esemenyek.md) · [Főoldal](../../README.md) · Következő: [04. Ütemezett feladatok →](04-utemezett-feladatok.md)

# 6.3 — Üzenetsorok: RabbitMQ

Az előző fejezetben látott `@Async` + esemény kombináció egy dolgot nem tud: túlélni egy
szerver-újraindítást, vagy elosztani a terhelést több worker-példány között. Erre kell egy
dedikált **üzenetsor-rendszer** (message broker) — ez az, amit Laravel-ben a Redis/database
queue driver és a Horizon ad.

## Laravel Redis queue / Horizon

```php
// job dispatch — a queue driver (redis, database, sqs) elmenti a jobot
dispatch(new SendWelcomeEmail($user))->onQueue('emails');
```

```bash
php artisan queue:work --queue=emails
```

A Horizon egy dashboard, ami ezt a Redis-alapú queue-t vizualizálja: hány job vár, hány worker
dolgozik, mik a sikertelen (failed) jobok, retry lehetőséggel.

## RabbitMQ alapfogalmak

A RabbitMQ egy dedikált, önálló szolgáltatás (nem a Spring alkalmazás folyamatán belül fut),
amivel a Spring alkalmazás hálózaton keresztül kommunikál. Három alapfogalom:

- **Queue** — maga az üzenetsor, ahol az üzenetek várakoznak feldolgozásra (~ Laravel `queue`)
- **Exchange** — az üzenet ide érkezik először, és routingszabályok alapján továbbítja a
  megfelelő queue(k)-ba (Laravel-ben ez a réteg rejtve van, a driver kezeli belsőleg)
- **Binding** — az exchange és a queue közti összekötő szabály

## Docker Compose RabbitMQ szolgáltatással

```yaml
services:
  rabbitmq:
    image: rabbitmq:3-management
    ports:
      - "5672:5672"     # AMQP protokoll port, ezen kommunikál az alkalmazás
      - "15672:15672"   # web management UI — ez a Horizon dashboard megfelelője
    environment:
      RABBITMQ_DEFAULT_USER: guest
      RABBITMQ_DEFAULT_PASS: guest
```

A `15672`-es porton elérhető webes felület (`http://localhost:15672`) mutatja a queue-kat,
üzenetszámokat, consumer-eket — ez a legközelebbi vizuális párhuzam a Horizon dashboard-dal.

## Spring AMQP: küldés és fogadás

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-amqp</artifactId>
</dependency>
```

```java
@Configuration
public class RabbitConfig {

    @Bean
    public Queue emailQueue() {
        return new Queue("email-queue", true);  // true = durable, túléli a broker újraindítását
    }
}
```

```java
@Service
public class EmailPublisher {
    private final RabbitTemplate rabbitTemplate;

    public EmailPublisher(RabbitTemplate rabbitTemplate) {
        this.rabbitTemplate = rabbitTemplate;
    }

    public void publishWelcomeEmail(String userEmail) {
        rabbitTemplate.convertAndSend("email-queue", userEmail);
    }
}
```

```java
@Component
public class EmailConsumer {

    @RabbitListener(queues = "email-queue")
    public void handleWelcomeEmail(String userEmail) {
        System.out.println("Üdvözlő email feldolgozása a queue-ból: " + userEmail);
    }
}
```

A `@RabbitListener` metódus egy **külön háttérfolyamatként viselkedik**, amit a Spring
automatikusan bekapcsol az alkalmazás indulásakor — ez felel meg a `php artisan queue:work`
worker-folyamat állandó futásának, csak itt nincs szükség külön folyamat indítására, ugyanabban
a JVM-ben fut, mint a webszerver (bár éles környezetben gyakran külön, dedikált worker
instance-okba szoktad szétválasztani a webréteget és a consumer-eket, pont úgy, ahogy Laravel-ben
is külön szervert futtatsz a `queue:work`-nek).

## Retry és failed message kezelés

RabbitMQ-ban ez nem "ingyen jár", mint a Horizon failed jobs táblája — explicit dead-letter
queue (DLQ) konfigurációt kell beállítanod, ahova a sikertelenül feldolgozott üzenetek
kerülnek egy megadott újrapróbálkozási szám után. Kezdőként elég tudni, hogy ez létezik, és hogy
ez az a pont, ahol a Horizon "failed jobs" felülete kényelmesebb élményt ad dobozból, mint a nyers
RabbitMQ — ezt a kényelmet Spring-ben magadnak kell felépítened, vagy egy magasabb szintű
absztrakciót (pl. Spring Cloud Stream) bevezetned.

## Kafka — mikor érdemes RabbitMQ helyett

A Kafka egy másik népszerű üzenetsor/esemény-stream rendszer, amit gyakran fogsz emlegetni
hallani Spring körökben. Kezdéshez **RabbitMQ-t javasolt választani**: a mentális modellje
(queue, egyszeri feldolgozás egy consumer által) sokkal közelebb áll a Laravel queue
gondolkodásmódhoz. A Kafka más problémára optimalizált — nagy átviteli sebességű, sok
fogyasztó által újraolvasható eseményfolyamokra (event sourcing, analitika, log-aggregáció) —,
ezt csak akkor érdemes bevezetni, ha konkrétan ilyen use case-ed van.

---

[← Előző: 02. Alkalmazás események](02-alkalmazas-esemenyek.md) · [Fázis index](../06-async-esemenyek.md) · [Főoldal](../../README.md) · Következő: [04. Ütemezett feladatok →](04-utemezett-feladatok.md)
