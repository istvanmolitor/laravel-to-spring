[← Fázis index](../01-spring-boot-alapok.md) · [Főoldal](../../README.md) · Következő: [02. Dependency Injection →](02-dependency-injection.md)

# 1.1 — Projekt indítása Spring Boot-tal

## Spring Initializr — a `composer create-project` megfelelője

Laravel-ben egy `composer create-project laravel/laravel app` paranccsal indulsz, ami lehúz egy
kész vázat. Spring-ben ugyanezt egy webes generátor, a **Spring Initializr**
(https://start.spring.io) végzi el — kiválasztod a build eszközt, a Java verziót és a
függőségeket, letöltesz egy `.zip`-et, és kicsomagolva már fut is.

VS Code-ban ez a **Spring Initializr Java Support** kiegészítővel (`vscjava.vscode-spring-initializr`,
a Java Extension Pack-nek is része) még kényelmesebb: `Ctrl+Shift+P` →
`Spring Initializr: Generate a Maven Project` — ugyanaz az űrlap végigkérdezve a paletta menüből,
a generált projekt automatikusan megnyílik.

### Ajánlott beállítások a kezdő projekthez

| Mező | Érték |
|---|---|
| Project | Maven |
| Language | Java |
| Spring Boot | legfrissebb stabil 3.x |
| Packaging | Jar |
| Java | 21 |

### Ajánlott függőségek (dependencies)

| Dependency | Mire kell | Laravel megfelelő |
|---|---|---|
| **Spring Web** | REST/MVC réteg, beépített Tomcat | maga a Laravel HTTP kernel |
| **Spring Data JPA** | ORM réteg (Hibernate) | Eloquent |
| **H2 Database** | in-memory adatbázis fejlesztéshez/teszthez | SQLite in-memory teszthez |
| **PostgreSQL Driver** | valódi DB driver | `pdo_pgsql` |
| **Validation** | Bean Validation (`@Valid`, `@NotNull` stb.) | Form Request szabályok |
| **Lombok** | boilerplate (getter/setter/konstruktor) generálás fordítási időben | nincs pontos megfelelő — leginkább a PHP 8 constructor promotion kényelmét pótolja |

## Mi generálódik — projekt anatómia

```
todo-api/
├── pom.xml                                    # függőségek, ~ composer.json
├── mvnw, mvnw.cmd                              # Maven wrapper — nem kell rendszerszinten Maven
├── src/
│   ├── main/
│   │   ├── java/hu/molitor/todoapi/
│   │   │   └── TodoApiApplication.java         # belépési pont, ~ public/index.php
│   │   └── resources/
│   │       ├── application.properties          # ~ .env + config/*.php összevonva
│   │       └── static/, templates/              # statikus fájlok / szerveroldali template-ek
│   └── test/
│       └── java/hu/molitor/todoapi/
│           └── TodoApiApplicationTests.java     # alap kontextus-betöltési teszt
```

A `mvnw`/`mvnw.cmd` a **Maven Wrapper** — ez biztosítja, hogy mindenki, aki klónozza a repót,
ugyanazt a Maven verziót használja rendszerszintű telepítés nélkül. Gondolj rá úgy, mint a Laravel
Sail/Docker megoldására a "nálam működik" probléma ellen, csak itt magára a build eszközre
vonatkozik, nem a teljes futtatókörnyezetre.

## `@SpringBootApplication` — a belépési pont

```java
package hu.molitor.todoapi;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class TodoApiApplication {
    public static void main(String[] args) {
        SpringApplication.run(TodoApiApplication.class, args);
    }
}
```

A `@SpringBootApplication` valójában három annotáció összevonása:

| Rész-annotáció | Mit csinál | Laravel megfelelő |
|---|---|---|
| `@Configuration` | jelzi, hogy ez az osztály maga is tartalmazhat bean-definíciókat | `AppServiceProvider` |
| `@EnableAutoConfiguration` | a classpath-on lévő könyvtárak alapján automatikusan beállít dolgokat (pl. ha lát H2-t, konfigurál egy DataSource-t) | Laravel package auto-discovery (`extra.laravel.providers` a `composer.json`-ban) |
| `@ComponentScan` | bejárja a csomagot (és alcsomagjait) a `@Component`/`@Service`/`@Controller` stb. annotált osztályokért, és regisztrálja őket a DI konténerbe | route/controller/provider automatikus felfedezése induláskor |

A `main` metódus egyetlen sora (`SpringApplication.run(...)`) indítja el a teljes alkalmazást:
beolvassa a konfigurációt, felépíti a DI konténert, elindítja a beágyazott webszervert.

## Beágyazott Tomcat vs. `php artisan serve` / nginx+php-fpm

Ez fontos architekturális különbség. Laravel-ben (és PHP-ban általában) a webszerver
(nginx/Apache + php-fpm, vagy fejlesztésben `php artisan serve`) **külön folyamat**, ami minden
kérésnél újra betölti/kiszolgálja az alkalmazást. Spring Boot-nál a webszerver
(alapértelmezetten **Tomcat**) **be van ágyazva magába az alkalmazásba** — a `.jar` fájl, amit a
build készít, egyszerre tartalmazza a te kódodat ÉS a webszervert. Nincs külön nginx/php-fpm
konfiguráció; egy `java -jar app.jar` paranccsal elindul egy teljes, magában álló webszerver.

```
Laravel:  nginx (folyamat) → php-fpm (worker pool) → betölti a Laravel bootstrap-et minden kérésnél
Spring:   java -jar app.jar → egyetlen JVM folyamat, a Tomcat szál-pool-lal fut BENNE
```

Ennek gyakorlati következménye: a Spring alkalmazás **hosszú életű folyamat**, az induláskor
egyszer felépül a DI konténer és marad a memóriában — nincs "minden kérésnél újra bootstrap",
mint PHP-nál (kivéve, ha OPcache/preloading-gal optimalizáltad). Ez az egyik oka, hogy Spring
alkalmazások indítása lassabb (néhány másodperc), cserébe a kiszolgálás gyorsabb, mert nem kell
újraépíteni semmit kérésenként.

## Fejlesztői ciklus: `./mvnw spring-boot:run`

```bash
./mvnw spring-boot:run
```

Ez elindítja az alkalmazást fejlesztői módban, alapértelmezetten a `8080`-as porton — a
`php artisan serve` közvetlen megfelelője.

### Spring Boot DevTools — hot reload

Add hozzá a `spring-boot-devtools` függőséget, és a legtöbb kódváltoztatás (controller, service)
automatikusan újratölti az alkalmazás-kontextust fájlmentéskor, anélkül hogy kézzel kellene
leállítanod/újraindítanod a szervert:

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-devtools</artifactId>
    <scope>runtime</scope>
    <optional>true</optional>
</dependency>
```

Ez közelíti azt az élményt, amit a `php artisan serve` natívan ad (mivel PHP amúgy is minden
kérésnél újraolvassa a fájlokat) — de fontos különbség, hogy itt egy **újraindítás** (restart)
történik a JVM-en belül, nem valódi hot-reload: gyorsabb, mint egy teljes JVM-indítás, de nem
azonnali, mint a PHP fájl-alapú kiszolgálása. Frontend Vite HMR-hez (ha van SPA a projektben) ennek
nincs köze — az továbbra is külön, saját dev szerverén fut, ahogy Laravel + Vite esetén is.

---

[← Fázis index](../01-spring-boot-alapok.md) · [Főoldal](../../README.md) · Következő: [02. Dependency Injection →](02-dependency-injection.md)
