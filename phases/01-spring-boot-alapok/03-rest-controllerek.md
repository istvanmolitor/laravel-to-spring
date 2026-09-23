[← Előző: 02. Dependency Injection](02-dependency-injection.md) · [Fázis index](../01-spring-boot-alapok.md) · [Főoldal](../../README.md) · Következő: [04. Konfiguráció →](04-konfiguracio.md)

# 1.3 — REST controllerek

Ez az a terület, ahol a Laravel route+controller tudásod szinte **1:1 átül** — csak a
route-definíció helye változik: Laravel-ben egy külön `routes/api.php` fájlban kötöd össze az
URL-t a controllerrel, Spring-ben az útvonal közvetlenül a controller metóduson van annotációként.

## `@RestController` és `@RequestMapping`

```php
// Laravel — routes/api.php
Route::prefix('api/todos')->group(function () {
    Route::get('/', [TodoController::class, 'index']);
    Route::get('/{id}', [TodoController::class, 'show']);
    Route::post('/', [TodoController::class, 'store']);
    Route::put('/{id}', [TodoController::class, 'update']);
    Route::delete('/{id}', [TodoController::class, 'destroy']);
});
```

```java
// Spring — a route közvetlenül a controlleren van, nincs külön route fájl
@RestController
@RequestMapping("/api/todos")
public class TodoController {

    @GetMapping
    public List<Todo> index() { ... }

    @GetMapping("/{id}")
    public Todo show(@PathVariable Long id) { ... }

    @PostMapping
    public Todo store(@RequestBody Todo todo) { ... }

    @PutMapping("/{id}")
    public Todo update(@PathVariable Long id, @RequestBody Todo todo) { ... }

    @DeleteMapping("/{id}")
    public void destroy(@PathVariable Long id) { ... }
}
```

A `@RequestMapping("/api/todos")` az osztály szintjén a közös prefix (mint a Laravel
`Route::prefix(...)->group(...)`), a metódusokon lévő `@GetMapping`/`@PostMapping`/stb. pedig a
HTTP metódus + alútvonal együtt.

A `@RestController` = `@Controller` + `@ResponseBody` — ez utóbbi azt jelenti, hogy minden
metódus visszatérési értéke **automatikusan JSON-ná szerializálódik** a válasz törzsébe, nincs
szükség explicit `response()->json(...)` hívásra, mint Laravel-ben.

## Paraméterek: `@PathVariable`, `@RequestParam`, `@RequestBody`

| Annotáció | Mire való | Laravel megfelelő |
|---|---|---|
| `@PathVariable` | URL útvonal-szegmens (`/todos/{id}`) | route paraméter automatikus injektálása (`function show(Todo $todo)` route-model binding, vagy `$id` explicit) |
| `@RequestParam` | query string paraméter (`?done=true`) | `$request->query('done')` / `$request->input('done')` |
| `@RequestBody` | a kérés törzse (tipikusan JSON), deszerializálva egy objektumba | `$request->validated()` / Form Request osztály |

```java
@GetMapping
public List<Todo> index(@RequestParam(required = false) Boolean done) {
    // GET /api/todos?done=true
    if (done != null) {
        return todoService.findByDone(done);
    }
    return todoService.findAll();
}

@PostMapping
public Todo store(@RequestBody @Valid TodoCreateRequest request) {
    // a JSON kérés törzse automatikusan Todo­CreateRequest objektummá alakul
    return todoService.create(request);
}
```

A `@Valid` a `@RequestBody` mellett automatikusan lefuttatja a Bean Validation szabályokat
(`@NotBlank`, `@Size` stb.) a beérkező objektumon — ez a Laravel Form Request `rules()`
metódusának megfelelője, bővebben a [3. fázis: Validáció](../03-validacio-hibakezeles.md)
részben.

## `ResponseEntity<T>` — explicit HTTP válasz vezérlés

Amikor nem elég az automatikus JSON-szerializáció, és explicit szeretnéd megadni a HTTP
státuszkódot vagy fejléceket, `ResponseEntity<T>`-t adsz vissza:

```php
// Laravel
public function store(TodoStoreRequest $request)
{
    $todo = Todo::create($request->validated());
    return response()->json($todo, 201);
}

public function show(int $id)
{
    $todo = Todo::find($id);
    if (!$todo) {
        return response()->json(['message' => 'Not found'], 404);
    }
    return response()->json($todo);
}
```

```java
// Spring
@PostMapping
public ResponseEntity<Todo> store(@RequestBody @Valid TodoCreateRequest request) {
    Todo created = todoService.create(request);
    return ResponseEntity.status(HttpStatus.CREATED).body(created);   // 201
}

@GetMapping("/{id}")
public ResponseEntity<Todo> show(@PathVariable Long id) {
    return todoService.findById(id)
            .map(ResponseEntity::ok)                    // 200, ha van találat
            .orElse(ResponseEntity.notFound().build());  // 404, ha nincs
}
```

Figyeld meg az `Optional<Todo>` (lásd [0.5](../00-java-alapok/05-optional.md)) és a
`ResponseEntity` láncolását `.map()`-pel — ez ugyanaz a funkcionális stílus, mint a
[Streamek és lambdák](../00-java-alapok/06-streamek-lambdak.md) fejezetben, csak itt egy HTTP
válasz felépítésére alkalmazva.

Ha a metódus egyszerűen csak az objektumot adja vissza (`Todo` a `ResponseEntity<Todo>` helyett),
Spring alapértelmezetten `200 OK`-t küld sikeres lefutás esetén, és a kivételkezelésről a
[3. fázis: Validáció, hibakezelés](../03-validacio-hibakezeles.md) részben tanult
`@ControllerAdvice` gondoskodik hiba esetén — egyelőre, ebben a fázisban, elég az explicit
`ResponseEntity` mintát ismerni.

## Teljes állapotmentesség (statelessness)

Fontos elvi különbség, amire érdemes már itt felkészülni: egy tipikus Spring REST API
**stateless** — nincs Laravel-szerű session-alapú `web` middleware csoport, nincs CSRF-token a
API végpontokon (ellentétben a Laravel session-alapú webes route-jaival). Minden kérés önmagában
hordozza a szükséges információt (pl. később a JWT tokent az `Authorization` fejlécben) — erről
részletesen a [4. fázis: Spring Security](../04-spring-security.md) részben lesz szó.

---

[← Előző: 02. Dependency Injection](02-dependency-injection.md) · [Fázis index](../01-spring-boot-alapok.md) · [Főoldal](../../README.md) · Következő: [04. Konfiguráció →](04-konfiguracio.md)
