[← Előző fázis: 0. Java alapok](00-java-alapok.md) · [Főoldal](../README.md) · Következő fázis: [02. Adatréteg: Spring Data JPA →](02-adatreteg-jpa.md)

# 1. fázis — Spring Boot alapok és a DI mentális modell

Időtartam: ~2-3 hét

Cél: egy egyszerű REST API felállítása, és megérteni, hogy a Spring DI miben más, mint a Laravel-é.
Ehhez a fázishoz a [0. fázis](00-java-alapok.md) Java nyelvi alapjai (típusrendszer, OOP,
Optional) szükségesek — innentől feltételezzük, hogy ezekkel már magabiztosan bánsz, és a
fókusz a Spring-specifikus fogalmakra kerül.

Ez a fázis is alfejezetekre van bontva, mindegyik konkrét PHP/Laravel összehasonlításokkal.
Haladj sorban — a záró gyakorlat (Todo API) az összes korábbi alfejezetre épít, és a roadmap
további fázisaiban tovább bővül:

1. [Projekt indítása](01-spring-boot-alapok/01-projekt-inditasa.md) —
   Spring Initializr, projekt anatómia, `@SpringBootApplication`, beágyazott Tomcat vs.
   `php artisan serve`, DevTools hot reload
2. [Dependency Injection](01-spring-boot-alapok/02-dependency-injection.md) —
   a Spring IoC konténer, component scanning, `@Service`/`@Repository`/`@Controller`,
   konstruktor-alapú DI a Laravel Facade-ok helyett, `@Configuration`+`@Bean` mint Service Provider
3. [REST controllerek](01-spring-boot-alapok/03-rest-controllerek.md) —
   `@RestController`, `@GetMapping`/`@PostMapping`/stb., `@PathVariable`/`@RequestParam`/`@RequestBody`,
   `ResponseEntity<T>`
4. [Konfiguráció](01-spring-boot-alapok/04-konfiguracio.md) —
   `application.yml`, profilok (`dev`/`prod`), `@Value`, típusos és validálható
   `@ConfigurationProperties`
5. [Záró gyakorlat: Todo API](01-spring-boot-alapok/05-gyakorlat-todo-api.md) —
   CRUD végpontok felállítása in-memory/H2 tárolással, konkrét kérés/válasz példákkal és
   `curl` teszteléssel — ez a projekt a további fázisokban tovább bővül
6. [Kiegészítés: Thymeleaf vs. Blade](01-spring-boot-alapok/06-sablon-renderesztes-thymeleaf.md)
   *(opcionális)* — szerver oldali renderelés, `@Controller` + nézetnév vs. `@RestController`,
   `th:text`/`th:if`/`th:each`/`th:field` fordítótábla, layout/fragmentek, CSRF form kezelés

---

[← Előző fázis: 0. Java alapok](00-java-alapok.md) · [Főoldal](../README.md) · Következő fázis: [02. Adatréteg: Spring Data JPA →](02-adatreteg-jpa.md)
