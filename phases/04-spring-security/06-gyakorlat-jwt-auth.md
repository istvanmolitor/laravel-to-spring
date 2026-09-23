[← Előző: 05. CORS](05-cors.md) · [Fázis index](../04-spring-security.md) · [Főoldal](../../README.md) · Következő fázis: [05. Tesztelés →](../05-teszteles.md)

# 4.6 — Záró gyakorlat: JWT auth a Todo API-hoz

Add hozzá a JWT-alapú autentikációt a korábbi fázisokban felépített Todo API-hoz: user
regisztráció/login endpointtal, védett route-okkal, és úgy, hogy minden user csak a saját
Todo-it lássa.

## 1. lépés — Auth végpontok

```
POST /api/auth/register
POST /api/auth/login
```

**Regisztráció kérés:**

```json
{
  "email": "kovacs@example.com",
  "password": "titkosjelszo123"
}
```

**Regisztráció/login válasz:**

```json
{
  "token": "eyJhbGciOiJIUzI1NiJ9.eyJzdWIiOiJrb3ZhY3NAZXhhbXBsZS5jb20i...",
  "email": "kovacs@example.com"
}
```

```java
public record RegisterRequest(
    @NotBlank @Email String email,
    @NotBlank @Size(min = 8) String password
) {}

public record AuthResponse(String token, String email) {}

@RestController
@RequestMapping("/api/auth")
public class AuthController {

    private final AuthService authService;
    private final JwtService jwtService;

    public AuthController(AuthService authService, JwtService jwtService) {
        this.authService = authService;
        this.jwtService = jwtService;
    }

    @PostMapping("/register")
    public AuthResponse register(@Valid @RequestBody RegisterRequest request) {
        User user = authService.register(request.email(), request.password());
        return new AuthResponse(jwtService.generateToken(user.getEmail()), user.getEmail());
    }

    @PostMapping("/login")
    public AuthResponse login(@Valid @RequestBody LoginRequest request) {
        User user = authService.authenticate(request.email(), request.password());
        return new AuthResponse(jwtService.generateToken(user.getEmail()), user.getEmail());
    }
}
```

Ez a [Bean Validation](../03-validacio-hibakezeles/01-bean-validation.md) és a
[DTO](../03-validacio-hibakezeles/02-dto-k-es-mapping.md) fejezetekben tanult mintákat használja —
a `record` alapú DTO-k és a `@Valid` validáció itt is ugyanúgy alkalmazandók, mint a Todo
végpontoknál.

## 2. lépés — `SecurityFilterChain` teljes összerakása

```java
@Configuration
@EnableWebSecurity
@EnableMethodSecurity
public class SecurityConfig {

    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http,
                                            JwtAuthFilter jwtAuthFilter,
                                            CorsConfigurationSource corsConfigurationSource) throws Exception {
        http
            .cors(cors -> cors.configurationSource(corsConfigurationSource))
            .csrf(csrf -> csrf.disable())
            .sessionManagement(s -> s.sessionCreationPolicy(SessionCreationPolicy.STATELESS))
            .authorizeHttpRequests(auth -> auth
                .requestMatchers("/api/auth/**").permitAll()
                .anyRequest().authenticated()
            )
            .addFilterBefore(jwtAuthFilter, UsernamePasswordAuthenticationFilter.class);

        return http.build();
    }

    @Bean
    public PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder();
    }
}
```

Ez a [Security alapok](01-security-alapok.md), a [JWT autentikáció](03-jwt-autentikacio.md) és a
[CORS](05-cors.md) fejezetekben külön-külön bemutatott darabok összeillesztése egyetlen
konfigurációs osztályba.

## 3. lépés — Todo lekérdezések szűrése a bejelentkezett userre

A [Réteges architektúra](../03-validacio-hibakezeles/04-reteges-architektura.md) fejezetben
felépített `TodoService`-t egészítsd ki: minden metódus a `SecurityContextHolder`-ből olvassa ki
az aktuális usert, és csak az ő Todo-ihoz enged hozzáférést (lásd
[Jogosultságkezelés](04-jogosultsagkezeles.md)):

```java
@Service
public class TodoService {

    private final TodoRepository todoRepository;
    private final UserRepository userRepository;

    // ... konstruktor

    private User currentUser() {
        String email = SecurityContextHolder.getContext().getAuthentication().getName();
        return userRepository.findByEmail(email).orElseThrow();
    }

    public List<Todo> findAllForCurrentUser() {
        return todoRepository.findByUserId(currentUser().getId());
    }

    public Todo create(TodoRequest request) {
        Todo todo = new Todo(request.title(), request.description(), currentUser());
        return todoRepository.save(todo);
    }

    public Todo getByIdOrThrow(Long id) {
        Todo todo = todoRepository.findById(id)
            .orElseThrow(() -> new TodoNotFoundException(id));

        if (!todo.getUser().getId().equals(currentUser().getId())) {
            throw new AccessDeniedException("Ez nem a te Todo-d");
        }
        return todo;
    }
}
```

## 4. lépés — Tesztelés `curl`-lal

```bash
# regisztráció
curl -X POST http://localhost:8080/api/auth/register \
  -H "Content-Type: application/json" \
  -d '{"email":"kovacs@example.com","password":"titkosjelszo123"}'

# a válaszban kapott tokent felhasználva:
curl http://localhost:8080/api/todos \
  -H "Authorization: Bearer eyJhbGciOiJIUzI1NiJ9..."

# token nélkül — 401-et kell kapnod
curl http://localhost:8080/api/todos
```

## Ellenőrző kérdések a fázis lezárásához

1. Mi a fő különbség a Sanctum token és a klasszikus JWT állapot-kezelése között, és ennek milyen
   következménye van a token visszavonhatóságára?
2. Miért kell `STATELESS` session policy-t beállítani JWT-alapú API-nál?
3. Mikor válaszd a Service rétegbeli explicit ellenőrzést a tulajdonos-alapú jogosultsághoz, és
   mikor a `@PreAuthorize` SpEL kifejezést?
4. Miért nem elég csak `hasRole('ADMIN')`-ot ellenőrizni egy olyan végpontnál, ahol a userek csak
   a saját adataikhoz férhetnek hozzá?

---

Ezzel a Todo API rendelkezik teljes autentikációval és tulajdonos-alapú jogosultsággal. Az
[5. fázisban](../05-teszteles.md) ehhez a rendszerhez írsz majd unit és integrációs teszteket, a
[6. fázisban](../06-async-esemenyek.md) pedig a regisztráció során aszinkron módon küldött
üdvözlő emailt vezeted be.

---

[← Előző: 05. CORS](05-cors.md) · [Fázis index](../04-spring-security.md) · [Főoldal](../../README.md) · Következő fázis: [05. Tesztelés →](../05-teszteles.md)
