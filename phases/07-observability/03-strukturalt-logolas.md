[← Előző: 02. Metrikák](02-metrikak.md) · [Fázis index](../07-observability.md) · [Főoldal](../../README.md) · Következő: [04. Záró gyakorlat →](04-gyakorlat-production-readiness.md)

# 7.3 — Strukturált logolás

## Laravel `Log::` facade ↔ SLF4J + Logback

```php
// PHP — Laravel
use Illuminate\Support\Facades\Log;

Log::info('Todo létrehozva', ['todoId' => $todo->id, 'userId' => $user->id]);
Log::warning('Lassú lekérdezés', ['durationMs' => $duration]);
Log::error('Fizetés sikertelen', ['orderId' => $order->id, 'exception' => $e]);
```

```java
// Java — SLF4J API (a tényleges implementáció Logback, de a kódod csak az SLF4J
// interfészt látja — ugyanaz az absztrakciós elv, mint a Micrometer a metrikáknál)
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

public class TodoService {
    private static final Logger log = LoggerFactory.getLogger(TodoService.class);

    public Todo create(TodoRequest request) {
        Todo todo = repository.save(new Todo(request.title()));
        log.info("Todo létrehozva: todoId={}, userId={}", todo.getId(), currentUserId());
        return todo;
    }
}
```

Amit érdemes rögtön megjegyezni: a `{}` a placeholder (nem string-konkatenáció!) — ez fontos
performancia szempontból, mert ha a `DEBUG` szint ki van kapcsolva, az SLF4J **nem** értékeli ki
a paramétereket, így nem fizetsz feleslegesen a stringépítés költségéért, ellentétben egy
`Log::debug("Todo: " . $todo->toJson())` hívással, ahol a `$todo->toJson()` mindig lefut, hiába
nem kerül sehova a kimenet.

## Log szintek

| Szint | Mikor használd | Laravel megfelelő |
|---|---|---|
| `TRACE` | rendkívül részletes, csak mély debughoz | ritkán használt PHP-ban is |
| `DEBUG` | fejlesztői részletek, élesben tipikusan kikapcsolva | `Log::debug()` |
| `INFO` | normál üzemi események (kérés feldolgozva, entitás létrehozva) | `Log::info()` |
| `WARN` | valami szokatlan, de nem hibás állapot | `Log::warning()` |
| `ERROR` | valódi hiba, ami beavatkozást igényelhet | `Log::error()` |

## Konfiguráció: `logback-spring.xml`

A Laravel `config/logging.php`-hoz hasonlóan itt is channel-szerű, profil-függő konfigurációt
építhetsz — csak XML-ben, nem PHP tömbben:

```xml
<!-- src/main/resources/logback-spring.xml -->
<configuration>
    <springProfile name="dev">
        <appender name="CONSOLE" class="ch.qos.logback.core.ConsoleAppender">
            <encoder>
                <pattern>%d{HH:mm:ss} %-5level %logger{36} - %msg%n</pattern>
            </encoder>
        </appender>
        <root level="INFO">
            <appender-ref ref="CONSOLE" />
        </root>
    </springProfile>

    <springProfile name="prod">
        <appender name="JSON" class="ch.qos.logback.core.ConsoleAppender">
            <encoder class="net.logstash.logback.encoder.LogstashEncoder" />
        </appender>
        <root level="INFO">
            <appender-ref ref="JSON" />
        </root>
    </springProfile>
</configuration>
```

Ez a `<springProfile>` blokk-pár azt jelenti: fejlesztésben ember által olvasható, színes,
egysoros logformátumot kapsz (mint a Laravel `single`/`daily` channel alapértelmezett kimenete),
élesben pedig JSON-t (`net.logstash.logback.encoder.LogstashEncoder`, külön dependencia kell
hozzá: `logstash-logback-encoder`).

## Miért éri meg a JSON formátum élesben?

Ha a logjaidat egy aggregáló rendszer (ELK stack — Elasticsearch/Logstash/Kibana, vagy Grafana
Loki) gyűjti össze, a JSON formátum azt jelenti, hogy minden mező (`level`, `logger`, `message`,
és a te egyedi kulcs-érték párjaid, pl. `todoId`, `userId`) **külön kereshető/szűrhető mezőként**
landol, nem csak egy nagy szövegblokként, amiben regex-eznél. Ez ugyanaz a motiváció, ami miatt a
Laravel `Log::info('msg', ['context' => 'array'])` context tömbjét is strukturáltan tárolod, nem
csak belesztringezed az üzenetbe — csak Spring-ben ez a strukturáltság konzisztensen végigvitt
konvenció egészen a log-aggregátorig.

---

[← Előző: 02. Metrikák](02-metrikak.md) · [Fázis index](../07-observability.md) · [Főoldal](../../README.md) · Következő: [04. Záró gyakorlat →](04-gyakorlat-production-readiness.md)
