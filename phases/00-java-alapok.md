[← Főoldal](../README.md) · Következő: [01. Spring Boot alapok és a DI mentális modell →](01-spring-boot-alapok.md)

# 0. fázis — Java alapok

Időtartam: ~1-2 hét

Cél: annyi Java nyelvi tudás, hogy a Spring kódot ne a szintaxis, hanem a koncepció nehezítse.

- Típusrendszer: statikus típusosság, primitívek vs. objektumok (`int` vs `Integer`), erős típusosság
  PHP-hoz képest (nincs laza `==`, nincs implicit type juggling)
- OOP Java-ban: interfészek vs. absztrakt osztályok, `final`, package-visibility, konstruktorok
- Generics (`List<String>`, `Optional<T>`) — ennek nincs igazi PHP megfelelője, erre szánj több időt
- `Optional<T>` mint a `null` kezelés Laravel-es "?." helyett
- Streamek és lambdák (`list.stream().filter(...).map(...).collect(...)`) — gondolj rá úgy, mint
  a Laravel Collection-ökre (`collect($items)->filter()->map()`), a gondolkodásmód szinte azonos
- Checked vs unchecked exceptionök (ez PHP-ban nem létezik, ez lesz az egyik legfurcsább rész)
- Build eszköz: válaszd a **Maven**-t kezdésnek (egyszerűbb, mint Gradle, jobban dokumentált)
- IDE: **IntelliJ IDEA Community** (a Spring ökoszisztéma szinte ehhez van optimalizálva)

## Gyakorlat

Írj pár kis konzolos programot (pl. egy egyszerű bank-szimulátor osztályokkal, kivételkezeléssel,
stream-alapú összegzésekkel) Spring nélkül, tisztán Java-ban.

---

[← Főoldal](../README.md) · Következő: [01. Spring Boot alapok és a DI mentális modell →](01-spring-boot-alapok.md)
