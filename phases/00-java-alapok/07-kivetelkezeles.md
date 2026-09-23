[← Előző: 06. Streamek és lambdák](06-streamek-lambdak.md) · [Fázis index](../00-java-alapok.md) · [Főoldal](../../README.md) · Következő: [08. Gyakorlat: bank szimulátor →](08-gyakorlat-bank-szimulator.md)

# 0.7 — Kivételkezelés

PHP-ban minden kivétel közös ősosztályból (`Throwable` → `Exception`/`Error`) származik, és a
`try/catch` blokkban bármelyiket elkaphatod, anélkül hogy a metódus szignatúrája jelezné, mit
dobhat. Java-ban ez a rész **szigorúbb és explicitebb** — ez lesz az egyik legfurcsább újdonság,
amivel találkozol.

## Checked vs unchecked exception — ez nem létezik PHP-ban

Java-ban a kivételeknek két nagy csoportja van:

```
Throwable
├── Error                          (súlyos, nem kezelendő — pl. OutOfMemoryError)
└── Exception
    ├── RuntimeException           ← UNCHECKED — nem kötelező kezelni/deklarálni
    │   ├── NullPointerException
    │   ├── IllegalArgumentException
    │   ├── IllegalStateException
    │   └── ...
    └── (minden más Exception)     ← CHECKED — KÖTELEZŐ kezelni vagy deklarálni!
        ├── IOException
        ├── SQLException
        └── ...
```

A **checked exception** a lényeg: ha egy metódus dobhat egy checked exceptiont, ezt a
szignatúrájában **kötelező deklarálnia** (`throws`), és minden hívónak vagy el kell kapnia, vagy
tovább kell dobnia. A fordító kényszeríti ki — ha elfelejted, a kód nem fordul le.

```java
import java.io.IOException;
import java.nio.file.Files;
import java.nio.file.Path;

// checked exception: a metódus szignatúrája jelzi, mit dobhat
public String readFile(String path) throws IOException {
    return Files.readString(Path.of(path));
}

// a hívónak kötelező reagálnia:
public void process() {
    try {
        String content = readFile("data.txt");
    } catch (IOException e) {
        System.err.println("Nem sikerült beolvasni: " + e.getMessage());
    }
}

// vagy továbbdobja, de akkor a saját szignatúrájában is jeleznie kell:
public void process() throws IOException {
    String content = readFile("data.txt");   // nincs try/catch, de a throws deklaráció kötelező
}
```

PHP-ban ehhez foghatót sosem kellett csinálnod — ott bármilyen exception "csendben átrepülhet"
egy függvényen, amíg valahol fel nem kapja egy `catch`. Java-ban a checked exceptionök a fordítót
használják dokumentációként és biztosítékként: **nem felejtheted el kezelni** egy olyan hibát,
amiről a könyvtár készítője tudta, hogy be fog következni (pl. fájl nem található, hálózati hiba).

Az **unchecked** (`RuntimeException` és leszármazottjai) ezzel szemben pont úgy viselkedik, mint
a PHP exceptionök — nem kötelező deklarálni vagy elkapni, bárhol felbukkanhat, tipikusan
programozói hibát vagy váratlan állapotot jelez:

```java
public void withdraw(double amount) {
    if (amount > balance) {
        throw new IllegalStateException("Nincs elég fedezet");  // unchecked, nincs throws kötelezettség
    }
    balance -= amount;
}
```

**Gyakorlati szabály, amit érdemes megjegyezni**: saját üzleti logikai hibáidhoz (pl.
"insufficient funds", "account not found") a legtöbb modern Java/Spring kódbázis
**unchecked exceptiont** használ (RuntimeException leszármazott), és a checked exceptionöket
inkább csak akkor, ha valóban külső, elkerülhetetlen körülményről van szó (fájl I/O, hálózat).
Ez konzisztens azzal, ahogy Laravel-ben is egyetlen exception hierarchiával dolgozol, csak itt
explicit döntened kell, melyik ágba illik a saját kivételed.

## `try` / `catch` / `finally`

```php
// PHP
try {
    $account->withdraw(1000);
} catch (InsufficientFundsException $e) {
    echo "Hiba: " . $e->getMessage();
} finally {
    logTransaction();
}
```

```java
// Java — szinte azonos
try {
    account.withdraw(1000);
} catch (InsufficientFundsException e) {
    System.out.println("Hiba: " + e.getMessage());
} finally {
    logTransaction();
}

// több kivételtípus egy catch ágban (Java 7+):
try {
    riskyOperation();
} catch (IOException | SQLException e) {
    System.err.println("Hiba: " + e.getMessage());
}
```

## Saját kivétel osztály

```java
public class InsufficientFundsException extends RuntimeException {
    public InsufficientFundsException(String message) {
        super(message);
    }
}

// használat
if (amount > balance) {
    throw new InsufficientFundsException(
        "A számla egyenlege (%.2f) nem fedezi a kivett összeget (%.2f)"
            .formatted(balance, amount)
    );
}
```

Ez majdnem pontosan úgy néz ki, mint a PHP megfelelője (`class InsufficientFundsException extends
Exception`), csak itt eldöntötted, hogy `RuntimeException`-ből (unchecked) származtatod, nem
`Exception`-ből (ami checked lenne).

## `try-with-resources` — automatikus erőforrás-felszabadítás

Ennek nincs pontos PHP megfelelője (leginkább a Laravel `DB::transaction(fn() => ...)`
automatikus commit/rollback zárásához hasonlítható gondolatilag). Bármi, ami implementálja az
`AutoCloseable` interfészt (fájlok, adatbázis kapcsolatok), automatikusan lezáródik a blokk
végén, hiba esetén is:

```java
try (var reader = Files.newBufferedReader(Path.of("data.txt"))) {
    String line = reader.readLine();
    // a reader automatikusan bezáródik a blokk végén, akkor is, ha exception repül
} catch (IOException e) {
    System.err.println("Hiba: " + e.getMessage());
}
```

Ez helyettesíti azt, amit PHP-ban kézzel írt `finally { fclose($handle); }` blokkal oldanál meg —
itt a nyelv garantálja a felszabadítást, nem kell rá emlékezned minden esetben.

---

[← Előző: 06. Streamek és lambdák](06-streamek-lambdak.md) · [Fázis index](../00-java-alapok.md) · [Főoldal](../../README.md) · Következő: [08. Gyakorlat: bank szimulátor →](08-gyakorlat-bank-szimulator.md)
