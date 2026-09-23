[← Előző: 04. Generics](04-generics.md) · [Fázis index](../00-java-alapok.md) · [Főoldal](../../README.md) · Következő: [06. Streamek és lambdák →](06-streamek-lambdak.md)

# 0.5 — `Optional<T>` — a null kezelés Java módra

## Hogy kezeled ezt most PHP-ban

```php
// nullsafe operator (PHP 8+)
$city = $user?->address?->city;

// null coalescing
$name = $user->name ?? "Ismeretlen";

// explicit null check
if ($account !== null) {
    $account->withdraw(100);
}
```

PHP-ban a `null` bármely típusú változó értéke lehet, hacsak nem jelölöd explicit nem-nullable
típussal, és még akkor is a régi kód tele lehet nem védett null-okkal. A `?->` és `??` operátorok
kényelmes eszközök, de semmi nem kényszerít ki null-check-et — ha elfelejted, futásidőben kapod
a `Call to a member function on null` hibát, gyakran csak élesben.

## A probléma Java-ban

Java-ban minden objektum-referencia (nem primitív) lehet `null`, és ha egy `null` referencián
hívsz metódust, `NullPointerException`-t (röviden **NPE**) kapsz — ez a Java világ legismertebb,
leggyakoribb futásidejű hibája:

```java
Account account = findAccountByOwner("Kovács");  // mi van, ha nincs ilyen owner?
account.withdraw(100);  // NullPointerException, ha account == null
```

A `null` **nem jelenik meg a típusban** — a metódus szignatúrájából (`Account
findAccountByOwner(String owner)`) nem derül ki, hogy visszaadhat-e `null`-t. Ez pontosan az a
probléma, amit az `Optional<T>` old meg: **a típus maga jelzi**, hogy az érték hiányozhat.

## `Optional<T>` alapok

```java
import java.util.Optional;

Optional<Account> findAccountByOwner(String owner) {
    Account account = ...; // keresés logika
    return Optional.ofNullable(account);   // account lehet null, az Optional becsomagolja
}

Optional<String> present = Optional.of("hello");   // sosem lehet null belül, NPE-t dob ha az lenne
Optional<String> empty = Optional.empty();          // explicit "nincs érték"
```

A visszaadott `Optional<Account>` típus **önmagában dokumentálja**, hogy a hívónak számolnia kell
azzal, hogy nincs találat — ez az, amit a Laravel `?Account` nullable típushint (PHP 7.1+) jelez,
csak itt egy külön "doboz" objektum reprezentálja, nem maga a nullable típus.

## Az Optional használata — ne csak `.get()`-elj mindenhol

Kezdő hiba, amit el fogsz követni párszor: `optional.get()`-et hívni ellenőrzés nélkül — ez
pontosan visszahozza az eredeti NPE problémát, csak becsomagolva. Az `Optional` igazi ereje a
funkcionális stílusú kezelésben van:

```java
Optional<Account> maybeAccount = findAccountByOwner("Kovács");

// rossz — ez csak áttolja a null problémát
Account account = maybeAccount.get();  // NoSuchElementException, ha üres

// jó — alapértelmezett érték megadása
Account account = maybeAccount.orElse(new Account("Ismeretlen", 0));

// jó — lusta kiértékelésű alapérték (csak akkor fut le, ha tényleg kell)
Account account = maybeAccount.orElseGet(() -> createDefaultAccount());

// jó — explicit hibát dobsz, ha nincs érték (kontrollált, dokumentált hiba)
Account account = maybeAccount.orElseThrow(() ->
    new NoSuchElementException("Nincs ilyen ügyfél: Kovács"));

// jó — csak akkor csinálsz valamit, ha van érték
maybeAccount.ifPresent(acc -> acc.withdraw(100));

// jó — van érték vagy nincs, mindkét ágat kezeled
maybeAccount.ifPresentOrElse(
    acc -> System.out.println("Talált: " + acc),
    () -> System.out.println("Nincs ilyen számla")
);
```

## `map` és `filter` — Laravel Collection-szerű láncolás

```java
Optional<Account> maybeAccount = findAccountByOwner("Kovács");

// map: ha van érték, transzformáld; ha nincs, marad üres
Optional<Double> maybeBalance = maybeAccount.map(Account::getBalance);

// filter: ha van érték, de nem felel meg a feltételnek, üressé válik
Optional<Account> richAccount = maybeAccount.filter(acc -> acc.getBalance() > 1000);

// láncolva, ahogy Laravel Collection pipeline-t építenél:
String summary = findAccountByOwner("Kovács")
    .filter(acc -> acc.getBalance() > 0)
    .map(acc -> acc.getOwner() + ": " + acc.getBalance())
    .orElse("Nincs aktív egyenleg");
```

Ez gondolatilag nagyon hasonlít a Laravel `collect($x)->when(...)->map(...)` láncolásra, csak itt
0 vagy 1 elemű "kollekcióról" van szó (van érték / nincs érték), nem tetszőleges méretűről.

## Mikor NE használj Optional-t

Fontos konvenció, amit be kell tartanod:

- **Ne** használj `Optional`-t osztály mezőkön (field-eken) — erre a `null` és a validáció a
  helyes eszköz, az `Optional` nem szerializálható jól, és nem erre találták ki
- **Ne** használj `Optional`-t metódusparaméterként — helyette írj két overloadot, vagy fogadd el
  a `null`-t és dokumentáld
- **Igen**, használd metódusok **visszatérési típusaként**, amikor a "nincs érték" egy legitim,
  várt eset (pl. `findById` — lehet, hogy nincs ilyen ID)

Ez a szabály nem véletlen bürokrácia — az `Optional`-t kifejezetten visszatérési érték
wrapper-nek tervezték, és a Spring Data JPA is pontosan így használja: `Optional<User>
findByEmail(String email)`.

---

[← Előző: 04. Generics](04-generics.md) · [Fázis index](../00-java-alapok.md) · [Főoldal](../../README.md) · Következő: [06. Streamek és lambdák →](06-streamek-lambdak.md)
