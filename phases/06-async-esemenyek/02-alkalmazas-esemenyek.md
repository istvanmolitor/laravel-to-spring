[← Előző: 01. Async alapok](01-async-alapok.md) · [Fázis index](../06-async-esemenyek.md) · [Főoldal](../../README.md) · Következő: [03. Üzenetsorok →](03-uzenetsorok.md)

# 6.2 — Alkalmazás események: `ApplicationEvent` és `@EventListener`

## Laravel Event/Listener

```php
// app/Events/UserRegistered.php
class UserRegistered
{
    public function __construct(public readonly User $user) {}
}

// app/Listeners/SendWelcomeEmail.php
class SendWelcomeEmail
{
    public function handle(UserRegistered $event): void
    {
        Mail::to($event->user)->send(new WelcomeMail($event->user));
    }
}

// EventServiceProvider-ben regisztrálva, vagy PHP 8 attribútummal:
// #[AsEventListener]

// kiváltás valahol a service rétegben:
event(new UserRegistered($user));
```

A Laravel event rendszer lazítja a kapcsolatot a "mi történt" (regisztráció) és a "mit kell rá
tenni" (email küldés, statisztika frissítés, stb.) között — a `AuthService`-nek fogalma sincs
róla, hány listener figyeli az eseményt, és mit csinálnak vele.

## Spring esemény rendszer

Ugyanez a minta Spring-ben, majdnem szó szerinti fordítással:

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
    private final ApplicationEventPublisher eventPublisher;

    public AuthService(ApplicationEventPublisher eventPublisher) {
        this.eventPublisher = eventPublisher;
    }

    public User register(RegisterRequest request) {
        User user = ...; // user létrehozása, mentése
        eventPublisher.publishEvent(new UserRegisteredEvent(user));
        return user;
    }
}
```

```java
@Component
public class WelcomeEmailListener {

    @EventListener
    public void onUserRegistered(UserRegisteredEvent event) {
        System.out.println("Üdvözlő email küldése: " + event.getUser().getEmail());
    }
}
```

Nincs külön "esemény osztály" ősosztályból kell származnia Spring 4.2 óta — bármilyen POJO
lehet esemény (korábban `ApplicationEvent`-ből kellett származtatni, ez a mai kódban már ritka).
A `@EventListener` metódus szignatúrájának paramétere határozza meg, melyik eseményre iratkozik
fel — ez a Spring reflection-alapú "type matching" mechanizmusa, nincs külön regisztrációs fájl,
mint a Laravel `EventServiceProvider $listen` tömbje (attribútum-alapú PHP 8 stílushoz hasonlóbb).

## `@TransactionalEventListener` — miért fontos ez

Ez az a részlet, amit könnyű elsőre kihagyni, és ami valódi hibaforrás lehet. Ha a
`register()` metódus `@Transactional`, és a `publishEvent(...)` hívás **a tranzakción belül**
történik, egy naiv `@EventListener` **azonnal lefut**, még mielőtt a tranzakció commitolna. Ha a
tranzakció aztán mégis rollback-el (pl. egy másik validáció elbukik később ugyanabban a
metódusban), a listener már elküldte az emailt egy olyan userről, ami végül **nem is jött
létre** az adatbázisban.

```java
@Component
public class WelcomeEmailListener {

    @Async
    @TransactionalEventListener(phase = TransactionPhase.AFTER_COMMIT)
    public void onUserRegistered(UserRegisteredEvent event) {
        // csak akkor fut le, ha a tranzakció SIKERESEN commitolt
        System.out.println("Üdvözlő email küldése: " + event.getUser().getEmail());
    }
}
```

Ennek nincs éles Laravel megfelelője, mert a Laravel `event()` hívás alapból nem tud semmit a
körülötte lévő DB tranzakcióról — ha ott hasonló hibát akarsz elkerülni, neked kell manuálisan a
tranzakció `DB::afterCommit(fn() => event(...))` mintáját használnod. Spring-ben ez a viselkedés
egyetlen annotációval, deklaratívan kérhető.

A `@Async` és `@TransactionalEventListener` együtt használva pontosan azt adja, amit a bevezető
gyakorlat első lépése kér: a listener a háttérben, és csak sikeres commit után fut le.

---

[← Előző: 01. Async alapok](01-async-alapok.md) · [Fázis index](../06-async-esemenyek.md) · [Főoldal](../../README.md) · Következő: [03. Üzenetsorok →](03-uzenetsorok.md)
