[← Előző: 2. Adatréteg: Spring Data JPA](02-adatreteg-jpa.md) · [Főoldal](../README.md) · Következő: [04. Spring Security →](04-spring-security.md)

# 3. fázis — Validáció, hibakezelés, réteges architektúra

Időtartam: ~1-2 hét

Cél: production-grade API struktúra, ahogy egy nagyobb Laravel projektben is elvárnád.

- Bean Validation: `@Valid`, `@NotNull`, `@Size`, `@Email` a DTO-kon — ez a Form Request
  validációs szabályainak felel meg
- DTO-k bevezetése (ne az Entity-t exponáld közvetlenül az API-n — ez fontosabb Spring-ben, mint
  Laravel-ben, mert nincs automatikus `$hidden`/`$fillable` védelmed)
- `@ControllerAdvice` + `@ExceptionHandler` — ez a Laravel `Handler.php`/exception rendering
  megfelelője, globális hibakezelés
- Réteges architektúra: Controller → Service → Repository — ez tisztább elválasztás, mint
  a legtöbb Laravel projekt "fat controller/fat model" mintája; szokj hozzá a service réteghez
- MapStruct vagy manuális mapper Entity↔DTO konverzióhoz

## Gyakorlat

Refaktoráld a Todo API-t tiszta réteges struktúrára, egységes hibaválasz formátummal.

---

[← Előző: 2. Adatréteg: Spring Data JPA](02-adatreteg-jpa.md) · [Főoldal](../README.md) · Következő: [04. Spring Security →](04-spring-security.md)
