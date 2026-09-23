[← Előző: 03. Spring teszt annotációk](03-spring-teszt-annotaciok.md) · [Fázis index](../05-teszteles.md) · [Főoldal](../../README.md) · Következő: [05. Záró gyakorlat: Todo API tesztek →](05-gyakorlat-todo-api-tesztek.md)

# 5.4 — Testcontainers

## A probléma, amit megold

Laravel projektekben tesztíráskor gyakran SQLite in-memory adatbázist használsz (`:memory:`),
mert gyors, és nem kell külön adatbázis szervert indítani a CI-hoz. Ennek ára van: az SQLite
**nem ugyanaz**, mint az éles PostgreSQL/MySQL — eltér a dátum/idő kezelés, a case-sensitivity
string összehasonlításnál, bizonyos SQL függvények (pl. `JSON_EXTRACT` variánsok), a tranzakciós
és lock viselkedés. Előfordulhat, hogy egy teszt zölden fut SQLite-tal, de éles PostgreSQL-en
másképp viselkedik — ez a fajta "teszt/éles divergencia" pont az a probléma, amit ez a fejezet
old meg.

## A megoldás: valódi adatbázis Docker konténerben, a teszthez

A **Testcontainers** könyvtár lehetővé teszi, hogy a tesztfutás elindítson egy valódi, izolált
PostgreSQL (vagy bármilyen más) Docker konténert, ami pontosan ugyanaz, mint amit élesben
használsz — a teszt ez ellen fut, majd a konténer automatikusan megszűnik a tesztfutás végén.
Ára: lassabb tesztfutás (Docker konténer indítása időbe kerül), cserébe garantáltan valós
adatbázis-viselkedést tesztelsz.

```xml
<!-- pom.xml -->
<dependency>
    <groupId>org.testcontainers</groupId>
    <artifactId>postgresql</artifactId>
    <version>1.19.7</version>
    <scope>test</scope>
</dependency>
<dependency>
    <groupId>org.testcontainers</groupId>
    <artifactId>junit-jupiter</artifactId>
    <version>1.19.7</version>
    <scope>test</scope>
</dependency>
```

```java
@Testcontainers
@SpringBootTest
class TodoRepositoryIntegrationTest {

    @Container
    static PostgreSQLContainer<?> postgres = new PostgreSQLContainer<>("postgres:16-alpine");

    @DynamicPropertySource
    static void configureDatasource(DynamicPropertyRegistry registry) {
        registry.add("spring.datasource.url", postgres::getJdbcUrl);
        registry.add("spring.datasource.username", postgres::getUsername);
        registry.add("spring.datasource.password", postgres::getPassword);
    }

    @Autowired
    private TodoRepository repository;

    @Test
    void savesAndRetrievesTodo() {
        Todo saved = repository.save(new Todo(null, "Bevásárlás", false, 1L));

        Optional<Todo> found = repository.findById(saved.getId());

        assertTrue(found.isPresent());
    }
}
```

Amit érdemes megérteni a fenti kódból:
- `@Container` egy statikus mezőn jelzi, hogy ezt a konténert kezelje a Testcontainers életciklusa
  (indítás a tesztosztály előtt, leállítás utána)
- `@DynamicPropertySource` futásidőben, a konténer tényleges (véletlenszerűen kiosztott) portja
  alapján állítja be a Spring datasource konfigurációját — nem tudod előre, milyen porton fog
  elindulni a konténer, ezért nem lehet ezt statikus `application.yml` értékként megadni

## Előfeltétel: Docker

A Testcontainers a helyi (vagy CI-beli) Docker daemon-t használja a konténerek indításához —
ugyanaz a Docker, amit valószínűleg már használsz a Laravel projektjeidhez `docker-compose.yml`
formájában (pl. Laravel Sail). Nincs extra infrastruktúra-igény, csak fusson a Docker.

## Miért éri meg a lassabb tesztfutás

- A repository/adatréteg tesztjeid **valódi SQL-t** futtatnak valódi PostgreSQL-en — ha egy
  `@Query`-ben natív SQL-t írtál, ami PostgreSQL-specifikus szintaxist használ, azt egy H2 vagy
  SQLite teszt nem venné észre, ha hibás
- CI-ban is ugyanígy működik (a legtöbb CI szolgáltatás — GitHub Actions, GitLab CI — támogatja
  Docker-in-Docker futtatást), így a build pipeline is valós adatbázis ellen tesztel
- Nem kell külön "teszt adatbázis konfigurációt" karbantartanod, ami eltér az éles
  konfigurációtól — a Testcontainers minden tesztfutáshoz friss, tiszta konténert ad

## Mikor NE használj Testcontainers-t

A `@DataJpaTest` beágyazott H2-vel ([5.3 fejezet](03-spring-teszt-annotaciok.md)) gyorsabb, és jó
választás, amikor csak azt akarod ellenőrizni, hogy a repository metódusaid szintaktikailag
helyesek és a mapping működik — nem minden tesztnek kell Testcontainers-t használnia. A
Testcontainers ott ér igazán sokat, ahol PostgreSQL-specifikus viselkedésre (dátumkezelés,
bizonyos constraint-ek, natív SQL) van szükséged, vagy a teljes integrációs tesztsorozatban,
ahol amúgy is a valós működést akarod igazolni.

---

[← Előző: 03. Spring teszt annotációk](03-spring-teszt-annotaciok.md) · [Fázis index](../05-teszteles.md) · [Főoldal](../../README.md) · Következő: [05. Záró gyakorlat: Todo API tesztek →](05-gyakorlat-todo-api-tesztek.md)
