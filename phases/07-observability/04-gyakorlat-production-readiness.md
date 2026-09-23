[← Előző: 03. Strukturált logolás](03-strukturalt-logolas.md) · [Fázis index](../07-observability.md) · [Főoldal](../../README.md) · Következő fázis: [08. Építs egy komplett projektet →](../08-projekt.md)

# 7.4 — Záró gyakorlat: production-readiness a Todo API-n

Ez a gyakorlat a korábbi fázisokban felépített Todo API-t egészíti ki azzal a minimális
megfigyelhetőségi réteggel, amit egy valódi, éles Spring Boot szolgáltatástól elvársz — pontosan
úgy, ahogy egy Laravel projektet sem engednél élesre Telescope/Horizon-szerű betekintés és
strukturált logolás nélkül.

## 1. lépés — Actuator bekötése

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
<dependency>
    <groupId>io.micrometer</groupId>
    <artifactId>micrometer-registry-prometheus</artifactId>
</dependency>
```

```yaml
# application.yml
management:
  endpoints:
    web:
      exposure:
        include: health, prometheus
  endpoint:
    health:
      show-details: when-authorized
```

Ellenőrizd: `curl http://localhost:8080/actuator/health` → `{"status":"UP"}`, és
`curl http://localhost:8080/actuator/prometheus` → Prometheus-formátumú metrikaszöveg.

## 2. lépés — Egyedi metrika a todo-létrehozáshoz

Egészítsd ki a `TodoService`-t (lásd [0.2 Metrikák](02-metrikak.md)) egy `Counter`-rel, ami minden
sikeres `create()` hívásnál növekszik:

```java
@Service
public class TodoService {

    private final TodoRepository repository;
    private final Counter todosCreatedCounter;

    public TodoService(TodoRepository repository, MeterRegistry registry) {
        this.repository = repository;
        this.todosCreatedCounter = Counter.builder("todos.created")
            .description("Létrehozott todo-k száma")
            .register(registry);
    }

    public Todo create(TodoRequest request) {
        Todo todo = repository.save(new Todo(request.title(), request.description()));
        todosCreatedCounter.increment();
        return todo;
    }
}
```

Hozz létre néhány todo-t az API-n keresztül, majd nézd meg, hogy a `todos_created_total` metrika
megjelenik-e a `/actuator/prometheus` kimenetben, a számláló pedig nő.

## 3. lépés — Profil-függő logolás beállítása

Hozz létre egy `src/main/resources/logback-spring.xml`-t (lásd [0.3 Strukturált
logolás](03-strukturalt-logolas.md)), ami:
- `dev` profilban ember által olvasható, egysoros konzol formátumot ír
- `prod` profilban JSON formátumot ír (`logstash-logback-encoder` dependenciával)

Indítsd el az alkalmazást `dev`, majd `prod` profillal (`spring.profiles.active=prod`), és
hasonlítsd össze a konzolkimenetet.

## 4. lépés — `System.out.println` lecserélése

Ha a korábbi fázisokban (pl. a 6. fázis async email-gyakorlatában) maradt olyan hely, ahol
`System.out.println`-nel "szimuláltál" egy műveletet (pl. email küldést), cseréld le SLF4J logger
hívásra:

```java
// előtte
System.out.println("Email elküldve: " + user.getEmail());

// utána
private static final Logger log = LoggerFactory.getLogger(EmailService.class);
// ...
log.info("Üdvözlő email elküldve: userId={}, email={}", user.getId(), user.getEmail());
```

Ez nem csak stilisztikai csere — a `System.out.println` kimenete nem megy át a Logback
formázón/appendereken, nem kerül JSON-ba élesben, és nem szűrhető log szint szerint. Egy
`println` egy Spring Boot alkalmazásban nagyjából annak felel meg, mintha Laravel-ben `echo`-val
írnál ki debug infót a `Log::` facade helyett — működik, de kikerüli az egész logolási
infrastruktúrát.

## Ellenőrző kérdések a fázis lezárásához

1. Miért nem exponálod alapból az `/actuator/env` végpontot élesen?
2. Mi a gyakorlati különbség a Micrometer/Prometheus és a Laravel Telescope célja között?
3. Miért placeholder (`{}`) alapú az SLF4J log hívás string-konkatenáció helyett?

---

Ezzel lezárult a Spring-specifikus tanulási fázisok sora. A [8. fázisban](../08-projekt.md) ezt a
teljes eszköztárat (Java alapok, Spring Boot, JPA, validáció/hibakezelés, Security, tesztelés,
async/queue, observability) egy önálló, valódi Laravel-projekt átportolására alkalmazod végig.

---

[← Előző: 03. Strukturált logolás](03-strukturalt-logolas.md) · [Fázis index](../07-observability.md) · [Főoldal](../../README.md) · Következő fázis: [08. Építs egy komplett projektet →](../08-projekt.md)
