[← Előző: 02. Felhasználó és jelszó](02-felhasznalo-es-jelszo.md) · [Fázis index](../04-spring-security.md) · [Főoldal](../../README.md) · Következő: [04. Jogosultságkezelés →](04-jogosultsagkezeles.md)

# 4.3 — JWT-alapú stateless autentikáció

## Sanctum token vs JWT — a fő koncepcionális különbség

```php
// Laravel Sanctum — bejelentkezéskor
$token = $user->createToken('api-token')->plainTextToken;
// a token egy adatbázis sorra mutat (personal_access_tokens tábla),
// minden kérésnél a Sanctum middleware DB lookup-ot végez, hogy a token érvényes-e
```

A Sanctum token **stateful**: a token maga csak egy kulcs, a tényleges hitelesítő adat az
adatbázisban van. Ez azt jelenti, hogy egy tokent bármikor azonnal vissza tudsz vonni
(`$token->delete()`), mert a szerver minden kéréskor ellenőrzi az adatbázist.

A klasszikus **JWT (JSON Web Token)** ezzel szemben **stateless**: a token maga tartalmazza az
összes szükséges adatot (pl. user ID, lejárati idő), digitálisan aláírva egy titkos kulccsal. A
szerver a token érvényességét **kizárólag az aláírás ellenőrzésével** dönti el, adatbázis lookup
nélkül.

| Szempont | Sanctum token | JWT (stateless) |
|---|---|---|
| Hol tárolódik az érvényesség | adatbázisban | a tokenben magában (aláírással védve) |
| Azonnali visszavonás | igen, egyszerű (`token->delete()`) | nem triviális (kell külön blacklist vagy rövid lejárat) |
| Skálázhatóság több szerver közt | DB lookup minden kérésnél (terhelés) | nincs DB lookup, könnyebben skálázható |
| Tipikus lejárat | hosszú, kézzel törölhető | rövid (percek-órák) + refresh token minta |

Kezdőként elég, ha tudod: a JWT gyorsabb és skálázhatóbb, de a visszavonás nehézkesebb — ezért
gyakori a "rövid élettartamú access token + hosszabb élettartamú refresh token" páros, amit
haladóbb szinten érdemes majd bevezetni.

## Függőség

```xml
<dependency>
    <groupId>io.jsonwebtoken</groupId>
    <artifactId>jjwt-api</artifactId>
    <version>0.12.5</version>
</dependency>
<dependency>
    <groupId>io.jsonwebtoken</groupId>
    <artifactId>jjwt-impl</artifactId>
    <version>0.12.5</version>
    <scope>runtime</scope>
</dependency>
<dependency>
    <groupId>io.jsonwebtoken</groupId>
    <artifactId>jjwt-jackson</artifactId>
    <version>0.12.5</version>
    <scope>runtime</scope>
</dependency>
```

## Token generálás bejelentkezéskor

```java
@Service
public class JwtService {

    private final SecretKey key = Keys.hmacShaKeyFor(
        "ez-egy-legalabb-256-bites-titkos-kulcs-legyen-production-ban".getBytes());

    public String generateToken(String email) {
        return Jwts.builder()
            .subject(email)
            .issuedAt(new Date())
            .expiration(new Date(System.currentTimeMillis() + 1000 * 60 * 60))  // 1 óra
            .signWith(key)
            .compact();
    }

    public String extractEmail(String token) {
        return Jwts.parser()
            .verifyWith(key)
            .build()
            .parseSignedClaims(token)
            .getPayload()
            .getSubject();
    }

    public boolean isValid(String token) {
        try {
            Jwts.parser().verifyWith(key).build().parseSignedClaims(token);
            return true;
        } catch (JwtException | IllegalArgumentException e) {
            return false;
        }
    }
}
```

Amit ez a Laravel `$user->createToken(...)` egysoros hívásával szemben látsz: itt te írod meg a
token felépítését (mit tartalmaz, mennyi ideig érvényes, mivel van aláírva) — ez több kód, de
teljes kontroll. Production kódban a titkos kulcsot **soha** ne írd be a forráskódba, hanem
`application.yml`/környezeti változóból olvasd be (lásd
[Konfiguráció kezelése](../01-spring-boot-alapok/04-konfiguracio.md)).

## A JWT filter — minden kérésnél lefut

```java
@Component
public class JwtAuthFilter extends OncePerRequestFilter {

    private final JwtService jwtService;
    private final UserDetailsService userDetailsService;

    public JwtAuthFilter(JwtService jwtService, UserDetailsService userDetailsService) {
        this.jwtService = jwtService;
        this.userDetailsService = userDetailsService;
    }

    @Override
    protected void doFilterInternal(HttpServletRequest request, HttpServletResponse response,
                                     FilterChain chain) throws ServletException, IOException {
        String authHeader = request.getHeader("Authorization");

        if (authHeader == null || !authHeader.startsWith("Bearer ")) {
            chain.doFilter(request, response);
            return;
        }

        String token = authHeader.substring(7);

        if (jwtService.isValid(token)) {
            String email = jwtService.extractEmail(token);
            UserDetails userDetails = userDetailsService.loadUserByUsername(email);

            var authToken = new UsernamePasswordAuthenticationToken(
                userDetails, null, userDetails.getAuthorities());
            SecurityContextHolder.getContext().setAuthentication(authToken);
        }

        chain.doFilter(request, response);
    }
}
```

Ezt a filtert a [Security alapoknál](01-security-alapok.md) látott `SecurityFilterChain`
konfigurációba kell beillesztened:

```java
@Bean
public SecurityFilterChain filterChain(HttpSecurity http, JwtAuthFilter jwtAuthFilter) throws Exception {
    http
        .csrf(csrf -> csrf.disable())
        .sessionManagement(s -> s.sessionCreationPolicy(SessionCreationPolicy.STATELESS))
        .authorizeHttpRequests(auth -> auth
            .requestMatchers("/api/auth/**").permitAll()
            .anyRequest().authenticated()
        )
        .addFilterBefore(jwtAuthFilter, UsernamePasswordAuthenticationFilter.class);

    return http.build();
}
```

Ez a `SecurityContextHolder.getContext().setAuthentication(...)` sor az, ami "beállítja", ki a
bejelentkezett user a kérés hátralévő részére — ezt fogod kiolvasni a
[jogosultságkezelésnél](04-jogosultsagkezeles.md) és a Todo lekérdezések szűrésénél.

---

[← Előző: 02. Felhasználó és jelszó](02-felhasznalo-es-jelszo.md) · [Fázis index](../04-spring-security.md) · [Főoldal](../../README.md) · Következő: [04. Jogosultságkezelés →](04-jogosultsagkezeles.md)
