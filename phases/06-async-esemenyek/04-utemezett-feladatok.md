[← Előző: 03. Üzenetsorok](03-uzenetsorok.md) · [Fázis index](../06-async-esemenyek.md) · [Főoldal](../../README.md) · Következő: [05. Záró gyakorlat →](05-gyakorlat-esemeny-es-queue.md)

# 6.4 — Ütemezett feladatok `@Scheduled`-del

## Laravel Task Scheduling

```php
// app/Console/Kernel.php
protected function schedule(Schedule $schedule): void
{
    $schedule->command('todos:cleanup-completed')->daily();
    $schedule->call(fn () => Todo::purgeOld())->weekly();
}
```

Ehhez Laravel-ben egyetlen crontab bejegyzés kell a szerveren:

```bash
* * * * * cd /path && php artisan schedule:run >> /dev/null 2>&1
```

Minden percben lefut a `schedule:run`, ami eldönti, melyik ütemezett feladatnak van itt az ideje.
Ez egy központi hely (`Kernel.php`), ahol **minden** ütemezett feladat definiálva van.

## `@Scheduled` Spring-ben

Spring-ben nincs központi "Kernel" osztály — az ütemezés közvetlenül a metóduson van
deklarálva, bárhol a kódbázisban:

```java
@Configuration
@EnableScheduling
public class SchedulingConfig {
}
```

```java
@Component
public class TodoCleanupTask {

    private final TodoRepository todoRepository;

    public TodoCleanupTask(TodoRepository todoRepository) {
        this.todoRepository = todoRepository;
    }

    @Scheduled(cron = "0 0 3 * * *")   // minden nap hajnali 3-kor
    public void cleanupCompletedTodos() {
        int deleted = todoRepository.deleteByDoneTrueAndUpdatedAtBefore(
            LocalDateTime.now().minusDays(30));
        System.out.println("Törölt lezárt todo-k: " + deleted);
    }

    @Scheduled(fixedRate = 60000)   // 60 másodpercenként, az előző futás KEZDETÉHEZ képest
    public void logQueueDepth() {
        System.out.println("Aktív todo-k száma: " + todoRepository.countByDoneFalse());
    }

    @Scheduled(fixedDelay = 60000)  // 60 másodperccel az előző futás VÉGE után
    public void slowHealthCheck() {
        // ...
    }
}
```

A cron kifejezés szintaxisa hasonló a Unix crontabhoz, de **6 mezős** (a másodperc az első
mező), nem 5 mezős, mint amit Laravel `->cron('* * * * *')` hívásnál megszoksz — ügyelj erre,
mert ez az egyik leggyakoribb elgépelési hiba.

## `fixedRate` vs `fixedDelay` — nincs pontos Laravel megfelelő

Ez a megkülönböztetés finomabb, mint amit Laravel Scheduling-ben megszoktál:

- **`fixedRate`**: az új futás az *előző futás kezdetéhez* képest indul a megadott időköz múlva
  — ha a feladat tovább tart, mint az időköz, a következő futás **azonnal** indul, amint az előző
  végzett (nincs átfedés, de nincs is "pihenő" a futások között)
- **`fixedDelay`**: az új futás az *előző futás végéhez* képest indul a megadott időköz múlva —
  mindig van garantált szünet a futások között

Laravel `->everyMinute()`, `->hourly()` stb. mögöttes viselkedése ehhez képest egyszerűbb, mert
ott a `schedule:run` percenkénti trigger dönti el, hogy "itt az idő"-e — nincs "előző futás még
fut-e" dilemma explicit módon kezelve, hacsak nem használod a `->withoutOverlapping()` módosítót.

## Miért nincs szükség központi crontab bejegyzésre

A legfontosabb strukturális különbség: a Spring Boot alkalmazás **saját maga a folyamat**, ami
folyamatosan fut (nem PHP-FPM worker-ek, amik kérésenként indulnak és állnak le) — ezért a JVM-en
belül futó ütemező szál magától tudja, mikor van itt egy feladat ideje, nincs szükség egy külső
cron démonra, ami időnként "megbökné" az alkalmazást. Ez közelebb áll ahhoz, mintha a Laravel
Horizon/queue worker folyamatosan futna a háttérben — csak itt ez az egész webalkalmazás
folyamatára is igaz, nem csak a queue workerre.

---

[← Előző: 03. Üzenetsorok](03-uzenetsorok.md) · [Fázis index](../06-async-esemenyek.md) · [Főoldal](../../README.md) · Következő: [05. Záró gyakorlat →](05-gyakorlat-esemeny-es-queue.md)
