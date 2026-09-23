[← Fázis index](../07-observability.md) · [Főoldal](../../README.md) · Következő: [02. Metrikák →](02-metrikak.md)

# 7.1 — Spring Boot Actuator

A Laravel világban a fejlesztés közbeni megfigyelhetőséget jellemzően a **Telescope**
(kérések, query-k, exceptionök, job-ok naplózása egy fejlesztői dashboardon) és a **Horizon**
(Redis queue worker-ek monitorozása) adja. Ezek elsősorban **emberi szemre** készültek — bejelentkezel
egy webes felületre, és böngészed a naplózott eseményeket.

A Spring Boot **Actuator** más célra való: **gépi fogyasztásra** — monitoring rendszerek
(Prometheus, Kubernetes liveness/readiness probe-ok, load balancerek) hívják géptől-gépig,
JSON válaszokkal. Nincs saját webes UI-ja (van egy egyszerű HAL böngésző, de nem erre való),
hanem egy sor HTTP végpontot exponál, amit külső eszközök olvasnak ki rendszeresen.

## Bekapcsolás

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```

Ennyi — a starter hozzáadása után az alkalmazás azonnal exponál néhány alap végpontot, alapértelmezetten
csak a `/actuator/health`-et publikusan.

## A legfontosabb beépített végpontok

| Végpont | Mit mutat | Laravel-es analógia |
|---|---|---|
| `/actuator/health` | fut-e az app, elérhető-e az adatbázis/egyéb függőség | nincs pontos megfelelő — leginkább egy saját `/up` route-hoz hasonlít, amit K8s health checkhez írnál |
| `/actuator/info` | build/verzió infó, amit te konfigurálsz | `php artisan --version` kimenete, csak HTTP-n |
| `/actuator/metrics` | JVM memória, CPU, HTTP kérésszám stb. | nincs beépített Laravel megfelelő |
| `/actuator/env` | aktív konfigurációs értékek (⚠️ érzékeny adat!) | `php artisan config:show` |
| `/actuator/loggers` | futásidőben átállítható log szintek | nincs pontos megfelelő |

```java
// application.yml
management:
  endpoints:
    web:
      exposure:
        include: health, info, metrics, prometheus
  endpoint:
    health:
      show-details: when-authorized
```

## Miért nem exponálod mind alapból?

Alapból a Spring Boot **csak** a `health` végpontot teszi publikusan elérhetővé — ez tudatos
biztonsági döntés. Az `/actuator/env` például kiírja az **összes** aktív konfigurációs értéket,
adatbázis jelszavakkal együtt (bár a Spring maszkolja a "sensitive" néven felismert kulcsokat,
pl. amikben `password` szerepel — de erre nem szabad vakon hagyatkozni). Ez ugyanaz a veszély, mint
amikor egy Laravel projektben véletlenül élesen bekapcsolva marad a `APP_DEBUG=true`, és egy
stack trace kiszivárogtatja a `.env` tartalmát egy hibaoldalon.

**Szabály**: csak azokat a végpontokat exponáld (`management.endpoints.web.exposure.include`),
amikre ténylegesen szükséged van (tipikusan `health`, `info`, `prometheus`), és éles
környezetben tedd az Actuator útvonalakat egy külön, belső hálózatról elérhető portra
(`management.server.port`), vagy védd Spring Security-vel — ugyanúgy, ahogy egy Telescope
dashboardot sem tennél ki védelem nélkül élesre.

## `/actuator/health` egyedi komponensekkel

```java
@Component
public class ExternalApiHealthIndicator implements HealthIndicator {

    @Override
    public Health health() {
        boolean reachable = checkExternalApi();
        return reachable
            ? Health.up().build()
            : Health.down().withDetail("reason", "external API unreachable").build();
    }
}
```

Ez automatikusan bekerül a `/actuator/health` összesített válaszába — ha bármelyik komponens
`DOWN`, az egész health check `DOWN`-t jelez, amit egy Kubernetes liveness/readiness probe azonnal
észlel, és nem irányít forgalmat a példány felé, amíg nem áll helyre.

---

[← Fázis index](../07-observability.md) · [Főoldal](../../README.md) · Következő: [02. Metrikák →](02-metrikak.md)
