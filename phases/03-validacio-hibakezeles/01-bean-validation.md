[← Fázis index](../03-validacio-hibakezeles.md) · [Főoldal](../../README.md) · Következő: [02. DTO-k és mapping →](02-dto-k-es-mapping.md)

# 3.1 — Bean Validation

Laravel-ben a bejövő adatok validálását a Form Request `rules()` metódusában írod le, szöveges
szabálylistával. Spring-ben ugyanezt a **Bean Validation** szabvány (`jakarta.validation`)
annotációkkal oldja meg, közvetlenül a DTO mezőin — a validáció a típus része lesz, nem egy
külön, szövegként megfogalmazott szabályhalmaz.

## Ugyanaz a szabályhalmaz mindkét nyelven

```php
// PHP — Laravel Form Request
class StoreTodoRequest extends FormRequest
{
    public function rules(): array
    {
        return [
            'title'       => 'required|string|max:255',
            'description' => 'nullable|string|max:2000',
            'ownerEmail'  => 'required|email',
            'priority'    => 'required|integer|min:1|max:5',
        ];
    }
}
```

```java
// Java — Spring Bean Validation
public record TodoRequest(
    @NotBlank @Size(max = 255) String title,
    @Size(max = 2000) String description,
    @NotBlank @Email String ownerEmail,
    @NotNull @Min(1) @Max(5) Integer priority
) {}
```

A controller oldalon a `@Valid` annotáció kapcsolja be az ellenőrzést:

```java
@PostMapping("/api/todos")
public ResponseEntity<TodoResponse> create(@Valid @RequestBody TodoRequest request) {
    // ha idáig eljutunk, a request GARANTÁLTAN érvényes —
    // a Spring már a metódus meghívása ELŐTT elkapja és elutasítja az érvénytelen kérést
    ...
}
```

Ez a legfontosabb mentális váltás: Laravel-ben a `rules()` egy külön osztályban, futásidőben
kiértékelt szabálylista, amit a controller elé kapcsolt middleware fut le. Spring-ben a validáció
**a típus (DTO) részévé válik** — ha látod a `TodoRequest` deklarációját, a validációs szabályok
is ott vannak melletted, nem egy másik fájlban kell keresned őket.

## A leggyakoribb annotációk

| Annotáció | Jelentés | Laravel megfelelő |
|---|---|---|
| `@NotNull` | nem lehet `null` | `required` (numerikus/objektum mezőknél) |
| `@NotBlank` | String, nem `null` és nem csak whitespace | `required` (string mezőknél) |
| `@NotEmpty` | Collection/String, nem `null` és nem üres | `required` |
| `@Size(min=, max=)` | String/Collection hossza | `min:`, `max:`, `string`/`array` |
| `@Min` / `@Max` | szám alsó/felső határa | `min:`, `max:` |
| `@Email` | email formátum | `email` |
| `@Pattern(regexp=)` | regex illesztés | `regex:` |
| `@Positive` / `@PositiveOrZero` | szám előjele | `min:0` / `min:1` |
| `@Past` / `@Future` | dátum a múltban/jövőben | `before:today` / `after:today` |

## Egyedi validátor írása

Ha egy beépített annotáció nem elég (pl. "a cím nem tartalmazhat tiltott szót"), saját
constraintet írhatsz — ez a Laravel egyedi validációs szabályainak (`Rule::make()` vagy egyedi
`Rule` osztály) felel meg:

```java
@Target({ElementType.FIELD})
@Retention(RetentionPolicy.RUNTIME)
@Constraint(validatedBy = NoBannedWordsValidator.class)
public @interface NoBannedWords {
    String message() default "A cím tiltott szót tartalmaz";
    Class<?>[] groups() default {};
    Class<? extends Payload>[] payload() default {};
}

public class NoBannedWordsValidator implements ConstraintValidator<NoBannedWords, String> {
    private static final List<String> BANNED = List.of("spam", "teszt123");

    @Override
    public boolean isValid(String value, ConstraintValidatorContext context) {
        if (value == null) return true;
        return BANNED.stream().noneMatch(value::contains);
    }
}

// használat: @NotBlank @NoBannedWords String title
```

Ez több boilerplate, mint egy Laravel `Rule::make(fn ($attribute, $value, $fail) => ...)`, de
cserébe a saját szabály is típusos, újrafelhasználható annotációvá válik, amit ugyanúgy rá tudsz
tenni bármelyik DTO mezőre.

## Mi történik érvénytelen kérésnél?

Alapból a Spring egy `MethodArgumentNotValidException`-t dob, amit — ha nem kezeled külön —
a keretrendszer egy `400 Bad Request` válasszá alakít, mezőnkénti hibaüzenetekkel. A
[3.3 — Globális hibakezelés](03-globalis-hibakezeles.md) fejezetben megmutatjuk, hogyan alakítod
ezt a Laravel automatikus `422 Unprocessable Entity` + `{"errors": {...}}` válaszához hasonló,
egységes formátumra.

---

[← Fázis index](../03-validacio-hibakezeles.md) · [Főoldal](../../README.md) · Következő: [02. DTO-k és mapping →](02-dto-k-es-mapping.md)
