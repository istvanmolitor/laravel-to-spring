[← Fázis index](../00-java-alapok.md) · [Főoldal](../../README.md) · Következő: [02. Típusrendszer →](02-tipusrendszer.md)

# 0.1 — Fejlesztői környezet és build eszközök

PHP-ban a `php` interpreter futtatja közvetlenül a forráskódot. Java-ban két lépés van:
**fordítás** (`.java` → `.class` bytecode) és **futtatás** (a JVM futtatja a bytecode-ot). Ez a
különbség sok mindent megmagyaráz abból, amit furcsának fogsz találni — pl. hogy típushibát már
fordításkor elkapsz, nem futásidőben, ahogy PHP-ban.

## JDK telepítése

A JDK (Java Development Kit) tartalmazza a fordítót (`javac`) és a futtatókörnyezetet (JVM).

Ajánlott: **Java 21 (LTS)** — ez a jelenlegi hosszú távú támogatású verzió, ezt fogja várni
a legtöbb friss Spring Boot 3.x projekt.

Verziókezeléshez használj **SDKMAN**-t — ez kb. az, amit a `phpbrew`/Herd/Valet jelent PHP verziók
kezelésére:

```bash
curl -s "https://get.sdkman.io" | bash
source "$HOME/.sdkman/bin/sdkman-init.sh"

sdk list java
sdk install java 21.0.4-tem
sdk use java 21.0.4-tem
```

Ellenőrzés:

```bash
java -version
javac -version
```

## Composer → Maven

| Composer | Maven |
|---|---|
| `composer.json` | `pom.xml` |
| `composer.lock` | benne van a `pom.xml`-ben rögzített verziókkal (nincs külön lock fájl klasszikusan) |
| `composer install` | `mvn install` (letölti a függőségeket és build-el) |
| `composer require vendor/pkg` | kézzel adsz hozzá egy `<dependency>` blokkot a `pom.xml`-hez |
| Packagist | Maven Central |
| `vendor/` | `~/.m2/repository` (globális, nem projektenkénti cache) |
| `composer dump-autoload` | nincs rá szükség — a package/import rendszer más elven működik |

A `pom.xml` deklaratív XML, ahol a projekt metaadatait és függőségeit írod le:

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0">
    <modelVersion>4.0.0</modelVersion>

    <groupId>hu.molitor</groupId>
    <artifactId>bank-szimulator</artifactId>
    <version>1.0.0</version>

    <properties>
        <maven.compiler.source>21</maven.compiler.source>
        <maven.compiler.target>21</maven.compiler.target>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.junit.jupiter</groupId>
            <artifactId>junit-jupiter</artifactId>
            <version>5.10.2</version>
            <scope>test</scope>
        </dependency>
    </dependencies>
</project>
```

Gyakori Maven parancsok (a `composer`/`artisan` megszokott reflexeid helyett):

```bash
mvn compile      # lefordítja a forráskódot
mvn test         # lefuttatja a teszteket
mvn package      # buildel egy futtatható .jar-t (target/ mappába)
mvn clean        # törli a build artifactokat
```

Alternatíva: **Gradle** — rugalmasabb, Groovy/Kotlin DSL-t használ XML helyett, de meredekebb
tanulási görbe. Kezdésnek maradj Maven-nél, a Spring világ nagy része mindkettőt egyformán
támogatja, könnyű lesz később váltani.

## Projekt struktúra: Laravel app/ vs Maven src/

Laravel-ben megszoktad, hogy a mappastruktúra konvenció-alapú (`app/Models`, `app/Http/Controllers`
stb.). Maven-nek is van egy szigorú konvenciója, az ún. **Standard Directory Layout**:

```
bank-szimulator/
├── pom.xml                          # ~ composer.json
├── src/
│   ├── main/
│   │   ├── java/                    # ~ app/ — a tényleges forráskód
│   │   │   └── hu/molitor/bank/
│   │   │       ├── Account.java
│   │   │       └── Bank.java
│   │   └── resources/               # ~ config/ + resources/ — konfig fájlok, nem-Java erőforrások
│   └── test/
│       ├── java/                    # ~ tests/ — a tesztek, tükrözik a main/java csomagszerkezetét
│       │   └── hu/molitor/bank/
│       │       └── AccountTest.java
│       └── resources/
└── target/                          # ~ vendor/ + build cache — generált fájlok, ne verzionáld
```

Fontos különbség: a Java **package** (csomag) rendszer közvetlenül leképezi a mappastruktúrát,
és a package névnek a fájl elején is szerepelnie kell — ez szigorúbb, mint a PSR-4 autoload Laravel-ben,
ahol a `namespace` és a mappa összefüggése konvenció, itt viszont a fordító kényszeríti ki.

```java
// src/main/java/hu/molitor/bank/Account.java
package hu.molitor.bank;

public class Account {
    // ...
}
```

Ez majdnem megegyezik a Laravel `namespace App\Models;` gondolattal, csak itt a package név
hagyományosan fordított domain (`hu.molitor.bank`), és **kötelezően** egyezik a fájl elérési
útjával — nincs `composer dump-autoload`, amivel elkened a hibát, a fordító azonnal hibát dob,
ha nem egyezik.

## IDE: IntelliJ IDEA

Töltsd le az **IntelliJ IDEA Community Edition**-t (ingyenes): https://www.jetbrains.com/idea/download/

Ez lesz a PhpStorm Java megfelelője (ugyanaz a JetBrains gyártja, a billentyűkombók és sok
funkció ismerős lesz). Az Ultimate verzió (fizetős) tartalmaz Spring-specifikus support-ot is,
de a Community Edition tökéletesen elég ehhez a fázishoz.

Első lépések:
1. `New Project` → `Maven` → válaszd ki a telepített JDK 21-et
2. Engedélyezd az auto-import-ot Maven változásoknál (`pom.xml` szerkesztésekor)
3. Ismerd meg a `Alt+Enter` (quick fix) és `Ctrl+Shift+A` (action search) kombókat — ezekre
   nagyon sokat fogsz támaszkodni, amíg a Java szintaxis nem válik reflexszé

## Hello World

```java
// src/main/java/hu/molitor/HelloWorld.java
package hu.molitor;

public class HelloWorld {
    public static void main(String[] args) {
        System.out.println("Hello, Java!");
    }
}
```

Amit érdemes azonnal megjegyezni PHP-hoz képest:
- Nincs `<?php` nyitótag, nincs kevert HTML/PHP fájl — minden `.java` fájl tisztán kód
- A fájl neve **kötelezően** meg kell egyezzen a benne lévő public osztály nevével
  (`HelloWorld.java` ↔ `class HelloWorld`) — ez PHP-ban csak konvenció (PSR-4), itt fordítási hiba,
  ha eltér
- A `main` metódus a belépési pont — ez az, amit az `artisan serve`/`index.php` jelent Laravel-ben,
  csak itt explicit metódusként írod le, nem a keretrendszer routolja oda a kérést

Futtatás parancssorból:

```bash
javac src/main/java/hu/molitor/HelloWorld.java -d target/classes
java -cp target/classes hu.molitor.HelloWorld
```

Gyakorlatban IntelliJ-ből fogod futtatni (zöld play gomb a `main` metódus mellett) — a fenti csak
azért fontos, hogy értsd, mi történik a háttérben a fordítás/futtatás két lépésében.

---

[← Fázis index](../00-java-alapok.md) · [Főoldal](../../README.md) · Következő: [02. Típusrendszer →](02-tipusrendszer.md)
