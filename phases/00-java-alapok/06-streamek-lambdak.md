[← Előző: 05. Optional](05-optional.md) · [Fázis index](../00-java-alapok.md) · [Főoldal](../../README.md) · Következő: [07. Kivételkezelés →](07-kivetelkezeles.md)

# 0.6 — Streamek és lambdák

Jó hír: ez az a terület, ahol a Laravel Collection tudásod **szinte közvetlenül átül** —
a gondolkodásmód (lánc-alapú, deklaratív adat-transzformáció) majdnem azonos. A Java Stream API
és a Laravel `Collection` osztály nagyon hasonló problémára adnak nagyon hasonló megoldást.

## Lambda szintaxis

Mielőtt a streamekre térnénk, nézzük a lambda (anonim függvény) szintaxist:

```php
// PHP arrow function / closure
$double = fn($x) => $x * 2;
$filter = function ($account) {
    return $account->getBalance() > 0;
};
```

```java
// Java lambda
Function<Integer, Integer> doubleIt = x -> x * 2;
Predicate<Account> hasPositiveBalance = account -> account.getBalance() > 0;

// több paraméter, kapcsos zárójeles törzs:
BiFunction<Integer, Integer, Integer> add = (a, b) -> {
    return a + b;
};
```

Nincs `fn`/`function` kulcsszó, nincs `use()` a külső változók befogásához (Java automatikusan
"befogja" a lambda környezetében lévő, effektíve final változókat) — ez leegyszerűsíti a PHP
closure `use (&$var)` referencia-vs-érték problémáját, mert Java-ban a befogott változó mindig
csak olvasható a lambdán belül.

## Laravel Collection ↔ Java Stream — táblázatos megfeleltetés

| Laravel Collection | Java Stream | Megjegyzés |
|---|---|---|
| `collect($items)` | `items.stream()` | Stream-et egy meglévő `Collection`-ből (List, Set) hozol létre |
| `->map(fn($x) => ...)` | `.map(x -> ...)` | elem-transzformáció |
| `->filter(fn($x) => ...)` | `.filter(x -> ...)` | szűrés predikátummal |
| `->reduce(fn($c, $x) => ..., $init)` | `.reduce(init, (c, x) -> ...)` | összesítés |
| `->sum()` | `.mapToDouble(...).sum()` | számszerű összesítéshez `mapToInt`/`mapToDouble` primitív stream kell |
| `->avg()` | `.mapToDouble(...).average()` | `OptionalDouble`-t ad vissza |
| `->sortBy(fn($x) => ...)` | `.sorted(Comparator.comparing(x -> ...))` | rendezés |
| `->pluck('name')` | `.map(x -> x.getName())` | nincs "mágikus" property-elérés, explicit getter kell |
| `->groupBy(fn($x) => ...)` | `.collect(Collectors.groupingBy(x -> ...))` | csoportosítás |
| `->first()` | `.findFirst()` | `Optional<T>`-et ad vissza, nem `null`-t! |
| `->count()` | `.count()` | |
| `->toArray()` | `.collect(Collectors.toList())` vagy `.toList()` (Java 16+) | Stream → lista visszaalakítás |
| `->unique()` | `.distinct()` | |
| `->contains(fn($x) => ...)` | `.anyMatch(x -> ...)` | |
| `->every(fn($x) => ...)` | `.allMatch(x -> ...)` | |
| `->implode(', ')` | `.collect(Collectors.joining(", "))` | csak String streamen |

## Gyakorlati példa: Laravel vs Java egymás mellett

```php
// PHP — az összes 1000-nél nagyobb egyenlegű számla tulajdonosának neve, vesszővel elválasztva
$names = collect($accounts)
    ->filter(fn($a) => $a->getBalance() > 1000)
    ->map(fn($a) => $a->getOwner())
    ->sort()
    ->implode(', ');
```

```java
// Java — ugyanaz
String names = accounts.stream()
    .filter(a -> a.getBalance() > 1000)
    .map(Account::getOwner)
    .sorted()
    .collect(Collectors.joining(", "));
```

Figyeld meg az `Account::getOwner` szintaxist — ez egy **method reference**, ami helyettesíti
az `a -> a.getOwner()` lambdát, amikor a lambda csak egy meglévő metódust hív meg. Ez olyasmi,
mint egy PHP-ban a `fn($a) => $a->getOwner()` rövidítése, csak explicit nyelvi konstrukció rá.

## Stream létrehozása

```java
List<Account> accounts = List.of(acc1, acc2, acc3);
Stream<Account> stream = accounts.stream();

// tömbből
int[] numbers = {1, 2, 3};
IntStream.of(numbers);

// tartományból
IntStream.range(1, 10);          // 1..9
IntStream.rangeClosed(1, 10);    // 1..10
```

## Terminális vs köztes műveletek

Fontos koncepció, aminek nincs igazán éles PHP Collection megfelelője: a Stream **lusta**
(lazy) — a köztes műveletek (`map`, `filter`, `sorted`) nem futnak le azonnal, csak amikor egy
**terminális** művelet (`collect`, `forEach`, `count`, `reduce`) meghívódik:

```java
Stream<Account> s = accounts.stream()
    .filter(a -> a.getBalance() > 0)   // még nem fut le semmi
    .map(Account::getOwner);            // még mindig nem fut le semmi

List<String> result = s.collect(Collectors.toList());  // ITT fut le minden, egyetlen menetben
```

Ez performancia szempontból hatékonyabb, mint a Laravel Collection, ami minden lánc-elemnél
azonnal materializálja a köztes tömböt — de a te szempontodból a fontos szabály: **egy Stream
csak egyszer fogyasztható el**. Ha kétszer próbálsz terminális műveletet hívni ugyanazon a
stream-en, `IllegalStateException`-t kapsz — mindig hozz létre új stream-et (`accounts.stream()`),
ha újra végig kell menned a kollekción.

## `Collectors` — a leggyakoribb "visszaalakítók"

```java
List<String> names = accounts.stream().map(Account::getOwner).collect(Collectors.toList());

Set<String> uniqueOwners = accounts.stream().map(Account::getOwner).collect(Collectors.toSet());

Map<String, Double> balanceByOwner = accounts.stream()
    .collect(Collectors.toMap(Account::getOwner, Account::getBalance));

Map<AccountType, List<Account>> byType = accounts.stream()
    .collect(Collectors.groupingBy(Account::getType));

double total = accounts.stream()
    .collect(Collectors.summingDouble(Account::getBalance));
```

## `forEach` — amikor nem transzformálsz, csak iterálsz

```java
accounts.forEach(account -> System.out.println(account));

// klasszikus for-each ciklus, ha jobban olvasható a konkrét esetben
for (Account account : accounts) {
    System.out.println(account);
}
```

Nincs éles szabály, mikor melyiket használd — a Stream API akkor a legjobb választás, amikor
transzformálsz/szűrsz/összesítesz, a hagyományos `for` ciklus pedig akkor, amikor egyszerű
mellékhatásos iterálásról van szó (pl. logolás), és a lánc nem adna hozzá olvashatóságot.

---

[← Előző: 05. Optional](05-optional.md) · [Fázis index](../00-java-alapok.md) · [Főoldal](../../README.md) · Következő: [07. Kivételkezelés →](07-kivetelkezeles.md)
