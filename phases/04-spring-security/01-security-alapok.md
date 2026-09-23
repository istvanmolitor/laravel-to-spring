[← Fázis index](../04-spring-security.md) · [Főoldal](../../README.md) · Következő: [02. Felhasználó és jelszó →](02-felhasznalo-es-jelszo.md)

# 4.1 — Spring Security alapok és a filter lánc

A Spring Security a beérkező HTTP kéréseket egy **filter láncon** (filter chain) engedi át, mielőtt
azok elérnék a controllert. Ez koncepcionálisan ugyanaz, mint a Laravel middleware pipeline, csak
más a deklarálás módja.

## Laravel middleware vs Spring Security filter chain

```php
// app/Http/Kernel.php
protected $middlewareGroups = [
    'api' => [
        \Illuminate\Routing\Middleware\ThrottleRequests::class . ':60,1',
        \Illuminate\Routing\Middleware\SubstituteBindings::class,
    ],
];

// routes/api.php
Route::middleware('auth:sanctum')->group(function () {
    Route::apiResource('todos', TodoController::class);
});

Route::post('/register', [AuthController::class, 'register']);
Route::post('/login', [AuthController::class, 'login']);
```

Laravel-ben a middleware-eket **route-onként vagy csoportosan, imperatívan** rendeled hozzá —
minden route explicit módon "felsorolja", milyen middleware-eken megy át.

```java
// Spring — egyetlen deklaratív konfigurációs osztály írja le a TELJES alkalmazás security szabályait
@Configuration
@EnableWebSecurity
public class SecurityConfig {

    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http
            .csrf(csrf -> csrf.disable())   // stateless JWT API-nál nincs szükség CSRF védelemre
            .sessionManagement(session ->
                session.sessionCreationPolicy(SessionCreationPolicy.STATELESS))
            .authorizeHttpRequests(auth -> auth
                .requestMatchers("/api/auth/register", "/api/auth/login").permitAll()
                .anyRequest().authenticated()
            );

        return http.build();
    }
}
```

Itt egyetlen `HttpSecurity` objektumon láncolt (fluent) metódushívásokkal írod le **az egész
alkalmazásra** vonatkozó szabályokat: melyik útvonal publikus, melyik igényel autentikációt,
hogyan kezeled a session-t. Ez a deklaratív, központosított stílus a fő különbség — nem kell
minden route-nál külön-külön middleware-t felsorolnod, egy helyen látod az egész biztonsági
modellt.

## A `SecurityFilterChain` mint bean

Fontos: a `filterChain` metódus egy **Spring bean-t** ad vissza (`@Bean` annotációval) — ez
illeszkedik abba, amit a [DI fejezetben](../01-spring-boot-alapok/02-dependency-injection.md)
tanultál a `@Configuration` + `@Bean` mintáról. A Spring Security a háttérben ezt a bean-t veszi
fel a saját beépített filter láncába, ami minden HTTP kérés előtt lefut, még azelőtt, hogy
bármelyik `@RestController` metódusa meghívódna.

## `csrf`, `sessionManagement`, `authorizeHttpRequests`

| Beállítás | Mit csinál | Laravel megfelelő |
|---|---|---|
| `.csrf(csrf -> csrf.disable())` | kikapcsolja a CSRF tokenes védelmet | Laravel API route-oknál (`routes/api.php`) ez eleve nincs bekapcsolva; web route-oknál (`routes/web.php`) a `VerifyCsrfToken` middleware felel meg neki |
| `.sessionManagement(...STATELESS)` | nem hoz létre/használ HTTP session-t, minden kérés önmagában hitelesítendő | ez felel meg a Sanctum "token-alapú" (nem session-cookie-alapú) módjának |
| `.authorizeHttpRequests(...)` | melyik útvonalhoz kell hitelesítés | `->middleware('auth:sanctum')` a route csoporton |

A `STATELESS` beállítás kulcsfontosságú JWT-alapú API-knál: azt mondja a Spring Security-nek,
hogy **ne** tartson fenn szerver oldali session állapotot — minden kérésnél a JWT tokenből kell
újra megállapítani, ki a hívó. Erről részletesen a [JWT autentikáció](03-jwt-autentikacio.md)
fejezetben lesz szó.

## Miért nem osztályokból áll össze, mint a Laravel middleware?

Laravel-ben minden middleware egy önálló osztály `handle($request, Closure $next)` metódussal,
amit láncba fűzöl. Spring Security-ben is léteznek egyedi filterek (`OncePerRequestFilter`
leszármazottak, ezt is használni fogod a JWT-nél), de az **alapkonfiguráció** — hogy mely
útvonalak nyilvánosak, milyen session-kezelést használsz, milyen auth providereket regisztrálsz —
egyetlen `HttpSecurity` builder-en keresztül, deklaratívan történik. Ez elsőre szokatlan lehet, de
gyorsan átlátod: egyetlen fájlban (`SecurityConfig`) látod a teljes biztonsági modellt, nem kell
végigkeresgélned route fájlokat és middleware regisztrációkat.

---

[← Fázis index](../04-spring-security.md) · [Főoldal](../../README.md) · Következő: [02. Felhasználó és jelszó →](02-felhasznalo-es-jelszo.md)
