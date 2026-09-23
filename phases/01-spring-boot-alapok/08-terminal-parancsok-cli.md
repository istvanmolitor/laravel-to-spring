[← Előző: 07. Kiegészítés: űrlap kezelés](07-urlap-kezeles-form-objektumok.md) · [Fázis index](../01-spring-boot-alapok.md) · [Főoldal](../../README.md) · Következő fázis: [02. Adatréteg: Spring Data JPA →](../02-adatreteg-jpa.md)

# 1.8 — Kiegészítés: terminál parancsok (Artisan-szerű CLI-k)

**Ez a fejezet is opcionális.** Laravel-ben megszoktad, hogy `php artisan make:command` egy
teljesen külön, névvel hívható parancsot generál (`php artisan todos:cleanup --older-than=30`),
amit akár manuálisan, akár a schedulerből futtatsz. Spring Bootban nincs egy az egyben ugyanez a
munkafolyamat, de több eszköz fedi le ugyanazt a szükségletet, más-más helyzetre.

## Alapfilozófia: nincs külön "artisan" belépési pont

Laravel-ben a `php artisan xyz` egy **teljesen más belépési pont**, mint a `php artisan serve`
(webszerver) — külön PHP process indul, ami csak a kért parancsot futtatja le, aztán kilép.

Spring Bootban a webszerver és a "parancssori mód" **ugyanabban a futtatható JAR-ban**, ugyanabban
a `main` metódusban van — az, hogy webszerver induljon-e, azon múlik, hogy van-e a classpath-on
webes starter (`spring-boot-starter-web`), illetve explicit konfiguráción. Nincs külön "artisan
belépési pont fájl", mint a Laravel `artisan` PHP script — a `SpringApplication.run(...)` ugyanaz
a belépési pont, ami a webszervert is indítja.

## `CommandLineRunner` / `ApplicationRunner` — induláskor lefutó kód

Ez a legegyszerűbb eset: olyan kód, aminek **egyszer, induláskor** kell lefutnia — ~ egy Laravel
`php artisan db:seed` egyszeri, kézi meghívása, vagy egy induláskor futó karbantartó script.

```php
// Laravel — egy egyszeri seed/setup script (nem igazi Artisan command, csak összehasonlításként)
// routes/console.php vagy egy dedikált Artisan command run() metódusában
Todo::factory()->count(10)->create();
```

```java
// Spring — CommandLineRunner bean, ami induláskor lefut
@Component
public class DemoDataSeeder implements CommandLineRunner {

    private final TodoRepository todoRepository;

    public DemoDataSeeder(TodoRepository todoRepository) {
        this.todoRepository = todoRepository;
    }

    @Override
    public void run(String... args) {
        if (todoRepository.count() == 0) {
            todoRepository.save(new Todo("Demo todo"));
        }
    }
}
```

A `run(String... args)` paraméterben a **parancssori argumentumokat** kapod meg nyers String
tömbként (`java -jar app.jar --seed --count=10` esetén `args = ["--seed", "--count=10"]`) — ez a
Laravel `$this->argument('name')`/`$this->option('name')` nyers, feldolgozatlan megfelelője.
Az `ApplicationRunner` ugyanez, csak egy már feldolgozott `ApplicationArguments` objektumot kapsz,
ahol `getOptionValues("count")` szerűen tudsz `--kulcs=érték` párokat kérdezni — közelebb áll a
Laravel `$this->option()`-höz, mint a nyers `CommandLineRunner`.

**Csapda**: ha a projektben van `spring-boot-starter-web`, minden `CommandLineRunner` **azután**
fut le, hogy a webszerver már elindult, és a JVM nem lép ki utána — ez tökéletes seedelésre, de
nem arra, hogy "fuss le egy dolgot, aztán állj le", mint egy klasszikus Artisan command. Ehhez
explicit ki kell léptetni a JVM-et (`System.exit(SpringApplication.exit(context))`), vagy webszerver
nélküli módban kell indítani:

```java
SpringApplication app = new SpringApplication(MyApp.class);
app.setWebApplicationType(WebApplicationType.NONE);  // ne induljon beágyazott Tomcat
app.run(args);
```

## Spring Shell — ez áll legközelebb az Artisan custom command mintához

Ha **több, névvel hívható, paraméterezhető, ismételten futtatható** parancsra van szükséged (pont,
amit `php artisan make:command` + saját `signature` ad), a **Spring Shell** a megfelelő eszköz —
nem a core Spring része, hanem külön projekt (`org.springframework.shell:spring-shell-starter`).

```php
// Laravel — app/Console/Commands/CleanupOldTodos.php
class CleanupOldTodos extends Command
{
    protected $signature = 'todos:cleanup {--older-than=30}';

    public function handle()
    {
        $days = (int) $this->option('older-than');
        $count = Todo::where('created_at', '<', now()->subDays($days))->delete();
        $this->info("{$count} régi todo törölve.");
    }
}
```

```java
// Spring Shell — TodoCommands.java
@ShellComponent
public class TodoCommands {

    private final TodoService todoService;

    public TodoCommands(TodoService todoService) {
        this.todoService = todoService;
    }

    @ShellMethod(key = "todos cleanup", value = "Régi todo-k törlése")
    public String cleanup(@ShellOption(defaultValue = "30") int olderThan) {
        int count = todoService.deleteOlderThan(olderThan);
        return count + " régi todo törölve.";
    }
}
```

Futtatva egy Spring Shell alkalmazás egy **interaktív promptot** ad (mint a Laravel `php artisan
tinker`, csak a te saját parancsaiddal is):

```
shell:> todos cleanup --older-than 7
5 régi todo törölve.
```

Vagy nem-interaktív módban, egyetlen parancsként:

```bash
java -jar app.jar todos cleanup --older-than 7
```

| Laravel Artisan | Spring |
|---|---|
| `php artisan make:command` | `@ShellComponent` + `@ShellMethod` osztály kézzel (nincs generátor, de nincs is rá szükség — kevés boilerplate) |
| `protected $signature = 'todos:cleanup {--older-than=30}'` | `@ShellMethod(key = "todos cleanup")` + metódusparaméter `@ShellOption(defaultValue = "30")`-val |
| `$this->option('older-than')` | egyszerű metódusparaméter (típusos: `int`, nem string) |
| `$this->info(...)`/`$this->error(...)` | metódus visszatérési értéke (String) jelenik meg a shell kimenetén |
| `php artisan tinker` | a Spring Shell interaktív promptja saját parancsokkal (nem general-purpose REPL, csak a definiált parancsok) |
| `php artisan list` | a Spring Shell `help` beépített parancsa |

## Ütemezett futtatás — kereszthivatkozás

Ha a cél az, hogy egy parancs **rendszeresen, magától** fusson (~ Laravel `Kernel::schedule()` +
`php artisan schedule:run` cronból), az nem ide tartozik, hanem a
[6.4 — Ütemezett feladatok](../06-async-esemenyek/04-utemezett-feladatok.md) fejezetbe
(`@Scheduled` + `@EnableScheduling`) — a `CommandLineRunner`/Spring Shell parancsokat onnan is
meg lehet hívni, ha a logikát egy külön service-be szervezed ki, amit mindkét belépési pont hív.

## Mikor tényleg kell ez neked

A Todo API záró gyakorlathoz **nem lesz rá szükség** — a REST API-t HTTP kérésekkel teszteled,
nem terminálból. A `CommandLineRunner` hasznos lehet fejlesztői demo adatok betöltésére (ahogy a
fenti `DemoDataSeeder`), a Spring Shell pedig akkor kerül elő, ha egy háttérrendszer-karbantartó
eszközkészletet (~ egy csapat belső Artisan command gyűjteménye) kell Java oldalon újraépítened.

---

[← Előző: 07. Kiegészítés: űrlap kezelés](07-urlap-kezeles-form-objektumok.md) · [Fázis index](../01-spring-boot-alapok.md) · [Főoldal](../../README.md) · Következő fázis: [02. Adatréteg: Spring Data JPA →](../02-adatreteg-jpa.md)
