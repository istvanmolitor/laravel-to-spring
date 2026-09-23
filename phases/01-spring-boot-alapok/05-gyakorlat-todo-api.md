[← Előző: 04. Konfiguráció](04-konfiguracio.md) · [Fázis index](../01-spring-boot-alapok.md) · [Főoldal](../../README.md) · Következő: [06. Kiegészítés: Thymeleaf →](06-sablon-renderesztes-thymeleaf.md)

# 1.5 — Záró gyakorlat: Todo API

Ez a gyakorlat egyetlen kis, de teljes REST API-ban gyakoroltatja be az 1. fázis minden elemét:
projekt-indítás, DI, controllerek, konfiguráció. **Ez a projekt a roadmap további fázisaiban
folyamatosan bővül** — érdemes verziókezelőbe (git) tenni már most.

## Entitás: `Todo`

| Mező | Típus | Megjegyzés |
|---|---|---|
| `id` | `Long` | auto-generált azonosító |
| `title` | `String` | kötelező, max 255 karakter |
| `description` | `String` | opcionális, hosszabb szöveg |
| `done` | `boolean` | alapértelmezetten `false` |
| `createdAt` | `LocalDateTime` | automatikusan beállítva létrehozáskor |

Ebben a fázisban egyelőre elég egy egyszerű, `@Entity` nélküli sima Java osztály (POJO) + egy
in-memory `Map<Long, Todo>` tárolás a service rétegben — a valódi JPA/H2 bekötés a
[2. fázis: Adatréteg](../02-adatreteg-jpa.md) témája. Ha korábban akarod kipróbálni a H2-t, az
alábbi `application.yml` résszel megteheted, de az entitás-annotálást (`@Entity`, `@Id` stb.)
hagyd a következő fázisra.

```java
public class Todo {
    private Long id;
    private String title;
    private String description;
    private boolean done;
    private LocalDateTime createdAt;

    // konstruktor, getterek/setterek (vagy Lombok @Data)
}
```

## Végpontok

| Metódus | Útvonal | Leírás |
|---|---|---|
| `GET` | `/api/todos` | az összes todo listája, opcionális `?done=true/false` szűréssel |
| `GET` | `/api/todos/{id}` | egy adott todo, `404` ha nincs ilyen `id` |
| `POST` | `/api/todos` | új todo létrehozása, `201 Created` |
| `PUT` | `/api/todos/{id}` | meglévő todo módosítása, `200 OK` vagy `404` |
| `DELETE` | `/api/todos/{id}` | todo törlése, `204 No Content` |

## Kérés/válasz példák

**`POST /api/todos`** — kérés törzse:

```json
{
  "title": "Spring Boot megtanulása",
  "description": "Fejezd be az 1. fázist a roadmap-ből"
}
```

Válasz (`201 Created`):

```json
{
  "id": 1,
  "title": "Spring Boot megtanulása",
  "description": "Fejezd be az 1. fázist a roadmap-ből",
  "done": false,
  "createdAt": "2026-09-23T14:30:00"
}
```

**`GET /api/todos`** — válasz (`200 OK`):

```json
[
  {
    "id": 1,
    "title": "Spring Boot megtanulása",
    "description": "Fejezd be az 1. fázist a roadmap-ből",
    "done": false,
    "createdAt": "2026-09-23T14:30:00"
  }
]
```

**`GET /api/todos/999`** (nem létező id) — válasz (`404 Not Found`).

**`PUT /api/todos/1`** — kérés törzse:

```json
{
  "title": "Spring Boot megtanulása",
  "description": "Fejezd be az 1. fázist a roadmap-ből",
  "done": true
}
```

## Ajánlott osztályszerkezet

```
src/main/java/hu/molitor/todoapi/
├── TodoApiApplication.java
├── Todo.java
├── TodoController.java     # @RestController, lásd 1.3
└── TodoService.java        # @Service, in-memory tárolás egyelőre, lásd 1.2
```

A `TodoController` konstruktoron keresztül injektálja a `TodoService`-t (lásd
[1.2 Dependency Injection](02-dependency-injection.md)) — ne hívj közvetlenül semmilyen tárolást a
controllerből, ez a réteges gondolkodás (Controller → Service) ebben a fázisban még nem szigorú
szabály (az a [3. fázis](../03-validacio-hibakezeles.md) témája), de már most szokj hozzá.

## `application.yml` — H2 beállítás (előkészítés a 2. fázishoz)

```yaml
spring:
  datasource:
    url: jdbc:h2:mem:tododb
    driver-class-name: org.h2.Driver
    username: sa
    password:
  h2:
    console:
      enabled: true
      path: /h2-console
  jpa:
    hibernate:
      ddl-auto: update
    show-sql: true

app:
  name: Todo API
```

A `h2.console.enabled: true` egy böngészős admin felületet nyit meg a `/h2-console` úton — ez
kábé a Laravel Tinker/adatbázis-kliens gyors, beépített megfelelője fejlesztés közben, hasznos
lesz, amint a [2. fázisban](../02-adatreteg-jpa.md) valódi JPA entitásokra állsz át.

## Kipróbálás `curl`-lal

```bash
# lista
curl http://localhost:8080/api/todos

# létrehozás
curl -X POST http://localhost:8080/api/todos \
  -H "Content-Type: application/json" \
  -d '{"title":"Spring Boot megtanulása","description":"1. fázis befejezése"}'

# egy elem lekérése
curl http://localhost:8080/api/todos/1

# módosítás — kész státuszra állítás
curl -X PUT http://localhost:8080/api/todos/1 \
  -H "Content-Type: application/json" \
  -d '{"title":"Spring Boot megtanulása","description":"1. fázis befejezése","done":true}'

# törlés
curl -X DELETE http://localhost:8080/api/todos/1 -w "%{http_code}\n"

# szűrés query paraméterrel
curl "http://localhost:8080/api/todos?done=false"
```

## Hogyan bővül ez a projekt a következő fázisokban

- **[2. fázis](../02-adatreteg-jpa.md)** — a `Todo` valódi `@Entity`-vé alakul, `User`–`Todo`
  kapcsolat (`@ManyToOne`/`@OneToMany`), valódi PostgreSQL + Flyway migrációk Docker Compose-ban
- **[3. fázis](../03-validacio-hibakezeles.md)** — Bean Validation a bemeneti DTO-kon, egységes
  hibaválasz `@ControllerAdvice`-szal, tiszta Controller → Service → Repository réteges refaktor
- **[4. fázis](../04-spring-security.md)** — JWT-alapú regisztráció/bejelentkezés, a todo-k
  a bejelentkezett felhasználóhoz kötve, védett végpontok
- **[5. fázis](../05-teszteles.md)** — unit tesztek a service rétegre, integrációs tesztek
  Testcontainers + `@SpringBootTest` kombinációval
- **[6. fázis](../06-async-esemenyek.md)** — pl. egy esemény (`TodoCompletedEvent`) és async
  listener, amikor egy todo készre vált

---

[← Előző: 04. Konfiguráció](04-konfiguracio.md) · [Fázis index](../01-spring-boot-alapok.md) · [Főoldal](../../README.md) · Következő: [06. Kiegészítés: Thymeleaf →](06-sablon-renderesztes-thymeleaf.md)
