[← Előző: 03. Repository réteg és lekérdezések](03-repository-es-lekerdezesek.md) · [Fázis index](../02-adatreteg-jpa.md) · [Főoldal](../../README.md) · Következő: [05. Tranzakciókezelés →](05-tranzakciok.md)

# 2.4 — Adatbázis migráció Flyway-jel

Az 1. fázisban H2 in-memory adatbázist használtunk, ahol Hibernate `ddl-auto: update`
beállítással automatikusan generálta a sémát az entitásokból. Ez kényelmes prototípusnak, de
**production közelében soha nem használod** — pontosan úgy, ahogy Laravel-ben sem `Schema::create`
hívogatnál kézzel minden induláskor, hanem verziózott migrációkat írsz.

## Artisan migration → Flyway

```php
// PHP — Artisan migráció
// database/migrations/2024_01_15_000000_create_todos_table.php
public function up(): void
{
    Schema::create('todos', function (Blueprint $table) {
        $table->id();
        $table->string('title');
        $table->text('description')->nullable();
        $table->boolean('done')->default(false);
        $table->timestamps();
    });
}

public function down(): void
{
    Schema::dropIfExists('todos');
}
```

```sql
-- Java/Spring — Flyway migráció
-- src/main/resources/db/migration/V1__create_todos_table.sql
CREATE TABLE todos (
    id BIGSERIAL PRIMARY KEY,
    title VARCHAR(255) NOT NULL,
    description TEXT,
    done BOOLEAN NOT NULL DEFAULT FALSE,
    created_at TIMESTAMP NOT NULL DEFAULT now()
);
```

A legnagyobb különbség: **nincs PHP DSL** (`Schema::create`, `Blueprint`) — közvetlenül SQL-t
írsz. Ez elsőre visszalépésnek tűnhet, de az előnye, hogy pontosan azt kapod, amit írtál, nincs
DSL-fordítási réteg, ami esetleg máshogy viselkedik adatbázis-motoronként.

## Fájlnév-konvenció — ez kritikus

```
src/main/resources/db/migration/
├── V1__create_users_table.sql
├── V2__create_todos_table.sql
├── V3__add_index_to_todos_user_id.sql
└── V4__add_priority_column_to_todos.sql
```

Szabályok:
- `V<verzió>__<leírás>.sql` — a dupla aláhúzás (`__`) kötelező elválasztó
- A verziószám **monoton növekvő**, és Flyway **sorrendben, egyszer** futtatja le mindet egy
  belső `flyway_schema_history` táblában nyilvántartva, melyik futott már le
- **Soha ne módosíts egy már lefuttatott migrációt** — Flyway checksum-mal ellenőrzi, hogy a fájl
  tartalma nem változott-e a legutóbbi futtatás óta, és hibát dob, ha igen. Ha hibát vétettél, egy
  **új** migrációt írsz, ami javítja (pl. `V5__fix_todos_priority_default.sql`) — ez szigorúbb,
  mint az Artisan világ, ahol simán újra futtathatod a `down()`+`up()` párt fejlesztés közben

## Nincs automatikus "down" — ez a legnagyobb mentális váltás

Laravel migrációban minden `up()`-hoz tartozik egy `down()`, amit a `php artisan migrate:rollback`
futtat. A Flyway ingyenes (Community) verziója **nem támogatja az automatikus rollback-et** — a
migrációk csak előre haladnak. Ha vissza kell vonnod egy változtatást, egy **új**, "javító"
migrációt írsz (pl. ha `V4` hozzáadott egy oszlopot, és ki kell venni, `V5__remove_priority_column`
törli azt).

Ez a gyakorlatban azt jelenti, hogy sokkal óvatosabban kell megterveznetek a séma-változtatásokat,
mint amit Laravel-ben a rollback biztonsági hálója mellett megszokhattál — ez tudatos tervezési
fegyelmet igényel, nem egy hiányosság, amit meg kell kerülni.

## Hogyan fut le

A Spring Boot induláskor automatikusan lefuttatja a még le nem futott migrációkat, ha a Flyway
dependency a `pom.xml`-ben van, és az `application.yml`-ben a megfelelő adatbázis-kapcsolat be
van állítva:

```yaml
spring:
  flyway:
    enabled: true
    locations: classpath:db/migration
  jpa:
    hibernate:
      ddl-auto: validate   # FONTOS: éles/Flyway mellett SOHA ne legyen "update" vagy "create"
```

A `ddl-auto: validate` azt jelenti: Hibernate ellenőrzi, hogy az entitásaid összhangban vannak-e
a ténylegesen migrált sémával, de **nem** módosítja azt — a séma egyetlen forrása mostantól a
Flyway migrációs fájlok, nem a `@Entity` annotációk. Ez felel meg annak az elvnek, hogy Laravel-ben
is a migrációs fájlok a séma "igazság forrása", nem az Eloquent modell.

## Docker Compose-hoz kötve

Fejlesztéskor a Flyway automatikusan lefut, amikor a Spring Boot app csatlakozik a
`docker-compose.yml`-ben futó PostgreSQL-hez — erről konkrét példa a fázis záró gyakorlatában,
[2.6 Gyakorlat](06-gyakorlat-todo-api-bovitese.md).

---

[← Előző: 03. Repository réteg és lekérdezések](03-repository-es-lekerdezesek.md) · [Fázis index](../02-adatreteg-jpa.md) · [Főoldal](../../README.md) · Következő: [05. Tranzakciókezelés →](05-tranzakciok.md)
