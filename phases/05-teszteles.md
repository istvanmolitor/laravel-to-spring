[← Előző fázis: 04. Spring Security](04-spring-security.md) · [Főoldal](../README.md) · Következő fázis: [06. Aszinkron munka, események, háttérfeladatok →](06-async-esemenyek.md)

# 5. fázis — Tesztelés

Időtartam: ~1-2 hét (párhuzamosan is tanulható a korábbi fázisokkal)

Cél: a Pest/PHPUnit tudásod JUnit 5 + Mockito ökoszisztémára fordítása.

Ez a fázis alfejezetekre van bontva, mindegyik konkrét PHP/Laravel összehasonlításokkal és
kódpéldákkal. Haladj sorban — a záró gyakorlat a korábbi fázisokban felépített Todo API-hoz ad
unit és integrációs teszteket:

1. [JUnit 5 alapok](05-teszteles/01-junit5-alapok.md) —
   `@Test`, `@BeforeEach`/`@AfterEach`, assertion-ök, paraméterezett tesztek — a Pest/PHPUnit
   szintaxis Java megfelelője
2. [Mockito](05-teszteles/02-mockito.md) —
   `@Mock`, `@InjectMocks`, `when().thenReturn()`, `verify()`, `ArgumentCaptor` — a Laravel
   Mockery/facade mock explicit, konstruktor-injection-alapú megfelelője
3. [Spring teszt annotációk](05-teszteles/03-spring-teszt-annotaciok.md) —
   `@SpringBootTest` vs `@WebMvcTest` vs `@DataJpaTest`, `MockMvc` — ahol Laravel-ben egységesen
   a `RefreshDatabase` trait-tel dolgozol, itt a teszt "súlya" szerint választasz annotációt
4. [Testcontainers](05-teszteles/04-testcontainers.md) —
   valós PostgreSQL konténerrel futó integrációs tesztek, szemben a Laravel SQLite in-memory
   gyakorlatával
5. [Záró gyakorlat: Todo API tesztek](05-teszteles/05-gyakorlat-todo-api-tesztek.md) —
   unit tesztek a service rétegre Mockitóval, teljes HTTP folyamatot lefedő integrációs tesztek
   Testcontainers + `@SpringBootTest` kombinációval

---

[← Előző fázis: 04. Spring Security](04-spring-security.md) · [Főoldal](../README.md) · Következő fázis: [06. Aszinkron munka, események, háttérfeladatok →](06-async-esemenyek.md)
