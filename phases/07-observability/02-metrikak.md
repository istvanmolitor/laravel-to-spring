[← Előző: 01. Actuator](01-actuator.md) · [Fázis index](../07-observability.md) · [Főoldal](../../README.md) · Következő: [03. Strukturált logolás →](03-strukturalt-logolas.md)

# 7.2 — Metrikák: Micrometer + Prometheus/Grafana

A Laravel ökoszisztémában nincs beépített, keretrendszer-szintű metrika-gyűjtő megoldás — ha
számszerű üzemeltetési adatokat (válaszidő, hibaarány, üzleti metrikák) akarsz gyűjteni és
vizualizálni, tipikusan külső APM eszközt vezetsz be (New Relic, Datadog, Blackfire), vagy kézzel
építesz valamit Redis/InfluxDB fölé. A Spring ökoszisztémában ez **beépített, elsőosztályú
funkció** — ez az egyik legnagyobb, kézzelfogható különbség a két világ "production-readiness"
elvárásai között.

## Micrometer — a mérőszám-absztrakció

A **Micrometer** olyan, mint egy "SLF4J a metrikáknak": egységes API-t ad, a háttérben pedig
tetszőleges monitoring backend csatlakoztatható (Prometheus, Datadog, CloudWatch, Graphite) anélkül,
hogy az alkalmazáskódot módosítanád. Spring Boot Actuator automatikusan regisztrál Micrometer
metrikákat minden HTTP kérésről, JVM memóriáról, adatbázis connection poolról.

```xml
<dependency>
    <groupId>io.micrometer</groupId>
    <artifactId>micrometer-registry-prometheus</artifactId>
</dependency>
```

```yaml
management:
  endpoints:
    web:
      exposure:
        include: health, prometheus
```

Ezután a `/actuator/prometheus` végpont Prometheus-formátumú szöveges metrikákat ad vissza —
ezt a Prometheus szerver rendszeresen (pl. 15 másodpercenként) lekérdezi ("scrape"), és eltárolja
idősorként.

## Egyedi metrika létrehozása

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

Egy `Timer`-rel a metódus futási idejét is mérheted:

```java
@Timed(value = "todo.service.create", description = "Todo létrehozás időtartama")
public Todo create(TodoRequest request) { ... }
```

Az `@Timed` annotációhoz szükség van a `spring-boot-starter-aop` dependenciára is, mert AOP proxy-n
keresztül méri az időt — ez ugyanaz a proxy-alapú mechanizmus, mint amit a `@Transactional`
esetében is látsz.

## Grafana dashboard — mit érdemes vizualizálni

A Prometheus önmagában csak eltárolja az idősorokat, a vizualizációt tipikusan **Grafana**
végzi, ami dashboardokat épít a Prometheus adatforrásból. Egy Todo API-hoz alap dashboard
panelek:

- HTTP kérés/válaszidő eloszlása végponttonként (Actuator automatikusan gyűjti)
- HTTP hibaarány (4xx/5xx státuszkódok aránya)
- `todos.created` — üzleti metrika, hány todo készült óránként
- JVM heap használat, garbage collection szünetek

Ez a fajta dashboard-építés koncepcionálisan annak felel meg, amit egy Laravel projektben
Grafana + egyedi Redis/InfluxDB metrika-küldés kombinációjával tudnál összerakni — csak itt a
keretrendszer maga már "kábelezve" van a Prometheus formátumhoz, nem kell a metrika-küldést
saját kódból megoldanod minden egyes mérőszámhoz.

---

[← Előző: 01. Actuator](01-actuator.md) · [Fázis index](../07-observability.md) · [Főoldal](../../README.md) · Következő: [03. Strukturált logolás →](03-strukturalt-logolas.md)
