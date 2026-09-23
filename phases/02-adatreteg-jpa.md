[← Előző fázis: 01. Spring Boot alapok és a DI mentális modell](01-spring-boot-alapok.md) · [Főoldal](../README.md) · Következő fázis: [03. Validáció, hibakezelés, réteges architektúra →](03-validacio-hibakezeles.md)

# 2. fázis — Adatréteg: Spring Data JPA

Időtartam: ~2-3 hét

Cél: az Eloquent tudásod átültetése JPA/Hibernate-re.

Ez a fázis alfejezetekre van bontva, mindegyik konkrét PHP/Laravel összehasonlításokkal és
kódpéldákkal. Haladj sorban — a záró gyakorlat a Todo API-t bővíti tovább, amit az 1. fázisban
építettél:

1. [Entity osztályok](02-adatreteg-jpa/01-entity-osztalyok.md) —
   `@Entity`, `@Table`, `@Id`, `@GeneratedValue`, `@Column` — az Eloquent modell explicit JPA
   megfelelője, Lombok-kal a boilerplate ellen
2. [Kapcsolatok modellezése](02-adatreteg-jpa/02-kapcsolatok.md) —
   `@OneToMany`/`@ManyToOne`/`@ManyToMany`, lazy vs eager loading, az N+1 probléma és a
   `JOIN FETCH` mint az Eloquent `with()` megfelelője
3. [Repository réteg és lekérdezések](02-adatreteg-jpa/03-repository-es-lekerdezesek.md) —
   `JpaRepository<T, ID>`, derived query methods, JPQL és natív `@Query`
4. [Flyway migrációk](02-adatreteg-jpa/04-flyway-migraciok.md) —
   verziózott SQL migrációk az Artisan migration helyett, és miért nincs automatikus rollback
5. [Tranzakciókezelés](02-adatreteg-jpa/05-tranzakciok.md) —
   `@Transactional` mint a `DB::transaction()` deklaratív megfelelője, dirty checking
6. [Záró gyakorlat: Todo API bővítése](02-adatreteg-jpa/06-gyakorlat-todo-api-bovitese.md) —
   User–Todo kapcsolat, Flyway migrációk, PostgreSQL Docker Compose-ban

---

[← Előző fázis: 01. Spring Boot alapok és a DI mentális modell](01-spring-boot-alapok.md) · [Főoldal](../README.md) · Következő fázis: [03. Validáció, hibakezelés, réteges architektúra →](03-validacio-hibakezeles.md)
