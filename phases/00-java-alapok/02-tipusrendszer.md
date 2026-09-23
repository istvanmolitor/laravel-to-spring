[← Előző: 01. Környezet és eszközök](01-kornyezet-es-eszkozok.md) · [Fázis index](../00-java-alapok.md) · [Főoldal](../../README.md) · Következő: [03. OOP alapok →](03-oop-alapok.md)

# 0.2 — Típusrendszer

Ez az a terület, ahol a legtöbb napi szintű súrlódásod lesz PHP-hoz képest — nem azért, mert
nehéz, hanem mert a reflexeid mást fognak várni. PHP 7+/8+ óta van type hinting, de opcionális és
sok helyen még mindig laza (`==`, implicit cast-ok). Java-ban a típusosság **kötelező és szigorú**,
és ez fordítási időben derül ki, nem futáskor.

## Statikus vs dinamikus típusosság

```php
// PHP — akár type hint nélkül is fut, csak figyelmeztet vagy semmit sem szól
function add($a, $b) {
    return $a + $b;
}

add(2, 3);        // 5
add("2", 3);      // 5 — implicit cast
add("kettő", 3);  // TypeError futásidőben (PHP 8-ban), vagy csendben NaN-szerű hiba régebbi verzióban
```

```java
// Java — a típus a szignatúra része, a fordító ellenőrzi build-kor
int add(int a, int b) {
    return a + b;
}

add(2, 3);        // OK
add("2", 3);      // fordítási hiba — ez a program el sem indul
```

Ez a legfontosabb mentális váltás: **a hibák egy jó része már a fordításnál kiderül**, mielőtt
egyetlen sor is lefutna. Ez elsőre lassabbnak tűnhet (több kód, több zajlás), de a gyakorlatban
kevesebb futásidejű meglepetést fogsz kapni, mint PHP-ban.

## Primitív típusok vs objektumok

PHP-ban minden érték kábé "ugyanúgy" viselkedik (scalar típusok: int, float, string, bool — de
nincs köztük éles határ objektum-szemantika szempontjából). Java-ban két világ van:

**Primitívek** — nem objektumok, nincs metódusuk, közvetlenül a memóriában tárolt érték:

| Típus | Méret | PHP megfelelő |
|---|---|---|
| `int` | 32 bit | `int` (de PHP int 64 bit rendszereken) |
| `long` | 64 bit | nagy `int`, PHP-ban nincs külön típus rá |
| `double` | 64 bit lebegőpontos | `float` |
| `boolean` | true/false | `bool` |
| `char` | egyetlen UTF-16 karakter | nincs pontos megfelelője (PHP-ban minden string) |
| `byte`, `short` | kisebb egészek | ritkán használod kezdőként |

**Wrapper osztályok** — minden primitívnek van objektum-párja: `Integer`, `Long`, `Double`,
`Boolean`, `Character`. Ezekre azért van szükség, mert a generikus típusok (`List<T>`) csak
objektumokkal működnek, primitívekkel nem:

```java
int x = 5;                  // primitív
Integer boxed = 5;          // wrapper objektum — "autoboxing", automatikus konverzió
List<Integer> numbers = new ArrayList<>();  // List<int> NEM létezik, kötelező a wrapper
numbers.add(5);              // itt is automatikus boxing történik a háttérben
```

Ez a PHP világból nézve furcsa plusz réteg, de a gyakorlatban a legtöbbször nem kell vele
foglalkoznod — a fordító elvégzi az autoboxing/unboxing konverziót automatikusan. Amire figyelj:
egy `Integer` lehet `null`, egy `int` **soha** — ez gyakori `NullPointerException` forrás, ha egy
adatbázisból jövő, esetleg hiányzó numerikus mezőt primitívbe próbálsz tenni.

## String összehasonlítás: a legklasszikusabb buktató

```php
// PHP — a == elég stringekre, tartalom szerint hasonlít
$a = "hello";
$b = "hello";
$a == $b;   // true
```

```java
// Java — a == referenciát hasonlít, NEM tartalmat!
String a = "hello";
String b = new String("hello");
a == b;          // false! (két különböző objektum a memóriában)
a.equals(b);      // true — ez a tartalom-összehasonlítás

// String literálok esetén a Java "string pool" miatt működhet a == is,
// de erre SOHA ne hagyatkozz, mindig .equals()-t használj
String c = "hello";
String d = "hello";
c == d;           // true — de ez implementációs részlet, ne erre építs!
```

**Szabály, amit be kell égetned**: objektumok (String, saját osztályok, wrapperek) összehasonlítására
mindig `.equals()`-t használj, `==`-t soha, kivéve ha kifejezetten azt akarod tudni, hogy két
referencia ugyanarra a memóriacímre mutat-e (ritkán akarod). Primitíveknél (`int`, `boolean`) a
`==` helyes és szükséges, mert azoknak nincs `.equals()` metódusuk.

## Típuskonverzió (casting)

```java
double d = 9.99;
int i = (int) d;       // explicit cast — 9 (levágja a törtrészt, nem kerekít!)

int x = 5;
double y = x;           // implicit — int-ből double-be mindig biztonságos (widening)

Object o = "hello";
String s = (String) o;  // objektum cast — futásidőben ClassCastException-t dobhat, ha nem stimmel
```

PHP-hoz képest itt nincs "csendes" implicit konverzió veszélyes irányba (pl. string→int, ha nem
numerikus a string) — az ilyesmi vagy fordítási hiba, vagy explicit cast-ot igényel, ami legalább
látható a kódban.

## `var` — local type inference

Java 10 óta használható a `var` kulcsszó lokális változóknál. **Ez nem dinamikus típusosság** —
a fordító ugyanúgy kikövetkezteti és rögzíti a típust fordításkor, csak nem kell kiírnod:

```java
var name = "Molitor István";   // a fordító tudja: String
var count = 5;                  // a fordító tudja: int
var accounts = new ArrayList<Account>();  // ArrayList<Account>

// name = 5;  // fordítási hiba — name típusa String marad, nem változik dinamikusan
```

Ne keverd össze a PHP dinamikus típusú változóival — a `var` csak írásmód-rövidítés, a mögöttes
típusosság ugyanolyan szigorú és statikus marad.

## Nullability

PHP-ban `null` bárhová kerülhet, és a hibák jó része csak futásidőben, néha csak élesben derül ki
(`Call to a member function on null`). Java-ban ugyanez `NullPointerException`-t dob, de mivel a
típusrendszer explicit, könnyebb védekezni ellene — erről bővebben a következő fejezetben,
az [Optional](05-optional.md) résznél.

---

[← Előző: 01. Környezet és eszközök](01-kornyezet-es-eszkozok.md) · [Fázis index](../00-java-alapok.md) · [Főoldal](../../README.md) · Következő: [03. OOP alapok →](03-oop-alapok.md)
