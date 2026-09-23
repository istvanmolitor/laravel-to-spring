[← Előző: 6. Async, események, háttérfeladatok](06-async-esemenyek.md) · [Főoldal](../README.md) · Következő: [08. Komplett projekt →](08-projekt.md)

# 7. fázis — Megfigyelhetőség és production-readiness

Időtartam: ~1 hét

- Spring Boot Actuator (`/actuator/health`, `/actuator/metrics`) — a Laravel Horizon/Telescope
  megfelelője, beépített
- Micrometer + Prometheus/Grafana integráció
- Strukturált logolás (Logback/SLF4J), log szintek — hasonló elv, mint Laravel `Log::` facade,
  csak konfigurációban XML/YAML alapú
- `@ConfigurationProperties` validáció, profil-alapú (`dev`/`staging`/`prod`) konfiguráció

---

[← Előző: 6. Async, események, háttérfeladatok](06-async-esemenyek.md) · [Főoldal](../README.md) · Következő: [08. Komplett projekt →](08-projekt.md)
