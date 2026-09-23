[← Előző: 06. Gyakorlat: Todo API bővítése](06-gyakorlat-todo-api-bovitese.md) · [Fázis index](../02-adatreteg-jpa.md) · [Főoldal](../../README.md) · Következő fázis: [03. Validáció, hibakezelés, réteges architektúra →](../03-validacio-hibakezeles.md)

# 2.7 — Kiegészítés: cache kezelés

**Ez a fejezet is opcionális.** A Laravel `Cache` facade-hoz (`Cache::remember()`, `Cache::put()`,
`Cache::forget()`) van egy közvetlen, egységes Spring megfelelő: a **Spring Cache abstraction** —
ugyanúgy egy egységes API sok különböző backend (driver) fölött, csak deklaratív, annotáció-alapú
formában, hasonlóan a [2.5 — Tranzakciókezelés](05-tranzakciok.md) `@Transactional`-jához.

## Bekapcsolás — ez explicit lépés, a Laravel-ben nincs ilyen

Laravel-ben a `Cache::` facade mindig elérhető, driver-t a `.env`/`config/cache.php` választja ki.
Spring-ben a cache mechanizmust **explicit be kell kapcsolni**:

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-cache</artifactId>
</dependency>
```

```java
@SpringBootApplication
@EnableCaching   // enélkül a @Cacheable annotációk némán hatástalanok maradnak
public class Application {
    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
}
```

Az `@EnableCaching` elfelejtése az egyik leggyakoribb kezdő hiba: nem dob hibát, egyszerűen a
`@Cacheable` metódusok minden hívásnál ténylegesen lefutnak, mintha cache ott sem lenne.

## Deklaratív cachelés: `@Cacheable`

```php
// Laravel — imperatív: te hívod meg a Cache::remember()-t
public function find(int $id): Todo
{
    return Cache::remember("todo:{$id}", 3600, fn () => Todo::findOrFail($id));
}
```

```java
// Spring — deklaratív: az annotáció köré AOP proxy épül, ami a hívás előtt megnézi a cache-t
@Service
public class TodoService {

    @Cacheable(value = "todos", key = "#id")
    public Todo findById(Long id) {
        return todoRepository.findById(id)
                .orElseThrow(() -> new TodoNotFoundException(id));
    }
}
```

A `value = "todos"` a cache neve (~ a Laravel cache kulcs-prefix elve, csak itt egy külön
névtérként konfigurálható, saját TTL-lel), a `key = "#id"` egy **SpEL kifejezés**, ami a metódus
`id` paraméteréből építi a cache kulcsot — ez a Laravel `"todo:{$id}"` string interpolációjának
felel meg, csak deklaratívan.

Fontos működésbeli csapda: mivel az `@Cacheable` egy **Spring AOP proxyn** keresztül működik
(ugyanaz a mechanizmus, mint a `@Transactional`-nál, lásd [2.5](05-tranzakciok.md)), **ugyanazon az
osztályon belüli, `this.`-en keresztüli hívás nem megy át a proxyn**, tehát nem cachelődik — csak
akkor működik, ha a metódust egy másik bean hívja meg (pl. a controller hívja a service-t).

## Invalidálás: `@CacheEvict` és `@CachePut`

```php
// Laravel
public function update(int $id, array $data): Todo
{
    $todo = Todo::findOrFail($id);
    $todo->update($data);
    Cache::forget("todo:{$id}");
    return $todo;
}

public function delete(int $id): void
{
    Todo::destroy($id);
    Cache::forget("todo:{$id}");
}
```

```java
// Spring
@CachePut(value = "todos", key = "#todo.id")   // frissíti a cache-t a friss értékkel, nem csak törli
public Todo update(Todo todo) {
    return todoRepository.save(todo);
}

@CacheEvict(value = "todos", key = "#id")      // törli a cache-ből (~ Cache::forget)
public void delete(Long id) {
    todoRepository.deleteById(id);
}
```

| Laravel `Cache` facade | Spring Cache abstraction |
|---|---|
| `Cache::remember($key, $ttl, $callback)` | `@Cacheable(value = "...", key = "...")` |
| `Cache::put($key, $value, $ttl)` | `@CachePut(value = "...", key = "...")` |
| `Cache::forget($key)` | `@CacheEvict(value = "...", key = "...")` |
| `Cache::tags(['todos'])->flush()` | nincs natív tag-támogatás — külön cache névvel vagy `allEntries = true`-val (teljes cache ürítés) kell kiváltani |
| `config/cache.php` driver (`file`, `array`, `redis`) | `CacheManager` implementáció (`ConcurrentMapCacheManager`, Caffeine, Redis) |
| `.env` `CACHE_DRIVER` | `application.yml` `spring.cache.type` |

## Backend választás: Caffeine (helyi) vs Redis (elosztott)

Alapértelmezésben (ha csak a `spring-boot-starter-cache`-t adod hozzá, külön `CacheManager` bean
nélkül) egy egyszerű, `ConcurrentHashMap`-alapú in-memory cache-t kapsz — ez a Laravel `array`
driver megfelelője: gyors, de nincs TTL, nincs kiürítési stratégia, és nem él túl egy
újraindítást. Fejlesztéshez elég, élesben nem javasolt.

**Caffeine** — egy nagy teljesítményű, helyi (JVM-en belüli) cache könyvtár, TTL-lel és
mérettel korlátozott kiürítéssel (~ a Laravel `file`/`array` drivernél praktikusabb, de még mindig
csak egyetlen instance-on belül él, nem osztott meg több worker között):

```xml
<dependency>
    <groupId>com.github.ben-manes.caffeine</groupId>
    <artifactId>caffeine</artifactId>
</dependency>
```

```yaml
spring:
  cache:
    type: caffeine
    caffeine:
      spec: maximumSize=500,expireAfterWrite=600s
```

**Redis** — ha több alkalmazás-instance között kell megosztott cache (~ Laravel `redis` driver,
horizontálisan skálázott worker mögött), ugyanaz a Redis szolgáltatás használható, amit a
[6.3 — Üzenetsorok](../06-async-esemenyek/03-uzenetsorok.md) fejezetben a RabbitMQ mellett
Docker Compose-ban már futtatsz (vagy egy külön Redis konténer ugyanazzal a mintával):

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>
```

```yaml
spring:
  cache:
    type: redis
  data:
    redis:
      host: localhost
      port: 6379
  cache.redis.time-to-live: 3600s
```

Csak a `spring.cache.type` értékét és a hozzá tartozó függőséget kell cserélni — a
`@Cacheable`/`@CachePut`/`@CacheEvict` annotációkban lévő üzleti kód **egy sort sem változik**,
amikor helyi Caffeine cache-ről elosztott Redis cache-re váltasz. Ez a Spring Cache abstraction
lényege, pontosan úgy, ahogy Laravel-ben is csak a `.env` `CACHE_DRIVER` értékét cserélnéd.

## Csapda: Entity-k cachelése és a lazy relationök

Ha közvetlenül egy `@Entity` objektumot cachelsz (Redis esetén ez szerializációt jelent), és annak
van `LAZY` kapcsolata (lásd [2.2 — Kapcsolatok](02-kapcsolatok.md)), a cache-ből visszaolvasott,
már nem Hibernate-menedzselt példányon a lazy mező elérése
`LazyInitializationException`-t dob — a Hibernate session, ami a lusta betöltést végezné, addigra
már lezárult. Ugyanez a probléma Laravel-ben is felmerül, ha egy Eloquent modellt relationökkel
együtt cachelsz, majd egy új kérésben próbálod a relationt lusta módon elérni — ott csendben új
lekérdezést indít, itt viszont kivételt dobsz. Emiatt gyakorlatban **DTO-kat cachelj Entity-k
helyett** (lásd [3.2 — DTO-k és mapping](../03-validacio-hibakezeles/02-dto-k-es-mapping.md),
később), amik már nem hordoznak lusta betöltésű proxy-referenciákat.

## Mikor tényleg kell ez neked

A Todo API záró gyakorlathoz nem szükséges — ott a hangsúly az adatrétegen van. Cache-elést akkor
érdemes bevezetni, ha profilozással (lásd [7. fázis](../07-observability.md), később) tényleges,
gyakran ismétlődő, drága lekérdezést azonosítasz — ugyanaz az elv, mint Laravel-ben: ne cachelj
"biztonságból", csak mért probléma megoldására.

---

[← Előző: 06. Gyakorlat: Todo API bővítése](06-gyakorlat-todo-api-bovitese.md) · [Fázis index](../02-adatreteg-jpa.md) · [Főoldal](../../README.md) · Következő fázis: [03. Validáció, hibakezelés, réteges architektúra →](../03-validacio-hibakezeles.md)
