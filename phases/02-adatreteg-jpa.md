[← Előző: 1. Spring Boot alapok](01-spring-boot-alapok.md) · [Főoldal](../README.md) · Következő: [03. Validáció, hibakezelés, réteges architektúra →](03-validacio-hibakezeles.md)

# 2. fázis — Adatréteg: Spring Data JPA

Időtartam: ~2-3 hét

Cél: az Eloquent tudásod átültetése JPA/Hibernate-re.

- Entity osztályok (`@Entity`, `@Table`, `@Id`, `@GeneratedValue`) — ez az Eloquent modell megfelelője,
  de itt **explicit** kell definiálnod mindent, amit Eloquent konvencióból kitalál
- Kapcsolatok: `@OneToMany`, `@ManyToOne`, `@ManyToMany` + `mappedBy`/`JoinColumn` — összevetve az
  Eloquent `hasMany`/`belongsTo`/`belongsToMany` relációival; itt kell figyelni a **lazy vs eager
  loading**-ra (`FetchType.LAZY` vs `EAGER`) — ez a Laravel N+1 problémának pont a Java megfelelője,
  csak itt explicit kell kezelni
- `JpaRepository<Entity, ID>` — a Laravel Eloquent query builder helyett itt interfészt írsz,
  a Spring generálja le a implementációt (derived query methods: `findByEmail`, `findByStatusAndActive`)
- JPQL és `@Query` annotáció — ez a raw query / `DB::table()` megfelelője, amikor a derived method
  nem elég
- Migráció: **Flyway** bevezetése (`db/migration/V1__init.sql` fájlok) — ez az Artisan migration
  párja, de itt SQL-t írsz közvetlenül, nincs PHP DSL
- Tranzakciókezelés: `@Transactional` — a Laravel `DB::transaction()` deklaratív megfelelője

## Gyakorlat

Bővítsd a Todo API-t user–todo kapcsolattal, Flyway migrációkkal, PostgreSQL-lel Docker
Compose-ban.

---

[← Előző: 1. Spring Boot alapok](01-spring-boot-alapok.md) · [Főoldal](../README.md) · Következő: [03. Validáció, hibakezelés, réteges architektúra →](03-validacio-hibakezeles.md)
