[← Előző: 05. Gyakorlat: Todo API](05-gyakorlat-todo-api.md) · [Fázis index](../01-spring-boot-alapok.md) · [Főoldal](../../README.md) · Következő fázis: [02. Adatréteg: Spring Data JPA →](../02-adatreteg-jpa.md)

# 1.6 — Kiegészítés: szerver oldali renderelés Thymeleaf-fel (Blade megfelelő)

**Ez a fejezet opcionális.** A teljes roadmap REST API-ra épül (a frontend külön SPA/mobil
kliens), ahol nincs szükség szerver oldali HTML renderelésre — a `TodoController` mindenhol
`@RestController`, ami JSON-t ad vissza. Ha viszont valaha egy klasszikus, szerver által
renderelt Spring MVC alkalmazást kell írnod (belső admin felület, egyszerű CRUD oldal,
e-mail sablon), a **Thymeleaf** a Spring világ Blade-je. Érdemes legalább egyszer végigcsinálni
ezt a fejezetet, hogy felismerd a mintát, ha találkozol vele.

## Alapfilozófia: natural templating vs. compile-to-PHP

A Blade fájlok (`.blade.php`) **fordítási lépésben** sima PHP-vá alakulnak (`storage/framework/views`
alá cache-elve) — a `.blade.php` fájl önmagában böngészőben megnyitva nem egy értelmes HTML,
mert a `{{ $todo->title }}` és `@foreach` direktívák nem HTML-szintaxis.

A Thymeleaf ezzel szemben **natural templating**: egy `.html` sablon önmagában is érvényes,
megnyitható HTML, a dinamikus részek plusz **attribútumokként** (`th:*`) épülnek be a normál HTML
tagekbe. Tervező/designer meg tudja nyitni böngészőben statikus adatokkal, és ugyanaz a fájl fut
élesben is dinamikus adatokkal.

```html
<!-- Blade — todos/show.blade.php — csak sablon-motorral értelmezhető -->
<h1>{{ $todo->title }}</h1>
<p class="@if($todo->done) done @endif">{{ $todo->description }}</p>
```

```html
<!-- Thymeleaf — todos/show.html — önmagában is érvényes HTML -->
<h1 th:text="${todo.title}">Placeholder cím</h1>
<p th:class="${todo.done} ? 'done'" th:text="${todo.description}">Placeholder leírás</p>
```

A `th:text` felülírja a tag statikus tartalmát futásidőben — fejlesztéskor a `Placeholder cím`
látszik böngészőben nyitva, élesben a valódi `todo.title` érték.

## Függőség és a "route" jellegű felállás

```xml
<!-- pom.xml -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-thymeleaf</artifactId>
</dependency>
```

A sablonok konvenció szerint `src/main/resources/templates/` alá kerülnek, `.html` kiterjesztéssel
— ez a Laravel `resources/views/` megfelelője. A statikus fájlok (CSS, JS, kép) a
`src/main/resources/static/` alá, ahogy Laravel-ben a `public/`.

A legfontosabb különbség a REST controllerekhez képest ([1.3](03-rest-controllerek.md)): itt
**nem** `@RestController`-t használsz, hanem sima `@Controller`-t, és a metódus **nem** az adatot,
hanem a **nézet nevét** (`String`) adja vissza — ez pontosan a Laravel controller
`return view(...)` mintája.

```php
// Laravel — TodoWebController.php
class TodoWebController extends Controller
{
    public function index()
    {
        $todos = Todo::all();
        return view('todos.index', ['todos' => $todos]);
    }

    public function show(int $id)
    {
        $todo = Todo::findOrFail($id);
        return view('todos.show', compact('todo'));
    }
}
```

```java
// Spring — TodoViewController.java
@Controller                              // nem @RestController!
@RequestMapping("/todos")
public class TodoViewController {

    private final TodoService todoService;

    public TodoViewController(TodoService todoService) {
        this.todoService = todoService;
    }

    @GetMapping
    public String index(Model model) {
        model.addAttribute("todos", todoService.findAll());
        return "todos/index";            // resources/templates/todos/index.html
    }

    @GetMapping("/{id}")
    public String show(@PathVariable Long id, Model model) {
        model.addAttribute("todo", todoService.findById(id).orElseThrow());
        return "todos/show";             // resources/templates/todos/show.html
    }
}
```

| Laravel | Spring + Thymeleaf |
|---|---|
| `return view('todos.index', ['todos' => $todos])` | `model.addAttribute("todos", todos); return "todos/index";` |
| `compact('todo')` | `model.addAttribute("todo", todo)` |
| pontnotáció: `todos.index` → `resources/views/todos/index.blade.php` | perjeles útvonal: `"todos/index"` → `resources/templates/todos/index.html` |
| `@RestController` vs. sima `Controller` (implicit, a `response()->json()` hívás dönt) | **explicit** különbség: `@RestController` (JSON) vs. `@Controller` + `String` visszatérés (nézet) |

Fontos csapda: ha egy `@Controller`-en belüli metódus JSON-t akarna visszaadni (pl. egy AJAX
végpont ugyanabban az osztályban), azt a metódust külön `@ResponseBody`-val kell megjelölni —
enélkül Spring **nézetnévnek** próbálja értelmezni a visszatérési stringet, és `TemplateInputException`-t
dobsz, mert nem talál ilyen nevű sablont.

## Kifejezések és direktívák — az igazi 1:1 fordítótábla

| Feladat | Blade | Thymeleaf |
|---|---|---|
| Szöveg kiíratás (escapelt) | `{{ $todo->title }}` | `th:text="${todo.title}"` |
| Szöveg kiíratás (nyers HTML, XSS-veszélyes) | `{!! $todo->description !!}` | `th:utext="${todo.description}"` |
| Feltétel | `@if($todo->done) ... @endif` | `th:if="${todo.done}"` |
| Feltétel ellentettje | `@unless($todo->done) ... @endunless` | `th:unless="${todo.done}"` |
| Ciklus | `@foreach($todos as $todo) ... @endforeach` | `th:each="todo : ${todos}"` |
| Attribútum feltételes beállítása | `class="{{ $todo->done ? 'done' : '' }}"` | `th:class="${todo.done} ? 'done' : ''"` |
| Link/URL generálás | `<a href="{{ route('todos.show', $todo->id) }}">` | `<a th:href="@{/todos/{id}(id=${todo.id})}">` |
| Formázott adat (dátum, szám) | Carbon `->format(...)` / `number_format()` | `${#temporals.format(todo.createdAt, 'yyyy-MM-dd')}` |
| Változó definiálása sablonon belül | `@php $x = 1; @endphp` (ritkán ajánlott) | `th:with="x=1"` |
| Fordítás/i18n | `__('messages.welcome')` / `@lang(...)` | `th:text="#{messages.welcome}"` |

```php
<!-- Blade — todos/index.blade.php -->
<ul>
    @foreach($todos as $todo)
        <li class="{{ $todo->done ? 'done' : '' }}">
            <a href="{{ route('todos.show', $todo->id) }}">{{ $todo->title }}</a>
            @if($todo->done)
                <span>✓ kész</span>
            @endif
        </li>
    @endforeach
</ul>
```

```html
<!-- Thymeleaf — todos/index.html -->
<ul>
    <li th:each="todo : ${todos}" th:class="${todo.done} ? 'done'">
        <a th:href="@{/todos/{id}(id=${todo.id})}" th:text="${todo.title}">Cím</a>
        <span th:if="${todo.done}">✓ kész</span>
    </li>
</ul>
```

Figyeld meg: a Thymeleaf a `todo.title`-t **getter-hívásként** oldja fel (`todo.getTitle()`) —
ugyanaz a rejtett konvenció, mint amikor Laravel-ben `$todo->title` mögött egy Eloquent
accessor vagy egyszerű property van, csak itt kötelező, mert Java-ban nincsenek publikus
property-k, mindig getter/setter van (lásd [0.3 — OOP alapok](../00-java-alapok/03-oop-alapok.md)).

## Layout és részletek: `@extends`/`@section`/`@include` → fragmentek

A Blade öröklődéses layout-modellt használ (`@extends`, `@section`, `@yield`), a Thymeleaf pedig
**fragment-alapú kompozíciót** (`th:insert`/`th:replace` egy névvel azonosított darabra hivatkozva)
— ez koncepcionálisan közelebb áll a Blade `@include`-hoz és a Laravel Components-hez, mint a
`@extends`-hez.

```php
<!-- Blade — layouts/app.blade.php -->
<html>
<head><title>@yield('title')</title></head>
<body>
    @include('partials.nav')
    <main>@yield('content')</main>
</body>
</html>

<!-- Blade — todos/index.blade.php -->
@extends('layouts.app')
@section('title', 'Todo lista')
@section('content')
    <ul>...</ul>
@endsection
```

```html
<!-- Thymeleaf — layouts/base.html (a layout dialektussal, thymeleaf-layout-dialect) -->
<html xmlns:th="http://www.thymeleaf.org" xmlns:layout="http://www.ualberta.ca/layout">
<head><title layout:title-pattern="$CONTENT_TITLE - Todo App">Alap cím</title></head>
<body>
    <div th:replace="~{partials/nav :: nav}"></div>
    <main layout:fragment="content">Ide kerül az oldal tartalma</main>
</body>
</html>

<!-- Thymeleaf — todos/index.html -->
<html xmlns:th="http://www.thymeleaf.org" xmlns:layout="http://www.ualberta.ca/layout"
      layout:decorate="~{layouts/base}">
<div layout:fragment="content">
    <ul>...</ul>
</div>
</html>
```

A layout dialektus (`thymeleaf-layout-dialect`) egy **külön, de nagyon elterjedt** kiegészítő
függőség — az alap Thymeleaf csak a `th:insert`/`th:replace` fragment-mechanizmust adja natívan,
a Laravel `@extends`-hez hasonló "dekoráció" ezzel a plusz könyvtárral lesz kényelmes. Kisebb,
ismétlődő darabokra (pl. egy nav sáv, egy todo-elem kártya) elég az alap fragment-hivatkozás:

```html
<!-- partials/nav.html -->
<nav th:fragment="nav">
    <a th:href="@{/todos}">Todos</a>
    <a th:href="@{/logout}">Kijelentkezés</a>
</nav>
```

| Laravel | Thymeleaf |
|---|---|
| `@extends('layouts.app')` + `@section`/`@yield` | `layout:decorate` + `layout:fragment` (külön `thymeleaf-layout-dialect` függőséggel) |
| `@include('partials.nav')` | `th:insert="~{partials/nav :: nav}"` (beszúrja a fragmentet a hívó tag *belsejébe*) |
| — (nincs pontos megfelelő) | `th:replace="~{partials/nav :: nav}"` (a hívó tagot *lecseréli* a fragmentre) |
| Blade komponens (`<x-alert type="error">`) | Thymeleaf fragment paraméterekkel: `th:fragment="alert(type)"` |

## Form kezelés és CSRF

```php
<!-- Blade — todos/create.blade.php -->
<form method="POST" action="{{ route('todos.store') }}">
    @csrf
    <input type="text" name="title" value="{{ old('title') }}">
    @error('title')
        <span class="error">{{ $message }}</span>
    @enderror
    <button type="submit">Mentés</button>
</form>
```

```html
<!-- Thymeleaf — todos/create.html -->
<form th:action="@{/todos}" th:object="${todoForm}" method="post">
    <input type="text" th:field="*{title}">
    <span th:if="${#fields.hasErrors('title')}" th:errors="*{title}" class="error"></span>
    <button type="submit">Mentés</button>
</form>
```

Két lényeges eltérés:

- **CSRF**: Blade-ben expliciten ki kell írni a `@csrf` direktívát a formba. Spring Security
  bekapcsolt CSRF védelme mellett a Thymeleaf a `th:action`-t használó `<form>` tagbe
  **automatikusan beszúrja** a rejtett CSRF input mezőt — nincs szükség kézi direktívára,
  amíg a Spring Security Thymeleaf integráció (`thymeleaf-extras-springsecurity6`) rajta van
  a classpath-on. Erről bővebben a [4. fázis: Spring Security](../04-spring-security.md)
  részben lesz szó.
- **`th:object` + `th:field`**: ez a Spring MVC "form backing object" mintája — egy Java osztály
  (pl. `TodoForm`), amit a controller ad át a modellnek, és a `*{title}` a `todoForm.getTitle()`
  /`setTitle()` getter/setter-párra kötődik automatikusan (két irányban: kiírja az értéket, és
  visszaolvasáskor beköti a beküldött form adatot). Ez közelebb áll a Laravel Form Request +
  `old()` páros kombinált funkciójához, mint bármelyik önmagában.

## Hibaüzenetek megjelenítése

```php
<!-- Blade -->
@if(session('success'))
    <div class="alert">{{ session('success') }}</div>
@endif
```

```html
<!-- Thymeleaf — a controller model.addAttribute("successMessage", "...")-ot állít be,
     vagy RedirectAttributes.addFlashAttribute(...) redirect utáni "flash" üzenethez -->
<div class="alert" th:if="${successMessage}" th:text="${successMessage}"></div>
```

A Laravel `session()->flash()` közvetlen megfelelője Spring MVC-ben a `RedirectAttributes`
paraméter `addFlashAttribute(...)` metódusa egy `redirect:`-tel visszatérő controller
metódusban — ez egyetlen következő kérésig él a session-ben, pontosan úgy, mint a Laravel
flash session adat.

## Mikor tényleg kell ez neked

Ebben a roadmapben **nem lesz rá szükség** a Todo API-hoz vagy a záró projekthez — mindenhol
`@RestController` + JSON a cél, ahogy [1.3-ban](03-rest-controllerek.md) tanultad. A Thymeleaf
ott kerül elő, ahol Spring egyébként is szerver oldali HTML-t generál a háttérben, még ha te nem
is írsz explicit sablont hozzá:

- a Spring Boot **hiba oldalak** (`/error`) alapból egy Whitelabel HTML-t adnak vissza, amit
  Thymeleaf `error.html` sablonnal lehet felülírni
- egyszerű **admin/belső eszköz** felületek, ahol nem éri meg külön SPA-t építeni
- **e-mail sablonok** (`spring-boot-starter-mail` + Thymeleaf kombinálva), analóg a Laravel
  Mailable + Blade markdown mail sablonokkal

Ha ezekkel találkozol, a fenti fordítótábla (`th:text`/`th:if`/`th:each`/`th:field` ↔
`{{ }}`/`@if`/`@foreach`/`old()`) lefedi a Blade tudásod 90%-ának az áthelyezését.

---

[← Előző: 05. Gyakorlat: Todo API](05-gyakorlat-todo-api.md) · [Fázis index](../01-spring-boot-alapok.md) · [Főoldal](../../README.md) · Következő fázis: [02. Adatréteg: Spring Data JPA →](../02-adatreteg-jpa.md)
