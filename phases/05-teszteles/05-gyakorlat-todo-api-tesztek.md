[← Előző: 04. Testcontainers](04-testcontainers.md) · [Fázis index](../05-teszteles.md) · [Főoldal](../../README.md) · Következő fázis: [06. Aszinkron munka, események, háttérfeladatok →](../06-async-esemenyek.md)

# 5.5 — Záró gyakorlat: tesztek a Todo API-hoz

A korábbi fázisokban felépítetted a Todo API-t (Todo és User entitás, PostgreSQL, Flyway,
réteges architektúra Controller/Service/Repository/DTO-kkal, JWT autentikáció). Ez a gyakorlat
két szinten fedi le teszttel: **unit tesztek** a service rétegre Mockitóval (gyors,
elszigetelt), és **integrációs tesztek** a teljes HTTP folyamatra Testcontainers +
`@SpringBootTest` kombinációval (lassabb, de valós végponttól végpontig futó ellenőrzés).

## 1. rész — `TodoServiceTest` (unit teszt, Mockito)

Feltételezett `TodoService` konstruktor: `TodoService(TodoRepository repository)`. Írj legalább
három tesztesetet:

### a) Sikeres létrehozás

```java
@ExtendWith(MockitoExtension.class)
class TodoServiceTest {

    @Mock
    private TodoRepository repository;

    @InjectMocks
    private TodoService service;

    @Test
    void createsNewTodoForCurrentUser() {
        Long userId = 1L;
        when(repository.save(any(Todo.class)))
            .thenAnswer(invocation -> invocation.getArgument(0));

        Todo created = service.create(userId, "Bevásárlás", "Tej, kenyér");

        assertEquals("Bevásárlás", created.getTitle());
        assertEquals(userId, created.getUserId());
        verify(repository).save(any(Todo.class));
    }
```

### b) Nem található todo — kivétel

```java
    @Test
    void throwsWhenTodoNotFound() {
        when(repository.findById(99L)).thenReturn(Optional.empty());

        assertThrows(TodoNotFoundException.class, () -> service.getByIdForUser(99L, 1L));
    }
```

### c) Jogosulatlan hozzáférés más user todo-jához

```java
    @Test
    void throwsWhenAccessingAnotherUsersTodo() {
        Todo othersTodo = new Todo(1L, "Más felhasználó todo-ja", false, 2L);
        when(repository.findById(1L)).thenReturn(Optional.of(othersTodo));

        assertThrows(AccessDeniedException.class, () -> service.getByIdForUser(1L, /* current user */ 1L));
    }
}
```

Ez a harmadik teszteset direktben a [4. fázisban](../04-spring-security/04-jogosultsagkezeles.md)
bevezetett tulajdonos-alapú jogosultságellenőrzést fedi le — a `TodoService`-nek dobnia kell egy
jogosultsági hibát, ha a bejelentkezett user nem a todo tulajdonosa.

## 2. rész — `TodoControllerIntegrationTest` (integrációs teszt, Testcontainers + MockMvc)

```java
@Testcontainers
@SpringBootTest
@AutoConfigureMockMvc
class TodoControllerIntegrationTest {

    @Container
    static PostgreSQLContainer<?> postgres = new PostgreSQLContainer<>("postgres:16-alpine");

    @DynamicPropertySource
    static void configureDatasource(DynamicPropertyRegistry registry) {
        registry.add("spring.datasource.url", postgres::getJdbcUrl);
        registry.add("spring.datasource.username", postgres::getUsername);
        registry.add("spring.datasource.password", postgres::getPassword);
    }

    @Autowired
    private MockMvc mockMvc;

    private String jwtToken;

    @BeforeEach
    void registerAndLogin() throws Exception {
        mockMvc.perform(post("/api/auth/register")
                .contentType(MediaType.APPLICATION_JSON)
                .content("""
                    {"email": "kovacs@example.com", "password": "titkos123"}
                    """))
            .andExpect(status().isCreated());

        MvcResult loginResult = mockMvc.perform(post("/api/auth/login")
                .contentType(MediaType.APPLICATION_JSON)
                .content("""
                    {"email": "kovacs@example.com", "password": "titkos123"}
                    """))
            .andExpect(status().isOk())
            .andReturn();

        jwtToken = JsonPath.read(loginResult.getResponse().getContentAsString(), "$.token");
    }

    @Test
    void createsAndRetrievesTodoThroughFullHttpFlow() throws Exception {
        mockMvc.perform(post("/api/todos")
                .header("Authorization", "Bearer " + jwtToken)
                .contentType(MediaType.APPLICATION_JSON)
                .content("""
                    {"title": "Bevásárlás", "description": "Tej, kenyér"}
                    """))
            .andExpect(status().isCreated())
            .andExpect(jsonPath("$.title").value("Bevásárlás"));

        mockMvc.perform(get("/api/todos")
                .header("Authorization", "Bearer " + jwtToken))
            .andExpect(status().isOk())
            .andExpect(jsonPath("$[0].title").value("Bevásárlás"));
    }

    @Test
    void rejectsUnauthenticatedRequest() throws Exception {
        mockMvc.perform(get("/api/todos"))
            .andExpect(status().isUnauthorized());
    }
}
```

Ez a teszt végigmegy egy teljes, valós folyamaton: regisztráció → bejelentkezés → JWT megszerzése
→ autentikált CRUD híváskezdeményezés → válasz ellenőrzése, valódi PostgreSQL adatbázis ellen.
Ez a Laravel `Feature` teszt (a `Unit` teszttel szemben) közvetlen megfelelője — ott is a teljes
HTTP réteget és az autentikációt végigfuttatnád egy `$this->actingAs($user)->postJson(...)`
hívással, itt a JWT tokent explicit meg kell szerezni és minden kéréshez csatolni.

## Futtatás

```bash
mvn test
```

Mivel a `TodoServiceTest` sima Mockito unit teszt (nincs Spring context, nincs Testcontainers),
ezredmásodpercek alatt lefut. A `TodoControllerIntegrationTest` Docker konténert indít és a
teljes Spring kontextust felhúzza — ez másodperceket vesz igénybe. Jó gyakorlat, ha a CI
pipeline-ban külön futtatod a kettőt (gyors unit tesztek minden commit-nál, lassabb integrációs
tesztek PR-onként vagy merge előtt) — ez ugyanaz a megfontolás, mint amikor Laravel-ben a `Unit`
és `Feature` teszt suite-okat külön csoportba szervezed.

---

A projekt a [6. fázisban](../06-async-esemenyek.md) async eseménykezelést kap: amikor egy
felhasználó regisztrál, egy háttérfolyamat (kezdetben egyszerű `@Async` metódus, majd
RabbitMQ-alapú üzenetsor) fog "üdvözlő emailt" küldeni — ezen keresztül ismered meg a Spring
esemény- és queue-rendszerét.

---

[← Előző: 04. Testcontainers](04-testcontainers.md) · [Fázis index](../05-teszteles.md) · [Főoldal](../../README.md) · Következő fázis: [06. Aszinkron munka, események, háttérfeladatok →](../06-async-esemenyek.md)
