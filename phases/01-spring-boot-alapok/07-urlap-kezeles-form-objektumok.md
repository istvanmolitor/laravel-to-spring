[← Előző: 06. Kiegészítés: Thymeleaf](06-sablon-renderesztes-thymeleaf.md) · [Fázis index](../01-spring-boot-alapok.md) · [Főoldal](../../README.md) · Következő: [08. Kiegészítés: terminál parancsok →](08-terminal-parancsok-cli.md)

# 1.7 — Kiegészítés: űrlap kezelés és form objektumok

**Ez a fejezet is opcionális**, és a [1.6 — Thymeleaf](06-sablon-renderesztes-thymeleaf.md) fejezetre
épül: ott a `th:object`/`th:field` szintaxist láttad, itt a mögötte lévő teljes form-feldolgozási
folyamatot bontjuk ki — validációs hibák megjelenítése, redirect-after-post minta, flash üzenetek,
fájlfeltöltés. Az utolsó rész (fájlfeltöltés) **REST API-ból is releváns**, nem csak szerver oldali
renderelésnél.

## Form-backing object: nem ugyanaz, mint a DTO

A [3.1 — Bean Validation](../03-validacio-hibakezeles/01-bean-validation.md) fejezetben (később)
DTO-kat fogsz validálni JSON body-n. Szerver oldali formnál hasonló elv, más mechanika: a
form mezői egy Java osztály (nem feltétlen `record`, mert a Spring bindernek üres konstruktor +
setterek kellenek a form-adatok visszaírásához) mezőire kötődnek automatikusan.

```php
// Laravel — StoreTodoRequest.php (Form Request)
class StoreTodoRequest extends FormRequest
{
    public function rules(): array
    {
        return [
            'title' => 'required|min:3|max:255',
            'dueDate' => 'nullable|date',
        ];
    }
}
```

```java
// Spring — TodoForm.java
public class TodoForm {

    @NotBlank
    @Size(min = 3, max = 255)
    private String title;

    private LocalDate dueDate;

    // getterek/setterek kellenek — a Spring binder ezeken keresztül tölti fel a mezőket
    public String getTitle() { return title; }
    public void setTitle(String title) { this.title = title; }
    public LocalDate getDueDate() { return dueDate; }
    public void setDueDate(LocalDate dueDate) { this.dueDate = dueDate; }
}
```

## `@ModelAttribute` + `@Valid` + `BindingResult`

A legfontosabb fogalmi különbség a Laravel Form Requesthez képest: Laravel-ben a validáció
**automatikusan** 422-t vagy redirectet dob hiba esetén, mielőtt a controller metódusod törzse
lefutna. Spring-ben ez **explicit** — te ellenőrzöd a `BindingResult`-ot, és te döntesz, mi történjen:

```php
// Laravel — a controller metódus törzse csak akkor fut le, ha a validáció átment
class TodoWebController extends Controller
{
    public function store(StoreTodoRequest $request)
    {
        Todo::create($request->validated());
        return redirect()->route('todos.index')->with('success', 'Létrehozva!');
    }
}
```

```java
// Spring — a validáció eredményét neked kell lekérdezned
@Controller
@RequestMapping("/todos")
public class TodoViewController {

    private final TodoService todoService;

    public TodoViewController(TodoService todoService) {
        this.todoService = todoService;
    }

    @PostMapping
    public String store(@Valid @ModelAttribute("todoForm") TodoForm form,
                         BindingResult bindingResult,
                         RedirectAttributes redirectAttributes) {
        if (bindingResult.hasErrors()) {
            // nincs redirect — ugyanazt a formot rendereljük újra, a hibákkal együtt
            return "todos/create";
        }
        todoService.create(form.getTitle(), form.getDueDate());
        redirectAttributes.addFlashAttribute("successMessage", "Létrehozva!");
        return "redirect:/todos";
    }

    @GetMapping("/new")
    public String createForm(Model model) {
        model.addAttribute("todoForm", new TodoForm());
        return "todos/create";
    }
}
```

Kulcsfontosságú részletek:

- A `BindingResult`-nak **közvetlenül** a `@Valid`-dal ellátott paraméter után kell következnie a
  metódus szignatúrájában — ha máshova teszed, Spring futásidőben `IllegalStateException`-t dob.
- Hiba esetén **nem** redirectelsz, hanem ugyanazt a nézetnevet adod vissza — ez azért fontos, mert
  a `th:errors` (lásd [1.6](06-sablon-renderesztes-thymeleaf.md)) csak akkor tud hibát megjeleníteni,
  ha a `BindingResult` a modellben marad, amit a Spring automatikusan betesz, ha a nézetnév
  ugyanaz marad, mint amiből a form jött.
- A `@ModelAttribute("todoForm")` név egyeznie kell a `th:object="${todoForm}"` névvel a sablonban.

## Redirect-after-post (PRG minta) és flash üzenetek

Sikeres mentés után **mindig redirectelj**, sose renderelj view-t közvetlenül POST után — ez a
Post/Redirect/Get minta, ami megvéd a duplikált submit-tól (F5 újratöltésnél a böngésző a redirect
utáni GET-et ismétli, nem a POST-ot). Ez ugyanaz a minta, mint amit Laravel-ben megszoktál, csak
ott a `redirect()->with(...)` egy lépésben csinálja.

| Laravel | Spring MVC |
|---|---|
| `redirect()->route('todos.index')` | `return "redirect:/todos";` |
| `->with('success', 'Létrehozva!')` | `redirectAttributes.addFlashAttribute("successMessage", "Létrehozva!")` |
| `session()->flash(...)` (implicit, egy kérésig él) | flash attribútum (implicit, egy kérésig él, a session-ön keresztül) |
| `old('title')` sikertelen validáció után | nem kell — a `BindingResult` hibás esetén nincs redirect, a form objektum már benne van a modellben a beírt értékekkel |

Fontos eltérés: Laravel-ben az `old()` azért kell, mert a validációs hiba **redirectet** vált ki
(GET-re), és a bevitt adatokat külön session flash-ből kell visszaolvasni. Spring-ben, mivel
hibánál **nincs redirect**, hanem közvetlen re-render, a `TodoForm` objektum már tartalmazza a
felhasználó által beírt (érvénytelen) adatokat — nincs szükség egy `old()`-hoz hasonló mechanizmusra.

## Fájlfeltöltés — ez REST API-ból is kell

```php
// Laravel
public function store(Request $request)
{
    $path = $request->file('avatar')->store('avatars', 'public');
    $user->update(['avatar_path' => $path]);
}
```

```java
// Spring — akár @Controller (form), akár @RestController (API) metódusban ugyanúgy működik
@PostMapping("/avatar")
public ResponseEntity<Void> uploadAvatar(@RequestParam("avatar") MultipartFile avatar) throws IOException {
    String filename = UUID.randomUUID() + "-" + avatar.getOriginalFilename();
    Path target = Path.of("uploads", filename);
    Files.copy(avatar.getInputStream(), target, StandardCopyOption.REPLACE_EXISTING);
    return ResponseEntity.ok().build();
}
```

A `MultipartFile` paraméter automatikusan bekötődik, ha a kérés `Content-Type` fejléce
`multipart/form-data` — sem `@RestController`, sem `@Controller` esetén nincs különbség ebben.
A méretkorlátot `application.yml`-ben állítod (a Laravel `upload_max_filesize`/`post_max_size`
php.ini beállítások megfelelője):

```yaml
spring:
  servlet:
    multipart:
      max-file-size: 10MB
      max-request-size: 10MB
```

HTML form oldalon a `<form>` tagnek `enctype="multipart/form-data"`-t kell kapnia, pontosan úgy,
mint Laravel-ben — ez böngésző-szintű követelmény, nem keretrendszer-specifikus.

## Mikor tényleg kell ez neked

A `BindingResult` + redirect-after-post minta csak akkor kerül elő, ha a
[1.6-ban](06-sablon-renderesztes-thymeleaf.md) leírt szerver oldali rendereléshez nyúlsz (admin
felület, belső eszköz). A fájlfeltöltés (`MultipartFile`) viszont a záró projektben
([8. fázis](../08-projekt.md)) is előjöhet, ha REST API-n keresztül kell képet/dokumentumot
fogadnod — azt már most érdemes megjegyezni.

---

[← Előző: 06. Kiegészítés: Thymeleaf](06-sablon-renderesztes-thymeleaf.md) · [Fázis index](../01-spring-boot-alapok.md) · [Főoldal](../../README.md) · Következő: [08. Kiegészítés: terminál parancsok →](08-terminal-parancsok-cli.md)
