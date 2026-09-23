[← Előző: 01. Projekt indítása](01-projekt-inditasa.md) · [Fázis index](../01-spring-boot-alapok.md) · [Főoldal](../../README.md) · Következő: [03. REST controllerek →](03-rest-controllerek.md)

# 1.2 — Dependency Injection és a Spring IoC konténer

## A Spring IoC konténer — a Service Container Spring-változata

Laravel-ben van egy **Service Container**, ami tudja, hogyan kell felépíteni egy objektumot a
függőségeivel együtt, és vagy explicit kéred (`app()->make(Foo::class)`), vagy a keretrendszer
automatikusan befecskendezi (pl. controller metódus paraméterébe, ha type-hintelt).

Spring-ben ugyanez a koncepció **IoC konténer** (Inversion of Control) néven fut — de itt sokkal
központibb szerepe van: **minden** objektum, amit Spring kezel ("bean"), a konténeren keresztül
jön létre és kap injektálva függőségeket, nincs "opcionális" mód, mint Laravel-ben, ahol simán
`new`-elhetsz is egy osztályt facade-ok helyett.

## Component scanning — automatikus felfedezés

```java
@Component
public class TodoValidator { ... }
```

A `@ComponentScan` (ami a `@SpringBootApplication`-be van beágyazva, lásd
[1.1](01-projekt-inditasa.md)) induláskor bejárja a fő alkalmazásosztály csomagját és
alcsomagjait, és minden `@Component`-tel (vagy annak specializációjával) jelölt osztályt
regisztrál a konténerbe, mint **bean**-t. Ez a Laravel automatikus package/provider
felfedezésének felel meg, csak itt osztály-szinten, nem csomag-szinten történik.

## `@Component`, `@Service`, `@Repository`, `@Controller` — szemantikai rétegek

Technikailag mind a négy ugyanazt csinálja (regisztrál egy bean-t a konténerbe) — a `@Service`,
`@Repository` és `@Controller` mind **meta-annotációként** a `@Component`-re épül. A különbség
tisztán **szemantikai jelzés**, ami olvashatóságot ad, és néhány esetben extra viselkedést kapcsol
be:

| Annotáció | Réteg | Extra viselkedés | Laravel megfelelő |
|---|---|---|---|
| `@Component` | általános, réteg-semleges | — | — |
| `@Service` | üzleti logika | — (tisztán dokumentációs célú) | Service osztály (konvenció, nincs keretrendszer-szintű megfelelője) |
| `@Repository` | adatelérési réteg | automatikusan lefordítja az adatbázis-specifikus kivételeket egységes Spring `DataAccessException`-re | Eloquent Repository minta (ha valaki bevezeti — Laravel-ben ez sem kötelező) |
| `@Controller` / `@RestController` | HTTP réteg | a `@RestController` automatikusan JSON-ná szerializálja a visszatérési értéket (`@ResponseBody` beépítve) | Controller osztály |

```java
@Service
public class TodoService {
    // üzleti logika
}

@Repository
public interface TodoRepository extends JpaRepository<Todo, Long> {
    // adatelérés — erről bővebben a 2. fázisban
}

@RestController
@RequestMapping("/api/todos")
public class TodoController {
    // HTTP réteg — erről bővebben a 3. fejezetben
}
```

## Konstruktor-alapú DI — miért ez a *best practice*

Ez az egyik legfontosabb szemléletbeli váltás Laravel-ből jőve.

```php
// Laravel — Facade: kényelmes, de rejtett globális állapotot használ
class TodoService
{
    public function markDone(int $id): void
    {
        $todo = Todo::findOrFail($id);   // Eloquent facade a háttérben
        $todo->done = true;
        $todo->save();

        Log::info("Todo #{$id} kész");   // Log facade — honnan jön? sehonnan explicit
    }
}
```

```php
// Laravel — explicit DI a konstruktorban (ez már közelebb áll a Spring gondolkodáshoz)
class TodoService
{
    public function __construct(
        private TodoRepository $repository,
        private LoggerInterface $logger,
    ) {}

    public function markDone(int $id): void
    {
        $todo = $this->repository->findOrFail($id);
        $todo->markDone();
        $this->repository->save($todo);
        $this->logger->info("Todo #{$id} kész");
    }
}
```

```java
// Spring — KIZÁRÓLAG ez a minta a javasolt, nincs "facade-szerű" kényelmi alternatíva
@Service
public class TodoService {

    private final TodoRepository repository;
    private final Logger logger;

    // ha egyetlen konstruktor van, a @Autowired opcionális — Spring automatikusan felismeri
    public TodoService(TodoRepository repository, Logger logger) {
        this.repository = repository;
        this.logger = logger;
    }

    public void markDone(Long id) {
        Todo todo = repository.findById(id).orElseThrow();
        todo.setDone(true);
        repository.save(todo);
        logger.info("Todo #" + id + " kész");
    }
}
```

Miért fontos ez neked, PHP fejlesztőként:

- **Nincs "mágikus" globális elérés.** A Laravel Facade-ok (`Log::`, `DB::`, `Cache::`) statikus
  proxy-k egy mögöttes, konténerből feloldott instance-ra — kényelmesek, de elrejtik az osztály
  valódi függőségeit, és megnehezítik a mockolást tesztben (bár a Laravel `Facade::shouldReceive()`
  segít ezen). Spring-ben **nincs ilyen mintát** — minden függőség explicit, látható a
  konstruktorban, és a teszteléshez natívan egyszerű mock-olni (lásd az [5. fázis:
  Tesztelés](../05-teszteles.md) részt).
- **A konstruktor önmagában dokumentáció.** Ha ránézel egy osztály konstruktorára, pontosan
  látod, mitől függ — nincs szükség elrejtett `app()->make(...)` hívások felkutatására a
  metódustörzsekben.
- **Immutabilitás.** A `final` mezők (lásd [0.3 OOP alapok](../00-java-alapok/03-oop-alapok.md))
  garantálják, hogy egy bean függőségei az objektum létrehozása után nem változnak.

Lombok-kal (amit az [1.1](01-projekt-inditasa.md) fejezetben hozzáadtál a projekthez) a fenti
konstruktor még rövidebb, a PHP 8 constructor promotion-höz hasonló kényelmet adva:

```java
@Service
@RequiredArgsConstructor   // Lombok: generál egy konstruktort minden final mezőhöz
public class TodoService {
    private final TodoRepository repository;
    private final Logger logger;
    // nincs kézzel írt konstruktor — Lombok legenerálja fordítási időben
}
```

## `@Configuration` + `@Bean` — a Service Provider megfelelője

Amikor egy külső könyvtár osztályát kell bean-né tenni (amit nem tudsz `@Component`-tel
annotálni, mert nem a te kódod), vagy amikor egyedi logikával kell felépíteni egy objektumot,
`@Configuration` osztályban `@Bean` metódusokat írsz:

```php
// Laravel Service Provider — register() fázisban kötöd be a konténerbe
class AppServiceProvider extends ServiceProvider
{
    public function register(): void
    {
        $this->app->singleton(PaymentGateway::class, function ($app) {
            return new StripeGateway(config('services.stripe.key'));
        });
    }
}
```

```java
// Spring — @Configuration osztály, @Bean metódusok
@Configuration
public class AppConfig {

    @Bean
    public PaymentGateway paymentGateway(
            @Value("${services.stripe.key}") String apiKey) {
        return new StripeGateway(apiKey);
    }
}
```

A `@Bean` metódus visszatérési értéke bekerül a konténerbe, és onnantól bárhová injektálható
(konstruktoron keresztül), pontosan úgy, mint a `@Component`-tel jelölt osztályok. A Spring
bean-ek alapértelmezetten **singleton** hatókörűek (egy instance az egész alkalmazás életciklusán
át) — ez megegyezik a Laravel `$this->app->singleton(...)` bindolással; ha minden feloldásnál új
instance kell (Laravel `$this->app->bind(...)` megfelelője), `@Scope("prototype")`-ot adsz hozzá.

---

[← Előző: 01. Projekt indítása](01-projekt-inditasa.md) · [Fázis index](../01-spring-boot-alapok.md) · [Főoldal](../../README.md) · Következő: [03. REST controllerek →](03-rest-controllerek.md)
