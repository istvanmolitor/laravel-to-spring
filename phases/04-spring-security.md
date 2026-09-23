[← Előző: 3. Validáció, hibakezelés, réteges architektúra](03-validacio-hibakezeles.md) · [Főoldal](../README.md) · Következő: [05. Tesztelés →](05-teszteles.md)

# 4. fázis — Spring Security

Időtartam: ~2-3 hét

Cél: a Sanctum/Passport tudásod átültetése — ez az egyik legmeredekebb tanulási görbe lesz.

- `SecurityFilterChain` bean-ek — a middleware pipeline megfelelője, de deklaratívabb
- Alap autentikáció, majd JWT-alapú stateless auth (`spring-boot-starter-security` + `jjwt` vagy
  `nimbus-jose-jwt`) — ez a Sanctum token-alapú auth párja
- `UserDetailsService`, `PasswordEncoder` (BCrypt — ugyanaz, mint Laravel alatt)
- Jogosultságkezelés: `@PreAuthorize`, role/authority alapú védelem — a Laravel Policy/Gate
  megfelelője, csak annotáció-alapú
- CORS konfiguráció — hasonló elven, mint Laravel `cors.php`, csak Java konfigban

## Gyakorlat

Adj JWT auth-ot a Todo API-hoz, user regisztráció/login endpointtal, védett route-okkal.

---

[← Előző: 3. Validáció, hibakezelés, réteges architektúra](03-validacio-hibakezeles.md) · [Főoldal](../README.md) · Következő: [05. Tesztelés →](05-teszteles.md)
