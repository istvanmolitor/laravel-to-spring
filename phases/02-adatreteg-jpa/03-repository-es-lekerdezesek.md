[← Előző: 02. Kapcsolatok modellezése](02-kapcsolatok.md) · [Fázis index](../02-adatreteg-jpa.md) · [Főoldal](../../README.md) · Következő: [04. Flyway migrációk →](04-flyway-migraciok.md)

# 2.3 — Repository réteg és lekérdezések

Eloquent-ben a query builder közvetlenül a modellosztályon él (`Todo::where(...)->get()`). Spring
Data JPA-ban ezt egy külön réteg, a **repository** végzi — ez egy interfész, aminek te csak a
szignatúráját írod meg, a Spring futásidőben generálja le mögé az implementációt.

## `JpaRepository<T, ID>` — mit kapsz ingyen

```java
public interface TodoRepository extends JpaRepository<Todo, Long> {
}
```

Ez az egyetlen sor máris ad neked egy csomó kész metódust, anélkül hogy egyetlen sort is
implementálnál:

```java
todoRepository.save(todo);            // insert vagy update (id alapján dönti el)
todoRepository.findById(1L);           // Optional<Todo>
todoRepository.findAll();              // List<Todo>
todoRepository.deleteById(1L);
todoRepository.count();
todoRepository.existsById(1L);
```

| Spring Data JPA | Eloquent megfelelő |
|---|---|
| `repository.save(entity)` | `$model->save()` |
| `repository.findById(id)` | `Todo::find($id)` — de itt `Optional<Todo>`-t kapsz, nem `null`-t! |
| `repository.findAll()` | `Todo::all()` |
| `repository.deleteById(id)` | `Todo::destroy($id)` |
| `repository.count()` | `Todo::count()` |

Figyeld meg: `findById` `Optional<Todo>`-t ad vissza, nem `null`-t vagy magát az entitást — ez
pontosan az [Optional](../00-java-alapok/05-optional.md) fejezetben tanult minta gyakorlati
alkalmazása. A kontroller/service rétegben ezt `.orElseThrow(...)`-zal fogod kezelni.

## Derived query methods — a metódusnév maga a lekérdezés

Ez a Spring Data legmeglepőbb, leginkább "mágikusnak" tűnő funkciója — de fontos, hogy ez sem
futásidejű mágia, hanem a Spring a metódusnév **szövegét elemzi** indításkor, és generál belőle
SQL-t:

```java
public interface TodoRepository extends JpaRepository<Todo, Long> {

    List<Todo> findByDone(boolean done);
    List<Todo> findByUserId(Long userId);
    List<Todo> findByUserIdAndDone(Long userId, boolean done);
    List<Todo> findByTitleContainingIgnoreCase(String keyword);
    Optional<Todo> findByIdAndUserId(Long id, Long userId);
    long countByUserId(Long userId);
    boolean existsByTitleAndUserId(String title, Long userId);
}
```

```php
// nagyjából ennek felel meg Eloquent query builderrel
Todo::where('done', $done)->get();
Todo::where('user_id', $userId)->get();
Todo::where('user_id', $userId)->where('done', $done)->get();
Todo::where('title', 'like', "%$keyword%")->get();
Todo::where('id', $id)->where('user_id', $userId)->first();
Todo::where('user_id', $userId)->count();
Todo::where('title', $title)->where('user_id', $userId)->exists();
```

A metódusnév felépítése kötött nyelvtant követ: `findBy` / `countBy` / `existsBy` +
mezőnév(ek) + opcionális módosítók (`Containing`, `IgnoreCase`, `GreaterThan`, `OrderBy...`) +
`And`/`Or` összekötők. Elsőre furcsa, hogy a "kód" egy metódusnévben él, de a gyakorlatban gyors
és olvasható — az IDE (VS Code Java kiegészítője vagy IntelliJ) validálja is a nevet a
mezőkhöz képest.

## JPQL és `@Query`

Amikor a derived method már nem elég kifejező, vagy összetettebb lekérdezés kell:

```java
public interface TodoRepository extends JpaRepository<Todo, Long> {

    @Query("SELECT t FROM Todo t WHERE t.user.id = :userId AND t.done = false ORDER BY t.createdAt DESC")
    List<Todo> findOpenTodosForUser(@Param("userId") Long userId);
}
```

Ez a **JPQL** (Jakarta Persistence Query Language) — figyeld meg, hogy `Todo t`-t és `t.user.id`-t
írunk, **nem** `todos` táblát és `user_id` oszlopot — a JPQL az entitásokon és azok mezőin
dolgozik, nem közvetlenül az adatbázis-sémán. Ez nagyjából a Laravel raw query builder
(`DB::table('todos')->where(...)`) és az Eloquent query builder közötti különbségnek felel meg,
csak fordítva: itt a JPQL az "entitás-szintű", és natív SQL-t is írhatsz, ha explicit kéred:

```java
@Query(value = "SELECT * FROM todos WHERE user_id = :userId AND done = false", nativeQuery = true)
List<Todo> findOpenTodosForUserNative(@Param("userId") Long userId);
```

Ez pontosan a Laravel `DB::select('SELECT * FROM todos WHERE user_id = ? AND done = false',
[$userId])` megfelelője — nyers SQL, amikor a JPQL/ORM már nem elég rugalmas (pl. adatbázis-specifikus
funkciók, komplex window function-ök).

## Mikor melyiket válaszd?

1. **Derived query method** — ha a lekérdezés egyszerű mező-alapú szűrés, ez a leggyorsabb és
   legolvashatóbb
2. **`@Query` JPQL-lel** — ha összetettebb (JOIN, aggregáció, egyedi rendezés), de még mindig
   entitás-szinten gondolkodsz
3. **`@Query(nativeQuery = true)`** — ha adatbázis-specifikus funkciót használsz, vagy a JPQL nem
   tudja kifejezni, amit szeretnél

Ez ugyanaz a döntési sorrend, mint amit Laravel-ben követsz: Eloquent metódus → query builder →
raw SQL, csak itt a "query builder" helyét a derived method + JPQL veszi át.

---

[← Előző: 02. Kapcsolatok modellezése](02-kapcsolatok.md) · [Fázis index](../02-adatreteg-jpa.md) · [Főoldal](../../README.md) · Következő: [04. Flyway migrációk →](04-flyway-migraciok.md)
