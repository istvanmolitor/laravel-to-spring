[← Előző: 04. Flyway migrációk](04-flyway-migraciok.md) · [Fázis index](../02-adatreteg-jpa.md) · [Főoldal](../../README.md) · Következő: [06. Gyakorlat: Todo API bővítése →](06-gyakorlat-todo-api-bovitese.md)

# 2.5 — Tranzakciókezelés

## Imperatív closure vs deklaratív annotáció

```php
// PHP — Laravel
DB::transaction(function () use ($userId, $title) {
    $user = User::findOrFail($userId);
    $todo = Todo::create(['title' => $title, 'user_id' => $user->id]);
    $user->increment('todo_count');
});
```

```java
// Java — Spring
@Service
@RequiredArgsConstructor
public class TodoService {

    private final UserRepository userRepository;
    private final TodoRepository todoRepository;

    @Transactional
    public Todo createTodo(Long userId, String title) {
        User user = userRepository.findById(userId)
            .orElseThrow(() -> new IllegalArgumentException("Nincs ilyen user"));

        Todo todo = new Todo(title, user);
        todoRepository.save(todo);

        user.setTodoCount(user.getTodoCount() + 1);
        // nincs explicit save() a user-re — a @Transactional metóduson belül
        // a "dirty checking" miatt Hibernate automatikusan észreveszi a változást
        // és a tranzakció végén UPDATE-eli

        return todo;
    }
}
```

A legszembetűnőbb különbség: Laravel-ben a tranzakciót egy **closure-be csomagolod** explicit
(`DB::transaction(function () { ... })`), Spring-ben egy **annotációval** (`@Transactional`) jelzed
a teljes metóduson, hogy tranzakcióban kell futnia. Ez utóbbi deklaratív — nem te írsz kódot a
commit/rollback logikára, a Spring egy proxy-n keresztül automatikusan köré csomagolja azt.

## Mi történik hiba esetén

Ha a `@Transactional` metóduson belül egy **unchecked exception** (RuntimeException vagy
leszármazottja) repül ki, Spring automatikusan **rollback**-eli a teljes tranzakciót — pontosan
úgy, mint amikor a Laravel `DB::transaction()` closure-jéből egy exception dobódik ki.

Fontos buktató: **checked exception esetén Spring alapból NEM rollback-el** — ez a Java checked/
unchecked exception rendszerének [korábban tanult](../00-java-alapok/07-kivetelkezeles.md)
következménye. Ha mégis checked exceptiont használnál (ritka eset), explicit kell jelezned:

```java
@Transactional(rollbackFor = SomeCheckedException.class)
public void riskyOperation() throws SomeCheckedException { ... }
```

Gyakorlati szabály: ha a saját üzleti kivételeidet (mint a korábbi
[bank szimulátor gyakorlat](../00-java-alapok/07-kivetelkezeles.md) `InsufficientFundsException`-je)
`RuntimeException`-ből származtatod — ahogy a Java fejezet is javasolta —, ez a probléma eleve
fel sem merül, mert az unchecked exceptionök mindig rollback-elnek.

## Dirty checking — amiről Laravel-ben nem kell gondolkodni

Fontos JPA-specifikus koncepció, aminek nincs Eloquent megfelelője: egy `@Transactional` metóduson
belül betöltött entitást (pl. `userRepository.findById(...)`) **nem kell explicit elmenteni**, ha
módosítod a mezőit — a Hibernate a tranzakció végén (commit-kor) automatikusan összeveti az
entitás állapotát az eredetivel, és ha eltérést talál, `UPDATE` SQL-t generál. Ez a "dirty
checking" mechanizmus. Eloquent-ben mindig explicit hívnod kell a `->save()`-et — itt viszont
elég a tranzakción belüli módosítás, a mentés automatikus.

## Propagation — csak felismerés szintjén

Advanced téma, amivel egyelőre elég felismerés szinten találkozni: a `@Transactional`-nak van egy
`propagation` paramétere, ami azt szabályozza, mi történjen, ha egy már tranzakcióban futó metódus
hív meg egy másik `@Transactional` metódust (csatlakozzon a meglévőhöz, vagy indítson újat). Az
alapértelmezett `Propagation.REQUIRED` (csatlakozik a meglévő tranzakcióhoz, vagy újat indít, ha
nincs) a legtöbb esetben pont azt csinálja, amit intuitívan várnál — egyelőre nem kell mélyebben
foglalkozni vele.

---

[← Előző: 04. Flyway migrációk](04-flyway-migraciok.md) · [Fázis index](../02-adatreteg-jpa.md) · [Főoldal](../../README.md) · Következő: [06. Gyakorlat: Todo API bővítése →](06-gyakorlat-todo-api-bovitese.md)
