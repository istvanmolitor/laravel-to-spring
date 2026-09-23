[← Előző: 03. REST controllerek](03-rest-controllerek.md) · [Fázis index](../01-spring-boot-alapok.md) · [Főoldal](../../README.md) · Következő: [05. Gyakorlat: Todo API →](05-gyakorlat-todo-api.md)

# 1.4 — Konfiguráció

## `.env` + `config/*.php` → `application.yml`

Laravel-ben két réteg van: a `.env` (titkok, környezetfüggő értékek) és a `config/*.php`
(strukturált, típusos elérésre előkészített konfiguráció, ami a `.env` értékeit olvassa be
`env()`-fel). Spring Boot-ban ez a két réteg egyetlen fájlba egyszerűsödik:
**`application.properties`** vagy **`application.yml`**.

```properties
# application.properties
spring.datasource.url=jdbc:h2:mem:tododb
spring.datasource.username=sa
app.name=Todo API
app.max-todos-per-user=100
```

```yaml
# application.yml — ugyanaz YAML formátumban, beágyazott struktúrával (ezt ajánlott használni)
spring:
  datasource:
    url: jdbc:h2:mem:tododb
    username: sa

app:
  name: Todo API
  max-todos-per-user: 100
```

A YAML változat olvashatóbb beágyazott kulcsoknál (ahogy a Laravel `config/app.php` tömbjei is
beágyazottak) — a legtöbb Spring projekt YAML-t használ, ezt ajánlott te is választani.

| Laravel | Spring Boot |
|---|---|
| `.env` | `application.yml` (fejlesztői/alap értékek) |
| `config/app.php`, `config/database.php` stb. | ugyanaz az `application.yml`, névtér-szerű kulcsokkal (`app.*`, `spring.datasource.*`) |
| `env('APP_NAME')` | környezeti változó felülírhatja: `APP_NAME` env var ↔ `app.name` property, Spring automatikusan összeköti |
| `config('app.name')` | `@Value("${app.name}")` vagy `@ConfigurationProperties` |

## Profilok — `dev`/`staging`/`prod`

Laravel-ben az `.env` fájl maga változik környezetenként (`.env.production` stb.), és az
`APP_ENV` határozza meg, melyik konfigurációs ág aktív. Spring Boot-ban ezt **profilok**
kezelik: külön `application-{profil}.yml` fájlokat írsz, amik **felülírják** az alap
`application.yml` értékeit:

```
src/main/resources/
├── application.yml              # közös, minden környezetben érvényes alapértékek
├── application-dev.yml          # fejlesztői felülírások (pl. H2, verbose logging)
└── application-prod.yml         # éles felülírások (pl. PostgreSQL, kevesebb log)
```

```yaml
# application-dev.yml
spring:
  datasource:
    url: jdbc:h2:mem:tododb
logging:
  level:
    root: DEBUG
```

```yaml
# application-prod.yml
spring:
  datasource:
    url: jdbc:postgresql://prod-db:5432/tododb
logging:
  level:
    root: WARN
```

Aktiválás:

```bash
./mvnw spring-boot:run -Dspring-boot.run.profiles=dev

# vagy környezeti változóval, ami production-ben a szokásos mód:
export SPRING_PROFILES_ACTIVE=prod
java -jar app.jar
```

## `@Value` — egyszerű, egyedi érték beolvasása

```java
@Service
public class TodoService {

    @Value("${app.max-todos-per-user}")
    private int maxTodosPerUser;

    // ...
}
```

Ez a leggyorsabb módja egyetlen konfig érték beolvasásának — nagyjából a `config('app.max_todos')`
egyszeri hívásának felel meg. Kis projektekben elfogadható, de sok `@Value` mező szanaszét egy
kódbázisban ugyanazt a problémát okozza, mint a `config()` hívások szétszórása Laravel-ben:
nehéz átlátni, milyen konfigurációtól függ az alkalmazás.

## `@ConfigurationProperties` — típusos konfig osztály

Ez a Spring saját megoldása arra, amire Laravel-ben nincs közvetlen, beépített párhuzam — egy
**típusos, validálható** konfigurációs osztály, ami egy `application.yml` névtér-ágát egy Java
objektumra képezi le:

```yaml
app:
  name: Todo API
  max-todos-per-user: 100
  admin-email: admin@example.com
```

```java
@ConfigurationProperties(prefix = "app")
@Validated
public class AppProperties {

    @NotBlank
    private String name;

    @Min(1)
    private int maxTodosPerUser;

    @Email
    private String adminEmail;

    // getterek/setterek (vagy Lombok @Data)
}
```

```java
@Configuration
@EnableConfigurationProperties(AppProperties.class)
public class AppConfig {
}
```

```java
@Service
@RequiredArgsConstructor
public class TodoService {
    private final AppProperties appProperties;   // konstruktoron keresztül injektálva, mint bármi más bean

    public void checkLimit(User user) {
        if (user.getTodoCount() >= appProperties.getMaxTodosPerUser()) {
            throw new TooManyTodosException();
        }
    }
}
```

Amit ez ad, és aminek nincs jó Laravel megfelelője: a `@Validated` annotációval a konfigurációs
értékek **induláskor validálódnak** (`@NotBlank`, `@Min`, `@Email` stb.) — ha valaki elront egy
`application.yml` értéket (pl. üresen hagyja az `admin-email`-t), az alkalmazás **el sem indul**,
világos hibaüzenettel. Laravel-ben egy hibás `.env` érték tipikusan csak akkor bukik ki, amikor
az adott kódág ténylegesen lefut — akár élesben, futásidőben.

Ez a minta ismerős lesz a [0.4 Generics](../00-java-alapok/04-generics.md) és
[0.3 OOP alapok](../00-java-alapok/03-oop-alapok.md) fejezetekből: egy sima Java osztály,
mezőkkel, gettert/settert generálva (Lombok `@Data`-val, lásd
[1.1](01-projekt-inditasa.md)), amit a Spring konténer tölt fel és validál neked induláskor.

---

[← Előző: 03. REST controllerek](03-rest-controllerek.md) · [Fázis index](../01-spring-boot-alapok.md) · [Főoldal](../../README.md) · Következő: [05. Gyakorlat: Todo API →](05-gyakorlat-todo-api.md)
