# REST API Design

[← Back to README](../README.md)

---

A well-designed REST API is predictable, consistent, and self-documenting. These conventions are the industry standard — follow them and consumers can integrate with minimal friction.

---

## Resource Naming

URLs identify **resources** (nouns), not actions (verbs). HTTP methods express the action.

```
# Good — plural nouns for collections
GET    /api/users           → list users
POST   /api/users           → create user
GET    /api/users/{id}      → get user
PUT    /api/users/{id}      → replace user
PATCH  /api/users/{id}      → partially update user
DELETE /api/users/{id}      → delete user

# Nested resources — relationships
GET    /api/users/{id}/orders          → orders for a user
POST   /api/users/{id}/orders          → create order for a user
GET    /api/users/{id}/orders/{orderId} → specific order

# Bad — verbs in the URL
POST /api/createUser        ✗
GET  /api/getUser/1         ✗
POST /api/deleteUser/1      ✗
```

### Naming conventions

- **Lowercase, hyphen-separated** for multi-word resources: `/api/order-items`, not `/api/orderItems`
- **Plural nouns** for collections: `/api/products`
- **Singular for singletons**: `/api/users/{id}/profile` (a user has one profile)

---

## HTTP Methods and Status Codes

### Methods

| Method | Purpose | Request Body | Idempotent? |
|--------|---------|-------------|------------|
| `GET` | Read | No | Yes |
| `POST` | Create | Yes | No |
| `PUT` | Replace (full update) | Yes | Yes |
| `PATCH` | Partial update | Yes | No |
| `DELETE` | Delete | No | Yes |

### Status codes

```
200 OK           — successful GET, PUT, PATCH
201 Created      — successful POST (include Location header)
204 No Content   — successful DELETE, or PUT with no body
400 Bad Request  — validation error, malformed input
401 Unauthorized — missing / invalid authentication
403 Forbidden    — authenticated but lacks permission
404 Not Found    — resource doesn't exist
409 Conflict     — duplicate, version conflict
422 Unprocessable Entity — valid format but semantic error
429 Too Many Requests    — rate limit exceeded
500 Internal Server Error — unexpected server failure
```

---

## Consistent Error Responses

Pick a structure and use it everywhere — never return raw stack traces.

```json
{
    "status": 400,
    "error": "Validation Failed",
    "message": "Request body has validation errors",
    "path": "/api/users",
    "timestamp": "2026-06-14T14:30:00Z",
    "errors": [
        { "field": "email",    "message": "must be a valid email" },
        { "field": "password", "message": "must be at least 8 characters" }
    ]
}
```

Spring Boot implementation:

```java
@RestControllerAdvice
public class GlobalExceptionHandler {

    @ExceptionHandler(MethodArgumentNotValidException.class)
    public ResponseEntity<ErrorResponse> handleValidation(
            MethodArgumentNotValidException ex, HttpServletRequest request) {

        List<FieldError> fieldErrors = ex.getBindingResult().getFieldErrors()
            .stream()
            .map(e -> new FieldError(e.getField(), e.getDefaultMessage()))
            .toList();

        ErrorResponse body = new ErrorResponse(
            400, "Validation Failed",
            "Request body has validation errors",
            request.getRequestURI(),
            Instant.now().toString(),
            fieldErrors);

        return ResponseEntity.badRequest().body(body);
    }

    @ExceptionHandler(NoSuchElementException.class)
    public ResponseEntity<ErrorResponse> handleNotFound(
            NoSuchElementException ex, HttpServletRequest request) {

        ErrorResponse body = new ErrorResponse(
            404, "Not Found", ex.getMessage(),
            request.getRequestURI(), Instant.now().toString(), List.of());

        return ResponseEntity.status(HttpStatus.NOT_FOUND).body(body);
    }
}

public record ErrorResponse(int status, String error, String message,
                             String path, String timestamp, List<FieldError> errors) {}
public record FieldError(String field, String message) {}
```

---

## Pagination and Filtering

Never return unbounded collections — always paginate.

### Query parameters

```
GET /api/products?page=0&size=20&sort=name,asc
GET /api/products?page=1&size=10&sort=price,desc&sort=name,asc
```

### Spring Data JPA with pagination

```java
@GetMapping
public ResponseEntity<Page<ProductDto>> list(
        @RequestParam(defaultValue = "0") int page,
        @RequestParam(defaultValue = "20") int size,
        @RequestParam(defaultValue = "name,asc") String[] sort) {

    Sort sorting = Sort.by(Arrays.stream(sort)
        .map(s -> {
            String[] parts = s.split(",");
            return parts.length > 1 && parts[1].equalsIgnoreCase("desc")
                ? Sort.Order.desc(parts[0])
                : Sort.Order.asc(parts[0]);
        })
        .toList());

    Pageable pageable = PageRequest.of(page, Math.min(size, 100), sorting);
    Page<Product> products = productRepository.findAll(pageable);
    return ResponseEntity.ok(products.map(this::toDto));
}
```

### Page response envelope

```json
{
    "content": [ ... ],
    "page": {
        "size": 20,
        "number": 0,
        "totalElements": 253,
        "totalPages": 13
    }
}
```

### Filtering

```
GET /api/products?category=electronics&minPrice=100&maxPrice=999&inStock=true
```

```java
@GetMapping
public List<ProductDto> search(
        @RequestParam Optional<String> category,
        @RequestParam Optional<Double> minPrice,
        @RequestParam Optional<Double> maxPrice,
        @RequestParam Optional<Boolean> inStock) {
    return productRepository.search(category.orElse(null), minPrice.orElse(null),
        maxPrice.orElse(null), inStock.orElse(null))
        .stream().map(this::toDto).toList();
}
```

---

## Versioning

Version the API so you can evolve it without breaking existing clients.

```
# URL versioning (most common)
/api/v1/users
/api/v2/users

# Header versioning
Accept: application/vnd.myapp.v2+json

# Query parameter
/api/users?version=2
```

Spring Boot URL versioning:

```java
@RestController
@RequestMapping("/api/v1/users")
public class UserControllerV1 { ... }

@RestController
@RequestMapping("/api/v2/users")
public class UserControllerV2 { ... }
```

---

## HATEOAS — Links in Responses

HATEOAS embeds links to related actions so clients can navigate without hardcoding URLs.

```java
// add spring-boot-starter-hateoas
@GetMapping("/{id}")
public EntityModel<UserDto> getById(@PathVariable Long id) {
    UserDto user = userService.findById(id);

    return EntityModel.of(user,
        linkTo(methodOn(UserController.class).getById(id)).withSelfRel(),
        linkTo(methodOn(UserController.class).getAll()).withRel("users"),
        linkTo(methodOn(OrderController.class).getByUser(id)).withRel("orders"));
}
```

Response:

```json
{
    "id": 1,
    "name": "Alice",
    "_links": {
        "self":   { "href": "/api/v1/users/1" },
        "users":  { "href": "/api/v1/users" },
        "orders": { "href": "/api/v1/users/1/orders" }
    }
}
```

---

## OpenAPI / Swagger Documentation

```xml
<dependency>
    <groupId>org.springdoc</groupId>
    <artifactId>springdoc-openapi-starter-webmvc-ui</artifactId>
    <version>2.5.0</version>
</dependency>
```

Swagger UI is available at `http://localhost:8080/swagger-ui.html`.

```java
@Operation(summary = "Get user by ID")
@ApiResponses({
    @ApiResponse(responseCode = "200", description = "User found"),
    @ApiResponse(responseCode = "404", description = "User not found",
                 content = @Content(schema = @Schema(implementation = ErrorResponse.class)))
})
@GetMapping("/{id}")
public ResponseEntity<UserDto> getById(
        @Parameter(description = "User ID", required = true)
        @PathVariable Long id) {
    return userService.findById(id)
        .map(ResponseEntity::ok)
        .orElse(ResponseEntity.notFound().build());
}
```

Global API info:

```java
@Bean
public OpenAPI openApi() {
    return new OpenAPI()
        .info(new Info()
            .title("My API")
            .version("1.0")
            .description("User and order management API"))
        .components(new Components()
            .addSecuritySchemes("bearerAuth",
                new SecurityScheme()
                    .type(SecurityScheme.Type.HTTP)
                    .scheme("bearer")
                    .bearerFormat("JWT")));
}
```

---

## REST API Design Summary

| Concern | Convention |
|---------|-----------|
| Resource names | Plural nouns, lowercase, hyphen-separated |
| HTTP methods | GET read, POST create, PUT replace, PATCH update, DELETE remove |
| Success codes | 200 OK, 201 Created, 204 No Content |
| Error codes | 400, 401, 403, 404, 409, 422, 429, 500 |
| Error body | Consistent JSON: status, error, message, path, timestamp, errors[] |
| Pagination | `?page=0&size=20&sort=field,dir` |
| Filtering | Query parameters |
| Versioning | `/api/v1/` prefix |
| Documentation | OpenAPI / Swagger (`springdoc-openapi`) |
| Discoverability | HATEOAS `_links` |

---

[← Back to README](../README.md)
