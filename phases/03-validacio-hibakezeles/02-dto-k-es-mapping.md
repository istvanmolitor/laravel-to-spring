[← Előző: 01. Bean Validation](01-bean-validation.md) · [Fázis index](../03-validacio-hibakezeles.md) · [Főoldal](../../README.md) · Következő: [03. Globális hibakezelés →](03-globalis-hibakezeles.md)

# 3.2 — DTO-k és mapping

## Miért ne exponáld az Entity-t közvetlenül

Laravel-ben megszokhattad, hogy egy Eloquent modellt simán visszaadsz a controllerből, és a
keretrendszer JSON-né szerializálja:

```php
// PHP — kényelmes, de kockázatos
public function show(Todo $todo)
{
    return $todo;   // minden mező kimegy, ami nincs a $hidden tömbben
}
```

Ez Laravel-ben is kockázatos (könnyű elfelejteni egy mezőt a `$hidden`/`$fillable` tömbből
kihagyni), de legalább van **valamilyen** beépített védőháló. Java/JPA `@Entity` osztályoknál
**nincs ilyen automatizmus** — ha egy entitást közvetlenül visszaadsz a controllerből, minden
mezője (jelszó hash, belső flag-ek, lazy kapcsolatok) kimehet, sőt a lazy kapcsolatok
szerializálása gyakran `LazyInitializationException`-t is dob, mert a JPA session már lezárult
mire a JSON-konverzió megtörténik.

**Szabály:** a controller réteg soha ne lásson `@Entity` osztályt közvetlenül — mindig DTO
(Data Transfer Object) megy ki és be rajta.

## `record` mint tömör DTO

Java 16+ óta van `record` — egy speciális, tömör osztálydeklaráció, ami pontosan DTO-khoz való:
automatikusan generál konstruktort, gettereket, `equals()`/`hashCode()`/`toString()`-ot.

```java
public record TodoResponse(Long id, String title, boolean done, LocalDateTime createdAt) {}

public record TodoRequest(
    @NotBlank @Size(max = 255) String title,
    @Size(max = 2000) String description
) {}
```

Ez körülbelül annak felel meg, amit Laravel-ben egy `TodoResource`/`TodoData` (Spatie
Laravel-Data) objektum jelent — egy dedikált, kizárólag adatátvitelre szolgáló típus, aminek nincs
üzleti logikája, csak mezői.

## Mapping Entity ↔ DTO — kézzel

```java
public class TodoMapper {
    public static TodoResponse toResponse(Todo todo) {
        return new TodoResponse(
            todo.getId(),
            todo.getTitle(),
            todo.isDone(),
            todo.getCreatedAt()
        );
    }

    public static Todo toEntity(TodoRequest request) {
        Todo todo = new Todo();
        todo.setTitle(request.title());
        todo.setDescription(request.description());
        return todo;
    }
}
```

Kis projektekben ez teljesen elfogadható és átlátható — pontosan az a szint, amit egy Laravel
`TodoResource::toArray()` metódusban is kézzel írnál meg.

## Mapping MapStruct-tal — nagyobb projektekhez

Ha sok entitásod/DTO-d van, a kézzel írt mapperek gyorsan monotonná válnak. A **MapStruct**
fordítási időben generál mapper implementációt egy interfész alapján:

```java
@Mapper(componentModel = "spring")
public interface TodoMapper {
    TodoResponse toResponse(Todo todo);
    Todo toEntity(TodoRequest request);
}
```

Ennyi — a MapStruct build-kor generál egy `TodoMapperImpl` osztályt, ami mezőnév-egyezés alapján
másolja át az értékeket (eltérő nevű mezőknél `@Mapping(source = "...", target = "...")`-tal
irányítod). Ez picit hasonlít arra, ahogy Laravel-ben a Spatie `laravel-data` csomag automatikus
transzformációt végez, csak itt fordítási időben (nem futásidőben reflection-nel) történik a
kódgenerálás, ami gyorsabb és a fordító azonnal jelez, ha egy mező nem illeszthető.

## Összevetés: Laravel API Resource

```php
// PHP — Laravel API Resource
class TodoResource extends JsonResource
{
    public function toArray($request): array
    {
        return [
            'id'         => $this->id,
            'title'      => $this->title,
            'done'       => $this->done,
            'created_at' => $this->created_at->toIso8601String(),
        ];
    }
}
```

```java
// Java — record + mapper, ugyanaz a szerep
public record TodoResponse(Long id, String title, boolean done, LocalDateTime createdAt) {}
```

A fő koncepcionális különbség: a Laravel Resource egy osztály, ami `toArray()`-jel maga végzi a
transzformációt (imperatív), míg a Java `record` egy tiszta adatszerkezet, a transzformációt
(a mapper) külön osztály végzi (deklaratív adat + különálló logika) — ez összhangban van a
[3.4 — Réteges architektúra](04-reteges-architektura.md) fejezetben tárgyalt felelősség-szétválasztási elvvel.

---

[← Előző: 01. Bean Validation](01-bean-validation.md) · [Fázis index](../03-validacio-hibakezeles.md) · [Főoldal](../../README.md) · Következő: [03. Globális hibakezelés →](03-globalis-hibakezeles.md)
