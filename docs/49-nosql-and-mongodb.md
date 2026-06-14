# NoSQL and MongoDB

[← Back to README](../README.md)

---

NoSQL databases store data in formats other than relational tables. **MongoDB** is the most popular document database — data is stored as JSON-like documents, making it natural for hierarchical, schema-flexible, or rapidly evolving data.

```mermaid
flowchart LR
    app["Java Application"]
    smd["Spring Data MongoDB"]
    driver["MongoDB Java Driver"]
    mongo["MongoDB\n(documents)"]

    app --> smd --> driver --> mongo
```

---

## When to Use MongoDB vs SQL

| MongoDB | SQL (PostgreSQL) |
|---------|-----------------|
| Flexible / evolving schema | Fixed, well-defined schema |
| Hierarchical / nested data | Highly relational data |
| High write throughput | Complex joins and transactions |
| Catalogue, CMS, user profiles | Financial records, inventory |
| Horizontal scale-out | Strong consistency required |

---

## Maven Dependency

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-mongodb</artifactId>
</dependency>
```

---

## Configuration

```yaml
spring:
  data:
    mongodb:
      uri: mongodb://localhost:27017/mydb
      # with auth
      # uri: mongodb://user:password@localhost:27017/mydb?authSource=admin
```

---

## Document Mapping

```java
import org.springframework.data.annotation.*;
import org.springframework.data.mongodb.core.mapping.*;
import org.springframework.data.mongodb.core.index.*;

@Document(collection = "products")   // maps to a MongoDB collection
public class Product {

    @Id
    private String id;               // MongoDB uses String ObjectId by default

    @Indexed(unique = true)
    private String sku;

    private String name;
    private String description;
    private double price;
    private int    stock;

    @DBRef                           // reference to another document
    private Category category;

    private List<String> tags;       // embedded array

    private ProductDetails details;  // embedded object (no join needed)

    @CreatedDate
    private Instant createdAt;

    @LastModifiedDate
    private Instant updatedAt;

    // constructors, getters, setters ...
}

// embedded sub-document — stored inside the product, not a separate collection
public class ProductDetails {
    private String  brand;
    private String  colour;
    private double  weight;
    private Map<String, String> specifications;
}
```

Enable auditing:

```java
@Configuration
@EnableMongoAuditing
public class MongoConfig {}
```

---

## Spring Data MongoDB Repository

```java
import org.springframework.data.mongodb.repository.MongoRepository;
import org.springframework.data.mongodb.repository.Query;

public interface ProductRepository extends MongoRepository<Product, String> {

    // derived query — Spring generates the MongoDB query
    Optional<Product> findBySku(String sku);
    List<Product>     findByPriceBetween(double min, double max);
    List<Product>     findByTagsContaining(String tag);
    List<Product>     findByCategoryIdOrderByPriceAsc(String categoryId);

    // custom query with MongoDB JSON syntax
    @Query("{ 'price': { $lte: ?0 }, 'stock': { $gt: 0 } }")
    List<Product> findAffordableInStock(double maxPrice);

    // count
    long countByCategory_Name(String categoryName);

    // exists
    boolean existsBySku(String sku);
}
```

---

## MongoTemplate — Full Query Control

`MongoTemplate` gives you access to the full MongoDB query API.

```java
import org.springframework.data.mongodb.core.MongoTemplate;
import org.springframework.data.mongodb.core.query.*;
import org.springframework.data.mongodb.core.aggregation.*;

@Service
public class ProductService {

    private final MongoTemplate mongo;

    public ProductService(MongoTemplate mongo) { this.mongo = mongo; }

    public List<Product> search(String keyword, double maxPrice, String tag) {
        Query query = new Query();

        // text search on name and description
        if (keyword != null) {
            query.addCriteria(new Criteria().orOperator(
                Criteria.where("name").regex(keyword, "i"),
                Criteria.where("description").regex(keyword, "i")
            ));
        }

        if (maxPrice > 0) {
            query.addCriteria(Criteria.where("price").lte(maxPrice));
        }

        if (tag != null) {
            query.addCriteria(Criteria.where("tags").in(tag));
        }

        query.with(Sort.by(Sort.Direction.ASC, "price"));
        query.limit(50);

        return mongo.find(query, Product.class);
    }

    public void updateStock(String productId, int delta) {
        Query query = Query.query(Criteria.where("id").is(productId));
        Update update = new Update().inc("stock", delta);   // atomic increment
        mongo.updateFirst(query, update, Product.class);
    }

    public void addTag(String productId, String tag) {
        Query  query  = Query.query(Criteria.where("id").is(productId));
        Update update = new Update().addToSet("tags", tag);  // add to array if not present
        mongo.updateFirst(query, update, Product.class);
    }
}
```

---

## Aggregation Pipeline

MongoDB aggregation pipelines transform documents through multiple stages.

```java
// average price per category
Aggregation agg = Aggregation.newAggregation(
    Aggregation.match(Criteria.where("stock").gt(0)),         // $match
    Aggregation.group("category.name")                        // $group
        .count().as("productCount")
        .avg("price").as("avgPrice")
        .max("price").as("maxPrice"),
    Aggregation.sort(Sort.by(Sort.Direction.DESC, "avgPrice")), // $sort
    Aggregation.limit(10)                                        // $limit
);

AggregationResults<CategoryStats> results =
    mongo.aggregate(agg, "products", CategoryStats.class);

List<CategoryStats> stats = results.getMappedResults();
```

```java
public record CategoryStats(String id, int productCount, double avgPrice, double maxPrice) {}
```

---

## Embedded vs Referenced Documents

```java
// EMBEDDED — address stored inside user document
// Good when: data is accessed together, no sharing between documents
@Document("users")
public class User {
    @Id private String id;
    private String   name;
    private Address  address;    // embedded
    private List<String> roles;  // embedded array
}

public class Address {
    private String street, city, country, postalCode;
}

// REFERENCED — order references user by ID
// Good when: many orders per user, order accessed independently
@Document("orders")
public class Order {
    @Id private String id;
    private String userId;       // manual reference — just store the ID
    private List<OrderItem> items;  // items embedded (belong to order)
    private double total;
}
```

---

## Indexing

```java
// compound index on class
@Document("products")
@CompoundIndex(def = "{'category': 1, 'price': -1}", name = "cat_price_idx")
public class Product { ... }

// single field index
@Indexed
private String sku;

// TTL index — document auto-deleted after expiry
@Indexed(expireAfterSeconds = 3600)
private Instant expiresAt;

// programmatic index creation
@Bean
CommandLineRunner createIndexes(MongoTemplate mongo) {
    return args -> mongo.indexOps(Product.class)
        .ensureIndex(new Index("sku", Sort.Direction.ASC).unique());
}
```

---

## Transactions (MongoDB 4.0+)

MongoDB supports multi-document ACID transactions on replica sets.

```java
@Service
public class TransferService {

    private final MongoTemplate mongo;
    private final MongoTransactionManager txManager;

    @Transactional
    public void transfer(String fromId, String toId, double amount) {
        Query from = Query.query(Criteria.where("id").is(fromId));
        Query to   = Query.query(Criteria.where("id").is(toId));

        mongo.updateFirst(from, new Update().inc("balance", -amount), Account.class);
        mongo.updateFirst(to,   new Update().inc("balance",  amount), Account.class);
    }
}

// register transaction manager
@Bean
MongoTransactionManager transactionManager(MongoDatabaseFactory factory) {
    return new MongoTransactionManager(factory);
}
```

---

## Running MongoDB Locally

```yaml
# compose.yml
services:
  mongo:
    image: mongo:7
    ports:
      - "27017:27017"
    environment:
      MONGO_INITDB_ROOT_USERNAME: admin
      MONGO_INITDB_ROOT_PASSWORD: secret
      MONGO_INITDB_DATABASE: mydb
    volumes:
      - mongo-data:/data/db

volumes:
  mongo-data:
```

```bash
docker compose up -d mongo

# connect via shell
docker exec -it <container> mongosh -u admin -p secret --authenticationDatabase admin

# inside mongosh
use mydb
db.products.find().limit(5)
db.products.countDocuments()
db.products.createIndex({ sku: 1 }, { unique: true })
```

---

## MongoDB Summary

| Concept | Spring Data / Annotation |
|---------|-------------------------|
| Document class | `@Document(collection="...")` |
| Primary key | `@Id` (String, ObjectId) |
| Embedded object | Plain field — no annotation needed |
| Index | `@Indexed`, `@CompoundIndex` |
| Repository | `MongoRepository<T, ID>` |
| Derived queries | `findBy...`, `countBy...`, `existsBy...` |
| Custom query | `@Query("{ 'field': ?0 }")` |
| Full control | `MongoTemplate` |
| Atomic update | `Update.inc()`, `addToSet()`, `pull()` |
| Aggregation | `Aggregation.newAggregation(...)` |
| Transactions | `@Transactional` + `MongoTransactionManager` |

---

[← Back to README](../README.md)
