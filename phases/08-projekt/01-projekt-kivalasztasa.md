[← Fázis index](../08-projekt.md) · [Főoldal](../../README.md) · Következő: [02. Technológiai stack →](02-technologiai-stack.md)

# 8.1 — Projekt kiválasztása

A roadmap 0-7. fázisában megtanultad a Java/Spring alapfogalmakat egy gyakorló "Todo API"
projekten keresztül. Ez a fázis más: itt egy **valós, önálló projektet** viszel végig — ideális
esetben egy már meglévő Laravel alkalmazásodat portolod át Spring Boot-ra. Ez a legjobb módja
annak, hogy a tudásod ne csak izolált gyakorlatokban, hanem egy teljes, koherens rendszerben
álljon össze.

## Miért éri meg egy meglévő projektet portolni (nem egy vadonatúj ötletet építeni)

Ha egy vadonatúj alkalmazást terveznél Spring Boot-ban, egyszerre kellene megküzdened a
**domain-tervezéssel** és az **új keretrendszer tanulásával** — ez két külön kognitív terhelés.
Ha viszont egy már ismert Laravel projektet portolsz, a domain logika, az adatmodell és az
elvárt viselkedés adott — csak a *hogyan* változik, a *mit* nem. Ez lehetővé teszi, hogy
közvetlenül összehasonlítsd a két megközelítést ugyanazon a problémán, és azonnal észreveszed,
ha valamit rosszul portoltál (mert tudod, mi a helyes viselkedés).

## Milyen projektet válassz — döntési szempontok

| Szempont | Ideális tartomány | Miért |
|---|---|---|
| Modellek/entitások száma | 5-15 | elég komplex, hogy több kapcsolattípust (1:N, N:M) gyakorolj, de nem hetekig tartó monstrum |
| Endpointok száma | 20-50 | reális méretű API felület, de 3-4 hét alatt kivitelezhető |
| Domain komplexitás | közepes: van üzleti logika, nem csak CRUD | ha csak CRUD, keveset tanulsz a service rétegről; ha túl komplex, elvész a fókusz a Spring tanulásáról |
| Külső integrációk | 0-2 (pl. email küldés, egy egyszerű fizetési vagy külső API hívás) | tanulságos (látod, hogyan néz ki egy HTTP kliens hívás Java-ban), de nem blokkoló, ha nincs hozzáférésed a külső szolgáltatáshoz |
| Meglévő tesztlefedettség | minél nagyobb, annál jobb | a meglévő PHPUnit/Pest tesztek dokumentálják az elvárt viselkedést — ezekből portolod a JUnit teszteket is, és ellenőrizheted, hogy a Java verzió ugyanazt csinálja |
| Authentikáció típusa | Sanctum vagy Passport, jelszavas login | közvetlenül megfeleltethető a 4. fázisban tanult JWT-alapú Spring Security megoldásnak |

## Gyors önértékelő checklist

Menj végig a saját Laravel projektjeiden ezzel a listával — minél több pipát kap egy projekt,
annál alkalmasabb:

- [ ] Kevesebb mint 15 Eloquent modell van benne
- [ ] Van legalább egy `hasMany`/`belongsTo` és legalább egy `belongsToMany` kapcsolat
- [ ] Van benne érdemi üzleti logika (validáció, számítás, állapotgép), nem csak "mentsd el a rekordot"
- [ ] A domain-t Te magad is jól ismered (nem kell közben a szakterületet is tanulnod)
- [ ] Van rajta legalább alap tesztlefedettség, vagy legalább jól dokumentált az elvárt viselkedés
- [ ] Nincs benne olyan Laravel-specifikus "mágia" (pl. bonyolult csomagok, egyedi Blade
      komponens-rendszer), aminek nincs értelme lefordítani egy API-only Spring alkalmazásba

## Ha nincs megfelelő saját projekted

Semmi baj — válassz egy ismerős, egyszerű domaint, amit könnyen el tudsz képzelni Laravel-ben
is, és tervezd meg "papíron" mindkét oldalon:

- **Blog/CMS** — Post, Category, Tag, Comment, User (szerző) — jó gyakorlat N:M kapcsolatokra
  (Post↔Tag) és 1:N-re (Post→Comment)
- **Könyvtárkezelő** — Book, Author, Member, Loan — állapotgép-szerű logika (kölcsönzés/visszahozás),
  jó gyakorlat üzleti szabályokra (nem kölcsönözhető, ha már ki van adva)
- **Egyszerű webshop** — Product, Category, Order, OrderItem, Customer — jó gyakorlat
  tranzakciókezelésre (rendelés leadása több táblát érint egyszerre)

A lényeg: bármelyiket választod, legyen benne elég reláció és üzleti logika ahhoz, hogy a
2-4. fázisban tanultakat (JPA kapcsolatok, réteges architektúra, validáció) valóban
alkalmazhasd, ne csak egy triviális CRUD-ot ismételj meg.

---

[← Fázis index](../08-projekt.md) · [Főoldal](../../README.md) · Következő: [02. Technológiai stack →](02-technologiai-stack.md)
