[← Előző fázis: 02. Adatréteg: Spring Data JPA](02-adatreteg-jpa.md) · [Főoldal](../README.md) · Következő fázis: [04. Spring Security →](04-spring-security.md)

# 3. fázis — Validáció, hibakezelés, réteges architektúra

Időtartam: ~1-2 hét

Cél: production-grade API struktúra, ahogy egy nagyobb Laravel projektben is elvárnád.

Ez a fázis alfejezetekre van bontva, mindegyik konkrét PHP/Laravel összehasonlításokkal és
kódpéldákkal. Haladj sorban — mindegyik épít az előzőre:

1. [Bean Validation](03-validacio-hibakezeles/01-bean-validation.md) —
   `@Valid`, `@NotNull`, `@Size`, `@Email` a DTO-kon, egyedi validátor írása — a Form Request
   `rules()` megfelelője
2. [DTO-k és mapping](03-validacio-hibakezeles/02-dto-k-es-mapping.md) —
   miért ne exponáld az Entity-t közvetlenül, `record` mint DTO, kézzel írt mapper vs MapStruct
3. [Globális hibakezelés](03-validacio-hibakezeles/03-globalis-hibakezeles.md) —
   `@ControllerAdvice` + `@ExceptionHandler`, egységes hibaválasz formátum — a `Handler.php` megfelelője
4. [Réteges architektúra](03-validacio-hibakezeles/04-reteges-architektura.md) —
   Controller → Service → Repository felelősség-elválasztás a Laravel "fat controller" mintával szemben
5. [Záró gyakorlat: refaktorálás](03-validacio-hibakezeles/05-gyakorlat-refaktoralas.md) —
   a Todo API átalakítása tiszta réteges struktúrára, DTO-kkal és egységes hibaválasszal

---

[← Előző fázis: 02. Adatréteg: Spring Data JPA](02-adatreteg-jpa.md) · [Főoldal](../README.md) · Következő fázis: [04. Spring Security →](04-spring-security.md)
