[← Előző fázis: 03. Validáció, hibakezelés, réteges architektúra](03-validacio-hibakezeles.md) · [Főoldal](../README.md) · Következő fázis: [05. Tesztelés →](05-teszteles.md)

# 4. fázis — Spring Security

Időtartam: ~2-3 hét

Cél: a Sanctum/Passport tudásod átültetése — ez az egyik legmeredekebb tanulási görbe lesz.

Ez a fázis alfejezetekre van bontva, mindegyik konkrét PHP/Laravel összehasonlításokkal és
kódpéldákkal. Haladj sorban — a záró gyakorlat a korábbi fázisokban felépített Todo API-hoz ad
teljes autentikációt:

1. [Security alapok és a filter lánc](04-spring-security/01-security-alapok.md) —
   `SecurityFilterChain`, `HttpSecurity` DSL — a Laravel middleware pipeline deklaratív megfelelője
2. [Felhasználó és jelszó](04-spring-security/02-felhasznalo-es-jelszo.md) —
   `UserDetailsService`, `PasswordEncoder`/BCrypt — a `Hash::make()`/`Authenticatable` explicit
   Java megfelelője
3. [JWT autentikáció](04-spring-security/03-jwt-autentikacio.md) —
   stateless token generálás/validálás, `OncePerRequestFilter` — a Sanctum stateful tokenjeivel
   szembeállítva
4. [Jogosultságkezelés](04-spring-security/04-jogosultsagkezeles.md) —
   `@PreAuthorize`, role-alapú és tulajdonos-alapú védelem — a Laravel Policy/Gate megfelelője
5. [CORS](04-spring-security/05-cors.md) —
   `CorsConfigurationSource` — a `config/cors.php` Java megfelelője
6. [Záró gyakorlat: JWT auth](04-spring-security/06-gyakorlat-jwt-auth.md) —
   regisztráció/login végpontok, teljes `SecurityFilterChain` összerakása, Todo lekérdezések
   szűrése a bejelentkezett userre

---

[← Előző fázis: 03. Validáció, hibakezelés, réteges architektúra](03-validacio-hibakezeles.md) · [Főoldal](../README.md) · Következő fázis: [05. Tesztelés →](05-teszteles.md)
