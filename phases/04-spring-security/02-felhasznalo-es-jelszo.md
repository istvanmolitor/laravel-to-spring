[← Előző: 01. Security alapok](01-security-alapok.md) · [Fázis index](../04-spring-security.md) · [Főoldal](../../README.md) · Következő: [03. JWT autentikáció →](03-jwt-autentikacio.md)

# 4.2 — Felhasználó-kezelés és jelszavak

## `UserDetailsService` — hogyan tölti be Spring a felhasználót

Laravel-ben az `Authenticatable` trait-et implementáló `User` Eloquent modell "magától" tudja,
hogyan kell betölteni és hitelesíteni egy felhasználót — a keretrendszer a modellt közvetlenül
használja. Spring Security-ben ez egy explicit **interfész implementációval** történik:

```java
public interface UserDetailsService {
    UserDetails loadUserByUsername(String username) throws UsernameNotFoundException;
}
```

```java
@Service
public class AppUserDetailsService implements UserDetailsService {

    private final UserRepository userRepository;

    public AppUserDetailsService(UserRepository userRepository) {
        this.userRepository = userRepository;
    }

    @Override
    public UserDetails loadUserByUsername(String email) {
        User user = userRepository.findByEmail(email)
            .orElseThrow(() -> new UsernameNotFoundException("Nincs ilyen felhasználó: " + email));

        return org.springframework.security.core.userdetails.User
            .withUsername(user.getEmail())
            .password(user.getPasswordHash())
            .authorities(Collections.emptyList())  // itt adnál role-okat, ha lennének
            .build();
    }
}
```

Fontos, hogy a paraméter neve `username`, még akkor is, ha nálad (mint a legtöbb modern
alkalmazásban) valójában **email** az azonosító — ez csak a Spring Security történelmi
elnevezése, a mögöttes logika bármi lehet, amit te írsz bele.

Ez a minta a Laravel `Authenticatable` trait-jének **explicit, kézzel megírt** megfelelője — ott a
keretrendszer konvencióból tudja, hogyan kell egy usert betölteni (`email` + `password` oszlop),
itt neked kell megírnod a lekérdezés logikáját, cserébe teljes kontrollod van rajta.

## `UserDetails` — mit "lát" a Security a userből

A `UserDetails` interfész az, amit a Spring Security belsőleg használ hitelesítéskor — nem
kötelező, hogy a saját `User` entitásod implementálja, mint fent láttad, elég egy beépített
`org.springframework.security.core.userdetails.User` objektumot visszaadnod. Haladóbb esetben a
saját `User` entitásod is implementálhatja közvetlenül (`implements UserDetails`), ekkor a
`getAuthorities()`, `isEnabled()`, `isAccountNonLocked()` stb. metódusokat neked kell
megvalósítanod.

## Jelszó hashelés: `PasswordEncoder`

```php
// Laravel
use Illuminate\Support\Facades\Hash;

$hashed = Hash::make($plainPassword);          // regisztrációkor
$isValid = Hash::check($plainPassword, $hashed); // bejelentkezéskor
```

```java
// Spring
@Bean
public PasswordEncoder passwordEncoder() {
    return new BCryptPasswordEncoder();
}

// regisztrációkor
String hashed = passwordEncoder.encode(plainPassword);

// bejelentkezéskor
boolean isValid = passwordEncoder.matches(plainPassword, hashed);
```

Jó hír: mindkét keretrendszer **alapból BCrypt**-et használ — az algoritmus és a biztonsági
garanciák gyakorlatilag azonosak, csak az API elnevezése más (`Hash::make`/`Hash::check` vs
`encode`/`matches`). A `PasswordEncoder`-t bean-ként regisztrálod (lásd
[Dependency Injection](../01-spring-boot-alapok/02-dependency-injection.md)), és a regisztrációs
`Service` osztályba konstruktor-injection-nel kapod meg, ugyanúgy, mint bármely más
függőséget.

## Regisztráció folyamata összerakva

```java
@Service
public class AuthService {

    private final UserRepository userRepository;
    private final PasswordEncoder passwordEncoder;

    public AuthService(UserRepository userRepository, PasswordEncoder passwordEncoder) {
        this.userRepository = userRepository;
        this.passwordEncoder = passwordEncoder;
    }

    public User register(String email, String rawPassword) {
        if (userRepository.findByEmail(email).isPresent()) {
            throw new IllegalStateException("Ez az email cím már foglalt");
        }

        User user = new User(email, passwordEncoder.encode(rawPassword));
        return userRepository.save(user);
    }
}
```

Ez majdnem sor-sorra megfelel annak, amit egy Laravel `RegisterController`/`AuthController`
`register()` metódusában írnál — a `Hash::make()` helyén `passwordEncoder.encode()` áll, az
Eloquent `User::create([...])` helyén `userRepository.save(new User(...))`.

---

[← Előző: 01. Security alapok](01-security-alapok.md) · [Fázis index](../04-spring-security.md) · [Főoldal](../../README.md) · Következő: [03. JWT autentikáció →](03-jwt-autentikacio.md)
