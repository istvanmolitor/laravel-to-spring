[← Előző: 07. Kivételkezelés](07-kivetelkezeles.md) · [Fázis index](../00-java-alapok.md) · [Főoldal](../../README.md) · Következő fázis: [01. Spring Boot alapok →](../01-spring-boot-alapok.md)

# 0.8 — Záró gyakorlat: bank szimulátor

Ez a gyakorlat az összes eddigi témát (típusrendszer, OOP, generics, Optional, streamek,
kivételkezelés) egyetlen, kis méretű, de teljes projektben gyakoroltatja be — tisztán Java-ban,
Spring nélkül. A cél nem az, hogy gyorsan végigmenj rajta, hanem hogy minden lépésnél tudatosan
alkalmazd az adott fejezet fogalmait.

## Projekt felállítása

Hozz létre egy új Maven projektet IntelliJ-ben (lásd [0.1 Környezet és eszközök](01-kornyezet-es-eszkozok.md)),
`junit-jupiter` test-scope függőséggel. Csomagszerkezet:

```
src/main/java/hu/molitor/bank/
├── Account.java
├── AccountType.java
├── SavingsAccount.java
├── CheckingAccount.java
├── Bank.java
├── Transaction.java
└── exception/
    ├── InsufficientFundsException.java
    └── AccountNotFoundException.java
src/test/java/hu/molitor/bank/
├── AccountTest.java
└── BankTest.java
```

## 1. lépés — `AccountType` enum ([0.3 OOP alapok](03-oop-alapok.md))

Hozz létre egy `AccountType` enumot `CHECKING` és `SAVINGS` értékekkel, mindegyikhez rendelj
kamatlábat (`double interestRate`), enum-konstruktorral.

## 2. lépés — `Account` absztrakt osztály ([0.3 OOP alapok](03-oop-alapok.md))

- Mezők: `owner` (String), `balance` (double), mindkettő `private`, `final` ahol értelmes
- Konstruktor, ami inicializálja őket
- `deposit(double amount)` — konkrét metódus, validálja hogy `amount > 0` (különben dobjon
  `IllegalArgumentException`-t)
- `withdraw(double amount)` — **absztrakt** metódus, a leszármazottak döntik el a szabályokat
  (pl. a `SavingsAccount` nem engedhet negatívba menni, a `CheckingAccount` engedhet egy
  túllépési keretet)
- `getBalance()`, `getOwner()` getterek
- `equals()`, `hashCode()`, `toString()` felülírása ([0.3](03-oop-alapok.md) alapján — használd
  IntelliJ generátorát, `Alt+Insert`)

## 3. lépés — `SavingsAccount` és `CheckingAccount` ([0.3 OOP alapok](03-oop-alapok.md))

Mindkettő `extends Account`, implementálják a `withdraw` metódust a saját szabályaik szerint.
A `CheckingAccount`-nak legyen egy `overdraftLimit` mezője (meddig mehet negatívba).

Ha a kivét szabályt sért, dobj egyedi kivételt:

## 4. lépés — Saját kivételosztályok ([0.7 Kivételkezelés](07-kivetelkezeles.md))

- `InsufficientFundsException extends RuntimeException` — akkor dobd, ha a kivét meghaladná az
  engedélyezett limitet
- `AccountNotFoundException extends RuntimeException` — a `Bank` osztály dobja, ha nem talál
  számlát egy adott tulajdonoshoz

Gondold át tudatosan: miért unchecked (`RuntimeException`) ezeket származtatni, és mikor
lenne indokolt checked exceptiont írni helyette? (Lásd az érvelést [0.7](07-kivetelkezeles.md)
alján.)

## 5. lépés — `Bank` osztály generikus gyűjteménnyel ([0.4 Generics](04-generics.md))

```java
public class Bank {
    private final List<Account> accounts = new ArrayList<>();

    public void addAccount(Account account) { ... }

    public Optional<Account> findByOwner(String owner) { ... }  // lásd 0.5. lépés

    public Account getByOwnerOrThrow(String owner) { ... }      // AccountNotFoundException-t dob
}
```

## 6. lépés — `Optional` a kereséshez ([0.5 Optional](05-optional.md))

A `findByOwner` visszatérési típusa legyen `Optional<Account>` — **ne** `null`-t adj vissza, ha
nincs találat. Írj rá egy másik metódust (`getByOwnerOrThrow`), ami az `Optional`
`.orElseThrow(...)` metódusával dob `AccountNotFoundException`-t, ha üres.

## 7. lépés — Riportok Stream API-val ([0.6 Streamek és lambdák](06-streamek-lambdak.md))

Implementáld a `Bank` osztályon:

- `double totalBalance()` — az összes számla egyenlegének összege
- `List<Account> accountsWithBalanceOver(double threshold)` — szűrt, egyenleg szerint csökkenő
  sorrendbe rendezett lista
- `Map<AccountType, List<Account>> groupByType()` — számlák típus szerint csoportosítva
- `String ownerSummary()` — az összes tulajdonos neve, ábécésorrendben, vesszővel elválasztva

Mindegyiket Stream API-val implementáld, ne hagyományos `for` ciklussal — a cél, hogy a
`map`/`filter`/`sorted`/`collect` láncolás a kezedre álljon, mielőtt Spring Data JPA
query-kkel és Laravel Collection-höz hasonló mintákkal dolgoznál tovább.

## 8. lépés — Unit tesztek JUnit 5-tel

Írj legalább:
- egy tesztet, ami sikeres `deposit`/`withdraw` folyamatot ellenőriz
- egy tesztet, ami `InsufficientFundsException`-t vár (`assertThrows`)
- egy tesztet, ami az `Optional`-t adó `findByOwner`-t ellenőrzi mindkét ágon (van találat / nincs
  találat)
- egy tesztet a `totalBalance()`/`groupByType()` stream-alapú riportokra

```java
@Test
void withdrawMoreThanBalanceThrows() {
    var account = new SavingsAccount("Kovács", 100.0);
    assertThrows(InsufficientFundsException.class, () -> account.withdraw(200.0));
}
```

Ez a JUnit 5 szintaxis koncepcionálisan megfelel a Pest `it('throws when ...', fn() =>
expect(fn() => ...)->toThrow(...))` mintájának — erről bővebben az [5. fázis:
Tesztelés](../05-teszteles.md) részben lesz szó, itt elég a `@Test` + `assertEquals`/`assertThrows`
alapokat használni.

## Kiterjesztési ötletek (opcionális)

- Adj hozzá `Transaction` osztályt, ami rögzíti a tranzakció típusát, összegét, időpontját
  (`java.time.LocalDateTime`), és minden `Account`-hoz egy `List<Transaction>` history-t
- Implementálj kamatszámítást (`applyInterest()`), ami az `AccountType` enum kamatlába alapján
  frissíti az egyenleget
- Írj egy `main` metódust, ami néhány számlát létrehoz, tranzakciókat hajt végre, és kiírja a
  riportokat konzolra

---

## Ellenőrző kérdések a fázis lezárásához

Ha ezekre magabiztosan tudsz válaszolni, kész vagy a következő fázisra:

1. Miért nem használható `==` String-ek tartalom szerinti összehasonlítására?
2. Mi a különbség egy `int` és egy `Integer` között, és miért van rá szükség generikus
   kollekcióknál?
3. Mikor használj `interface`-t, és mikor `abstract class`-t?
4. Miért nem lehet `List<int>`-et írni, csak `List<Integer>`-t?
5. Mikor helyes `Optional<T>`-et visszatérési típusként használni, és mikor nem ajánlott
   mezőként/paraméterként?
6. Mi a különbség egy köztes (`map`, `filter`) és egy terminális (`collect`, `forEach`) Stream
   művelet között?
7. Mi a különbség checked és unchecked exception között, és melyiket választanád saját üzleti
   logikai hibákhoz?

---

[← Előző: 07. Kivételkezelés](07-kivetelkezeles.md) · [Fázis index](../00-java-alapok.md) · [Főoldal](../../README.md) · Következő fázis: [01. Spring Boot alapok →](../01-spring-boot-alapok.md)
