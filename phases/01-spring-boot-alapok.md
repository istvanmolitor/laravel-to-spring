[← Előző: 0. Java alapok](00-java-alapok.md) · [Főoldal](../README.md) · Következő: [02. Adatréteg: Spring Data JPA →](02-adatreteg-jpa.md)

# 1. fázis — Spring Boot alapok és a DI mentális modell

Időtartam: ~2-3 hét

Cél: egy egyszerű REST API felállítása, és megérteni, hogy a Spring DI miben más, mint a Laravel-é.

- Spring Initializr-rel (start.spring.io) hozz létre egy projektet: `Spring Web`, `Spring Data JPA`,
  `H2`/`PostgreSQL`, `Validation`, `Lombok`
- `@SpringBootApplication`, a component scan mechanizmus — ez a Laravel autodiscovery-jének felel meg
- `@Component`, `@Service`, `@Repository`, `@Controller`/`@RestController` — mikor melyiket
- Konstruktor-alapú Dependency Injection (ez a Spring-ben *best practice*, ellentétben a Laravel
  facade-okkal, amik rejtett globális állapotot használnak) — szokj le a "mágikus" elérésről
- `@RestController` + `@GetMapping`/`@PostMapping`/`@PathVariable`/`@RequestParam`/`@RequestBody`
  — ez majdnem 1:1 megfeleltethető a Laravel route+controller párosnak
- `application.yml` konfiguráció és profilok (`dev`, `prod`) — a Laravel `.env` + `config/` párja
- `@ConfigurationProperties` — típusos konfig osztályok, ennek nincs jó Laravel megfelelője,
  de gondolj rá úgy, mint egy validált `config()` helper-re

## Gyakorlat

Építs egy "Todo API"-t CRUD végpontokkal, in-memory H2 adatbázissal.

---

[← Előző: 0. Java alapok](00-java-alapok.md) · [Főoldal](../README.md) · Következő: [02. Adatréteg: Spring Data JPA →](02-adatreteg-jpa.md)
