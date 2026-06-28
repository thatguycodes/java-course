# Spring Data Elasticsearch Deep Dive

[← Back to README](../README.md)

---

Spring Data Elasticsearch provides a repository abstraction and a rich `ElasticsearchOperations` API over the Elasticsearch Java client. Beyond basic CRUD, it exposes full-text search with query DSL, aggregations, highlight, pagination, nested objects, and index lifecycle management — all with strongly typed Java models.

```mermaid
flowchart LR
    service["Spring Service"]
    repo["ElasticsearchRepository\nor ElasticsearchOperations"]
    client["Elasticsearch\nJava Client"]
    cluster["Elasticsearch\nCluster"]

    service --> repo --> client --> cluster
    cluster -- "hits + aggs + highlights" --> client --> repo --> service
```

---

## Dependency

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-elasticsearch</artifactId>
</dependency>
```

```yaml
# application.yml
spring:
  elasticsearch:
    uris: http://localhost:9200
    username: elastic
    password: changeme
    connection-timeout: 5s
    socket-timeout: 30s
```

---

## Document Mapping

```java
@Document(indexName = "products")
@Setting(settingPath = "elasticsearch/products-settings.json")
public class ProductDocument {

    @Id
    private String id;

    @Field(type = FieldType.Text, analyzer = "english")
    private String name;

    @Field(type = FieldType.Text, analyzer = "english")
    private String description;

    @Field(type = FieldType.Keyword)
    private String sku;

    @Field(type = FieldType.Keyword)
    private String category;

    @Field(type = FieldType.Double)
    private BigDecimal price;

    @Field(type = FieldType.Integer)
    private int stockLevel;

    @Field(type = FieldType.Boolean)
    private boolean active;

    @Field(type = FieldType.Date, format = DateFormat.date_time)
    private Instant createdAt;

    // Nested type — each tag is a separate object in the inverted index
    @Field(type = FieldType.Nested)
    private List<Tag> tags;

    // Object type — flattened into parent document
    @Field(type = FieldType.Object)
    private Dimensions dimensions;
}

@Document(indexName = "")  // nested — not a top-level document
public class Tag {
    @Field(type = FieldType.Keyword) private String name;
    @Field(type = FieldType.Keyword) private String value;
}
```

---

## Repository

```java
public interface ProductSearchRepository
        extends ElasticsearchRepository<ProductDocument, String> {

    // Derived query: find by category and active flag
    List<ProductDocument> findByCategoryAndActiveTrue(String category);

    // Full-text search via @Query (Elasticsearch query DSL as JSON)
    @Query("""
        {
          "multi_match": {
            "query": "?0",
            "fields": ["name^3", "description", "sku"],
            "type": "best_fields",
            "fuzziness": "AUTO"
          }
        }
        """)
    Page<ProductDocument> searchByText(String query, Pageable pageable);

    // Range query via @Query
    @Query("""
        {
          "range": {
            "price": { "gte": ?0, "lte": ?1 }
          }
        }
        """)
    List<ProductDocument> findByPriceRange(double min, double max);

    // Count active products per category
    long countByCategoryAndActiveTrue(String category);
}
```

---

## ElasticsearchOperations — Complex Queries

```java
@Service
@RequiredArgsConstructor
public class ProductSearchService {

    private final ElasticsearchOperations operations;

    // Full-text + filter + pagination + highlighting
    public SearchResult<ProductDocument> search(ProductSearchRequest req) {
        Query query = NativeQuery.builder()
            .withQuery(q -> q.bool(b -> b
                .must(m -> m.multiMatch(mm -> mm
                    .query(req.text())
                    .fields("name^3", "description", "sku")
                    .fuzziness("AUTO")))
                .filter(buildFilters(req))))
            .withHighlightQuery(HighlightQuery.of(h -> h
                .withHighlight(hl -> hl
                    .fields("name", Map.of())
                    .fields("description", Map.of())
                    .preTags("<em>")
                    .postTags("</em>"))))
            .withSort(Sort.by(req.sortField()).descending())
            .withPageable(PageRequest.of(req.page(), req.size()))
            .build();

        SearchHits<ProductDocument> hits = operations.search(query, ProductDocument.class);

        List<SearchHitResult<ProductDocument>> results = hits.getSearchHits().stream()
            .map(hit -> new SearchHitResult<>(hit.getContent(), hit.getHighlightFields()))
            .toList();

        return new SearchResult<>(results, hits.getTotalHits(), req.page(), req.size());
    }

    private List<co.elastic.clients.elasticsearch._types.query_dsl.Query> buildFilters(
            ProductSearchRequest req) {
        List<co.elastic.clients.elasticsearch._types.query_dsl.Query> filters = new ArrayList<>();

        if (req.category() != null) {
            filters.add(Query.of(q -> q.term(t -> t.field("category").value(req.category()))));
        }
        if (req.minPrice() != null) {
            filters.add(Query.of(q -> q.range(r -> r.field("price")
                .gte(JsonData.of(req.minPrice())))));
        }
        if (req.maxPrice() != null) {
            filters.add(Query.of(q -> q.range(r -> r.field("price")
                .lte(JsonData.of(req.maxPrice())))));
        }
        filters.add(Query.of(q -> q.term(t -> t.field("active").value(true))));

        return filters;
    }
}
```

---

## Aggregations

```java
@Service
@RequiredArgsConstructor
public class ProductAggregationService {

    private final ElasticsearchOperations operations;

    public CategoryFacets getCategoryFacets(String searchText) {
        Query query = NativeQuery.builder()
            .withQuery(q -> searchText != null
                ? q.match(m -> m.field("description").query(searchText))
                : q.matchAll(m -> m))
            .withAggregation("by_category", Aggregation.of(a -> a
                .terms(t -> t.field("category").size(20))))
            .withAggregation("price_ranges", Aggregation.of(a -> a
                .range(r -> r.field("price")
                    .ranges(
                        AggregationRange.of(ra -> ra.to("50")),
                        AggregationRange.of(ra -> ra.from("50").to("200")),
                        AggregationRange.of(ra -> ra.from("200"))
                    ))))
            .withAggregation("avg_price", Aggregation.of(a -> a
                .avg(avg -> avg.field("price"))))
            .withMaxResults(0)   // aggregations only, no hits needed
            .build();

        SearchHits<ProductDocument> hits = operations.search(query, ProductDocument.class);

        return CategoryFacets.fromAggregations(hits.getAggregations());
    }
}
```

---

## Index Management

```java
@Component
@RequiredArgsConstructor
public class ProductIndexManager {

    private final ElasticsearchOperations operations;
    private final ElasticsearchConverter converter;

    public void createIndex() {
        IndexOperations indexOps = operations.indexOps(ProductDocument.class);
        if (!indexOps.exists()) {
            indexOps.createWithMapping();
        }
    }

    // Re-index with zero downtime (index alias pattern)
    public void reindex() {
        IndexOperations indexOps = operations.indexOps(ProductDocument.class);
        String newIndexName = "products-" + Instant.now().toEpochMilli();

        // 1. Create new index
        indexOps.create(Settings.builder().build(),
            indexOps.createMapping(ProductDocument.class));

        // 2. Populate new index
        // (bulk indexing happens here)

        // 3. Swap alias atomically
        indexOps.addAlias(AliasData.of(AliasActionParameters.builder()
            .withAliases("products")
            .build()));
    }

    public void deleteIndex() {
        operations.indexOps(ProductDocument.class).delete();
    }
}
```

---

## Bulk Indexing

```java
@Service
@RequiredArgsConstructor
public class ProductBulkIndexService {

    private final ElasticsearchOperations operations;

    @Async
    public CompletableFuture<BulkResult> bulkIndex(List<ProductDocument> products) {
        List<IndexQuery> queries = products.stream()
            .map(doc -> new IndexQueryBuilder()
                .withId(doc.getId())
                .withObject(doc)
                .build())
            .toList();

        List<IndexedObjectInformation> results = operations.bulkIndex(queries,
            IndexCoordinates.of("products"));

        long indexed = results.stream().filter(r -> r.id() != null).count();
        return CompletableFuture.completedFuture(
            new BulkResult(indexed, products.size() - indexed));
    }
}
```

---

## Nested Object Query

```java
// Query nested objects (tags) — must use nested query context
public List<ProductDocument> findByTag(String tagName, String tagValue) {
    Query query = NativeQuery.builder()
        .withQuery(q -> q.nested(n -> n
            .path("tags")
            .query(nq -> nq.bool(b -> b
                .must(m -> m.term(t -> t.field("tags.name").value(tagName)))
                .must(m -> m.term(t -> t.field("tags.value").value(tagValue)))))
            .scoreMode(ChildScoreMode.Avg)))
        .build();

    return operations.search(query, ProductDocument.class)
        .getSearchHits()
        .stream()
        .map(SearchHit::getContent)
        .toList();
}
```

---

## Spring Data Elasticsearch Summary

| Concept | Detail |
|---------|--------|
| `@Document(indexName)` | Maps a class to an Elasticsearch index |
| `@Field(type = FieldType.Text)` | Analyzed full-text field — tokenized for search |
| `@Field(type = FieldType.Keyword)` | Exact-match field — used for filters, aggregations, sorting |
| `@Field(type = FieldType.Nested)` | Nested object array — each element has its own inverted index |
| `ElasticsearchRepository` | Spring Data repository with derived queries and `@Query` |
| `NativeQuery` | Programmatic query builder using the full Elasticsearch Query DSL |
| Highlights | `.withHighlightQuery(...)` marks matching terms with custom pre/post tags |
| Aggregations | `terms`, `range`, `avg`, `date_histogram` — faceted navigation and analytics |
| `withMaxResults(0)` | Return only aggregations, no document hits |
| Bulk indexing | `operations.bulkIndex(queries, indexCoordinates)` — high-throughput ingestion |
| Alias pattern | Create new index → populate → swap alias atomically — zero-downtime re-index |

---

[← Back to README](../README.md)
