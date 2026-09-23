[← Előző: 01. Projekt kiválasztása](01-projekt-kivalasztasa.md) · [Fázis index](../08-projekt.md) · [Főoldal](../../README.md) · Következő: [03. Migrációs stratégia →](03-migracios-strategia.md)

# 8.2 — Technológiai stack

A javasolt stack nem véletlenszerű — minden eleme egy korábbi roadmap-fázisban lett bevezetve.
A cél, hogy ezen a projekten ne új technológiákat tanulj, hanem a már megtanultakat kombináld
egy koherens, éles minőségű rendszerré.

## Az összeállítás

| Réteg | Választás | Melyik fázisban tanultad |
|---|---|---|
| Nyelv/futtatókörnyezet | Java 21 (LTS) | [0. fázis](../../00-java-alapok.md) |
| Keretrendszer | Spring Boot 3.x | [1. fázis](../01-spring-boot-alapok.md) |
| Build eszköz | Maven | [0.1](../00-java-alapok/01-kornyezet-es-eszkozok.md) |
| Adatréteg | Spring Data JPA + Hibernate | [2. fázis](../02-adatreteg-jpa.md) |
| Adatbázis | PostgreSQL | [2. fázis](../02-adatreteg-jpa.md) |
| Migráció | Flyway | [2. fázis](../02-adatreteg-jpa.md) |
| Architektúra | Controller → Service → Repository + DTO-k | [3. fázis](../03-validacio-hibakezeles.md) |
| Autentikáció | Spring Security + JWT | [4. fázis](../04-spring-security.md) |
| Tesztelés | JUnit 5 + Mockito + Testcontainers | [5. fázis](../05-teszteles.md) |
| Háttérfolyamatok | `@Async`/`@EventListener`, esetleg RabbitMQ | [6. fázis](../06-async-esemenyek.md) |
| Megfigyelhetőség | Spring Boot Actuator + strukturált logolás | [7. fázis](../07-observability.md) |
| Konténerizáció | Docker Compose (app + db + opcionálisan Redis/RabbitMQ) | [2. fázis](../02-adatreteg-jpa.md) gyakorlata |
| CI | GitHub Actions | [8.4](04-ci-cd.md) — itt, most vezetjük be |

## Miért pont ezeket — röviden indokolva

**Java 21, nem a legújabb verzió minden áron.** LTS (Long-Term Support) kiadás — ez az, amit
egy induló Spring csapat is választana, mert kiszámítható a támogatási időtáv, és minden
friss Spring Boot 3.x dokumentáció/tutorial erre épül.

**PostgreSQL, nem H2 vagy MySQL.** A gyakorló Todo API-nál (1. fázis) H2-vel kezdtél, hogy
gyorsan indulhass, majd a 2. fázisban átálltál PostgreSQL-re — ezen a komplett projekten már
kezdettől PostgreSQL-t használj, mert ez az, amit egy éles Spring csapat is tipikusan választ, és
a H2/PostgreSQL viselkedésbeli eltérései (típuskezelés, dátumformátum) csak felesleges
meglepetést okoznának most.

**Flyway, nem Hibernate `ddl-auto: update`.** A Hibernate tud automatikusan sémát generálni
fejlesztés közben, de éles projektben ez kockázatos (nem kontrollált, nem verziózott
séma-változás) — pontosan úgy, ahogy Laravel-ben sem hagynád, hogy Eloquent "kitalálja" a
séma szerkezetét migrációk nélkül.

**Spring Security + saját JWT megoldás, nem OAuth2/Keycloak.** Egy komplett külső identity
provider (Keycloak, Auth0) bevezetése önmagában egy külön tanulási görbe — a 4. fázisban
megtanult, saját kezűleg felépített JWT-megoldás pontosan annyi komplexitást ad, amennyi egy
Sanctum/Passport-tal ekvivalens Laravel projekt portolásához kell, se többet, se kevesebbet.

**Testcontainers a tesztekhez.** A Laravel oldalon valószínűleg SQLite in-memory-t vagy egy
dedikált teszt-adatbázist használtál — itt, mivel a projekt már PostgreSQL-specifikus SQL-t is
tartalmazhat (Flyway migrációkban), a Testcontainers biztosítja, hogy a tesztek ugyanazon az
adatbázis-motoron fussanak, mint amin élesben futna a rendszer.

**Docker Compose, nem natívan telepített PostgreSQL/RabbitMQ.** Ez megegyezik azzal az élménnyel,
amit Laravel Sail vagy Docker-alapú Laravel fejlesztői környezet nyújt — egy paranccsal
(`docker compose up`) elindul a teljes függőség-halmaz, mindenki gépén ugyanúgy.

## Projekt-vázlat felállítása

Spring Initializr-rel (lásd [1.1 fázis](../01-spring-boot-alapok/01-projekt-inditasa.md)) hozd
létre a projektet az alábbi dependenciákkal:

- `Spring Web`
- `Spring Data JPA`
- `PostgreSQL Driver`
- `Flyway Migration`
- `Spring Security`
- `Validation`
- `Lombok`
- `Spring Boot Actuator`

Ezután add hozzá kézzel a `pom.xml`-hez a JWT könyvtárat (pl. `jjwt-api`/`jjwt-impl`/`jjwt-jackson`),
a Testcontainers dependenciákat (`testcontainers`, `postgresql`, `junit-jupiter` — mind `test`
scope-ban), és ha a projekted igényli, a `spring-boot-starter-amqp`-t a RabbitMQ integrációhoz.

---

[← Előző: 01. Projekt kiválasztása](01-projekt-kivalasztasa.md) · [Fázis index](../08-projekt.md) · [Főoldal](../../README.md) · Következő: [03. Migrációs stratégia →](03-migracios-strategia.md)
