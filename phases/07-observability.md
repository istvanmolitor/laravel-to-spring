[← Előző fázis: 06. Aszinkron munka, események, háttérfeladatok](06-async-esemenyek.md) · [Főoldal](../README.md) · Következő fázis: [08. Építs egy komplett projektet →](08-projekt.md)

# 7. fázis — Megfigyelhetőség és production-readiness

Időtartam: ~1 hét

Ez a fázis alfejezetekre van bontva, mindegyik konkrét PHP/Laravel összehasonlításokkal és
kódpéldákkal. Haladj sorban — a záró gyakorlat a korábbi fázisokban felépített Todo API-t
egészíti ki minimális éles-üzemi megfigyelhetőséggel:

1. [Spring Boot Actuator](07-observability/01-actuator.md) —
   `/actuator/health`, `/actuator/metrics` — gépi fogyasztásra szánt megfelelője a Laravel
   Telescope/Horizon emberi dashboardjainak
2. [Metrikák: Micrometer + Prometheus/Grafana](07-observability/02-metrikak.md) —
   egyedi `Counter`/`Timer` metrikák, amikhez nincs beépített Laravel megfelelő
3. [Strukturált logolás](07-observability/03-strukturalt-logolas.md) —
   SLF4J + Logback, profil-függő JSON logolás — a `Log::` facade explicit, konfigurálhatóbb
   megfelelője
4. [Záró gyakorlat: production-readiness](07-observability/04-gyakorlat-production-readiness.md) —
   Actuator, egyedi metrika és profil-alapú logolás bekötése a Todo API-ba

---

[← Előző fázis: 06. Aszinkron munka, események, háttérfeladatok](06-async-esemenyek.md) · [Főoldal](../README.md) · Következő fázis: [08. Építs egy komplett projektet →](08-projekt.md)
