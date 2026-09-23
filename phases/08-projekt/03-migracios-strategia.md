[← Előző: 02. Technológiai stack](02-technologiai-stack.md) · [Fázis index](../08-projekt.md) · [Főoldal](../../README.md) · Következő: [04. CI/CD →](04-ci-cd.md)

# 8.3 — Migrációs stratégia

**Ne "big bang" migrációként gondolj erre.** Ne próbáld egyszerre lefordítani az egész Laravel
projektet — dolgozz **funkció-szigetenként**: válassz ki egy erőforrást (pl. csak a `User` +
autentikáció), vidd végig a teljes vertikumon (entitás → réteges architektúra → auth → teszt),
győződj meg róla, hogy működik, és csak utána lépj a következő erőforrásra. Ez pontosan az a
inkrementális megközelítés, amit egy nagyobb Laravel refaktornál is választanál.

## Lépésről lépésre — visszautalva a roadmap fázisaira

### 1. Domain modellek portolása ([2. fázis](../02-adatreteg-jpa.md) tudása)

Vedd sorra az Eloquent modelljeidet, és alakítsd JPA entitássá. Konkrét buktatók, amikre
PHP fejlesztőként figyelj:

| Laravel/Eloquent | Spring/JPA megfelelő | Megjegyzés |
|---|---|---|
| `$timestamps = true` (automatikus `created_at`/`updated_at`) | `@CreationTimestamp` / `@UpdateTimestamp` (Hibernate annotációk) | explicit ki kell írnod, nincs automatikus konvenció |
| Soft delete (`SoftDeletes` trait, `deleted_at`) | `@SQLDelete` + `@Where(clause = "deleted_at IS NULL")` vagy explicit `deletedAt` mező + minden query-ben szűrés | nincs "trait-szintű" automatizmus, tudatosan kell megtervezni |
| `$casts = ['data' => 'array']` (JSON mező) | `@Column(columnDefinition = "jsonb")` + egy converter (`AttributeConverter`) | Postgres `jsonb` típushoz kell egy explicit konverter osztály |
| `$fillable`/`$guarded` (mass assignment védelem) | nincs rá szükség, mert nem az Entity-t populálod közvetlenül kérésből, hanem a [3. fázisban](../03-validacio-hibakezeles.md) tanult DTO-t | a DTO-alapú tervezés eleve kizárja ezt a problémaosztályt |
| Model Observer / `booted()` hook | `@PrePersist`/`@PreUpdate` JPA lifecycle callback, vagy explicit service-réteg logika | célszerűbb service-rétegbe tenni, ne az entitásba rejtett mellékhatást |
| Accessor/Mutator (`getFullNameAttribute()`) | sima Java metódus az entitáson, vagy DTO mapping logika | nincs "mágikus" property-hozzáférés, mindig explicit metódushívás |

### 2. Réteges architektúra felállítása ([3. fázis](../03-validacio-hibakezeles.md))

Minden Laravel Controller metódusra gondolj úgy, mint ami valójában két felelősséget kever
(HTTP kezelés + üzleti logika) — validáld, hogy Spring oldalon ez explicit szét legyen választva:
`Controller` (HTTP be/kimenet) → `Service` (üzleti logika, `@Transactional`) → `Repository`
(adatelérés). A Form Request validációs szabályaidat Bean Validation annotációkká alakítod a
DTO-kon, a Form Request `authorize()` logikáját pedig a Service rétegbe vagy Spring Security
`@PreAuthorize`-ba.

### 3. Autentikáció/autorizáció portolása ([4. fázis](../04-spring-security.md))

Ha Sanctum vagy Passport tokent használtál, a portolás gondolatilag egyszerű: a login endpoint
JWT-t ad vissza token helyett. Nehezebb rész a **jogosultságkezelés** — ha sok Laravel Policy-d
van, ezek mindegyikét explicit ellenőrzéssé vagy `@PreAuthorize` SpEL kifejezéssé kell alakítanod.
Javaslat: kezdd a legegyszerűbb, "csak a saját erőforrását láthatja" mintájú policykkal, ezek
portolása mechanikus.

### 4. Tesztek átírása ([5. fázis](../05-teszteles.md))

Ha van meglévő PHPUnit/Pest tesztlefedettséged, ez **arany erőforrás** — nem kell kitalálnod,
mit kell tesztelni, csak le kell fordítanod a Laravel HTTP tesztjeidet `MockMvc`/`@SpringBootTest`
integrációs tesztekre, a unit tesztjeidet pedig Mockito-alapú service tesztekre. Ha nincs meglévő
tesztlefedettséged, ez jó alkalom pótolni — de ne hagyd az egész projekt végére, írd meg
erőforrásonként, ahogy portolod.

### 5. Háttérfolyamatok portolása ([6. fázis](../06-async-esemenyek.md))

Vedd sorra a Laravel Job-jaidat és Listener-eidet. Egyszerű, gyors háttérfeladatokhoz elég az
`@Async`; ha volt dedikált queue workered (Horizon), fontold meg RabbitMQ bevezetését — de csak
akkor, ha a projekted mérete indokolja, ne vezesd be feleslegesen egy kis projektnél.

### 6. Observability bekötése ([7. fázis](../07-observability.md))

Ez az utolsó, viszonylag gyors lépés: Actuator health check, alap logolás profil szerint. Ha a
Laravel projektben volt Telescope/Horizon-alapú megfigyelésed, itt az Actuator + Micrometer veszi
át a szerepét.

## Gyakorlati tanács a sorrendre

Javasolt sorrend egy konkrét erőforráson belül: **entitás → repository → service (üzleti
logika egyszerű teszttel) → controller + DTO → auth/jogosultság → integrációs teszt**. Ne kezdd
a controllerrel — enélkül a mögöttes réteg még nem létezik, és a Laravel reflexed ("route,
majd controller") itt félrevezet, mert Spring-ben az adatréteg és a service a stabilabb,
korábban lezárható alap.

---

[← Előző: 02. Technológiai stack](02-technologiai-stack.md) · [Fázis index](../08-projekt.md) · [Főoldal](../../README.md) · Következő: [04. CI/CD →](04-ci-cd.md)
