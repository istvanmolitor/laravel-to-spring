[← Előző: 04. Réteges architektúra](04-reteges-architektura.md) · [Fázis index](../03-validacio-hibakezeles.md) · [Főoldal](../../README.md) · Következő fázis: [04. Spring Security →](../04-spring-security.md)

# 3.5 — Záró gyakorlat: refaktorálás

Cél: refaktoráld a Todo API-t (1–2. fázisban felépített projekt) tiszta réteges struktúrára,
DTO-kkal és egységes hibaválasz formátummal.

## Célcsomag-struktúra

```
src/main/java/hu/molitor/todoapi/
├── controller/
│   └── TodoController.java
├── service/
│   └── TodoService.java
├── repository/
│   └── TodoRepository.java
├── entity/
│   ├── Todo.java
│   └── User.java
├── dto/
│   ├── TodoRequest.java
│   ├── TodoResponse.java
│   └── ErrorResponse.java
└── exception/
    ├── TodoNotFoundException.java
    ├── UserNotFoundException.java
    └── GlobalExceptionHandler.java
```

## `TodoService` váza

```java
@Service
public class TodoService {
    private final TodoRepository todoRepository;

    public TodoService(TodoRepository todoRepository) {
        this.todoRepository = todoRepository;
    }

    public List<TodoResponse> findAllByUser(Long userId) {
        return todoRepository.findByUserId(userId).stream()
            .map(TodoMapper::toResponse)
            .toList();
    }

    public TodoResponse findById(Long id) {
        Todo todo = todoRepository.findById(id)
            .orElseThrow(() -> new TodoNotFoundException(id));
        return TodoMapper.toResponse(todo);
    }

    @Transactional
    public TodoResponse create(TodoRequest request, Long userId) {
        Todo todo = TodoMapper.toEntity(request);
        todo.setUserId(userId);
        return TodoMapper.toResponse(todoRepository.save(todo));
    }

    @Transactional
    public TodoResponse update(Long id, TodoRequest request) {
        Todo todo = todoRepository.findById(id)
            .orElseThrow(() -> new TodoNotFoundException(id));
        todo.setTitle(request.title());
        todo.setDescription(request.description());
        return TodoMapper.toResponse(todoRepository.save(todo));
    }

    @Transactional
    public void delete(Long id) {
        if (!todoRepository.existsById(id)) {
            throw new TodoNotFoundException(id);
        }
        todoRepository.deleteById(id);
    }
}
```

## `TodoNotFoundException`

```java
public class TodoNotFoundException extends RuntimeException {
    public TodoNotFoundException(Long id) {
        super("Nem található todo ezzel az azonosítóval: " + id);
    }
}
```

## `GlobalExceptionHandler` — teljes kód 3 esetre

```java
@RestControllerAdvice
public class GlobalExceptionHandler {

    @ExceptionHandler(TodoNotFoundException.class)
    public ResponseEntity<ErrorResponse> handleTodoNotFound(TodoNotFoundException ex) {
        return ResponseEntity.status(HttpStatus.NOT_FOUND)
            .body(new ErrorResponse(404, ex.getMessage(), Instant.now()));
    }

    @ExceptionHandler(UserNotFoundException.class)
    public ResponseEntity<ErrorResponse> handleUserNotFound(UserNotFoundException ex) {
        return ResponseEntity.status(HttpStatus.NOT_FOUND)
            .body(new ErrorResponse(404, ex.getMessage(), Instant.now()));
    }

    @ExceptionHandler(MethodArgumentNotValidException.class)
    public ResponseEntity<ValidationErrorResponse> handleValidation(MethodArgumentNotValidException ex) {
        Map<String, String> errors = new HashMap<>();
        ex.getBindingResult().getFieldErrors()
            .forEach(e -> errors.put(e.getField(), e.getDefaultMessage()));
        return ResponseEntity.unprocessableEntity()
            .body(new ValidationErrorResponse(422, "Validációs hiba", errors));
    }
}
```

## DTO-k `@Valid` validációval

```java
public record TodoRequest(
    @NotBlank @Size(max = 255) String title,
    @Size(max = 2000) String description
) {}

public record TodoResponse(Long id, String title, String description, boolean done) {}

public record ErrorResponse(int status, String message, Instant timestamp) {}

public record ValidationErrorResponse(int status, String message, Map<String, String> fieldErrors) {}
```

## `TodoController` — csak HTTP be-/kimenet

```java
@RestController
@RequestMapping("/api/todos")
public class TodoController {
    private final TodoService todoService;

    public TodoController(TodoService todoService) {
        this.todoService = todoService;
    }

    @GetMapping("/{id}")
    public TodoResponse getById(@PathVariable Long id) {
        return todoService.findById(id);
    }

    @PostMapping
    public ResponseEntity<TodoResponse> create(@Valid @RequestBody TodoRequest request) {
        TodoResponse created = todoService.create(request, currentUserId());
        return ResponseEntity.status(HttpStatus.CREATED).body(created);
    }

    @PutMapping("/{id}")
    public TodoResponse update(@PathVariable Long id, @Valid @RequestBody TodoRequest request) {
        return todoService.update(id, request);
    }

    @DeleteMapping("/{id}")
    public ResponseEntity<Void> delete(@PathVariable Long id) {
        todoService.delete(id);
        return ResponseEntity.noContent().build();
    }
}
```

(A `currentUserId()` helper egyelőre lehet egy ideiglenes placeholder — a 4. fázisban a JWT
autentikáció bevezetésekor ez a bejelentkezett felhasználó tényleges azonosítójává válik.)

## Ellenőrzés

Futtasd a projektet, és `curl`-lel próbáld ki:

```bash
curl -X POST http://localhost:8080/api/todos \
  -H "Content-Type: application/json" \
  -d '{"title": ""}'
# várt válasz: 422, mezőnkénti hibaüzenettel a "title" mezőre

curl http://localhost:8080/api/todos/9999
# várt válasz: 404, egységes ErrorResponse formátumban
```

Ez a réteges struktúra (Controller → Service → Repository, DTO-k, központi hibakezelés) a
következő fázisokban tovább bővül: a [4. fázisban](../04-spring-security.md) JWT autentikációt kap
(a `currentUserId()` valódi implementációt), az [5. fázisban](../05-teszteles.md) pedig ez a
tiszta réteges felépítés teszi könnyen tesztelhetővé a `TodoService`-t Mockito-val, HTTP réteg
nélkül.

---

[← Előző: 04. Réteges architektúra](04-reteges-architektura.md) · [Fázis index](../03-validacio-hibakezeles.md) · [Főoldal](../../README.md) · Következő fázis: [04. Spring Security →](../04-spring-security.md)
