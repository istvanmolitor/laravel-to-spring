[← Előző: 02. Típusrendszer](02-tipusrendszer.md) · [Fázis index](../00-java-alapok.md) · [Főoldal](../../README.md) · Következő: [04. Generics →](04-generics.md)

# 0.3 — OOP alapok Java-ban

Az OOP fogalmak (osztály, öröklés, interfész, encapsulation) ugyanazok, amiket Laravel-ből
ismersz — a PHP OOP modellje nagyrészt a Java-ét másolta le anno. A különbségek inkább
szigorúsági szintben és konvenciókban vannak.

## Osztályok és konstruktorok

```php
// PHP
class Account {
    private string $owner;
    private float $balance;

    public function __construct(string $owner, float $balance) {
        $this->owner = $owner;
        $this->balance = $balance;
    }
}
```

```java
// Java
public class Account {
    private String owner;
    private double balance;

    public Account(String owner, double balance) {
        this.owner = owner;
        this.balance = balance;
    }
}
```

Fő különbségek:
- Nincs `__construct` — a konstruktor neve **megegyezik az osztály nevével**
- Nincs `$this->prop = $prop` "constructor promotion" rövidítés (PHP 8-ban `public function
  __construct(private string $owner)`) — Java-ban ki kell írnod a mezőt és a hozzárendelést is,
  *kivéve* ha Lombok-ot használsz (erről a Spring fázisban lesz szó, `@RequiredArgsConstructor`)
- Minden mezőt és metódust explicit láthatósággal kell ellátni — nincs "alapértelmezett public",
  mint PHP-ban class property-knél régebben volt

## Láthatósági szintek (access modifierek)

PHP-ban hármat ismersz: `public`, `protected`, `private`. Java-ban **négy** szint van, mert van egy
plusz, "package-private" nevű, amit akkor kapsz, ha **nem írsz ki semmit**:

| Modifier | Ki éri el | PHP megfelelő |
|---|---|---|
| `public` | bárki | `public` |
| `protected` | ugyanaz a package + leszármazott osztályok bárhol | `protected` (de PHP-ban nincs package-fogalom) |
| *(nincs kulcsszó)* — package-private | csak ugyanabban a package-ben lévő osztályok | nincs PHP megfelelője |
| `private` | csak az adott osztály | `private` |

```java
package hu.molitor.bank;

class InternalHelper {   // nincs "public" előtte → csak a hu.molitor.bank package-en belül látható
    void doSomething() { }  // nincs modifier → package-private metódus
}
```

Ez a package-private szint az, amivel PHP-ban sosem találkoztál — gondolj rá úgy, mint egy
"csak ezen a modulon belüli" láthatóságra, amit Laravel-ben legfeljebb konvencióval (namespace
dokumentációval) tudnál jelezni, itt viszont a fordító kikényszeríti.

## `final` kulcsszó

```java
public final class Account { }        // nem lehet leszármazni belőle
public class Account {
    public final void deposit() { }   // nem lehet felülírni (override-olni) leszármazottban
}
public void method(final String x) {  // x nem újra-értékadható a metóduson belül
}
```

Ha követted a Laravel közösség "final class by default" trendjét (Adam Wathan, Spatie stílus),
ez ismerős lesz — a Java világban a `final` osztály még inkább alapértelmezett gyakorlat
könyvtáraknál és DTO-knál, mert explicit jelzi: ez az osztály nem öröklésre, hanem kompozícióra
készült.

## Interfészek és absztrakt osztályok

```java
public interface PaymentMethod {
    void pay(double amount);

    // Java 8 óta lehet default implementáció is az interfészben:
    default String describe() {
        return "Generic payment method";
    }
}

public abstract class BaseAccount {
    protected double balance;

    public abstract void withdraw(double amount);  // kötelező implementálni

    public double getBalance() {                    // konkrét, örökölhető metódus
        return balance;
    }
}

public class CheckingAccount extends BaseAccount implements PaymentMethod {
    @Override
    public void withdraw(double amount) {
        balance -= amount;
    }

    @Override
    public void pay(double amount) {
        withdraw(amount);
    }
}
```

Ez koncepcionálisan majdnem 1:1 megegyezik a PHP `interface`/`abstract class` párossal —
`extends` egy osztályt (csak egyet, Java-ban sincs többszörös osztály-öröklés), `implements`
tetszőleges számú interfészt. Az `@Override` annotáció nem kötelező, de mindig írd ki — a fordító
ellenőrzi, hogy valóban felülírsz egy meglévő metódust, és elgépelés esetén hibát dob (PHP-ban
ez a fajta védőháló nincs meg).

## `equals()`, `hashCode()`, `toString()`

PHP-ban az objektum-összehasonlítás (`==` objektumokon tartalom szerint, `===` referencia szerint)
és a `__toString()` magic method ismerős. Java-ban ez explicit metódusok felülírásával működik:

```java
public class Account {
    private final String owner;
    private final double balance;

    // ... konstruktor ...

    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (!(o instanceof Account other)) return false;
        return Double.compare(balance, other.balance) == 0
            && owner.equals(other.owner);
    }

    @Override
    public int hashCode() {
        return Objects.hash(owner, balance);
    }

    @Override
    public String toString() {
        return "Account{owner='" + owner + "', balance=" + balance + "}";
    }
}
```

Fontos szabály: ha felülírod `equals()`-t, **mindig** írd felül `hashCode()`-ot is (kontraktus:
két egyenlő objektumnak ugyanaz kell legyen a hashCode-ja, különben `HashMap`/`HashSet` rosszul
fog viselkedni). Ezt a párt a gyakorlatban Lombok-kal vagy IDE-generálással szoktad megoldani, nem
kézzel írod — VS Code-ban jobb klikk → `Source Action...` → `Generate hashCode() and equals()`.

## Enumok

PHP 8.1 óta van enum, Java-ban ez sokkal régebbi és gazdagabb funkciójú:

```java
public enum AccountType {
    CHECKING, SAVINGS, CREDIT;
}

public enum AccountType {
    CHECKING(0.0),
    SAVINGS(0.02),
    CREDIT(-0.15);

    private final double interestRate;

    AccountType(double interestRate) {           // konstruktor az enumon belül
        this.interestRate = interestRate;
    }

    public double getInterestRate() {
        return interestRate;
    }
}

// használat
AccountType type = AccountType.SAVINGS;
switch (type) {
    case CHECKING -> System.out.println("Checking account");
    case SAVINGS -> System.out.println("Savings account");
    case CREDIT -> System.out.println("Credit account");
}
```

A Java enum minden konstansa egyben egy szingleton objektum-példány is, saját mezőkkel és
metódusokkal — ez erősebb, mint a PHP `enum: string`/`enum: int` backed enum, ahol csak egy
skalár értéket társítasz.

## Statikus tagok

```java
public class AccountUtils {
    public static final double MIN_BALANCE = 0.0;  // ~ konstans, mint PHP `const`

    public static boolean isValidAmount(double amount) {  // ~ PHP static function
        return amount > 0;
    }
}

// hívás: nincs "self::" vagy "static::" nüansz, mindig az osztály nevével hívod
AccountUtils.isValidAmount(100.0);
```

---

[← Előző: 02. Típusrendszer](02-tipusrendszer.md) · [Fázis index](../00-java-alapok.md) · [Főoldal](../../README.md) · Következő: [04. Generics →](04-generics.md)
