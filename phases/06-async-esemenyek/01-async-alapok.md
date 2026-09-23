[← Fázis index](../06-async-esemenyek.md) · [Főoldal](../../README.md) · Következő: [02. Alkalmazás események →](02-alkalmazas-esemenyek.md)

# 6.1 — Aszinkron végrehajtás `@Async`-kal

## Hogy csinálod ezt most Laravel-ben

```php
// egy job dispatch-elése — a queue worker dolgozza fel a háttérben
dispatch(new SendWelcomeEmail($user));

// vagy job osztály
class SendWelcomeEmail implements ShouldQueue
{
    public function __construct(private User $user) {}

    public function handle(): void
    {
        Mail::to($this->user)->send(new WelcomeMail($this->user));
    }
}
```

A `dispatch()` egy sort ír be a `jobs` táblába (vagy Redis-be, SQS-be, attól függően milyen
drivert használsz), és egy külön `php artisan queue:work` folyamat veszi fel és dolgozza fel.
Ez **perzisztens**: ha a worker folyamat összeomlik vagy a szerver újraindul, a be nem dolgozott
jobok megmaradnak és újra feldolgozásra kerülnek, amint a worker újra elindul.

## `@Async` Spring-ben

A Spring legegyszerűbb aszinkron megoldása, az `@Async`, ezzel szemben **nem perzisztens** — egy
in-memory thread pool-on fut ugyanabban a JVM folyamatban, mint a webalkalmazás maga:

```java
@Configuration
@EnableAsync
public class AsyncConfig {
}
```

```java
@Service
public class EmailService {

    @Async
    public void sendWelcomeEmail(User user) {
        // ez a metódus egy külön szálon fut, a hívó azonnal visszakapja a vezérlést
        System.out.println("Üdvözlő email küldése: " + user.getEmail());
    }
}
```

```java
@Service
public class AuthService {
    private final EmailService emailService;

    public AuthService(EmailService emailService) {
        this.emailService = emailService;
    }

    public User register(RegisterRequest request) {
        User user = ...; // user létrehozása, mentése
        emailService.sendWelcomeEmail(user);  // nem blokkol, azonnal visszatér
        return user;
    }
}
```

Fontos technikai részlet, ami PHP-ból nézve furcsa lehet: az `@Async` **csak akkor működik**, ha
a metódust kívülről hívod (egy másik Spring bean-ből), mert a Spring egy proxy objektumon
keresztül fogja közbe a hívást. Ha egy osztály a saját `@Async` metódusát hívja `this.metodus()`
formában, az **szinkron módon fog lefutni** — ez az egyik leggyakoribb kezdő hiba, amivel
találkozni fogsz.

## `CompletableFuture<T>` — amikor kell az eredmény

Ha a hívónak szüksége van az aszinkron metódus eredményére (nem csak "tűzd el és felejtsd el"),
a visszatérési típus `CompletableFuture<T>` legyen:

```java
@Async
public CompletableFuture<Boolean> checkEmailDeliverability(String email) {
    boolean deliverable = ...; // időigényes külső API hívás
    return CompletableFuture.completedFuture(deliverable);
}

// hívó oldalon
CompletableFuture<Boolean> future = emailService.checkEmailDeliverability(user.getEmail());
future.thenAccept(result -> System.out.println("Kézbesíthető: " + result));
```

Ez koncepcionálisan a PHP világ Promise/async mintáihoz hasonlít (amivel PHP-ban ritkán
találkozol natívan, inkább ReactPHP/Swoole környezetben), Java-ban viszont a `CompletableFuture`
a nyelv/standard könyvtár beépített, general-purpose eszköze erre.

## A korlát, ami a következő fejezethez vezet

Az `@Async` tökéletes olyan feladatokhoz, amik gyorsak, nem kritikusak, és ahol elfogadható, hogy
egy szerver-újraindítás esetén elveszik a folyamatban lévő munka (pl. egy cache frissítés
elindítása háttérben). Amint viszont **garanciát** akarsz arra, hogy egy email tényleg
kiküldésre kerül akkor is, ha a szerver közben újraindul, vagy több worker-példány között akarod
elosztani a terhelést — pontosan azt a problémát, amit Laravel-ben a Redis/database queue driver
és a Horizon old meg —, egy dedikált üzenetsor-rendszerre lesz szükséged. Erről szól a
[6.3 Üzenetsorok](03-uzenetsorok.md) fejezet.

---

[← Fázis index](../06-async-esemenyek.md) · [Főoldal](../../README.md) · Következő: [02. Alkalmazás események →](02-alkalmazas-esemenyek.md)
