[← Előző: 04. Ütemezett feladatok](04-utemezett-feladatok.md) · [Fázis index](../06-async-esemenyek.md) · [Főoldal](../../README.md) · Következő fázis: [07. Megfigyelhetőség és production-readiness →](../07-observability.md)

# 6.5 — Záró gyakorlat: esemény, majd üzenetsor

Eredeti specifikáció: *"Adj egy 'email küldés regisztrációkor' eseményt async listener-rel, majd
cseréld le RabbitMQ-alapú queue-ra."* Ez a gyakorlat két lépésben építi fel ugyanazt a
funkciót — előbb a legegyszerűbb Spring-es megoldással, aztán egy valódi üzenetsorral —, hogy
tapasztald meg közvetlenül, mikor és miért éri meg a bonyolultabb eszközre váltani.

## 1. lépés — `@TransactionalEventListener` + `@Async`

Előfeltétel: a 4. fázisban elkészült `AuthService.register(RegisterRequest request)` metódus,
ami elmenti az új `User`-t.

```java
public class UserRegisteredEvent {
    private final User user;

    public UserRegisteredEvent(User user) {
        this.user = user;
    }

    public User getUser() {
        return user;
    }
}
```

```java
@Service
public class AuthService {
    private final UserRepository userRepository;
    private final PasswordEncoder passwordEncoder;
    private final ApplicationEventPublisher eventPublisher;

    public AuthService(UserRepository userRepository, PasswordEncoder passwordEncoder,
                        ApplicationEventPublisher eventPublisher) {
        this.userRepository = userRepository;
        this.passwordEncoder = passwordEncoder;
        this.eventPublisher = eventPublisher;
    }

    @Transactional
    public User register(RegisterRequest request) {
        User user = new User(request.email(), passwordEncoder.encode(request.password()));
        userRepository.save(user);
        eventPublisher.publishEvent(new UserRegisteredEvent(user));
        return user;
    }
}
```

```java
@Component
public class WelcomeEmailListener {

    @Async
    @TransactionalEventListener(phase = TransactionPhase.AFTER_COMMIT)
    public void onUserRegistered(UserRegisteredEvent event) {
        System.out.println("[EMAIL] Üdvözlő email küldése: " + event.getUser().getEmail());
    }
}
```

Ne felejtsd el a `@EnableAsync`-et a konfigurációs osztályra tenni. Ellenőrizd: regisztrálj egy
tesztfelhasználót a `POST /api/auth/register` végponton, és figyeld meg a konzolban, hogy a log
üzenet **egy másik szálon** (a log elé kiírt thread névből látszik) és **a válasz visszaküldése
után röviddel** jelenik meg, nem blokkolva a kérést.

## 2. lépés — átalakítás RabbitMQ-ra

Cseréld le a `WelcomeEmailListener`-t egy publisher/consumer párra:

```java
@Configuration
public class RabbitConfig {

    @Bean
    public Queue emailQueue() {
        return new Queue("email-queue", true);
    }
}
```

```java
@Component
public class WelcomeEmailListener {
    private final RabbitTemplate rabbitTemplate;

    public WelcomeEmailListener(RabbitTemplate rabbitTemplate) {
        this.rabbitTemplate = rabbitTemplate;
    }

    @TransactionalEventListener(phase = TransactionPhase.AFTER_COMMIT)
    public void onUserRegistered(UserRegisteredEvent event) {
        rabbitTemplate.convertAndSend("email-queue", event.getUser().getEmail());
    }
}
```

```java
@Component
public class WelcomeEmailConsumer {

    @RabbitListener(queues = "email-queue")
    public void handleWelcomeEmail(String userEmail) {
        System.out.println("[EMAIL, queue-ból] Üdvözlő email küldése: " + userEmail);
    }
}
```

Egészítsd ki a `docker-compose.yml`-t egy `rabbitmq` szolgáltatással (lásd [6.3
Üzenetsorok](03-uzenetsorok.md)), és próbáld ki: indítsd el a konténereket, regisztrálj egy
usert, majd nézd meg a `http://localhost:15672` management felületen, hogy az üzenet megjelenik-e
a `email-queue`-ban, és a consumer feldolgozza-e.

## Mikor éri meg RabbitMQ-ra váltani

Ennél a kis Todo API-nál gyakorlati szempontból egyik megoldás sem "jobb" abszolút értelemben —
a döntés a következő szempontoktól függ:

| Szempont | `@Async` + esemény | RabbitMQ |
|---|---|---|
| Túléli-e a szerver újraindítását egy folyamatban lévő üzenet | nem | igen (durable queue) |
| Elosztható-e több worker-példány között | nem | igen |
| Van-e beépített retry/dead-letter kezelés | nem, kézzel kell megírni | igen, konfigurálható |
| Bevezetési/üzemeltetési költség | nulla, beépített | külön infrastruktúra (broker) kell |
| Monitorozhatóság | nincs dashboard | van (management UI) |

Ökölszabály: amíg egy feladat elvesztése egy szerver-újraindításkor elfogadható kockázat (pl. egy
"utolsó bejelentkezés időpontja" mező frissítése), maradj `@Async`-nál. Amint egy feladat
elvesztése üzleti szempontból problémás (email kiküldés, számlázás, értesítés) — pontosan úgy,
ahogy Laravel-ben is emiatt választanál Redis/database queue drivert `sync` helyett —, térj át
dedikált üzenetsorra.

## Mi jön ezután

A 7. fázisban ez a rendszer megfigyelhetőséget kap: metrikákat arról, hány üzenet vár a
queue-ban, health check-eket a RabbitMQ kapcsolat állapotára, és strukturált logolást minden
eddigi komponensre.

---

[← Előző: 04. Ütemezett feladatok](04-utemezett-feladatok.md) · [Fázis index](../06-async-esemenyek.md) · [Főoldal](../../README.md) · Következő fázis: [07. Megfigyelhetőség és production-readiness →](../07-observability.md)
