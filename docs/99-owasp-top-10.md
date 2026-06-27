# OWASP Top 10 for Java

[← Back to README](../README.md)

---

The **OWASP Top 10** is the industry-standard list of the most critical web application security risks. This doc covers each category with Java/Spring examples of the vulnerability and how to prevent it.

---

## A01 — Broken Access Control

Users accessing resources or actions they shouldn't be allowed to.

```java
// VULNERABLE — no ownership check
@GetMapping("/api/orders/{id}")
public Order getOrder(@PathVariable UUID id) {
    return orderRepo.findById(id).orElseThrow();  // any user can read any order
}

// FIXED — verify the order belongs to the authenticated user
@GetMapping("/api/orders/{id}")
public Order getOrder(@PathVariable UUID id,
                      @AuthenticationPrincipal UserDetails user) {
    Order order = orderRepo.findById(id).orElseThrow();
    if (!order.getCustomerId().equals(user.getUsername())) {
        throw new ResponseStatusException(HttpStatus.FORBIDDEN);
    }
    return order;
}
```

```java
// Spring Security method-level access control
@PreAuthorize("hasRole('ADMIN') or #order.customerId == authentication.name")
public Order updateOrder(Order order, UpdateCommand cmd) { ... }

// Repository-level filtering — never return what you don't own
@Query("SELECT o FROM Order o WHERE o.id = :id AND o.customerId = :userId")
Optional<Order> findByIdAndCustomerId(UUID id, String userId);
```

---

## A02 — Cryptographic Failures

Sensitive data transmitted or stored without adequate encryption.

```java
// VULNERABLE — MD5 / SHA-1 for passwords
String hash = DigestUtils.md5Hex(password);   // NEVER do this

// FIXED — bcrypt with cost factor ≥ 12
@Bean
public PasswordEncoder passwordEncoder() {
    return new BCryptPasswordEncoder(12);
}

// VULNERABLE — storing credit card numbers in plain text
user.setCreditCard(cardNumber);

// FIXED — encrypt sensitive fields at rest
@Convert(converter = AesEncryptedConverter.class)
private String creditCardLast4;   // store only last 4, encrypt what you must keep
```

```yaml
# Force HTTPS
server:
  ssl:
    enabled: true
  http2:
    enabled: true

# HSTS header — tell browsers to always use HTTPS
spring:
  security:
    headers:
      hsts:
        include-subdomains: true
        max-age-seconds: 31536000
```

---

## A03 — Injection

Untrusted data sent to an interpreter (SQL, LDAP, OS command, etc.).

```java
// VULNERABLE — SQL injection
String query = "SELECT * FROM users WHERE name = '" + username + "'";
// username = "' OR '1'='1" → returns all users

// FIXED — parameterised query
@Query("SELECT u FROM User u WHERE u.username = :username")
Optional<User> findByUsername(@Param("username") String username);

// FIXED — JDBC PreparedStatement
PreparedStatement ps = conn.prepareStatement(
    "SELECT * FROM users WHERE username = ?");
ps.setString(1, username);

// VULNERABLE — OS command injection
Runtime.getRuntime().exec("convert " + userFilename);

// FIXED — never pass user input to exec(); use ProcessBuilder with arg list
new ProcessBuilder("convert", sanitizedFilename).start();
```

---

## A04 — Insecure Design

Missing or ineffective security controls at the design level.

```java
// Principle of least privilege — only expose what's needed
@RestController
@RequestMapping("/api/users")
public class UserController {

    // VULNERABLE — exposes internal fields including password hash
    @GetMapping("/{id}")
    public User getUser(@PathVariable Long id) {
        return userRepo.findById(id).orElseThrow();
    }

    // FIXED — return a DTO that excludes sensitive fields
    @GetMapping("/{id}")
    public UserResponse getUser(@PathVariable Long id) {
        return userRepo.findById(id)
            .map(UserResponse::from)
            .orElseThrow();
    }
}

// Rate-limit sensitive endpoints (see Rate Limiting doc)
@PostMapping("/api/auth/login")
@RateLimited(maxRequests = 10, windowSeconds = 60)
public ResponseEntity<TokenResponse> login(@RequestBody LoginRequest req) { ... }
```

---

## A05 — Security Misconfiguration

Defaults left enabled, verbose error messages, missing security headers.

```java
@Bean
public SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
    http
        // Security headers
        .headers(headers -> headers
            .contentSecurityPolicy(csp -> csp
                .policyDirectives("default-src 'self'; script-src 'self'"))
            .frameOptions(frame -> frame.deny())           // X-Frame-Options: DENY
            .xssProtection(xss -> xss.enable())
            .referrerPolicy(ref -> ref
                .policy(ReferrerPolicyHeaderWriter.ReferrerPolicy.STRICT_ORIGIN)))
        // Never expose stack traces in API responses
        .exceptionHandling(ex -> ex
            .authenticationEntryPoint((req, res, e) ->
                res.sendError(HttpServletResponse.SC_UNAUTHORIZED)))
    return http.build();
}
```

```yaml
# application.yml — disable actuator endpoints that expose internals
management:
  endpoints:
    web:
      exposure:
        include: health, info   # NOT 'env', 'beans', 'mappings' in production
  endpoint:
    health:
      show-details: never       # don't reveal DB/disk status to unauthenticated users

# Never leak exception details
server:
  error:
    include-stacktrace: never
    include-message: never
    include-binding-errors: never
```

---

## A06 — Vulnerable and Outdated Components

Using libraries with known CVEs.

```xml
<!-- Dependency version scanner -->
<plugin>
    <groupId>org.owasp</groupId>
    <artifactId>dependency-check-maven</artifactId>
    <version>10.0.3</version>
    <configuration>
        <failBuildOnCVSS>7</failBuildOnCVSS>   <!-- fail on HIGH severity -->
        <suppressionFile>dependency-check-suppressions.xml</suppressionFile>
    </configuration>
    <executions>
        <execution>
            <goals><goal>check</goal></goals>
        </execution>
    </executions>
</plugin>
```

```bash
mvn dependency-check:check   # generates target/dependency-check-report.html
```

GitHub Dependabot auto-creates PRs for vulnerable dependencies — enable it in `.github/dependabot.yml`:

```yaml
version: 2
updates:
  - package-ecosystem: maven
    directory: /
    schedule:
      interval: weekly
    open-pull-requests-limit: 10
```

---

## A07 — Identification and Authentication Failures

Weak passwords, no MFA, broken session management.

```java
// FIXED — strong password policy via Bean Validation
@Pattern(
    regexp = "^(?=.*[a-z])(?=.*[A-Z])(?=.*\\d)(?=.*[@$!%*?&])[A-Za-z\\d@$!%*?&]{12,}$",
    message = "Password must be 12+ chars with upper, lower, digit, and special character"
)
String password;

// FIXED — invalidate session on login (session fixation prevention)
http.sessionManagement(session -> session
    .sessionFixation(SessionManagementConfigurer.SessionFixationConfigurer::newSession)
    .maximumSessions(1)            // one active session per user
    .maxSessionsPreventsLogin(false));  // new login invalidates old session

// FIXED — JWT with short expiry + refresh token rotation
public TokenPair issueTokens(User user) {
    String accessToken  = jwtUtil.generate(user, Duration.ofMinutes(15));
    String refreshToken = refreshTokenService.create(user);   // stored, rotated on use
    return new TokenPair(accessToken, refreshToken);
}
```

---

## A08 — Software and Data Integrity Failures

Deserializing untrusted data, using unsigned artifacts.

```java
// VULNERABLE — Java object deserialization of untrusted data
ObjectInputStream ois = new ObjectInputStream(untrustedStream);
Object obj = ois.readObject();   // arbitrary code execution if malicious

// FIXED — never deserialize untrusted Java serialized objects
// Use JSON (Jackson) with a known type instead:
Order order = objectMapper.readValue(json, Order.class);

// FIXED — validate before deserializing
@JsonTypeInfo(use = JsonTypeInfo.Id.NAME)
@JsonSubTypes({
    @JsonSubTypes.Type(value = PlaceOrderCommand.class, name = "PlaceOrder"),
    @JsonSubTypes.Type(value = CancelOrderCommand.class, name = "CancelOrder")
})
public sealed interface OrderCommand permits PlaceOrderCommand, CancelOrderCommand {}
```

Verify artifact integrity in CI:

```yaml
- name: Verify Maven wrapper checksum
  run: sha256sum -c .mvn/wrapper/maven-wrapper.properties.sha256
```

---

## A09 — Security Logging and Monitoring Failures

Not logging security events; not alerting on suspicious patterns.

```java
@Component
public class SecurityAuditListener implements ApplicationListener<AbstractAuthenticationEvent> {

    @Override
    public void onApplicationEvent(AbstractAuthenticationEvent event) {
        if (event instanceof AuthenticationSuccessEvent e) {
            log.info("LOGIN_SUCCESS user={} ip={}",
                e.getAuthentication().getName(),
                getIp());
        } else if (event instanceof AuthenticationFailureBadCredentialsEvent e) {
            log.warn("LOGIN_FAILURE user={} ip={}",
                e.getAuthentication().getName(),
                getIp());
        }
    }
}

// Log security-relevant events — use structured logging for easy querying
log.warn("PRIVILEGE_ESCALATION_ATTEMPT user={} resource={} action={}",
    user, resourceId, action);
log.info("PASSWORD_CHANGED user={}", username);
log.info("MFA_ENABLED user={}", username);
```

---

## A10 — Server-Side Request Forgery (SSRF)

The server fetches a URL supplied by the user — attacker points it at internal services.

```java
// VULNERABLE — fetches any URL the user provides
@GetMapping("/api/proxy")
public String proxy(@RequestParam String url) {
    return restClient.get().uri(url).retrieve().body(String.class);
}
// Attacker: /api/proxy?url=http://169.254.169.254/latest/meta-data/
// → fetches AWS instance metadata credentials

// FIXED — allowlist of permitted hosts
@GetMapping("/api/proxy")
public String proxy(@RequestParam String url) {
    URI uri = URI.create(url);
    Set<String> allowed = Set.of("api.partner.com", "cdn.example.com");
    if (!allowed.contains(uri.getHost())) {
        throw new ResponseStatusException(HttpStatus.BAD_REQUEST,
            "URL host not permitted");
    }
    return restClient.get().uri(uri).retrieve().body(String.class);
}
```

---

## OWASP Summary

| # | Risk | Key Fix |
|---|------|---------|
| A01 | Broken Access Control | Ownership checks, `@PreAuthorize`, filter at repository layer |
| A02 | Cryptographic Failures | BCrypt ≥ 12 rounds, TLS everywhere, HSTS |
| A03 | Injection | Parameterised queries, `@Param`, never concat user input |
| A04 | Insecure Design | DTOs (never expose entities), rate limiting, least privilege |
| A05 | Security Misconfiguration | Security headers, hide stack traces, restrict Actuator |
| A06 | Vulnerable Components | OWASP Dependency-Check, Dependabot, pin versions |
| A07 | Auth Failures | Strong passwords, session fixation, short JWT TTL |
| A08 | Integrity Failures | No Java deserialization of untrusted data; use JSON |
| A09 | Logging Failures | Log auth events, failed access, privilege changes |
| A10 | SSRF | Allowlist permitted hosts; never fetch arbitrary user-supplied URLs |

---

[← Back to README](../README.md)
