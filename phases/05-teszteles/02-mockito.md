[← Előző: 01. JUnit 5 alapok](01-junit5-alapok.md) · [Fázis index](../05-teszteles.md) · [Főoldal](../../README.md) · Következő: [03. Spring teszt annotációk →](03-spring-teszt-annotaciok.md)

# 5.2 — Mockito

A `TodoService` unit tesztjénél nem akarod, hogy a teszt valódi adatbázist érjen el — a
`TodoRepository`-t "hamisítani" (mockolni) akarod, hogy kontrollált bemenetekkel tesztelhesd a
service réteg üzleti logikáját, elszigetelve az adatréteg valós viselkedésétől. Erre való a
Mockito, a Java világ Mockery-je.

## Laravel Mockery vs Mockito

```php
// Laravel — Mockery
$repository = Mockery::mock(TodoRepository::class);
$repository->shouldReceive('findById')
    ->with(1)
    ->andReturn(new Todo(1, 'Bevásárlás', false));

$service = new TodoService($repository);
$todo = $service->getById(1);

expect($todo->getTitle())->toBe('Bevásárlás');
```

```java
// Java — Mockito
import org.junit.jupiter.api.extension.ExtendWith;
import org.mockito.Mock;
import org.mockito.junit.jupiter.MockitoExtension;
import static org.mockito.Mockito.when;

@ExtendWith(MockitoExtension.class)
class TodoServiceTest {

    @Mock
    private TodoRepository repository;

    @InjectMocks
    private TodoService service;

    @Test
    void returnsTodoById() {
        when(repository.findById(1L)).thenReturn(Optional.of(new Todo(1L, "Bevásárlás", false)));

        Todo todo = service.getById(1L);

        assertEquals("Bevásárlás", todo.getTitle());
    }
}
```

A `@Mock` létrehozza a hamis `TodoRepository`-t, az `@InjectMocks` pedig automatikusan
példányosítja a `TodoService`-t, és a konstruktorán keresztül beinjektálja a mockolt
függőségeket — ez pontosan a [1. fázisban](../01-spring-boot-alapok/02-dependency-injection.md)
tárgyalt konstruktor-alapú DI-t használja ki, csak most a Spring container helyett Mockito
állítja össze a példányt.

## `when...thenReturn` és a Laravel facade-mockolás különbsége

Laravel-ben gyakran facade-okat mockolsz közvetlenül (`Todo::shouldReceive('find')->andReturn(...)`),
ami a globális, statikus-szerű hozzáférés miatt lehetséges, de el is rejti a függőséget — a
tesztből nem látszik explicit, hogy a `TodoService` mitől függ, csak a mock-elvárásokból derül ki.

Spring-ben nincs ilyen "mockold a statikus elérést" minta, mert nincsenek facade-ok — mindig egy
konkrét, konstruktoron át kapott objektumot (`TodoRepository repository`) mockolsz. Ez explicitebb:
a `TodoServiceTest` szignatúrájából (a `@Mock` mezőkből) azonnal látszik, mitől függ a tesztelt
osztály.

## `verify` — hívás megtörténtének ellenőrzése

```java
@Test
void deletesTodoAndLogsIt() {
    Todo todo = new Todo(1L, "Bevásárlás", false);
    when(repository.findById(1L)).thenReturn(Optional.of(todo));

    service.delete(1L);

    verify(repository).delete(todo);          // ellenőrzi, hogy pontosan egyszer meghívódott
    verify(repository, never()).save(any());  // ellenőrzi, hogy NEM hívódott meg
}
```

Ez a Mockery `shouldHaveReceived()`/`shouldNotHaveReceived()` párjának felel meg — amikor nem a
visszatérési értéket, hanem azt akarod ellenőrizni, hogy egy mellékhatás (pl. egy metódushívás)
valóban megtörtént-e.

## `ArgumentCaptor` — mivel hívták meg pontosan?

```java
@Test
void savesTodoWithTrimmedTitle() {
    service.create("  Bevásárlás  ", "leírás");

    ArgumentCaptor<Todo> captor = ArgumentCaptor.forClass(Todo.class);
    verify(repository).save(captor.capture());

    assertEquals("Bevásárlás", captor.getValue().getTitle());  // trim megtörtént-e
}
```

Ennek nincs túl elterjedt, közvetlen Mockery megfelelője — Laravel-ben ilyenkor tipikusan egy
closure-alapú matcher-t adnál a `shouldReceive('save')->with(function ($todo) { ... })`
hívásba. Az `ArgumentCaptor` ugyanezt teszi explicitebb, kétlépéses formában: előbb elkapod az
argumentumot, utána külön assertelsz rajta.

## Mikor NE mockolj

Ugyanaz a szabály érvényes, mint Laravel-ben: a `TodoService` unit tesztjeiben mockold a
`TodoRepository`-t (mert az adatréteg elérése), de **ne** mockolj egyszerű, mellékhatás nélküli
objektumokat (pl. egy DTO-t vagy egy `Todo` entitást) — azokat egyszerűen példányosítsd. A
mockolás azokra a függőségekre való, amik I/O-t végeznek (adatbázis, hálózat, fájlrendszer) vagy
amiknek a viselkedését explicit kontrollálni akarod a tesztben.

---

[← Előző: 01. JUnit 5 alapok](01-junit5-alapok.md) · [Fázis index](../05-teszteles.md) · [Főoldal](../../README.md) · Következő: [03. Spring teszt annotációk →](03-spring-teszt-annotaciok.md)
