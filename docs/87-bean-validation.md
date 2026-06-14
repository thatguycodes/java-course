# Bean Validation (Jakarta Validation)

[← Back to README](../README.md)

---

**Jakarta Bean Validation** (formerly Java Bean Validation, JSR-380) provides a declarative way to validate objects using annotations. Spring Boot auto-configures validation for REST request bodies, method parameters, and service-layer objects.

---

## Maven Dependency

```xml
<!-- Included transitively with spring-boot-starter-web -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-validation</artifactId>
</dependency>
```

---

## Built-In Constraints

```java
public record CreateUserRequest(

    @NotBlank(message = "Username is required")
    @Size(min = 3, max = 50)
    String username,

    @NotBlank
    @Email(message = "Must be a valid email address")
    String email,

    @NotBlank
    @Size(min = 8, message = "Password must be at least 8 characters")
    @Pattern(regexp = ".*[A-Z].*", message = "Password must contain an uppercase letter")
    String password,

    @NotNull
    @Min(value = 18, message = "Must be at least 18")
    @Max(value = 120)
    Integer age,

    @NotNull
    @Past(message = "Date of birth must be in the past")
    LocalDate dateOfBirth,

    @NotEmpty
    @Size(max = 10)
    List<@NotBlank String> roles,

    @DecimalMin("0.00")
    @DecimalMax("999999.99")
    @Digits(integer = 6, fraction = 2)
    BigDecimal balance
) {}
```

### Common Constraints Quick Reference

| Annotation | Applies to | Meaning |
|------------|-----------|---------|
| `@NotNull` | Any | Field must not be null |
| `@NotBlank` | String | Not null, not empty, not whitespace-only |
| `@NotEmpty` | String, Collection | Not null, not empty |
| `@Size(min, max)` | String, Collection, array | Length/size within range |
| `@Min` / `@Max` | Numeric | Value range (inclusive) |
| `@DecimalMin` / `@DecimalMax` | Numeric, String | Decimal value range |
| `@Positive` / `@PositiveOrZero` | Numeric | Must be positive |
| `@Negative` / `@NegativeOrZero` | Numeric | Must be negative |
| `@Email` | String | Valid email format |
| `@Pattern(regexp)` | String | Matches regex |
| `@Past` / `@Future` | Date/Time | In the past/future |
| `@PastOrPresent` / `@FutureOrPresent` | Date/Time | Now or past/future |
| `@Digits(integer, fraction)` | Numeric | Max integer + fraction digits |
| `@AssertTrue` / `@AssertFalse` | boolean | Must be true/false |

---

## Validating REST Request Bodies

```java
@RestController
@RequestMapping("/api/users")
public class UserController {

    @PostMapping
    public ResponseEntity<UserResponse> create(
            @RequestBody @Valid CreateUserRequest request) {
        // If validation fails, MethodArgumentNotValidException is thrown
        // before this method body executes
        return ResponseEntity.status(201).body(userService.create(request));
    }

    @PutMapping("/{id}")
    public ResponseEntity<UserResponse> update(
            @PathVariable Long id,
            @RequestBody @Valid UpdateUserRequest request) {
        return ResponseEntity.ok(userService.update(id, request));
    }
}
```

### Global Validation Error Handler

```java
@RestControllerAdvice
public class ValidationExceptionHandler {

    @ExceptionHandler(MethodArgumentNotValidException.class)
    public ResponseEntity<ValidationErrorResponse> handleValidationErrors(
            MethodArgumentNotValidException ex) {

        Map<String, List<String>> errors = ex.getBindingResult()
            .getFieldErrors()
            .stream()
            .collect(Collectors.groupingBy(
                FieldError::getField,
                Collectors.mapping(FieldError::getDefaultMessage, Collectors.toList())));

        return ResponseEntity.badRequest()
            .body(new ValidationErrorResponse("Validation failed", errors));
    }

    @ExceptionHandler(ConstraintViolationException.class)
    public ResponseEntity<ValidationErrorResponse> handleConstraintViolations(
            ConstraintViolationException ex) {

        Map<String, List<String>> errors = ex.getConstraintViolations()
            .stream()
            .collect(Collectors.groupingBy(
                v -> v.getPropertyPath().toString(),
                Collectors.mapping(ConstraintViolation::getMessage, Collectors.toList())));

        return ResponseEntity.badRequest()
            .body(new ValidationErrorResponse("Validation failed", errors));
    }
}

public record ValidationErrorResponse(String message, Map<String, List<String>> errors) {}
```

**Response example:**
```json
{
  "message": "Validation failed",
  "errors": {
    "email": ["Must be a valid email address"],
    "password": ["Password must be at least 8 characters", "Password must contain an uppercase letter"],
    "age": ["Must be at least 18"]
  }
}
```

---

## Method-Level Validation

```java
@Service
@Validated   // enables method-level validation
public class UserService {

    public UserResponse create(@Valid CreateUserRequest request) { ... }

    public UserResponse findById(@Positive Long id) { ... }

    public List<UserResponse> findByAge(
            @Min(0) int minAge,
            @Max(150) int maxAge) { ... }
}
```

---

## Custom Constraint

```java
// 1. Define the annotation
@Target({ElementType.FIELD, ElementType.PARAMETER})
@Retention(RetentionPolicy.RUNTIME)
@Constraint(validatedBy = PhoneNumberValidator.class)
@Documented
public @interface PhoneNumber {
    String message() default "Invalid phone number";
    Class<?>[] groups() default {};
    Class<? extends Payload>[] payload() default {};
    String region() default "ZA";
}

// 2. Implement the validator
public class PhoneNumberValidator
        implements ConstraintValidator<PhoneNumber, String> {

    private String region;

    @Override
    public void initialize(PhoneNumber annotation) {
        this.region = annotation.region();
    }

    @Override
    public boolean isValid(String value, ConstraintValidatorContext ctx) {
        if (value == null) return true;  // @NotNull handles null check
        return value.matches("\\+?[0-9]{7,15}");
    }
}

// 3. Use it
public record ContactRequest(
    @NotBlank String name,
    @PhoneNumber(region = "US") String phone
) {}
```

---

## Cross-Field Validation

```java
// 1. Class-level annotation
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Constraint(validatedBy = PasswordMatchValidator.class)
public @interface PasswordMatch {
    String message() default "Passwords do not match";
    Class<?>[] groups() default {};
    Class<? extends Payload>[] payload() default {};
}

// 2. Class-level validator
public class PasswordMatchValidator
        implements ConstraintValidator<PasswordMatch, ChangePasswordRequest> {

    @Override
    public boolean isValid(ChangePasswordRequest req, ConstraintValidatorContext ctx) {
        if (req.newPassword() == null || req.confirmPassword() == null) return true;
        boolean match = req.newPassword().equals(req.confirmPassword());
        if (!match) {
            ctx.disableDefaultConstraintViolation();
            ctx.buildConstraintViolationWithTemplate("Passwords do not match")
               .addPropertyNode("confirmPassword")
               .addConstraintViolation();
        }
        return match;
    }
}

// 3. Apply at class level
@PasswordMatch
public record ChangePasswordRequest(
    @NotBlank String currentPassword,
    @NotBlank @Size(min = 8) String newPassword,
    @NotBlank String confirmPassword
) {}
```

---

## Validation Groups

Groups let you apply different constraints for different operations (create vs update):

```java
public interface OnCreate {}
public interface OnUpdate {}

public record UserRequest(

    @NotBlank(groups = OnCreate.class)  // required only on create
    String username,

    @NotNull(groups = {OnCreate.class, OnUpdate.class})
    @Email
    String email,

    @NotBlank(groups = OnCreate.class)  // required only on create
    String password
) {}

@PostMapping
public ResponseEntity<UserResponse> create(
        @RequestBody @Validated(OnCreate.class) UserRequest request) { ... }

@PutMapping("/{id}")
public ResponseEntity<UserResponse> update(
        @PathVariable Long id,
        @RequestBody @Validated(OnUpdate.class) UserRequest request) { ... }
```

---

## Cascaded Validation

Validate nested objects with `@Valid`:

```java
public record OrderRequest(

    @NotBlank
    String customerId,

    @NotEmpty
    @Valid   // cascade validation into each OrderLineRequest
    List<OrderLineRequest> lines
) {}

public record OrderLineRequest(

    @NotBlank
    String productId,

    @Positive
    int quantity,

    @NotNull @DecimalMin("0.01")
    BigDecimal price
) {}
```

---

## Programmatic Validation

```java
@Service
public class ValidationService {

    private final jakarta.validation.Validator validator;

    public ValidationService(jakarta.validation.Validator validator) {
        this.validator = validator;
    }

    public <T> void validate(T object) {
        Set<ConstraintViolation<T>> violations = validator.validate(object);
        if (!violations.isEmpty()) {
            throw new ConstraintViolationException(violations);
        }
    }
}
```

---

## Bean Validation Summary

| Feature | Detail |
|---------|--------|
| `@Valid` | Triggers validation on the annotated field or parameter |
| `@Validated` | Spring-specific; enables method-level validation and group support |
| `@NotBlank` vs `@NotEmpty` vs `@NotNull` | Blank checks whitespace; Empty checks length; Null only checks null |
| Custom constraint | `@Constraint(validatedBy = ...)` annotation + `ConstraintValidator<A, T>` |
| Cross-field | Class-level `@Constraint` with access to the whole object |
| Groups | Different constraints for create vs update via `@Validated(Group.class)` |
| Cascade | `@Valid` on nested objects or collection elements |
| Error handling | `MethodArgumentNotValidException` (MVC) / `ConstraintViolationException` (service) |

---

[← Back to README](../README.md)
