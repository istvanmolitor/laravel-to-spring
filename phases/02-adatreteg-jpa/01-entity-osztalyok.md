[← Fázis index](../02-adatreteg-jpa.md) · [Főoldal](../../README.md) · Következő: [02. Kapcsolatok modellezése →](02-kapcsolatok.md)

# 2.1 — Entity osztályok

Az 1. fázisban felállítottad a Todo API-t, H2 adatbázissal. Most jön a réteg, ami a legtöbb napi
Laravel-munkádat jelenti: az adatelérés. Az Eloquent modellek helyét itt a **JPA entitások**
veszik át — ugyanaz a probléma (objektum ↔ tábla leképezés), de más filozófiával.

## A legfontosabb mentális váltás

Eloquent-ben egy modell a lehető legkevesebb kóddal is működik, mert a keretrendszer *konvencióból*
kitalálja a tábla nevét, az oszlopokat, a primary key-t. JPA-ban (Hibernate-tel, ami a Spring Boot
alapértelmezett JPA implementációja) **mindent explicit kell deklarálnod** — ez több boilerplate,
cserébe semmi nem "mágia", és a fordító sok hibát elkap, amit Eloquent-ben csak futásidőben vennél
észre.

```php
// PHP — Eloquent modell
class Todo extends Model
{
    protected $table = 'todos';                 // opcionális, konvencióból "todos" lenne
    protected $fillable = ['title', 'description', 'done'];
    protected $casts = [
        'done' => 'boolean',
        'created_at' => 'datetime',
    ];
}
```

```java
// Java — JPA entitás
@Entity
@Table(name = "todos")
public class Todo {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(nullable = false, length = 255)
    private String title;

    @Column(length = 2000)
    private String description;

    @Column(nullable = false)
    private boolean done = false;

    @Column(name = "created_at", nullable = false, updatable = false)
    private LocalDateTime createdAt = LocalDateTime.now();

    protected Todo() {
        // JPA-nak kell egy paraméter nélküli konstruktor (proxy-k, reflection miatt)
    }

    public Todo(String title, String description) {
        this.title = title;
        this.description = description;
    }

    // gettereket lásd lent
}
```

## Annotációk sorban

| Annotáció | Szerep | Eloquent megfelelő |
|---|---|---|
| `@Entity` | jelzi, hogy ez az osztály egy adatbázis táblára képződik le | `extends Model` |
| `@Table(name = "...")` | explicit tábla név, ha nem az osztálynévből (kisbetűs, többes szám) derülne ki | `protected $table` |
| `@Id` | jelzi a primary key mezőt | `protected $primaryKey` (alapból `id`) |
| `@GeneratedValue(strategy = GenerationType.IDENTITY)` | auto-increment stratégia (adatbázisra bízott) | Eloquent alapból ezt feltételezi |
| `@Column(...)` | oszlop-szintű finomhangolás: `nullable`, `length`, `name`, `unique` | `$fillable`/migráció oszlop definíció keveréke |

`@Table` és `@Column` **elhagyható**, ha megfelel neked a Hibernate alapértelmezett naming
stratégiája (osztálynév → snake_case táblanév, mezőnév → snake_case oszlopnév) — de kezdőként
érdemes mindent kiírni explicit, amíg nem ismered pontosan a konvenciót, mert egy eltérés itt nem
"csendes hiba" lesz, mint Eloquent-ben, hanem induláskor azonnal `SchemaManagementException`-t
kapsz, ha a tábla/oszlop nem egyezik.

## Nincs `$fillable`/`$guarded`, nincs mass assignment védelem

Ez fontos biztonsági különbség. Eloquent-ben a `$fillable` tömb védi a modellt a tömeges
tulajdonság-beállítástól (`Todo::create($request->all())` csak a fillable mezőket engedi be).
JPA entitásoknak **nincs ilyen beépített védelme** — egy entitás mezői simán settelhetők, ha van
setter. Emiatt (és más okokból is) a Spring világban szinte sosem az entitást fogadod be
közvetlenül a kontrollerben, hanem egy külön DTO-t — erről részletesen a
[3. fázis: DTO-k és mapping](../03-validacio-hibakezeles/02-dto-k-es-mapping.md) részben lesz szó.
Egyelőre elég tudni: **az Entity nem egyenlő az API request/response formátumával**.

## Getterek, setterek — nincs `__get`/`__set` mágia

PHP-ban megszokhattad, hogy Eloquent modelleken `$todo->title` egyszerűen működik a `__get` magic
method miatt. Java-ban nincs ilyen — minden mezőhöz explicit gettert (és ha kell, settert) kell
írnod:

```java
public Long getId() { return id; }
public String getTitle() { return title; }
public void setTitle(String title) { this.title = title; }
public boolean isDone() { return done; }          // boolean getter konvenció: "is" prefix, nem "get"
public void setDone(boolean done) { this.done = done; }
```

Ez sok gépelést jelent — a gyakorlatban a Spring projektek túlnyomó többsége **Lombok**-ot használ,
ami annotációkból generálja ezeket fordítási időben:

```java
@Entity
@Table(name = "todos")
@Getter
@Setter
@NoArgsConstructor
public class Todo {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String title;
    private String description;
    private boolean done = false;
    private LocalDateTime createdAt = LocalDateTime.now();
}
```

A `@Getter`/`@Setter`/`@NoArgsConstructor` Lombok annotációk pont azt a boilerplate-et tüntetik el,
amit fent kézzel kiírtunk — ezt már az 1. fázisban a projekt dependenciái közé vetted fel
(`Lombok` starter). Innentől a roadmap további entitás-példáiban Lombok-kal rövidített formát
fogunk használni, hacsak nincs ok kiírni a teljes kódot.

---

[← Fázis index](../02-adatreteg-jpa.md) · [Főoldal](../../README.md) · Következő: [02. Kapcsolatok modellezése →](02-kapcsolatok.md)
