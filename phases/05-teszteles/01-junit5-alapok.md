[← Fázis index](../05-teszteles.md) · [Főoldal](../../README.md) · Következő: [02. Mockito →](02-mockito.md)

# 5.1 — JUnit 5 alapok

A tesztelési *gondolkodásmód* nem változik PHP-hoz képest — amit Pest-ben vagy PHPUnit-ban
megszoktál (arrange-act-assert, egy teszt egy dolgot ellenőriz, leíró tesztnevek), az itt is
érvényes. Ami változik, az a szintaxis és az, hogy Java-ban a tesztek is lefordított, típusos
kód, nem futásidőben összerakott closure-ök.

## Alapszerkezet

```php
// Pest
it('deposits money into the account', function () {
    $account = new Account('Kovács', 100);
    $account->deposit(50);
    expect($account->getBalance())->toBe(150.0);
});
```

```java
// JUnit 5
import org.junit.jupiter.api.Test;
import static org.junit.jupiter.api.Assertions.assertEquals;

class AccountTest {
    @Test
    void depositsMoneyIntoTheAccount() {
        Account account = new Account("Kovács", 100);
        account.deposit(50);
        assertEquals(150.0, account.getBalance());
    }
}
```

Fő különbségek:
- Nincs closure-alapú `it('...', function () {...})` regisztráció — minden teszt egy `@Test`
  annotációval jelölt metódus egy teszt osztályban
- A metódusnév maga a "leírás" — nincs külön string címke, ezért a metódusneveket camelCase-ben,
  de olvashatóan írd (`depositsMoneyIntoTheAccount`, `throwsWhenBalanceIsInsufficient`).
  `@DisplayName("...")` annotációval adhatsz emberi olvasásra szánt címet is, ha a metódusnév
  önmagában nem elég beszédes
- Az `assertEquals(expected, actual)` sorrendje fordított ahhoz képest, ahogy a Pest
  `expect($actual)->toBe($expected)` olvastatja — ez elsőre sok hibás asserthez vezet, amíg be nem
  gyakorlod

## Gyakori assertion-ök

```java
assertEquals(150.0, account.getBalance());
assertTrue(account.isActive());
assertFalse(account.isClosed());
assertNull(result);
assertNotNull(account);
assertThrows(InsufficientFundsException.class, () -> account.withdraw(1000));
assertAll(
    () -> assertEquals("Kovács", account.getOwner()),
    () -> assertEquals(150.0, account.getBalance())
);
```

Az `assertThrows` a Pest `expect(fn() => ...)->toThrow(...)` mintájának felel meg — mindkettő egy
lambdát/closure-t vár, ami a kivételt dobó hívást tartalmazza.

## Életciklus-annotációk

```java
class TodoServiceTest {

    private TodoService service;

    @BeforeEach
    void setUp() {
        service = new TodoService(new InMemoryTodoRepository());
    }

    @AfterEach
    void tearDown() {
        // erőforrás felszabadítás, ha kell
    }

    @Test
    void createsNewTodo() {
        // ...
    }
}
```

| JUnit 5 | Pest | Mikor fut |
|---|---|---|
| `@BeforeEach` | `beforeEach(function () {...})` | minden teszt előtt |
| `@AfterEach` | `afterEach(function () {...})` | minden teszt után |
| `@BeforeAll` (static metódus) | `beforeAll(function () {...})` | egyszer, az osztály összes tesztje előtt |
| `@AfterAll` (static metódus) | `afterAll(function () {...})` | egyszer, az osztály összes tesztje után |

A `@BeforeEach` a leggyakoribb — ide kerül a "friss állapot minden teszthez" logika, ahogy egy
Pest `beforeEach`-ben is a teszt-fixture-öket állítanád össze.

## Paraméterezett tesztek

```php
// Pest
it('validates amount', function (float $amount, bool $expected) {
    expect(Account::isValidAmount($amount))->toBe($expected);
})->with([
    [100.0, true],
    [-5.0, false],
    [0.0, false],
]);
```

```java
// JUnit 5
@ParameterizedTest
@CsvSource({
    "100.0, true",
    "-5.0, false",
    "0.0, false"
})
void validatesAmount(double amount, boolean expected) {
    assertEquals(expected, AccountUtils.isValidAmount(amount));
}
```

Ez ugyanazt a "adatvezérelt teszt" mintát valósítja meg, mint a Pest `->with([...])` dataset-je,
csak Java-ban explicit annotáció (`@CsvSource`, vagy összetettebb esetekre `@MethodSource`) adja
meg a bemeneti adatokat.

## Maven integráció

```bash
mvn test              # az összes teszt lefuttatása
mvn test -Dtest=TodoServiceTest   # csak egy osztály tesztjei
```

Ez a `php artisan test` / `./vendor/bin/pest` megfelelője — VS Code-ban a bal oldali **Testing**
nézetben (Erlenmeyer-lombik ikon), IntelliJ-ben pedig a sor elején megjelenő zöld play gombbal
ugyanígy egyenként vagy osztályszinten is futtathatod/debugolhatod a teszteket.

---

[← Fázis index](../05-teszteles.md) · [Főoldal](../../README.md) · Következő: [02. Mockito →](02-mockito.md)
