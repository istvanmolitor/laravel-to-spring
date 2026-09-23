[← Előző: 03. JWT autentikáció](03-jwt-autentikacio.md) · [Fázis index](../04-spring-security.md) · [Főoldal](../../README.md) · Következő: [05. CORS →](05-cors.md)

# 4.4 — Jogosultságkezelés

## Role-alapú védelem: `@PreAuthorize`

```php
// Laravel Gate
Gate::define('manage-todos', fn (User $user) => $user->role === 'admin');

if (Gate::allows('manage-todos')) { ... }
```

```java
// Spring Security
@PreAuthorize("hasRole('ADMIN')")
@DeleteMapping("/api/admin/todos/{id}")
public void deleteAnyTodo(@PathVariable Long id) { ... }
```

A `@PreAuthorize` a metódus **meghívása előtt** ellenőrzi a megadott SpEL (Spring Expression
Language) kifejezést — ha hamis, `AccessDeniedException`-t dob, mielőtt a metódus törzse
egyáltalán lefutna. Ahhoz, hogy ez működjön, a `SecurityConfig` osztályon engedélyezned kell:

```java
@Configuration
@EnableMethodSecurity   // ez kapcsolja be a @PreAuthorize feldolgozását
public class SecurityConfig { ... }
```

`hasRole('ADMIN')` és `hasAuthority('ROLE_ADMIN')` ugyanazt csinálja — a `hasRole` automatikusan
hozzáfűzi a `ROLE_` prefixet, a `hasAuthority` nem. Ez apró, de gyakori zavart okozó részlet:
kezdőként maradj a `hasRole`-nál, amíg nincs okod a különbségre építeni.

## Tulajdonos-alapú jogosultság — nem csak role kérdés

A Todo API-ban a legfontosabb szabály nem az, hogy "van-e ADMIN role-od", hanem hogy **a saját
Todo-idat éred-e el**. Ez a Laravel Policy klasszikus esete:

```php
// app/Policies/TodoPolicy.php
class TodoPolicy
{
    public function view(User $user, Todo $todo): bool
    {
        return $user->id === $todo->user_id;
    }
}

// Controller
public function show(Todo $todo)
{
    $this->authorize('view', $todo);
    return new TodoResource($todo);
}
```

Spring-ben ennek **nincs egyetlen kanonikus, beépített megfelelője** — két elterjedt megközelítés
van, és Spring-ben inkább az elsőt szokás választani API-knál:

**1. Explicit ellenőrzés a Service rétegben** (ajánlott, egyszerűbb és olvashatóbb):

```java
@Service
public class TodoService {

    public Todo getByIdForCurrentUser(Long todoId) {
        String currentEmail = SecurityContextHolder.getContext()
            .getAuthentication().getName();

        Todo todo = todoRepository.findById(todoId)
            .orElseThrow(() -> new TodoNotFoundException(todoId));

        if (!todo.getUser().getEmail().equals(currentEmail)) {
            throw new AccessDeniedException("Ez nem a te Todo-d");
        }

        return todo;
    }
}
```

**2. `@PreAuthorize` SpEL kifejezéssel, ami a paraméterre hivatkozik** (tömörebb, de nehezebben
olvasható/tesztelhető, mert a logika egy string kifejezésbe van rejtve):

```java
@PreAuthorize("@todoSecurity.isOwner(#todoId, authentication.name)")
@GetMapping("/api/todos/{todoId}")
public TodoResponse getTodo(@PathVariable Long todoId) { ... }
```

Kezdőként az **1. megközelítést** javasolt választani — a Service rétegbe írt explicit ellenőrzés
könnyebben olvasható, egyszerűbben tesztelhető (lásd az [5. fázis:
Tesztelés](../05-teszteles.md)-t), és nem igényel SpEL kifejezések elsajátítását.

## `SecurityContextHolder` — a "ki vagyok bejelentkezve" elérése

Ez az, amiből a [JWT filter](03-jwt-autentikacio.md) beállította az aktuális usert:

```java
Authentication authentication = SecurityContextHolder.getContext().getAuthentication();
String email = authentication.getName();  // a UserDetails username-je (nálunk: email)
```

Ez a Laravel `auth()->user()` / `Auth::id()` közvetlen megfelelője — csak itt nem egy globális
helper függvényt hívsz, hanem egy statikus context holderből olvasod ki, ami a szálhoz (thread)
kötött állapotot tárol a kérés teljes életciklusára.

---

[← Előző: 03. JWT autentikáció](03-jwt-autentikacio.md) · [Fázis index](../04-spring-security.md) · [Főoldal](../../README.md) · Következő: [05. CORS →](05-cors.md)
