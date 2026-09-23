[← Előző: 03. Globális hibakezelés](03-globalis-hibakezeles.md) · [Fázis index](../03-validacio-hibakezeles.md) · [Főoldal](../../README.md) · Következő: [05. Gyakorlat: refaktorálás →](05-gyakorlat-refaktoralas.md)

# 3.4 — Réteges architektúra

## A Laravel "fat controller"/"fat model" csapda

Sok Laravel projektben (főleg kisebb, gyorsan növő kódbázisokban) az üzleti logika két helyre
szivárog szét: a controllerbe (mert "gyors odaírni") vagy a modellbe (mert "az Eloquent modell úgyis
kéznél van"). Ez működik egy darabig, de idővel nehezen tesztelhető, nehezen újrahasználható kódhoz
vezet:

```php
// PHP — tipikus "fat controller"
class TodoController extends Controller
{
    public function store(Request $request)
    {
        $validated = $request->validate([...]);

        $todo = new Todo();
        $todo->title = $validated['title'];
        $todo->user_id = auth()->id();
        $todo->save();

        if ($todo->priority > 3) {
            Mail::to(auth()->user())->send(new HighPriorityTodoMail($todo));
        }

        return new TodoResource($todo);
    }
}
```

A jobb Laravel gyakorlat itt is bevezet egy Service/Action réteget — ez ugyanaz az elv, amit a
Spring alapból kikényszerít.

## A három réteg Spring-ben

```
Controller  →  Service  →  Repository
(HTTP)         (üzleti logika)  (adatelérés)
```

- **Controller**: kizárólag HTTP be-/kimenetet kezel — kérés fogadása, DTO validálása, service
  hívása, válasz összeállítása. **Nincs benne üzleti logika.**
- **Service**: az üzleti szabályok, tranzakciókezelés (`@Transactional`), a döntések helye
  ("ha priority > 3, küldj emailt").
- **Repository**: kizárólag adatelérés (`JpaRepository`), nem tud semmit a HTTP rétegről vagy az
  üzleti szabályokról.

```java
@RestController
@RequestMapping("/api/todos")
public class TodoController {
    private final TodoService todoService;

    public TodoController(TodoService todoService) {   // konstruktor injection
        this.todoService = todoService;
    }

    @PostMapping
    public ResponseEntity<TodoResponse> create(@Valid @RequestBody TodoRequest request) {
        TodoResponse response = todoService.create(request);
        return ResponseEntity.status(HttpStatus.CREATED).body(response);
    }
}

@Service
public class TodoService {
    private final TodoRepository todoRepository;
    private final MailService mailService;

    public TodoService(TodoRepository todoRepository, MailService mailService) {
        this.todoRepository = todoRepository;
        this.mailService = mailService;
    }

    @Transactional
    public TodoResponse create(TodoRequest request) {
        Todo todo = TodoMapper.toEntity(request);
        Todo saved = todoRepository.save(todo);

        if (saved.getPriority() > 3) {
            mailService.sendHighPriorityNotification(saved);
        }

        return TodoMapper.toResponse(saved);
    }
}
```

A controller most **egyetlen dolgot csinál**: fogadja a kérést, hívja a service-t, visszaadja a
választ. Minden döntés (mit jelent egy "magas prioritású" todo, mikor küldünk emailt) a
service rétegben van — ez az, amit teszteléskor ki tudsz cserélni/mockolni anélkül, hogy egy HTTP
kérést kellene szimulálnod (lásd az [5. fázis: Tesztelés](../05-teszteles.md) részt).

## Miért kényszerítettebb ez Spring-ben, mint Laravel-ben

Laravel-ben technikailag semmi nem állítja meg, hogy a controller közvetlenül az Eloquent
modellel dolgozzon — a keretrendszer megengedi, sőt a legtöbb tutorial ezt mutatja be
egyszerűségért cserébe. Spring-ben a `JpaRepository` interfészek és a Dependency Injection
mechanizmus miatt már a projekt felállításakor természetes lesz külön Service osztályokat írni —
ez nem szigorúbb szabály, hanem a keretrendszer **konvenciója és a közösségi gyakorlat**
egyaránt ebbe az irányba tereli a kódot.

**Fontos**: ez nem jelenti azt, hogy Laravel-ben rossz gyakorlat lenne Service osztályokat
bevezetni — pont ellenkezőleg, sok érett Laravel csapat pontosan ugyanezt a hármas felosztást
(Controller → Service/Action → Eloquent) használja. A különbség csak annyi, hogy Spring-ben ez
szinte alapértelmezett, Laravel-ben tudatos döntés.

---

[← Előző: 03. Globális hibakezelés](03-globalis-hibakezeles.md) · [Fázis index](../03-validacio-hibakezeles.md) · [Főoldal](../../README.md) · Következő: [05. Gyakorlat: refaktorálás →](05-gyakorlat-refaktoralas.md)
