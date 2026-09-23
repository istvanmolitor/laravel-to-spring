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

## IDE: VS Code vagy IntelliJ IDEA

Mindkettő tökéletesen alkalmas a teljes roadmap végigcsinálására, Spring Boot projektekkel
együtt — nem kell köztük választanod, bármelyikkel (vagy akár váltogatva) haladhatsz. Az
**IntelliJ IDEA Community Edition** ingyenes (nem kell rá előfizetés, csak az Ultimate verzió
fizetős), és mivel ugyanaz a JetBrains gyártja, mint a PhpStorm-ot, sok billentyűkombó és
funkció ismerős lesz belőle. A **VS Code** könnyebb, gyorsabban indul, és ha egyébként is azt
használod más projektekhez, nem kell külön IDE-t futtatnod párhuzamosan.

### Opció A: Visual Studio Code

Telepítsd a Java kiegészítőcsomagot:

```bash
code --install-extension vscjava.vscode-java-pack
```

Ez az **"Extension Pack for Java"** (Microsoft/Red Hat), ami egyben tartalmazza:
- **Language Support for Java(TM) by Red Hat** — szintaxis-kiemelés, autocomplete, refaktorálás
- **Debugger for Java** — töréspontok, léptetés
- **Test Runner for Java** — JUnit tesztek futtatása/debugolása közvetlenül az editorból
- **Maven for Java** — `pom.xml` kezelés, Maven életciklus parancsok a paletta menüből
- **Project Manager for Java**

**Projekt létrehozása — A) irányított folyamattal:**

`Ctrl+Shift+P` → `Java: Create Java Project` → `Maven` → archetípus:
`maven-archetype-quickstart` → add meg a groupId-ot (`hu.molitor`) és artifactId-ot
(`bank-szimulator`) → válassz mappát.

Ez legenerál egy alap `pom.xml`-t (általában JUnit 4-es függőséggel — cseréld le a fenti JUnit 5
Jupiter blokkra), plusz egy minta `App.java`/`AppTest.java` fájlt.

**Projekt létrehozása — B) kézzel, a fenti `pom.xml`-lel:**

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

**Futtatás és tesztelés:**

- Nyisd meg a `main` metódust tartalmazó osztályt — fölötte megjelenik egy `Run | Debug`
  CodeLens link, arra kattintva lefut, a `Debug`-bal töréspontokat is tehetsz
- JUnit tesztekhez: a bal oldali sávban egy Erlenmeyer-lombik ikon (**Testing** nézet) — ott
  listázva látod az összes `@Test` metódust, egyenként vagy csoportosan futtathatod/debugolhatod
- Terminálból ugyanúgy működik, mint bármelyik Maven projektnél: `mvn compile`, `mvn test`,
  `mvn package`

### Opció B: IntelliJ IDEA

Töltsd le a Community Edition-t: https://www.jetbrains.com/idea/download/ — Ubuntu-n a
legkényelmesebb a Snap csomag (`sudo snap install intellij-idea-community --classic`) vagy a
JetBrains Toolbox App, ha több JetBrains terméket is kezelnél verziózva.

**Projekt létrehozása:**

`File → New → Project` → bal oldalon `Maven` → válaszd ki a telepített JDK 21-et → `Create`.
Ez automatikusan legenerálja a `pom.xml`-t és a `src/main/java`/`src/test/java` struktúrát — a
JUnit 5 függőséget ugyanúgy kézzel kell hozzáadnod a `pom.xml`-hez, a fenti tartalommal.

**Futtatás és tesztelés:**

- Zöld play gomb jelenik meg a `main` metódus és minden `@Test` metódus mellett a sor elején —
  arra kattintva lefut, a bogár ikonnal debug módban
- `Alt+Enter` (quick fix) és `Ctrl+Shift+A` (action search) a két legfontosabb kombó, amíg a
  Java szintaxis nem válik reflexszé
- `equals()`/`hashCode()`/`toString()` generálásához: `Alt+Insert` a fájlban, majd válaszd ki a
  kívánt generátort
- Engedélyezd az auto-importot Maven változásoknál (`pom.xml` szerkesztésekor felugró
  értesítésben, vagy `Maven` panel → `Reload All Maven Projects`)

## Az IntelliJ-vel generált projekt: `org.example` és a build folyamat

Ha az imént `File → New → Project → Maven` varázslóval hoztál létre egy projektet (mondjuk
`Teszt` néven), valószínűleg ezt kaptad:

```
~/IdeaProjects/Teszt/
├── pom.xml
└── src/main/java/org/example/Main.java
```

Két dolog azonnal szúrja a szemet, ha PHP-ból jössz: honnan jön az `org.example`, és mit jelent
pontosan az, hogy "buildelni" kell a forrást.

### Miért `org.example`?

Ez **nem valamiféle Java-konvenció**, hanem az IntelliJ New Project varázslójának kitöltött
placeholder GroupId-ja, amit akkor kapsz, ha a létrehozáskor nem írtad felül a "GroupId" mezőt.
Önmagában semmit nem jelent — nem a te domained, nem köt semmilyen szervezethez, pusztán egy
minta érték, amivel a varázsló ki tudja tölteni a `pom.xml`-t és a kezdő `Main.java` csomagját.

A GroupId (és az ebből lévő Java package) hagyományosan a **fordított domained**, pontosan úgy,
ahogy a `composer.json` `"name"` mezőjében is `vendor/package` formát használsz, csak itt egy
szinttel korábban, magában a forráskódban is meg kell jelennie. Ha nincs saját domained, bármi
egyedi jó választás: `io.github.<felhasznalonev>`, vagy egyszerűen a neved kisbetűvel
összefűzve, pl. `istvanmolitor`.

**Hogyan cseréld le a meglévő `org.example`-t:**

1. A bal oldali Project fán navigálj a `src/main/java/org/example` csomaghoz
2. Jobb klikk rajta → `Refactor` → `Rename...` → válaszd a **"Rename package"** opciót (ne csak a
   mappát nevezd át kézzel!) → írd be az új nevet, pl. `hu.molitor`
3. Az IntelliJ ekkor **minden fájlban** átírja a `package org.example;` sort és minden importot,
   ami rá hivatkozott — ez a Refactor funkció lényege, ne kézzel, mappaátnevezéssel csináld
4. A `pom.xml`-ben is érdemes frissíteni a `<groupId>org.example</groupId>` sort ugyanerre, hogy
   konzisztens maradjon (ez funkcionálisan nem kötelező — a Java package-et nem a `pom.xml`
   vezérli —, de zavaró, ha nem egyezik)

A jövőben egyszerűbb elkerülni az egészet: projekt létrehozásakor írd be explicit a saját
GroupId-odat, mielőtt a `Create`-re kattintasz.

### Hogyan buildelődik ténylegesen a forráskód

A parancs, amit futtattál —

```bash
javac src/main/java/org/example/Main.java -d target/classes
```

— **közvetlenül a `javac` fordítót** hívja meg, teljesen megkerülve mind IntelliJ-t, mind a
Maven-t. Ez működik egyetlen függőség nélküli fájlnál, de nem ez a szokásos munkamód, és nem
skálázódik: nem fordítja le automatikusan az összes `.java` fájlt, és nem veszi figyelembe a
`pom.xml`-ben deklarált függőségeket (pl. JUnit-ot) a classpath összeállításánál.

Két reális út van build-elésre egy Maven projektben:

1. **A zöld play gomb IntelliJ-ben** — ez alapértelmezetten **nem a Maven-t hívja meg**, hanem
   IntelliJ saját, beépített inkrementális Java fordítóját használja (gyorsabb, mert csak a
   változott fájlokat fordítja újra). Ha azt szeretnéd, hogy a Run/Debug gomb ténylegesen
   `mvn`-en keresztül fusson (pl. mert egy Maven plugin viselkedésére vagy kíváncsi), ezt itt
   kapcsolhatod be: `Settings → Build, Execution, Deployment → Build Tools → Maven → Runner` →
   `Delegate IDE build/run actions to Maven`.
2. **Explicit Maven parancs** — `mvn compile` a terminálból (vagy a jobb oldali Maven panelen
   `Lifecycle → compile` duplakattintással). Ez lefordítja **az összes** forrásfájlt a megfelelő
   könyvtárszerkezetbe, a `pom.xml` függőségeivel együtt számított classpath-tal — ez a helyes,
   skálázódó megfelelője annak, amit kézzel a `javac` hívással próbáltál elérni.

### Hova kerül a build — a `target/` mappa

A Maven Standard Directory Layout szerint **minden** generált fájl a `target/` mappába kerül,
sosem a forrás mellé:

```
target/
├── classes/                          # lefordított .class fájlok, package szerinti almappákban
│   └── org/example/Main.class        # ide célzott pontosan a te "-d target/classes" kapcsolód
├── test-classes/                     # lefordított teszt .class fájlok (mvn test után)
├── generated-sources/                # annotáció-feldolgozók generált forrása, ha van ilyen
└── Teszt-1.0-SNAPSHOT.jar            # mvn package után: a csomagolt, futtatható jar
```

Ez a mappa **soha nem kerül verziókezelésbe** — Git-ben legyen `.gitignore`-ban —, mert bármikor
újra elő tud állni a forráskódból, pontosan úgy, ahogy a Laravel `vendor/` mappáját sem
verziózod, csak a `composer.json`-t/`composer.lock`-ot.

```bash
mvn clean      # törli a teljes target/ mappát
```

Hasznos, ha "furcsa", megmagyarázhatatlan hibákat kapsz, és biztos akarsz lenni benne, hogy nincs
elavult, korábbi futásból visszamaradt build artifact a háttérben.

## Hello World

Az alábbi példa a `hu.molitor` package nevet használja — ha az előző szakasz szerint már
átnevezted az IntelliJ-generált `org.example`-t, nálad ez lesz a saját package neved; ha még nem,
nyugodtan hagyd `org.example`-nek, a lényeg ugyanaz marad.

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

Bontsuk le pontosan, mi történik itt.

**1. sor — fordítás:**

| Rész | Jelentés |
|---|---|
| `javac` | a Java fordító — forráskódból (`.java`) bytecode-ot (`.class`) állít elő |
| `src/main/java/hu/molitor/HelloWorld.java` | a lefordítandó forrásfájl elérési útja |
| `-d target/classes` | hova kerüljön a lefordított `.class` fájl (`-d` = "destination") |

A `javac` beolvassa a fájl elején lévő `package hu.molitor;` sort, és a `-d`-vel megadott
célkönyvtárban **automatikusan újraépíti ezt a csomag-struktúrát alkönyvtárakként** — nem oda
kerül a `.class`, ahol a forrás van, hanem: `target/classes/hu/molitor/HelloWorld.class`.

**2. sor — futtatás:**

| Rész | Jelentés |
|---|---|
| `java` | a JVM indítóparancsa — ez futtatja a lefordított bytecode-ot |
| `-cp target/classes` | **classpath** — hol keresse a JVM a `.class` fájlokat |
| `hu.molitor.HelloWorld` | a teljesen minősített osztálynév (`csomag.OsztályNév`), **nem** fájlútvonal |

Ez a legfontosabb fogalmi váltás PHP-hoz képest: PHP-ban egy **fájlt** futtatsz
(`php public/index.php`), Java-ban egy **osztályt**, aminek a nevét a csomagjával együtt, pontokkal
elválasztva adod meg. A JVM ebből vezeti le, hol keresse a `.class` fájlt: a pontokat
könyvtár-elválasztóra cseréli (`hu.molitor.HelloWorld` → `hu/molitor/HelloWorld`), hozzáfűzi a
`.class` kiterjesztést, és a `-cp`-ben megadott gyökér alatt keresi:
`target/classes/hu/molitor/HelloWorld.class`. Ez picit hasonlít a Composer PSR-4
namespace→mappa leképezésére, csak ezt itt maga a JVM végzi el minden indításkor, nem egy
generált `autoload.php`.

Ha az IntelliJ-generált `org.example` csomagban dolgozol, ugyanez a két parancs így néz ki
(feltéve, hogy a fájl `Main.java`, ahogy IntelliJ alapból elnevezi):

```bash
javac src/main/java/org/example/Main.java -d target/classes
java -cp target/classes org.example.Main
```

Ez a két parancs **egyetlen fájlra** működik, kézzel. Amint két vagy több `.java` fájlod van,
amik hivatkoznak egymásra, mindkettőt egyszerre kell látnia a `javac`-nak a classpath-on,
különben `cannot find symbol` hibát kapsz — ezt oldja meg automatikusan a `mvn compile` (végigmegy
az összes `src/main/java` alatti fájlon, összeállítja a classpath-ot a `pom.xml` függőségeiből).

Gyakorlatban az IDE-ből fogod futtatni (VS Code-ban a `main` metódus fölötti `Run` CodeLens
linkkel, IntelliJ-ben a sor elején megjelenő zöld play gombbal) — a fenti csak azért fontos, hogy
értsd, mi történik a háttérben a fordítás/futtatás két lépésében.

---

[← Fázis index](../00-java-alapok.md) · [Főoldal](../../README.md) · Következő: [02. Típusrendszer →](02-tipusrendszer.md)
