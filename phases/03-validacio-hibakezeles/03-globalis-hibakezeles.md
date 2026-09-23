[← Előző: 02. DTO-k és mapping](02-dto-k-es-mapping.md) · [Fázis index](../03-validacio-hibakezeles.md) · [Főoldal](../../README.md) · Következő: [04. Réteges architektúra →](04-reteges-architektura.md)

# 3.3 — Globális hibakezelés

## Laravel oldalon

```php
// PHP — app/Exceptions/Handler.php
public function register(): void
{
    $this->renderable(function (TodoNotFoundException $e, $request) {
        return response()->json(['message' => $e->getMessage()], 404);
    });
}
```

Validációs hibáknál Laravel automatikusan `422 Unprocessable Entity`-t ad `{"message": "...",
"errors": {"title": ["A title mező kötelező."]}}` formátumban, anélkül hogy bármit írnod kellene.

## Spring oldalon: `@ControllerAdvice`

Spring-ben nincs egyetlen központi `Handler` osztály — helyette egy `@ControllerAdvice`-szal
jelölt osztályt írsz, amiben `@ExceptionHandler`-rel jelölt metódusok fogják el az egyes
kivételtípusokat, **az összes controllerre vonatkozóan**:

```java
@RestControllerAdvice
public class GlobalExceptionHandler {

    @ExceptionHandler(TodoNotFoundException.class)
    public ResponseEntity<ErrorResponse> handleNotFound(TodoNotFoundException ex) {
        var body = new ErrorResponse(404, ex.getMessage(), Instant.now());
        return ResponseEntity.status(HttpStatus.NOT_FOUND).body(body);
    }

    @ExceptionHandler(MethodArgumentNotValidException.class)
    public ResponseEntity<ValidationErrorResponse> handleValidation(MethodArgumentNotValidException ex) {
        Map<String, String> fieldErrors = new HashMap<>();
        ex.getBindingResult().getFieldErrors().forEach(error ->
            fieldErrors.put(error.getField(), error.getDefaultMessage())
        );
        var body = new ValidationErrorResponse(422, "Validációs hiba", fieldErrors);
        return ResponseEntity.unprocessableEntity().body(body);
    }

    @ExceptionHandler(Exception.class)
    public ResponseEntity<ErrorResponse> handleGeneric(Exception ex) {
        var body = new ErrorResponse(500, "Váratlan hiba történt", Instant.now());
        return ResponseEntity.internalServerError().body(body);
    }
}
```

A `@RestControllerAdvice` a `@ControllerAdvice` + `@ResponseBody` kombinációja — automatikusan
JSON-ná szerializálja a visszaadott objektumot, pont úgy, mint egy normál `@RestController`.

## Egységes hibaválasz DTO

```java
public record ErrorResponse(int status, String message, Instant timestamp) {}

public record ValidationErrorResponse(int status, String message, Map<String, String> fieldErrors) {}
```

Cél, hogy **minden** hibaválasz (legyen az 404, 422 vagy 500) ugyanazt az alapvázat kövesse — ez
megkönnyíti a frontend/kliens oldali hibakezelést, pont ahogy egy jól megtervezett Laravel API-nál
is konzisztens hibaformátumra törekszel.

## Összevetés táblázatban

| Laravel | Spring |
|---|---|
| `app/Exceptions/Handler.php` | `@RestControllerAdvice` osztály |
| `renderable(fn ($e, $request) => ...)` | `@ExceptionHandler(SomeException.class)` metódus |
| automatikus 422 validációs válasz | `@ExceptionHandler(MethodArgumentNotValidException.class)` — magadnak kell megírni |
| `abort(404, '...')` egy controllerben | `throw new TodoNotFoundException(...)`, a kezelés máshol történik |
| egy fájlban minden eset | tetszőlegesen több `@ExceptionHandler` metódus egy vagy több `@ControllerAdvice` osztályban |

A legnagyobb mentális váltás: Laravel-ben a hibakezelés **egy célzottan erre kialakított osztály**
köré épül, ami sok esetben "ki van pipálva" a keretrendszer által (pl. validáció). Spring-ben
**mindent explicit kezelsz** — cserébe teljes kontrollod van afelett, hogy pontosan milyen
formátumban, milyen HTTP státusszal, milyen logolással megy ki egy adott hiba.

---

[← Előző: 02. DTO-k és mapping](02-dto-k-es-mapping.md) · [Fázis index](../03-validacio-hibakezeles.md) · [Főoldal](../../README.md) · Következő: [04. Réteges architektúra →](04-reteges-architektura.md)
