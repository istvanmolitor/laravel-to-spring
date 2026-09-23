[← Előző: 7. Observability, production-readiness](07-observability.md) · [Főoldal](../README.md)

# 8. fázis — Építs egy komplett projektet

Időtartam: ~3-4 hét

A legfontosabb lépés: ne csak izolált gyakorlatokat csinálj, hanem építs végig egy valós méretű
alkalmazást, amiben minden korábbi fázis eleme összeáll. Ötlet: portold át egy már meglévő,
kisebb Laravel projektedet Spring Boot-ra — így közvetlenül összehasonlíthatod a két
megközelítést ugyanazon a domain logikán.

## Javasolt technológiai stack az első komplett projekthez

- Spring Boot 3.x + Java 21 (LTS)
- Spring Data JPA + PostgreSQL + Flyway
- Spring Security + JWT
- Docker Compose (app + db + esetleg Redis/RabbitMQ)
- JUnit 5 + Mockito + Testcontainers
- GitHub Actions CI (build + teszt)

---

[← Előző: 7. Observability, production-readiness](07-observability.md) · [Főoldal](../README.md)
