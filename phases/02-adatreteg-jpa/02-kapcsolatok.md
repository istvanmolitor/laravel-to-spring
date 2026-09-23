[← Előző: 01. Entity osztályok](01-entity-osztalyok.md) · [Fázis index](../02-adatreteg-jpa.md) · [Főoldal](../../README.md) · Következő: [03. Repository réteg és lekérdezések →](03-repository-es-lekerdezesek.md)

# 2.2 — Kapcsolatok modellezése

A Todo API-hoz most hozzáadunk egy `User` entitást — minden todo-nak legyen egy tulajdonosa. Ez a
klasszikus egy-a-többhöz (one-to-many) kapcsolat, amit Eloquent-ben `hasMany`/`belongsTo`
párossal ismersz.

## `@OneToMany` / `@ManyToOne`

```php
// PHP — Eloquent
class User extends Model {
    public function todos(): HasMany {
        return $this->hasMany(Todo::class);
    }
}

class Todo extends Model {
    public function user(): BelongsTo {
        return $this->belongsTo(User::class);
    }
}
```

```java
// Java — JPA
@Entity
@Table(name = "users")
@Getter @Setter @NoArgsConstructor
public class User {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String email;
    private String name;

    @OneToMany(mappedBy = "user", cascade = CascadeType.ALL, orphanRemoval = true)
    private List<Todo> todos = new ArrayList<>();
}

@Entity
@Table(name = "todos")
@Getter @Setter @NoArgsConstructor
public class Todo {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String title;
    private boolean done = false;

    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "user_id", nullable = false)
    private User user;
}
```

Amit érdemes megfigyelni:
- A **`@JoinColumn`** oldal (itt `Todo.user`) a kapcsolat "tulajdonosa" — ez felel meg annak,
  hogy melyik táblában van az idegen kulcs oszlop (`user_id`). Ez pontosan olyan, mint Eloquent-ben
  a `belongsTo` oldal.
- A **`mappedBy = "user"`** a `User.todos` oldalon azt mondja: "ez a kapcsolat fordítottja, a
  gazda oldal a `Todo` entitás `user` mezője" — ennek nincs Eloquent megfelelője, mert Eloquent-ben
  mindkét irány önállóan, metódushívással van definiálva, itt viszont a két oldal explicit
  hivatkozik egymásra a JPA mapping szintjén.
- **`cascade = CascadeType.ALL, orphanRemoval = true`**: ha törlöd a `User`-t, a hozzá tartozó
  `Todo`-k is törlődnek — ez körülbelül a Laravel `$table->foreignId('user_id')->constrained()
  ->cascadeOnDelete()` migrációs beállításnak felel meg, csak itt az entitás szintjén, nem az SQL
  migrációban deklarálod (bár a DB constraint-et is be kell állítani Flyway migrációban, lásd
  [2.4 Flyway migrációk](04-flyway-migraciok.md)).

## `@ManyToMany`

Ha lenne pl. `Tag` entitásod, amit több `Todo`-hoz is hozzá lehet rendelni:

```java
@Entity
public class Todo {
    @ManyToMany
    @JoinTable(
        name = "todo_tags",
        joinColumns = @JoinColumn(name = "todo_id"),
        inverseJoinColumns = @JoinColumn(name = "tag_id")
    )
    private Set<Tag> tags = new HashSet<>();
}
```

Ez az Eloquent `belongsToMany` + automatikus pivot tábla mintájának felel meg, csak itt a pivot
tábla nevét (`todo_tags`) és oszlopait explicit kell megadnod a `@JoinTable`-ben.

## Lazy vs eager loading — az N+1 probléma Java módra

Ez a legfontosabb rész ebben a fejezetben, mert itt könnyű elrontani a teljesítményt.

```php
// PHP — klasszikus N+1 hiba Eloquent-ben
$todos = Todo::all();               // 1 query
foreach ($todos as $todo) {
    echo $todo->user->name;         // +N query, minden todo-nál külön lekérdezés!
}

// megoldás: eager loading
$todos = Todo::with('user')->get(); // 2 query összesen (todos + users IN (...))
```

JPA-ban ugyanez a probléma létezik, csak itt a `FetchType` explicit kikényszeríti, hogy tudatosan
dönts:

```java
@ManyToOne(fetch = FetchType.LAZY)   // alapértelmezett @ManyToOne-nál: EAGER — DE mindig írd ki LAZY-ra!
private User user;

@OneToMany(mappedBy = "user")         // alapértelmezett @OneToMany-nál: LAZY
private List<Todo> todos;
```

| Kapcsolat típusa | Alapértelmezett FetchType | Ajánlás |
|---|---|---|
| `@ManyToOne` | `EAGER` | Írd explicit `LAZY`-ra — az `EAGER` alapértelmezett a leggyakoribb N+1 forrás |
| `@OneToMany` | `LAZY` | Maradhat `LAZY`, ez a biztonságos alapeset |
| `@ManyToMany` | `LAZY` | Maradhat `LAZY` |

`LAZY` mellett a kapcsolódó entitás csak akkor töltődik be az adatbázisból, amikor **ténylegesen
hozzáférsz** (`todo.getUser().getName()`) — ha ezt egy ciklusban teszed N elemre, ugyanúgy N+1
lekérdezést kapsz, mint Eloquent-ben `with()` nélkül.

### Megoldás: `JOIN FETCH`

```java
public interface TodoRepository extends JpaRepository<Todo, Long> {

    @Query("SELECT t FROM Todo t JOIN FETCH t.user WHERE t.done = false")
    List<Todo> findAllOpenWithUser();
}
```

A `JOIN FETCH` pontosan az Eloquent `with('user')` megfelelője — egyetlen lekérdezésben, JOIN-nal
hozza be a kapcsolódó entitást is, elkerülve az N+1-et. A repository rétegről (ahol ezt a
`@Query`-t írod) a következő fejezetben lesz szó részletesen.

**Gyakorlati szabály**: minden `@ManyToOne`/`@OneToMany` mezőnél tudatosan gondold át — kell-e a
kapcsolódó entitás minden lekérdezésnél, vagy csak néha? Alapértelmezésként mindig `LAZY`, és ahol
tényleg kell a kapcsolódó adat, ott explicit `JOIN FETCH`-csel vagy Spring Data
`@EntityGraph`-fal (haladóbb, később érdemes megismerni) told be egy lekérdezésbe.

---

[← Előző: 01. Entity osztályok](01-entity-osztalyok.md) · [Fázis index](../02-adatreteg-jpa.md) · [Főoldal](../../README.md) · Következő: [03. Repository réteg és lekérdezések →](03-repository-es-lekerdezesek.md)
