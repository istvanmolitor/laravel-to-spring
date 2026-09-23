[← Előző: 05. Tranzakciókezelés](05-tranzakciok.md) · [Fázis index](../02-adatreteg-jpa.md) · [Főoldal](../../README.md) · Következő: [07. Kiegészítés: cache kezelés →](07-cache-kezeles.md)

# 2.6 — Záró gyakorlat: Todo API bővítése

Cél: bővítsd az 1. fázisban épített Todo API-t user–todo kapcsolattal, Flyway migrációkkal, és
cseréld le a H2 in-memory adatbázist valódi PostgreSQL-re, Docker Compose-ban futtatva.

## 1. lépés — `User` entitás

```java
@Entity
@Table(name = "users")
@Getter @Setter @NoArgsConstructor
public class User {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(nullable = false, unique = true)
    private String email;

    @Column(nullable = false)
    private String name;

    @OneToMany(mappedBy = "user", cascade = CascadeType.ALL, orphanRemoval = true)
    private List<Todo> todos = new ArrayList<>();
}
```

## 2. lépés — `Todo` entitás bővítése `user` kapcsolattal

```java
@Entity
@Table(name = "todos")
@Getter @Setter @NoArgsConstructor
public class Todo {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(nullable = false)
    private String title;

    private String description;

    @Column(nullable = false)
    private boolean done = false;

    @Column(name = "created_at", nullable = false, updatable = false)
    private LocalDateTime createdAt = LocalDateTime.now();

    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "user_id", nullable = false)
    private User user;

    public Todo(String title, User user) {
        this.title = title;
        this.user = user;
    }
}
```

## 3. lépés — Flyway migrációk

`src/main/resources/db/migration/V1__create_users_table.sql`:

```sql
CREATE TABLE users (
    id BIGSERIAL PRIMARY KEY,
    email VARCHAR(255) NOT NULL UNIQUE,
    name VARCHAR(255) NOT NULL
);
```

`src/main/resources/db/migration/V2__create_todos_table.sql`:

```sql
CREATE TABLE todos (
    id BIGSERIAL PRIMARY KEY,
    title VARCHAR(255) NOT NULL,
    description TEXT,
    done BOOLEAN NOT NULL DEFAULT FALSE,
    created_at TIMESTAMP NOT NULL DEFAULT now(),
    user_id BIGINT NOT NULL REFERENCES users(id) ON DELETE CASCADE
);

CREATE INDEX idx_todos_user_id ON todos(user_id);
```

Figyeld meg: az `ON DELETE CASCADE` itt, az SQL szintjén garantálja azt, amit az entitás oldalán a
`cascade = CascadeType.ALL, orphanRemoval = true` fejez ki — a kettő együtt ad teljes védelmet,
mindkét réteget (adatbázis + alkalmazás) érdemes összhangban tartani.

## 4. lépés — `docker-compose.yml` PostgreSQL-lel

```yaml
services:
  db:
    image: postgres:16
    environment:
      POSTGRES_DB: todo_api
      POSTGRES_USER: todo_api
      POSTGRES_PASSWORD: secret
    ports:
      - "5432:5432"
    volumes:
      - todo_api_data:/var/lib/postgresql/data

volumes:
  todo_api_data:
```

Indítás: `docker compose up -d`

## 5. lépés — `application.yml` PostgreSQL kapcsolat

```yaml
spring:
  datasource:
    url: jdbc:postgresql://localhost:5432/todo_api
    username: todo_api
    password: secret
  jpa:
    hibernate:
      ddl-auto: validate
    show-sql: true
  flyway:
    enabled: true
    locations: classpath:db/migration
```

Ne felejtsd el a `pom.xml`-hez hozzáadni a PostgreSQL drivert és a Flyway starter-t, ha az 1.
fázisban még csak H2-vel dolgoztál:

```xml
<dependency>
    <groupId>org.postgresql</groupId>
    <artifactId>postgresql</artifactId>
    <scope>runtime</scope>
</dependency>
<dependency>
    <groupId>org.flywaydb</groupId>
    <artifactId>flyway-core</artifactId>
</dependency>
<dependency>
    <groupId>org.flywaydb</groupId>
    <artifactId>flyway-database-postgresql</artifactId>
</dependency>
```

## 6. lépés — repository-k és végpontok bővítése

- `UserRepository extends JpaRepository<User, Long>`, `findByEmail(String email)` derived query
- `TodoRepository`-ban: `findByUserId(Long userId)`, `findByIdAndUserId(Long id, Long userId)`
- Módosítsd a meglévő CRUD végpontokat úgy, hogy a `Todo` létrehozásához/lekérdezéséhez explicit
  `userId`-t is kelljen megadni (egyelőre kérésparaméterként — a 4. fázisban, a JWT auth
  bevezetésekor ez majd az autentikált userből fog jönni, nem explicit paraméterből)

## Ellenőrzés

```bash
docker compose up -d
./mvnw spring-boot:run

curl -X POST localhost:8080/api/users -d '{"email":"kovacs@example.com","name":"Kovács"}' -H 'Content-Type: application/json'
curl -X POST localhost:8080/api/todos -d '{"title":"Boltba menni","userId":1}' -H 'Content-Type: application/json'
curl localhost:8080/api/todos?userId=1
```

Ellenőrizd az induláskor keletkező logban, hogy a Flyway lefuttatta-e mindkét migrációt
(`Successfully applied 2 migrations`), és hogy a Hibernate `show-sql: true` mellett a
`findByUserId` hívás **egyetlen** SQL lekérdezést generál, nem N+1-et.

---

Ez a Todo API a következő fázisokban tovább alakul: a 3. fázisban réteges architektúrára
(Controller → Service → Repository) és DTO-kra refaktoráljuk, a 4.-ben JWT-alapú autentikációt
kap, hogy a `userId`-t ne kérésparaméterként, hanem a bejelentkezett userből vegye.

---

[← Előző: 05. Tranzakciókezelés](05-tranzakciok.md) · [Fázis index](../02-adatreteg-jpa.md) · [Főoldal](../../README.md) · Következő: [07. Kiegészítés: cache kezelés →](07-cache-kezeles.md)
