[← Főoldal](../README.md) · Következő fázis: [01. Spring Boot alapok és a DI mentális modell →](01-spring-boot-alapok.md)

# 0. fázis — Java alapok

Időtartam: ~1-2 hét

Cél: annyi Java nyelvi tudás, hogy a Spring kódot ne a szintaxis, hanem a koncepció nehezítse.
PHP/Laravel háttérrel a legtöbb OOP és architektúra-fogalom ismerős lesz — ez a fázis kifejezetten
azokra a pontokra fókuszál, ahol a Java nyelvi modellje **ténylegesen eltér** a PHP-től
(statikus típusosság, generics, checked exceptionök), nem ismétli át az alap OOP-t a nulláról.

Ez a fázis alfejezetekre van bontva, mindegyik konkrét PHP/Laravel összehasonlításokkal és
kódpéldákkal. Haladj sorban — mindegyik épít az előzőre:

1. [Fejlesztői környezet és build eszközök](00-java-alapok/01-kornyezet-es-eszkozok.md) —
   JDK, Maven (Composer megfelelője), VS Code és IntelliJ IDEA beállítása, projektstruktúra, Hello World
2. [Típusrendszer](00-java-alapok/02-tipusrendszer.md) —
   statikus típusosság, primitívek vs. objektumok, `String` összehasonlítás buktatói, `var`
3. [OOP alapok](00-java-alapok/03-oop-alapok.md) —
   láthatósági szintek, `final`, interfészek/absztrakt osztályok, `equals`/`hashCode`/`toString`, enum
4. [Generics](00-java-alapok/04-generics.md) —
   `List<T>`, saját generikus osztályok, miért nincs erre szükség PHP-ban
5. [`Optional<T>`](00-java-alapok/05-optional.md) —
   null-kezelés Java módra, mikor és hogyan használd
6. [Streamek és lambdák](00-java-alapok/06-streamek-lambdak.md) —
   Laravel Collection ↔ Stream API táblázatos megfeleltetés
7. [Kivételkezelés](00-java-alapok/07-kivetelkezeles.md) —
   checked vs unchecked exception, try-with-resources
8. [Záró gyakorlat: bank szimulátor](00-java-alapok/08-gyakorlat-bank-szimulator.md) —
   egy összefoglaló projekt, ami az összes fenti témát egyetlen kis alkalmazásban gyakoroltatja be,
   ellenőrző kérdésekkel a fázis lezárásához

---

[← Főoldal](../README.md) · Következő fázis: [01. Spring Boot alapok és a DI mentális modell →](01-spring-boot-alapok.md)
