[← Előző: 03. OOP alapok](03-oop-alapok.md) · [Fázis index](../00-java-alapok.md) · [Főoldal](../../README.md) · Következő: [05. Optional →](05-optional.md)

# 0.4 — Generics

Ez az a témakör, aminek **nincs igazi PHP megfelelője**, ezért érdemes rá extra időt szánni.
PHP-ban egy tömb bármit tartalmazhat, és legfeljebb PHPDoc kommenttel (`@param Account[] $accounts`)
jelzed a szándékodat — ezt semmi nem kényszeríti ki, csak a statikus analízis eszközök (PHPStan,
Psalm) tudják ellenőrizni, ha használod őket. Java-ban a generikus típusok a nyelv **beépített,
fordító által kikényszerített** részei.

## Miért kell ez?

```php
// PHP — futásidőben derül csak ki, ha valami "nem oda illő" kerül a tömbbe
function totalBalance(array $accounts): float {
    $sum = 0;
    foreach ($accounts as $account) {
        $sum += $account->getBalance();   // ha $account nem Account, itt robban
    }
    return $sum;
}
```

```java
// Java — a List<Account> garantálja fordítási időben, hogy csak Account kerülhet bele
double totalBalance(List<Account> accounts) {
    double sum = 0;
    for (Account account : accounts) {
        sum += account.getBalance();      // biztosan Account, nincs futásidejű meglepetés
    }
    return sum;
}

List<Account> accounts = new ArrayList<>();
accounts.add(new Account("Kovács", 1000));
// accounts.add("hello");  // FORDÍTÁSI HIBA — ezt a hibát PHP-ban csak futásidőben kapnád el
```

## Alapszintaxis

```java
List<String> names = new ArrayList<>();
Map<String, Integer> ages = new HashMap<>();
Optional<Account> maybeAccount = Optional.empty();

// a "gyémánt operátor" <> jobb oldalon üresen hagyható Java 7+ óta,
// a fordító kikövetkezteti a bal oldali deklarációból
```

A `<T>` a típusparaméter — gondolj rá úgy, mint egy "kitöltendő sablonra": a `List<T>` azt mondja,
"lista, aminek az elemtípusát majd megadjuk használatkor". Ez nem PHP `array` docblock-kommentár,
hanem valódi, fordító által ellenőrzött típusinformáció.

## Saját generikus osztály írása

```java
public class Box<T> {
    private T content;

    public void put(T content) {
        this.content = content;
    }

    public T get() {
        return content;
    }
}

Box<String> stringBox = new Box<>();
stringBox.put("hello");
String value = stringBox.get();    // nincs szükség cast-ra, a fordító tudja, hogy String

Box<Account> accountBox = new Box<>();
accountBox.put(new Account("Nagy", 500));
```

Több típusparaméter is lehet, ahogy egy PHP-ban a `Collection`-ökben kulcs-érték párokat
kezelnél:

```java
public class Pair<K, V> {
    private final K key;
    private final V value;

    public Pair(K key, V value) {
        this.key = key;
        this.value = value;
    }

    public K getKey() { return key; }
    public V getValue() { return value; }
}

Pair<String, Double> balance = new Pair<>("Kovács", 1500.0);
```

## Generikus metódusok

```java
public class ListUtils {
    public static <T> T firstOrNull(List<T> list) {
        return list.isEmpty() ? null : list.get(0);
    }
}

String first = ListUtils.firstOrNull(names);     // T = String
Account firstAcc = ListUtils.firstOrNull(accounts); // T = Account
```

## Wildcard-ok — csak felismerés szintjén

Haladóbb téma, amivel majd Spring kódban fogsz találkozni, de elsőre elég felismerni:

```java
// "valamilyen Account vagy annak leszármazottja listája, csak olvasásra"
void printAll(List<? extends Account> accounts) { ... }

// "valamilyen lista, amibe Account-ot (vagy ősosztályát) lehet írni"
void addDefaults(List<? super Account> accounts) { ... }
```

Ökölszabály (amit "PECS" néven szokás megjegyezni: *Producer Extends, Consumer Super*): ha a
paraméterből csak olvasol, `? extends`; ha csak beleírsz, `? super`. Kezdőként elég, ha tudod,
hogy ez létezik, és felismered Spring/könyvtár kódban — saját magadnak ritkán kell írnod.

## Type erasure — érdekesség, nem gyakorlati akadály

A Java generics fordítási időben létezik, futásidőben a JVM "elfelejti" a konkrét típusparamétert
(ún. *type erasure*) — ez performancia és visszafelé-kompatibilitási döntés volt a nyelv
történetében. Gyakorlati következmény: nem tudsz futásidőben rákérdezni, hogy egy `List<String>`
vagy `List<Integer>` volt-e eredetileg (`list.getClass()` mindkettőnél ugyanazt adja vissza).
Kezdőként ez ritkán fog problémát okozni, de ha egyszer furcsa `unchecked cast` warningot látsz,
ez áll a háttérben.

---

[← Előző: 03. OOP alapok](03-oop-alapok.md) · [Fázis index](../00-java-alapok.md) · [Főoldal](../../README.md) · Következő: [05. Optional →](05-optional.md)
