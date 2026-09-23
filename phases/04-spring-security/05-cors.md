[← Előző: 04. Jogosultságkezelés](04-jogosultsagkezeles.md) · [Fázis index](../04-spring-security.md) · [Főoldal](../../README.md) · Következő: [06. Gyakorlat: JWT auth →](06-gyakorlat-jwt-auth.md)

# 4.5 — CORS konfiguráció

Ha a frontended (pl. egy külön porton futó Vue/React app) más originről hívja az API-t, mint
ahonnan kiszolgálod, be kell állítanod a CORS (Cross-Origin Resource Sharing) szabályokat —
ugyanaz a probléma, amit Laravel-ben a `config/cors.php` fájllal oldasz meg.

## Laravel `config/cors.php`

```php
return [
    'paths' => ['api/*'],
    'allowed_methods' => ['*'],
    'allowed_origins' => ['http://localhost:5173'],
    'allowed_headers' => ['*'],
    'supports_credentials' => true,
];
```

## Spring Security CORS konfiguráció

```java
@Bean
public CorsConfigurationSource corsConfigurationSource() {
    CorsConfiguration config = new CorsConfiguration();
    config.setAllowedOrigins(List.of("http://localhost:5173"));
    config.setAllowedMethods(List.of("GET", "POST", "PUT", "DELETE", "OPTIONS"));
    config.setAllowedHeaders(List.of("*"));
    config.setAllowCredentials(true);

    UrlBasedCorsConfigurationSource source = new UrlBasedCorsConfigurationSource();
    source.registerCorsConfiguration("/api/**", config);
    return source;
}
```

Ezt a bean-t be kell kötnöd a [Security alapoknál](01-security-alapok.md) látott
`SecurityFilterChain`-be:

```java
@Bean
public SecurityFilterChain filterChain(HttpSecurity http,
                                        CorsConfigurationSource corsConfigurationSource) throws Exception {
    http
        .cors(cors -> cors.configurationSource(corsConfigurationSource))
        .csrf(csrf -> csrf.disable())
        // ... a többi beállítás a korábbi fejezetekből
        ;
    return http.build();
}
```

## A `paths`/`registerCorsConfiguration` mintaillesztés

Figyeld meg, hogy mindkét keretrendszerben **útvonal-minta alapján** korlátozod a CORS
szabályokat: Laravel-ben a `paths => ['api/*']`, Spring-ben a
`source.registerCorsConfiguration("/api/**", config)`. Ez azt jelzi, hogy csak az `/api/`
prefixű végpontokra vonatkozik a szabály — statikus erőforrásokra vagy egyéb route-okra nem.

## Gyakori hiba: CORS vs 401/403 összekeverése

Ha a böngésző konzolban CORS hibát látsz, az **nem** azt jelenti, hogy hibás a JWT tokened vagy a
jogosultságod — a CORS ellenőrzés a böngészőben, **a válasz fejlécei alapján** történik, még
azelőtt, hogy a hitelesítési logikád egyáltalán lefutna a preflight (`OPTIONS`) kérésnél. Ha CORS
hibát kapsz, először mindig a `corsConfigurationSource` beállítást ellenőrizd, ne a JWT filtert.

---

[← Előző: 04. Jogosultságkezelés](04-jogosultsagkezeles.md) · [Fázis index](../04-spring-security.md) · [Főoldal](../../README.md) · Következő: [06. Gyakorlat: JWT auth →](06-gyakorlat-jwt-auth.md)
