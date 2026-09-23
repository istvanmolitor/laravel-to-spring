[← Előző: 4. Spring Security](04-spring-security.md) · [Főoldal](../README.md) · Következő: [06. Async, események, háttérfeladatok →](06-async-esemenyek.md)

# 5. fázis — Tesztelés

Időtartam: ~1-2 hét (párhuzamosan is tanulható a korábbi fázisokkal)

Cél: a Pest/PHPUnit tudásod JUnit 5 + Mockito ökoszisztémára fordítása.

- JUnit 5 alapok: `@Test`, `@BeforeEach`, `assertEquals` stb. — koncepcionálisan azonos PHPUnit-tal
- Mockito: `@Mock`, `@InjectMocks`, `when().thenReturn()` — a Laravel `Mockery`/facade mock
  megfelelője
- `@SpringBootTest` vs `@WebMvcTest` vs `@DataJpaTest` — a Spring külön annotációkkal szegmentálja
  a teszt "súlyát", ahol Laravel-ben inkább a `RefreshDatabase` trait-tel egységesen dolgozol
- Testcontainers — valós PostgreSQL/Redis konténerrel futó integrációs tesztek, ennek nincs
  elterjedt Laravel megfelelője (ott inkább SQLite in-memory-t használsz teszthez), de sokkal
  megbízhatóbb

## Gyakorlat

Írj unit teszteket a service rétegre Mockito-val, és integrációs teszteket
Testcontainers + `@SpringBootTest` kombinációval.

---

[← Előző: 4. Spring Security](04-spring-security.md) · [Főoldal](../README.md) · Következő: [06. Async, események, háttérfeladatok →](06-async-esemenyek.md)
