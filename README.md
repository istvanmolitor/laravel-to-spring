# Laravel → Java Spring roadmap

Cél: éles, produkciós szintű Spring Boot tudás elérése meglévő erős Laravel/PHP háttérrel.
A becsült időtartamok napi ~1-2 óra tanulással számolnak, gyorsítható teljes idős munkával.

Alapelv: nem a nyelvet és a keretrendszert külön tanulod meg, hanem fogalompárokban gondolkodsz.
Laravel-ben már ismersz MVC-t, DI-t, ORM-et, middleware-t, service providereket — a Spring ezeket
más szintaxissal, de hasonló gondolatmenettel oldja meg.

---

## Fogalmi térkép (Laravel → Spring)

| Laravel | Spring / Java világ |
|---|---|
| Composer | Maven / Gradle |
| Artisan | Spring Boot CLI / `./mvnw` |
| Service Container (DI) | Spring IoC Container |
| Service Provider | `@Configuration` osztály |
| Facade | nincs közvetlen megfelelő — inkább `@Autowired` / konstruktor injection |
| Eloquent ORM | Spring Data JPA / Hibernate |
| Migrations | Flyway / Liquibase |
| Route + Controller | `@RestController` + `@RequestMapping`/`@GetMapping` |
| Middleware | `Filter` / `Interceptor` / Spring Security `SecurityFilterChain` |
| Form Request validáció | `@Valid` + Bean Validation (`jakarta.validation`) |
| Blade | Thymeleaf (ha kell szerver oldali render; API-nál nem releváns) |
| Job / Queue (Redis, SQS) | Spring `@Async`, Spring Batch, Spring Cloud Stream, JMS/Kafka |
| Event/Listener | `ApplicationEvent` + `@EventListener` |
| Config (.env) | `application.yml`/`application.properties` + `@ConfigurationProperties` |
| Tinker | Spring Shell / egyszerű JUnit teszt |
| PHPUnit / Pest | JUnit 5 + Mockito |
| Sanctum/Passport | Spring Security + JWT (`spring-security-oauth2` / `jjwt`) |
| Laravel Mix/Vite | ugyanaz marad (frontend külön réteg) |
| Horizon | Spring Boot Actuator + Micrometer |
| `php artisan serve` | beépített embedded Tomcat, `./mvnw spring-boot:run` |

Ezt a táblázatot érdemes a fejedben tartani — minden fázisban vissza fogsz rá utalni.

---

## Fázisok

Minden fázis külön fájlban van kidolgozva, a fájlok között előre/vissza linkekkel lehet lépkedni.

0. [Java alapok](phases/00-java-alapok.md) — ~1-2 hét
1. [Spring Boot alapok és a DI mentális modell](phases/01-spring-boot-alapok.md) — ~2-3 hét
2. [Adatréteg: Spring Data JPA](phases/02-adatreteg-jpa.md) — ~2-3 hét
3. [Validáció, hibakezelés, réteges architektúra](phases/03-validacio-hibakezeles.md) — ~1-2 hét
4. [Spring Security](phases/04-spring-security.md) — ~2-3 hét
5. [Tesztelés](phases/05-teszteles.md) — ~1-2 hét
6. [Aszinkron munka, események, háttérfeladatok](phases/06-async-esemenyek.md) — ~1-2 hét
7. [Megfigyelhetőség és production-readiness](phases/07-observability.md) — ~1 hét
8. [Építs egy komplett projektet](phases/08-projekt.md) — ~3-4 hét

---

## Ajánlott tananyagok

- **Hivatalos Spring dokumentáció és "guides"** (spring.io/guides) — rövid, célzott, gyakorlati
  útmutatók, minden fázishoz van belőle
- **Baeldung** (baeldung.com) — a Spring világ "Laravel Daily/Laracasts"-je, rendkívül alapos
  cikkek szinte minden témában, ezt fogod legtöbbet használni napi szinten
- Könyv: *"Spring Boot in Action"* vagy *"Spring in Action" (Craig Walls)* — jó strukturált
  bevezető, ha könyvből szeretsz tanulni
- **Java generics és Optional**: külön szánj rá időt, ez okozza a legtöbb kezdeti súrlódást
  PHP fejlesztőknek

---

## Gyakori buktatók PHP/Laravel fejlesztőknek

- **Nincs "mágia"**: a Spring explicit konfigurációt vár ott, ahol Laravel konvencióból kitalálja
  (pl. entitás kapcsolatok, oszlopnevek) — ez elsőre lassabbnak tűnik, de kevesebb rejtett hibát
  okoz
- **Erős típusosság mindenhol**: nincs laza összehasonlítás, nincs implicit cast — a compiler
  sok hibát kiszűr, amit PHP-ban futásidőben vennél észre
- **N+1 probléma explicit kezelése**: `FetchType.LAZY` alapértelmezett a `@ManyToOne`-nál
  kivéve, `@OneToMany`-nél LAZY az alapértelmezett is — mindig gondold át, mikor kell `JOIN FETCH`
- **Checked exceptionök**: eleinte zavaróak lesznek, de gyorsan megszokod a `throws` deklarációkat
- **Build/restart ciklus lassabb**, mint PHP-nál — használj Spring Boot DevTools-t a hot reload-hoz,
  hogy közelítsd a Laravel fejlesztői élményt

---

## Becsült összidő

~3-4 hónap napi 1-2 óra tanulással a 0-7. fázisig, plusz 1 hónap a komplett projekthez.
Mivel erős Laravel/OOP/backend architektúra tudásod van, a legtöbb koncepcionális elem gyorsan
átragad — a nehézséget a Java nyelvi sajátosságok (típusosság, generics, verbosity) és a Spring
ökoszisztéma méretének megszokása fogja jelenteni, nem az architekturális gondolkodás.
