[← Előző: 02. Mockito](02-mockito.md) · [Fázis index](../05-teszteles.md) · [Főoldal](../../README.md) · Következő: [04. Testcontainers →](04-testcontainers.md)

# 5.3 — Spring teszt annotációk: `@SpringBootTest`, `@WebMvcTest`, `@DataJpaTest`

Laravel-ben tesztíráskor tipikusan egyféle "súlyt" használsz: a `RefreshDatabase`/
`DatabaseTransactions` trait-tel felszerelt `TestCase`-t, és onnantól minden teszt (legyen az
egy modell logikáját vagy egy teljes HTTP végpontot ellenőrző teszt) ugyanabban a keretben fut,
ugyanolyan "súlyú" bootstrap-pal. Spring-ben ez tudatosan szegmentált: külön annotációk indítanak
kisebb vagy nagyobb alkalmazáskontextust, attól függően, mit tesztelsz — ez gyorsabb tesztfutást
tesz lehetővé, mert nem kell mindenhez a teljes alkalmazást felhúzni.

## `@SpringBootTest` — a teljes alkalmazáskontextus

```java
@SpringBootTest
class TodoApplicationIntegrationTest {
    @Autowired
    private TodoService todoService;

    @Test
    void applicationContextLoads() {
        assertNotNull(todoService);
    }
}
```

Ez felhúzza a **teljes** Spring alkalmazáskontextust (minden bean, minden konfiguráció) —
leginkább a Laravel teljes `TestCase`-hez hasonlít, ahol a teljes alkalmazás fut a teszt alatt.
Lassabb, mint a lentebbi szűkített változatok, ezért csak ott használd, ahol tényleg a teljes
integrációra van szükség (pl. egy teljes HTTP kérés-válasz folyamat végigtesztelésére).

## `@WebMvcTest` — csak a web réteg

```java
@WebMvcTest(TodoController.class)
class TodoControllerTest {

    @Autowired
    private MockMvc mockMvc;

    @MockBean
    private TodoService todoService;

    @Test
    void returnsTodoList() throws Exception {
        when(todoService.findAll()).thenReturn(List.of(new Todo(1L, "Bevásárlás", false)));

        mockMvc.perform(get("/api/todos"))
            .andExpect(status().isOk())
            .andExpect(jsonPath("$[0].title").value("Bevásárlás"));
    }
}
```

Csak a controller réteget és a hozzá kapcsolódó web infrastruktúrát (routing, JSON
szerializáció, validáció) tölti be — a service réteg `@MockBean`-ként mockolva van. Ez a Laravel
`$this->getJson('/api/todos')->assertOk()->assertJsonPath('0.title', 'Bevásárlás')` mintájának
felel meg, csak itt explicit ki kell mockolnod a mögöttes service-t, mert nem a teljes
alkalmazás fut.

## `@DataJpaTest` — csak a JPA/adatréteg

```java
@DataJpaTest
class TodoRepositoryTest {

    @Autowired
    private TodoRepository repository;

    @Test
    void findsTodosByUserId() {
        repository.save(new Todo(null, "Bevásárlás", false, 1L));

        List<Todo> todos = repository.findByUserId(1L);

        assertEquals(1, todos.size());
    }
}
```

Csak a JPA réteget és egy beágyazott H2 adatbázist tölt be — gyors, mert nincs web réteg vagy
security konfiguráció betöltve. Minden teszt metódus **automatikusan tranzakcióban fut, és a
végén rollback-elődik** — ez pontosan az, amit a Laravel `RefreshDatabase`/`DatabaseTransactions`
trait ad neked, csak itt ez az annotáció alapértelmezett viselkedése, nem kell külön bekapcsolni.

## Táblázatos összefoglaló

| Annotáció | Mit tölt be | Sebesség | Laravel megfelelő gondolat |
|---|---|---|---|
| `@SpringBootTest` | teljes alkalmazáskontextus | lassú | teljes `TestCase` HTTP teszttel |
| `@WebMvcTest` | csak web réteg (controller, JSON, validáció) | közepes | HTTP teszt mockolt service-ekkel |
| `@DataJpaTest` | csak JPA réteg + beágyazott H2 | gyors | modell/repository teszt `RefreshDatabase`-zel |
| sima unit teszt + Mockito | semmi Spring context, csak POJO-k | leggyorsabb | tiszta unit teszt Mockery-vel |

## `MockMvc` — HTTP szintű asszertek

```java
mockMvc.perform(post("/api/todos")
        .contentType(MediaType.APPLICATION_JSON)
        .content("""
            {"title": "Bevásárlás", "description": "Tej, kenyér"}
            """))
    .andExpect(status().isCreated())
    .andExpect(jsonPath("$.title").value("Bevásárlás"));
```

Ez a `$this->postJson('/api/todos', [...])->assertCreated()->assertJsonPath('title',
'Bevásárlás')` közvetlen megfelelője — a `jsonPath(...)` a Laravel `assertJsonPath`
funkcióját adja, JSONPath szintaxissal.

---

[← Előző: 02. Mockito](02-mockito.md) · [Fázis index](../05-teszteles.md) · [Főoldal](../../README.md) · Következő: [04. Testcontainers →](04-testcontainers.md)
