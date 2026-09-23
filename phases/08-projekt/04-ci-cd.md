[← Előző: 03. Migrációs stratégia](03-migracios-strategia.md) · [Fázis index](../08-projekt.md) · [Főoldal](../../README.md) · Következő: [05. Definition of Done →](05-definition-of-done.md)

# 8.4 — CI/CD GitHub Actions-szel

Ha korábban Laravel projekteken már beállítottál GitHub Actions CI-t (`composer install`,
`php artisan test`), a struktúra koncepcionálisan ismerős lesz — ugyanaz a "checkout → függőségek
telepítése → tesztek futtatása → build" pipeline, csak Maven/JUnit lépésekkel.

## Alap workflow: teszt minden push-nál

```yaml
# .github/workflows/ci.yml
name: CI

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  test:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v4

      - name: Set up JDK 21
        uses: actions/setup-java@v4
        with:
          java-version: '21'
          distribution: 'temurin'
          cache: maven

      - name: Run tests
        run: ./mvnw test
```

Fontos: a Testcontainers-alapú integrációs tesztekhez (lásd [5. fázis](../05-teszteles.md))
Docker daemon kell a futtató környezetben. A GitHub Actions `ubuntu-latest` runner-ein a Docker
alapból elérhető, tehát a Testcontainers tesztek külön beállítás nélkül lefutnak — ez az egyik
ok, amiért a `ubuntu-latest` runner a javasolt választás (nem kell "Docker-in-Docker" trükközés).

Ez összevethető azzal, ahogy egy Laravel CI workflow-ban a `php artisan test` előtt esetleg egy
MySQL/PostgreSQL service container-t indítasz (`services: mysql: image: mysql:8`) — itt a
Testcontainers pontosan ezt csinálja, csak a teszt kód maga indítja el a konténert, nem a CI
workflow konfigurációja.

## Bővített workflow: build + Docker image

```yaml
# .github/workflows/ci.yml (folytatás)
  build:
    needs: test
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main'

    steps:
      - uses: actions/checkout@v4

      - name: Set up JDK 21
        uses: actions/setup-java@v4
        with:
          java-version: '21'
          distribution: 'temurin'
          cache: maven

      - name: Build jar
        run: ./mvnw package -DskipTests

      - name: Build Docker image
        run: docker build -t my-app:${{ github.sha }} .
```

Egy egyszerű `Dockerfile` a Spring Boot alkalmazáshoz:

```dockerfile
FROM eclipse-temurin:21-jre
WORKDIR /app
COPY target/*.jar app.jar
ENTRYPOINT ["java", "-jar", "app.jar"]
```

Ez a két lépéses build (`test` job, majd csak sikeres teszt után `build` job, ami `main`
ágon fut) megegyezik azzal a Laravel gyakorlattal, ahol a `composer install` + `php artisan
test` után esetleg egy Docker image-et vagy deploy artifactot építesz — a `needs: test` kulcsszó
biztosítja, hogy hibás teszt esetén ne épüljön image.

## Mit NE tegyél bele ebbe a fázisba

A tényleges deploy lépést (pl. egy szerverre vagy felhőbe pusholás) hagyd ki ennek a
gyakorlóprojektnek a köréből, hacsak nincs kifejezetten hova deployolnod — a cél itt a **zöld CI
pipeline** elérése, ami bizonyítja, hogy a build és a tesztek megbízhatóan, automatizáltan
lefutnak, nem a teljes production deployment infrastruktúra felépítése.

---

[← Előző: 03. Migrációs stratégia](03-migracios-strategia.md) · [Fázis index](../08-projekt.md) · [Főoldal](../../README.md) · Következő: [05. Definition of Done →](05-definition-of-done.md)
