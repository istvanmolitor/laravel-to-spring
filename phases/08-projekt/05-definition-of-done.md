[← Előző: 04. CI/CD](04-ci-cd.md) · [Fázis index](../08-projekt.md) · [Főoldal](../../README.md)

# 8.5 — Definition of Done

Ez a checklist segít eldönteni, hogy a migrációs projekted "készen van" — nem feltétlenül
tökéletes vagy teljes körű, hanem elég érett ahhoz, hogy azt mondd: sikeresen átültettél egy
valós Laravel alkalmazást Spring Boot-ra, és production-grade szemlélettel dolgoztál.

## Funkcionalitás

- [ ] Az eredeti Laravel projekt összes fő endpointjának van Spring megfelelője
- [ ] Minden endpoint ugyanazt a válasz-struktúrát/adatot adja vissza, mint az eredeti (legalább
      a fő mezők szintjén)
- [ ] A validációs szabályok (kötelező mezők, formátum-ellenőrzések) át lettek ültetve Bean
      Validation-re, és ugyanazokat az eseteket utasítják el
- [ ] Az üzleti logika edge case-ei (pl. "nem törölhető, ha van hozzá kapcsolódó rekord") le
      vannak fedve

## Adatréteg

- [ ] Minden Eloquent reláció megfelelő JPA `@OneToMany`/`@ManyToOne`/`@ManyToMany` párral
      rendelkezik
- [ ] Végignézted a fő listázó/riport endpointokat N+1 probléma szempontjából (lásd
      [2.2 fázis](../02-adatreteg-jpa/02-kapcsolatok.md)), és ahol szükséges, `JOIN FETCH`-et
      vagy explicit lekérdezést használsz
- [ ] Minden séma-változás Flyway migrációban van rögzítve, nem `ddl-auto: update`-tel generált
- [ ] A migrációk tiszta adatbázisból lefuttatva hibamentesen létrehozzák a teljes sémát

## Biztonság

- [ ] A bejelentkezés/regisztráció végpontok működnek, JWT tokent adnak vissza
- [ ] A védett végpontok valóban elutasítják a token nélküli/érvénytelen tokenes kéréseket
- [ ] A tulajdonos-alapú jogosultság-ellenőrzés működik (egy user nem érheti el más user
      erőforrásait, ha az eredeti Laravel projektben sem tehette)
- [ ] CORS be van állítva a frontendhez (ha van ilyen), nem `*` engedélyezéssel élesben
- [ ] Jelszavak BCrypt-tel vannak hashelve, sosem kerülnek vissza válaszban

## Tesztek

- [ ] A kritikus üzleti logikai útvonalak (a legfontosabb 5-10 eset) unit teszttel le vannak
      fedve Mockitóval
- [ ] Legalább a fő erőforrásokra van integrációs teszt Testcontainers + `@SpringBootTest`
      kombinációval, ami a teljes HTTP-kérés/válasz ciklust ellenőrzi
- [ ] A tesztek megbízhatóan, reprodukálhatóan zöldek (nem "flaky")

## Observability

- [ ] `/actuator/health` endpoint elérhető és helyes állapotot jelez
- [ ] Van legalább alapszintű strukturált logolás a fő üzleti műveleteknél (létrehozás, törlés,
      hibák)
- [ ] Az érzékeny adatok (jelszó, token) nem kerülnek logba

## CI

- [ ] A GitHub Actions workflow minden push-nál lefut és zöld
- [ ] A build (`mvn package`) sikeresen előállítja a futtatható artifactot

---

## Ezzel lezártad a teljes roadmapet

Ha ez a checklist nagyrészt ki van pipálva, sikeresen végigmentél a teljes Laravel → Spring
átálláson: a Java nyelvi alapoktól ([0. fázis](../../00-java-alapok.md)) a Spring Boot
alapokon, adatrétegen, réteges architektúrán, biztonságon, teszteken és aszinkron
folyamatokon át egészen egy komplett, éles minőségű rendszerig. A Laravel-es háttered nem
lett feleslegessé — a legtöbb architekturális döntést pontosan azért tudtad meghozni gyorsan,
mert már ismerted a mögöttes problémát (rétegzés, validáció, auth, N+1), csak most már Java és
Spring eszközökkel is tudod megoldani. Innentől a legjobb módja a további fejlődésnek, ha valós
munkában, csapatban dolgozol Spring projekteken, és a Baeldung/hivatalos dokumentáció (lásd a
[főoldal ajánlott tananyagok](../../README.md#ajánlott-tananyagok) szakaszát) segítségével
mélyíted el a témákat, amelyekkel a saját projektedben ténylegesen találkozol.

---

[← Előző: 04. CI/CD](04-ci-cd.md) · [Fázis index](../08-projekt.md) · [Főoldal](../../README.md)
