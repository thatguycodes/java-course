# Spring Data JPA Projections & Specifications

[← Back to README](../README.md)

---

**Projections** let you query only the columns you need instead of loading full entities. **Specifications** let you build dynamic, type-safe WHERE clauses without string concatenation. Together they cover the full spectrum from simple read-models to complex filter UIs.

```mermaid
flowchart LR
    repo["JpaRepository +\nJpaSpecificationExecutor"]
    proj["Projection\n(interface / DTO / dynamic)"]
    spec["Specification&lt;T&gt;\n(predicate builder)"]
    db["Database"]

    repo -->|findBy...| proj --> db
    repo -->|findAll(spec, pageable)| spec --> db
```

---

## Interface Projections

Spring Data creates a proxy at runtime that implements the interface and maps column names to getter methods.

```java
// Closed projection — only declared fields are SELECTed
public interface OrderSummary {
    UUID getId();
    String getStatus();
    BigDecimal getTotal();
}

public interface OrderRepository extends JpaRepository<Order, UUID> {
    List<OrderSummary> findByCustomerId(UUID customerId);
    Optional<OrderSummary> findProjectedById(UUID id);
}
```

```java
// Open projection — can combine fields with SpEL
public interface OrderWithCustomerName {
    UUID getId();
    BigDecimal getTotal();

    @Value("#{target.customer.firstName + ' ' + target.customer.lastName}")
    String getCustomerFullName();   // SpEL computed from the full entity
}
```

---

## DTO Projections (Class-Based)

A constructor expression avoids proxy overhead and is ideal for serialisation:

```java
// DTO record
public record OrderSummaryDto(UUID id, String status, BigDecimal total) {}

public interface OrderRepository extends JpaRepository<Order, UUID> {

    // JPQL constructor expression
    @Query("""
        SELECT new com.example.OrderSummaryDto(o.id, o.status, o.total)
        FROM Order o
        WHERE o.customerId = :customerId
        """)
    List<OrderSummaryDto> findSummariesByCustomer(@Param("customerId") UUID customerId);
}
```

---

## Dynamic Projections

The projection type is chosen at the call site — one repository method serves many use cases:

```java
public interface OrderRepository extends JpaRepository<Order, UUID> {
    <T> List<T> findByCustomerId(UUID customerId, Class<T> type);
    <T> Optional<T> findById(UUID id, Class<T> type);
}

// Callers choose the projection
List<OrderSummary>    summaries = repo.findByCustomerId(id, OrderSummary.class);
List<OrderSummaryDto> dtos      = repo.findByCustomerId(id, OrderSummaryDto.class);
List<Order>           entities  = repo.findByCustomerId(id, Order.class);
```

---

## Specifications — Dynamic Queries

`JpaSpecificationExecutor<T>` adds `findAll(Specification<T>)`, `findOne(Specification<T>)`, and `count(Specification<T>)`.

```java
public interface OrderRepository
        extends JpaRepository<Order, UUID>,
                JpaSpecificationExecutor<Order> {}
```

### Building Specifications

```java
public class OrderSpecs {

    public static Specification<Order> hasStatus(String status) {
        return (root, query, cb) ->
            status == null ? cb.conjunction()
                           : cb.equal(root.get("status"), status);
    }

    public static Specification<Order> forCustomer(UUID customerId) {
        return (root, query, cb) ->
            customerId == null ? cb.conjunction()
                               : cb.equal(root.get("customerId"), customerId);
    }

    public static Specification<Order> totalGreaterThan(BigDecimal min) {
        return (root, query, cb) ->
            min == null ? cb.conjunction()
                        : cb.greaterThan(root.get("total"), min);
    }

    public static Specification<Order> createdAfter(Instant date) {
        return (root, query, cb) ->
            date == null ? cb.conjunction()
                         : cb.greaterThanOrEqualTo(root.get("createdAt"), date);
    }

    // JOIN-based specification
    public static Specification<Order> customerNameLike(String fragment) {
        return (root, query, cb) -> {
            if (fragment == null) return cb.conjunction();
            Join<Order, Customer> customer = root.join("customer", JoinType.INNER);
            return cb.like(cb.lower(customer.get("name")),
                           "%" + fragment.toLowerCase() + "%");
        };
    }
}
```

### Composing Specifications

```java
@Service
@RequiredArgsConstructor
public class OrderQueryService {

    private final OrderRepository repo;

    public Page<Order> search(OrderSearchRequest req, Pageable pageable) {
        Specification<Order> spec = Specification
            .where(OrderSpecs.hasStatus(req.status()))
            .and(OrderSpecs.forCustomer(req.customerId()))
            .and(OrderSpecs.totalGreaterThan(req.minTotal()))
            .and(OrderSpecs.createdAfter(req.createdAfter()))
            .and(OrderSpecs.customerNameLike(req.customerName()));

        return repo.findAll(spec, pageable);
    }

    public long countPending() {
        return repo.count(OrderSpecs.hasStatus("PENDING"));
    }
}
```

---

## Combining Specifications with Projections

```java
// Specification + Projection together — Spring Data 3.x
public interface OrderRepository
        extends JpaRepository<Order, UUID>,
                JpaSpecificationExecutor<Order> {

    <T> Page<T> findBy(Specification<Order> spec, Function<FluentQuery.FetchableFluentQuery<Order>, Page<T>> queryFunction);
}

// Usage
Page<OrderSummary> page = repo.findBy(
    OrderSpecs.hasStatus("CONFIRMED"),
    q -> q.as(OrderSummary.class).page(pageable));
```

---

## Native Query Projections

```java
public interface RevenueByDay {
    LocalDate getDay();
    BigDecimal getRevenue();
    Long getOrderCount();
}

@Query(value = """
    SELECT DATE(created_at) AS day,
           SUM(total)       AS revenue,
           COUNT(*)         AS order_count
    FROM orders
    WHERE created_at >= :since
    GROUP BY DATE(created_at)
    ORDER BY day
    """, nativeQuery = true)
List<RevenueByDay> revenueByDay(@Param("since") LocalDate since);
```

---

## Querydsl Integration (Type-Safe Alternative)

```xml
<dependency>
    <groupId>com.querydsl</groupId>
    <artifactId>querydsl-jpa</artifactId>
    <classifier>jakarta</classifier>
</dependency>
<dependency>
    <groupId>com.querydsl</groupId>
    <artifactId>querydsl-apt</artifactId>
    <classifier>jakarta</classifier>
    <scope>provided</scope>
</dependency>
```

```java
// APT generates QOrder from the @Entity class
public interface OrderRepository
        extends JpaRepository<Order, UUID>,
                QuerydslPredicateExecutor<Order> {}

// Usage — compile-time safe, IDE auto-complete
QOrder order = QOrder.order;

BooleanExpression predicate = order.status.eq("CONFIRMED")
    .and(order.total.goe(new BigDecimal("100")))
    .and(order.createdAt.after(Instant.now().minus(Duration.ofDays(7))));

Iterable<Order> results = repo.findAll(predicate);
```

---

## Projection vs Entity — When to Use Each

| Approach | Use When |
|----------|---------|
| Full entity | Mutating the data; complex domain logic |
| Interface projection | Read-only; avoiding N+1 on nested objects |
| DTO projection (record) | Serialising directly to API response; performance-critical |
| Dynamic projection | Single repository method serving multiple callers |
| `Specification<T>` | User-driven search filters; optional/dynamic conditions |
| Querydsl | Complex type-safe queries; refactor-safe column references |

---

## JPA Projections & Specifications Summary

| Concept | Detail |
|---------|--------|
| Closed interface projection | Proxy over selected columns — no unnecessary fields loaded |
| Open interface projection (`@Value`) | SpEL expressions combining multiple fields |
| DTO / record projection | Constructor expression in JPQL — no proxy overhead |
| Dynamic projection (`Class<T>`) | Same query method, different return type at call site |
| `Specification<T>` | Composable `(root, query, cb) -> Predicate` — `and()`, `or()`, `not()` |
| `JpaSpecificationExecutor` | Mix-in interface adding `findAll(Specification, Pageable)` |
| `cb.conjunction()` | "Always true" predicate — safe no-op when filter is null |
| Native query projection | Interface mapped by column alias from `@Query(nativeQuery=true)` |
| Querydsl | APT-generated `QEntity` classes for compile-time safe predicates |

---

[← Back to README](../README.md)
