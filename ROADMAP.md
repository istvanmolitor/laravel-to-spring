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

## 0. fázis — Java alapok (1-2 hét)

Cél: annyi Java nyelvi tudás, hogy a Spring kódot ne a szintaxis, hanem a koncepció nehezítse.

- Típusrendszer: statikus típusosság, primitívek vs. objektumok (`int` vs `Integer`), erős típusosság
  PHP-hoz képest (nincs laza `==`, nincs implicit type juggling)
- OOP Java-ban: interfészek vs. absztrakt osztályok, `final`, package-visibility, konstruktorok
- Generics (`List<String>`, `Optional<T>`) — ennek nincs igazi PHP megfelelője, erre szánj több időt
- `Optional<T>` mint a `null` kezelés Laravel-es "?." helyett
- Streamek és lambdák (`list.stream().filter(...).map(...).collect(...)`) — gondolj rá úgy, mint
  a Laravel Collection-ökre (`collect($items)->filter()->map()`), a gondolkodásmód szinte azonos
- Checked vs unchecked exceptionök (ez PHP-ban nem létezik, ez lesz az egyik legfurcsább rész)
- Build eszköz: válaszd a **Maven**-t kezdésnek (egyszerűbb, mint Gradle, jobban dokumentált)
- IDE: **IntelliJ IDEA Community** (a Spring ökoszisztéma szinte ehhez van optimalizálva)

Gyakorlat: írj pár kis konzolos programot (pl. egy egyszerű bank-szimulátor osztályokkal,
kivételkezeléssel, stream-alapú összegzésekkel) Spring nélkül, tisztán Java-ban.

---

## 1. fázis — Spring Boot alapok és a DI mentális modell (2-3 hét)

Cél: egy egyszerű REST API felállítása, és megérteni, hogy a Spring DI miben más, mint a Laravel-é.

- Spring Initializr-rel (start.spring.io) hozz létre egy projektet: `Spring Web`, `Spring Data JPA`,
  `H2`/`PostgreSQL`, `Validation`, `Lombok`
- `@SpringBootApplication`, a component scan mechanizmus — ez a Laravel autodiscovery-jének felel meg
- `@Component`, `@Service`, `@Repository`, `@Controller`/`@RestController` — mikor melyiket
- Konstruktor-alapú Dependency Injection (ez a Spring-ben *best practice*, ellentétben a Laravel
  facade-okkal, amik rejtett globális állapotot használnak) — szokj le a "mágikus" elérésről
- `@RestController` + `@GetMapping`/`@PostMapping`/`@PathVariable`/`@RequestParam`/`@RequestBody`
  — ez majdnem 1:1 megfeleltethető a Laravel route+controller párosnak
- `application.yml` konfiguráció és profilok (`dev`, `prod`) — a Laravel `.env` + `config/` párja
- `@ConfigurationProperties` — típusos konfig osztályok, ennek nincs jó Laravel megfelelője,
  de gondolj rá úgy, mint egy validált `config()` helper-re

Gyakorlat: építs egy "Todo API"-t CRUD végpontokkal, in-memory H2 adatbázissal.

---

## 2. fázis — Adatréteg: Spring Data JPA (2-3 hét)

Cél: az Eloquent tudásod átültetése JPA/Hibernate-re.

- Entity osztályok (`@Entity`, `@Table`, `@Id`, `@GeneratedValue`) — ez az Eloquent modell megfelelője,
  de itt **explicit** kell definiálnod mindent, amit Eloquent konvencióból kitalál
- Kapcsolatok: `@OneToMany`, `@ManyToOne`, `@ManyToMany` + `mappedBy`/`JoinColumn` — összevetve az
  Eloquent `hasMany`/`belongsTo`/`belongsToMany` relációival; itt kell figyelni a **lazy vs eager
  loading**-ra (`FetchType.LAZY` vs `EAGER`) — ez a Laravel N+1 problémának pont a Java megfelelője,
  csak itt explicit kell kezelni
- `JpaRepository<Entity, ID>` — a Laravel Eloquent query builder helyett itt interfészt írsz,
  a Spring generálja le a implementációt (derived query methods: `findByEmail`, `findByStatusAndActive`)
- JPQL és `@Query` annotáció — ez a raw query / `DB::table()` megfelelője, amikor a derived method
  nem elég
- Migráció: **Flyway** bevezetése (`db/migration/V1__init.sql` fájlok) — ez az Artisan migration
  párja, de itt SQL-t írsz közvetlenül, nincs PHP DSL
- Tranzakciókezelés: `@Transactional` — a Laravel `DB::transaction()` deklaratív megfelelője

Gyakorlat: bővítsd a Todo API-t user–todo kapcsolattal, Flyway migrációkkal, PostgreSQL-lel Docker
Compose-ban.

---

## 3. fázis — Validáció, hibakezelés, réteges architektúra (1-2 hét)

Cél: production-grade API struktúra, ahogy egy nagyobb Laravel projektben is elvárnád.

- Bean Validation: `@Valid`, `@NotNull`, `@Size`, `@Email` a DTO-kon — ez a Form Request
  validációs szabályainak felel meg
- DTO-k bevezetése (ne az Entity-t exponáld közvetlenül az API-n — ez fontosabb Spring-ben, mint
  Laravel-ben, mert nincs automatikus `$hidden`/`$fillable` védelmed)
- `@ControllerAdvice` + `@ExceptionHandler` — ez a Laravel `Handler.php`/exception rendering
  megfelelője, globális hibakezelés
- Réteges architektúra: Controller → Service → Repository — ez tisztább elválasztás, mint
  a legtöbb Laravel projekt "fat controller/fat model" mintája; szokj hozzá a service réteghez
- MapStruct vagy manuális mapper Entity↔DTO konverzióhoz

Gyakorlat: refaktoráld a Todo API-t tiszta réteges struktúrára, egységes hibaválasz formátummal.

---

## 4. fázis — Spring Security (2-3 hét)

Cél: a Sanctum/Passport tudásod átültetése — ez az egyik legmeredekebb tanulási görbe lesz.

- `SecurityFilterChain` bean-ek — a middleware pipeline megfelelője, de deklaratívabb
- Alap autentikáció, majd JWT-alapú stateless auth (`spring-boot-starter-security` + `jjwt` vagy
  `nimbus-jose-jwt`) — ez a Sanctum token-alapú auth párja
- `UserDetailsService`, `PasswordEncoder` (BCrypt — ugyanaz, mint Laravel alatt)
- Jogosultságkezelés: `@PreAuthorize`, role/authority alapú védelem — a Laravel Policy/Gate
  megfelelője, csak annotáció-alapú
- CORS konfiguráció — hasonló elven, mint Laravel `cors.php`, csak Java konfigban

Gyakorlat: adj JWT auth-ot a Todo API-hoz, user regisztráció/login endpointtal, védett route-okkal.

---

## 5. fázis — Tesztelés (1-2 hét, párhuzamosan is tanulható a korábbi fázisokkal)

Cél: a Pest/PHPUnit tudásod JUnit 5 + Mockito ökoszisztémára fordítása.

- JUnit 5 alapok: `@Test`, `@BeforeEach`, `assertEquals` stb. — koncepcionálisan azonos PHPUnit-tal
- Mockito: `@Mock`, `@InjectMocks`, `when().thenReturn()` — a Laravel `Mockery`/facade mock
  megfelelője
- `@SpringBootTest` vs `@WebMvcTest` vs `@DataJpaTest` — a Spring külön annotációkkal szegmentálja
  a teszt "súlyát", ahol Laravel-ben inkább a `RefreshDatabase` trait-tel egységesen dolgozol
- Testcontainers — valós PostgreSQL/Redis konténerrel futó integrációs tesztek, ennek nincs
  elterjedt Laravel megfelelője (ott inkább SQLite in-memory-t használsz teszthez), de sokkal
  megbízhatóbb

Gyakorlat: írj unit teszteket a service rétegre Mockito-val, és integrációs teszteket
Testcontainers + `@SpringBootTest` kombinációval.

---

## 6. fázis — Aszinkron munka, események, háttérfeladatok (1-2 hét)

Cél: a Laravel Queue/Job/Event rendszer Spring megfelelőinek megismerése.

- `@Async` + `@EnableAsync` — egyszerű háttérfeladatok, mint a Laravel `dispatch()`
- `ApplicationEvent` + `@EventListener` (és `@TransactionalEventListener`) — a Laravel
  Event/Listener rendszer megfelelője
- Üzenetsor bevezetése: RabbitMQ (`spring-boot-starter-amqp`) vagy Kafka — ez a Laravel Redis
  queue/Horizon megfelelője, csak itt dedikált broker infrastruktúrával dolgozol
- Ütemezett feladatok: `@Scheduled` — a Laravel Task Scheduling (`schedule()`) párja

Gyakorlat: adj egy "email küldés regisztrációkor" eseményt async listener-rel, majd cseréld le
RabbitMQ-alapú queue-ra.

---

## 7. fázis — Megfigyelhetőség és production-readiness (1 hét)

- Spring Boot Actuator (`/actuator/health`, `/actuator/metrics`) — a Laravel Horizon/Telescope
  megfelelője, beépített
- Micrometer + Prometheus/Grafana integráció
- Strukturált logolás (Logback/SLF4J), log szintek — hasonló elv, mint Laravel `Log::` facade,
  csak konfigurációban XML/YAML alapú
- `@ConfigurationProperties` validáció, profil-alapú (`dev`/`staging`/`prod`) konfiguráció

---

## 8. fázis — Építs egy komplett projektet (3-4 hét)

A legfontosabb lépés: ne csak izolált gyakorlatokat csinálj, hanem építs végig egy valós méretű
alkalmazást, amiben minden korábbi fázis eleme összeáll. Ötlet: portold át egy már meglévő,
kisebb Laravel projektedet Spring Boot-ra — így közvetlenül összehasonlíthatod a két
megközelítést ugyanazon a domain logikán.

Javasolt technológiai stack az első komplett projekthez:
- Spring Boot 3.x + Java 21 (LTS)
- Spring Data JPA + PostgreSQL + Flyway
- Spring Security + JWT
- Docker Compose (app + db + esetleg Redis/RabbitMQ)
- JUnit 5 + Mockito + Testcontainers
- GitHub Actions CI (build + teszt)

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
