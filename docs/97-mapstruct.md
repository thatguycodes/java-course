# MapStruct

[← Back to README](../README.md)

---

**MapStruct** is a compile-time code generator that creates type-safe, performant mapper implementations from annotated interfaces. It eliminates hand-written, error-prone DTO↔entity conversion code with zero runtime overhead — the generated code is plain Java method calls, not reflection.

---

## Maven Dependency

```xml
<dependency>
    <groupId>org.mapstruct</groupId>
    <artifactId>mapstruct</artifactId>
    <version>1.6.0</version>
</dependency>
<dependency>
    <groupId>org.mapstruct</groupId>
    <artifactId>mapstruct-processor</artifactId>
    <version>1.6.0</version>
    <scope>provided</scope>
</dependency>

<!-- If also using Lombok — processor order matters -->
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-compiler-plugin</artifactId>
    <configuration>
        <annotationProcessorPaths>
            <path>
                <groupId>org.projectlombok</groupId>
                <artifactId>lombok</artifactId>
            </path>
            <path>
                <groupId>org.mapstruct</groupId>
                <artifactId>mapstruct-processor</artifactId>
                <version>1.6.0</version>
            </path>
        </annotationProcessorPaths>
    </configuration>
</plugin>
```

Lombok must appear **before** MapStruct in the processor list so Lombok generates getters/setters before MapStruct reads them.

---

## Basic Mapping

```java
// Entity
@Entity
public class Order {
    @Id private UUID id;
    private String customerId;
    private BigDecimal total;
    private OrderStatus status;
    private Instant createdAt;
}

// DTO
public record OrderResponse(
    UUID id,
    String customerId,
    BigDecimal total,
    String status,
    Instant createdAt
) {}

// Mapper — MapStruct generates the implementation
@Mapper(componentModel = "spring")   // makes it a Spring bean
public interface OrderMapper {

    OrderResponse toResponse(Order order);

    List<OrderResponse> toResponseList(List<Order> orders);
}
```

**Generated code (simplified):**
```java
@Component
public class OrderMapperImpl implements OrderMapper {

    @Override
    public OrderResponse toResponse(Order order) {
        if (order == null) return null;
        return new OrderResponse(
            order.getId(),
            order.getCustomerId(),
            order.getTotal(),
            order.getStatus() == null ? null : order.getStatus().name(),
            order.getCreatedAt()
        );
    }
}
```

---

## Field Name Mapping

```java
public record UserResponse(String fullName, String emailAddress) {}

@Mapper(componentModel = "spring")
public interface UserMapper {

    @Mapping(source = "name",  target = "fullName")
    @Mapping(source = "email", target = "emailAddress")
    UserResponse toResponse(User user);
}
```

---

## Ignoring Fields

```java
@Mapper(componentModel = "spring")
public interface ProductMapper {

    @Mapping(target = "id", ignore = true)          // don't copy id on create
    @Mapping(target = "createdAt", ignore = true)   // set by JPA
    Product toEntity(CreateProductRequest request);

    ProductResponse toResponse(Product product);
}
```

---

## Expressions and Constants

```java
@Mapper(componentModel = "spring")
public interface OrderMapper {

    @Mapping(target = "createdAt", expression = "java(java.time.Instant.now())")
    @Mapping(target = "status", constant = "PENDING")
    @Mapping(target = "id", expression = "java(java.util.UUID.randomUUID())")
    Order toEntity(CreateOrderRequest request);
}
```

---

## Nested Object Mapping

```java
public class Order {
    private UUID id;
    private Customer customer;
    private Address shippingAddress;
}

public record OrderResponse(UUID id, CustomerDto customer, AddressDto shippingAddress) {}

@Mapper(componentModel = "spring",
        uses = {CustomerMapper.class, AddressMapper.class})  // delegate to other mappers
public interface OrderMapper {
    OrderResponse toResponse(Order order);
}
```

MapStruct automatically delegates `Customer → CustomerDto` and `Address → AddressDto` to the referenced mappers.

---

## Update Mapping (Merge into Existing Object)

```java
@Mapper(componentModel = "spring")
public interface ProductMapper {

    // Updates fields of an existing entity from a request DTO
    @Mapping(target = "id", ignore = true)
    @Mapping(target = "createdAt", ignore = true)
    void updateFromRequest(UpdateProductRequest request, @MappingTarget Product product);
}

// Usage in service
@Transactional
public Product update(UUID id, UpdateProductRequest request) {
    Product product = repo.findById(id).orElseThrow();
    productMapper.updateFromRequest(request, product);   // merges changes in place
    return repo.save(product);
}
```

---

## Custom Conversion Methods

```java
@Mapper(componentModel = "spring")
public interface OrderMapper {

    OrderResponse toResponse(Order order);

    // MapStruct picks up this method automatically for String → OrderStatus conversions
    default OrderStatus toStatus(String value) {
        return value == null ? null : OrderStatus.valueOf(value.toUpperCase());
    }

    // Format Instant to readable string
    default String formatInstant(Instant instant) {
        return instant == null ? null
            : DateTimeFormatter.ISO_INSTANT.format(instant);
    }
}
```

---

## Using a Spring Service Inside a Mapper

```java
@Mapper(componentModel = "spring")
public abstract class OrderMapper {

    @Autowired
    protected CustomerRepository customerRepo;  // inject Spring beans

    public abstract OrderResponse toResponse(Order order);

    @AfterMapping   // runs after the generated mapping
    protected void enrichWithCustomerName(Order order,
                                          @MappingTarget OrderResponse.Builder resp) {
        customerRepo.findById(order.getCustomerId())
            .ifPresent(c -> resp.customerName(c.getFullName()));
    }
}
```

---

## Mapping Collections and Maps

```java
@Mapper(componentModel = "spring")
public interface TagMapper {

    // List
    List<TagDto> toDtoList(List<Tag> tags);

    // Set
    Set<TagDto> toDtoSet(Set<Tag> tags);

    // Map values
    Map<String, TagDto> toDtoMap(Map<String, Tag> tags);

    // Individual element — used by the above automatically
    TagDto toDto(Tag tag);
}
```

---

## Null Handling Strategy

```java
@Mapper(
    componentModel = "spring",
    nullValuePropertyMappingStrategy = NullValuePropertyMappingStrategy.IGNORE,
    // ↑ skip null source fields when updating — keep existing target value
    nullValueCheckStrategy = NullValueCheckStrategy.ALWAYS
)
public interface ProductMapper {

    @BeanMapping(nullValuePropertyMappingStrategy = NullValuePropertyMappingStrategy.IGNORE)
    void updateFromRequest(UpdateProductRequest request, @MappingTarget Product product);
}
```

---

## Testing Mappers

```java
// MapStruct mappers are plain classes — test without Spring context
class OrderMapperTest {

    private final OrderMapper mapper = Mappers.getMapper(OrderMapper.class);

    @Test
    void mapsOrderToResponse() {
        Order order = new Order();
        order.setId(UUID.randomUUID());
        order.setCustomerId("CUST-1");
        order.setTotal(new BigDecimal("99.99"));
        order.setStatus(OrderStatus.CONFIRMED);

        OrderResponse response = mapper.toResponse(order);

        assertThat(response.id()).isEqualTo(order.getId());
        assertThat(response.customerId()).isEqualTo("CUST-1");
        assertThat(response.total()).isEqualByComparingTo("99.99");
        assertThat(response.status()).isEqualTo("CONFIRMED");
    }
}
```

---

## MapStruct Summary

| Feature | Annotation / Config |
|---------|-------------------|
| Define mapper | `@Mapper(componentModel = "spring")` |
| Rename field | `@Mapping(source = "name", target = "fullName")` |
| Ignore field | `@Mapping(target = "id", ignore = true)` |
| Constant value | `@Mapping(target = "status", constant = "PENDING")` |
| Java expression | `@Mapping(target = "id", expression = "java(UUID.randomUUID())")` |
| Delegate to other mapper | `@Mapper(uses = {AddressMapper.class})` |
| Update existing object | `void update(Dto dto, @MappingTarget Entity entity)` |
| Post-mapping hook | `@AfterMapping` on a method with `@MappingTarget` |
| Null source strategy | `NullValuePropertyMappingStrategy.IGNORE` |
| Test without Spring | `Mappers.getMapper(MyMapper.class)` |

---

[← Back to README](../README.md)
