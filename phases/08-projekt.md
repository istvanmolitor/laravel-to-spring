[← Előző fázis: 07. Megfigyelhetőség és production-readiness](07-observability.md) · [Főoldal](../README.md)

# 8. fázis — Építs egy komplett projektet

Időtartam: ~3-4 hét

A legfontosabb lépés: ne csak izolált gyakorlatokat csinálj, hanem építs végig egy valós méretű
alkalmazást, amiben minden korábbi fázis eleme összeáll. Ez a fázis más jellegű, mint az
előzőek — nem új Java/Spring koncepciókat tanít, hanem a projektmenedzsment és az alkalmazási
stratégia áll a középpontban: hogyan válassz egy valós (ideális esetben már meglévő) Laravel
projektet, és hogyan portold át Spring Boot-ra, felhasználva mindazt, amit az 1-7. fázisban
tanultál.

1. [Projekt kiválasztása](08-projekt/01-projekt-kivalasztasa.md) —
   szempontok és checklist egy meglévő Laravel projekt átportolásra való alkalmasságának
   felméréséhez, alternatívák ha nincs megfelelő saját projekted
2. [Technológiai stack](08-projekt/02-technologiai-stack.md) —
   a javasolt stack (Spring Boot 3.x, JPA, PostgreSQL, Flyway, Spring Security+JWT, Testcontainers,
   GitHub Actions) részletes indoklással, visszautalva minden korábbi fázisra
3. [Migrációs stratégia](08-projekt/03-migracios-strategia.md) —
   lépésről lépésre terv funkció-szigetenkénti portolásra, konkrét Eloquent→JPA buktatókkal
   (timestampek, soft delete, mass assignment, accessor/mutator)
4. [CI/CD](08-projekt/04-ci-cd.md) —
   GitHub Actions workflow Maven build-del, teszteléssel és Docker image-építéssel
5. [Definition of Done](08-projekt/05-definition-of-done.md) —
   önértékelési checklist a projekt lezárásához, kategorizálva funkcionalitás, adatréteg,
   biztonság, tesztek, observability és CI szerint

---

[← Előző fázis: 07. Megfigyelhetőség és production-readiness](07-observability.md) · [Főoldal](../README.md)
