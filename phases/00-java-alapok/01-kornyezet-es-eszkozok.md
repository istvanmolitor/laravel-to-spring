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

Ubuntu-n a legegyszerűbb az APT-os csomagkezelő:

```bash
sudo apt update
sudo apt install openjdk-21-jdk
```

Ellenőrzés:

```bash
java -version
javac -version
```

Ha több Java verziót is kezelni akarsz párhuzamosan (pl. egy régebbi projekthez Java 17 kell), arra
való a **SDKMAN** — ez kb. az, amit a `phpbrew`/Herd/Valet jelent PHP verziók kezelésére:

```bash
curl -s "https://get.sdkman.io" | bash
source "$HOME/.sdkman/bin/sdkman-init.sh"

sdk list java
sdk install java 21.0.4-tem
sdk use java 21.0.4-tem
```

Kezdésnek az APT-os telepítés is tökéletesen elég, ne bonyolítsd túl feleslegesen.

## Composer → Maven

Ubuntu-n a Maven is APT-ból települ:

```bash
sudo apt install maven
mvn -version
```

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

## IDE: Visual Studio Code (Ubuntu)

Megjegyzés: az **IntelliJ IDEA Community Edition** ingyenes (nem kell rá előfizetés,
csak az Ultimate verzió fizetős) — ha egyszer úgy döntenél, hogy váltanál, ugyanez a Maven-alapú
workflow ott is működik. De VS Code-dal is tökéletesen végig lehet csinálni a teljes roadmapot,
Spring Boot projektekkel együtt.

### Java kiegészítők telepítése

```bash
code --install-extension vscjava.vscode-java-pack
```

Ez az **"Extension Pack for Java"** (Microsoft/Red Hat), ami egyben tartalmazza:
- **Language Support for Java(TM) by Red Hat** — szintaxis-kiemelés, autocomplete, refaktorálás
- **Debugger for Java** — töréspontok, léptetés
- **Test Runner for Java** — JUnit tesztek futtatása/debugolása közvetlenül az editorból
- **Maven for Java** — `pom.xml` kezelés, Maven életciklus parancsok a paletta menüből
- **Project Manager for Java**

### Projekt létrehozása

**A) VS Code-ból, irányított folyamattal:**

`Ctrl+Shift+P` → `Java: Create Java Project` → `Maven` → archetípus:
`maven-archetype-quickstart` → add meg a groupId-ot (`hu.molitor`) és artifactId-ot
(`bank-szimulator`) → válassz mappát.

Ez legenerál egy alap `pom.xml`-t (általában JUnit 4-es függőséggel — cseréld le a fenti JUnit 5
Jupiter blokkra), plusz egy minta `App.java`/`AppTest.java` fájlt.

**B) Kézzel, a fenti `pom.xml`-lel:**

```bash
mkdir -p bank-szimulator/src/main/java/hu/molitor/bank
mkdir -p bank-szimulator/src/test/java/hu/molitor/bank
cd bank-szimulator
```

Hozd létre a `pom.xml`-t a fenti tartalommal (Write/szerkesztő), majd nyisd meg a mappát:

```bash
code .
```

VS Code felismeri, hogy Maven projektről van szó, és a Java Language Server elindítja az
indexelést (ezt a jobb alsó sarokban egy folyamatjelző mutatja).

### Futtatás és tesztelés VS Code-ban

- Nyisd meg a `main` metódust tartalmazó osztályt — fölötte megjelenik egy `Run | Debug`
  CodeLens link, arra kattintva lefut, a `Debug`-bal töréspontokat is tehetsz
- JUnit tesztekhez: a bal oldali sávban egy Erlenmeyer-lombik ikon (**Testing** nézet) — ott
  listázva látod az összes `@Test` metódust, egyenként vagy csoportosan futtathatod/debugolhatod
- Terminálból ugyanúgy működik, mint bármelyik Maven projektnél: `mvn compile`, `mvn test`,
  `mvn package`

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

Gyakorlatban VS Code-ból fogod futtatni (a `main` metódus fölötti `Run` CodeLens linkkel) — a
fenti csak azért fontos, hogy értsd, mi történik a háttérben a fordítás/futtatás két lépésében.

---

[← Fázis index](../00-java-alapok.md) · [Főoldal](../../README.md) · Következő: [02. Típusrendszer →](02-tipusrendszer.md)
