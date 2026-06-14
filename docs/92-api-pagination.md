# API Pagination

[← Back to README](../README.md)

---

Pagination prevents returning millions of rows in a single response. There are three main strategies — **offset/page**, **keyset (seek)**, and **cursor** — each with different tradeoffs for correctness, performance, and API design.

---

## Offset / Page-Number Pagination

The simplest approach: skip N rows and take the next page.

```
GET /api/products?page=2&size=20
```

### Spring Data — `Pageable`

```java
// Repository
public interface ProductRepository extends JpaRepository<Product, UUID> {

    Page<Product> findByCategory(String category, Pageable pageable);

    @Query("SELECT p FROM Product p WHERE p.price < :maxPrice ORDER BY p.name")
    Page<Product> findCheaperThan(@Param("maxPrice") BigDecimal maxPrice, Pageable pageable);
}

// Controller
@GetMapping("/api/products")
public ResponseEntity<Page<ProductResponse>> list(
        @RequestParam(required = false) String category,
        @PageableDefault(size = 20, sort = "name") Pageable pageable) {

    Page<Product> page = category != null
        ? productRepo.findByCategory(category, pageable)
        : productRepo.findAll(pageable);

    return ResponseEntity.ok(page.map(ProductResponse::from));
}
```

**Request:**
```
GET /api/products?page=0&size=10&sort=name,asc
```

**Response:**
```json
{
  "content": [...],
  "page": {
    "size": 10,
    "number": 0,
    "totalElements": 347,
    "totalPages": 35
  }
}
```

### Offset Pagination — Tradeoffs

| Pro | Con |
|-----|-----|
| Simple to implement | Slow for high page numbers (`OFFSET 10000` scans 10 000 rows) |
| `totalPages` / `totalElements` available | Rows can shift between pages if data changes (duplicates / skips) |
| Supports random access to any page | Not suitable for real-time feeds |

---

## Keyset / Seek Pagination

Instead of skipping rows, filter from the last seen value. Much faster and stable under concurrent inserts.

```
GET /api/products?afterId=abc-123&size=20
```

### Implementation

```java
@Repository
public interface ProductRepository extends JpaRepository<Product, UUID> {

    // Keyset on a sortable column (createdAt + id for uniqueness)
    @Query("""
        SELECT p FROM Product p
        WHERE (p.createdAt < :afterTs)
           OR (p.createdAt = :afterTs AND p.id < :afterId)
        ORDER BY p.createdAt DESC, p.id DESC
        """)
    List<Product> findPageAfter(
        @Param("afterTs") Instant afterTs,
        @Param("afterId") UUID afterId,
        Pageable pageable);
}

@GetMapping("/api/products")
public ResponseEntity<KeysetPage<ProductResponse>> list(
        @RequestParam(required = false) UUID afterId,
        @RequestParam(required = false) Instant afterTs,
        @RequestParam(defaultValue = "20") int size) {

    List<Product> products = afterId != null
        ? productRepo.findPageAfter(afterTs, afterId,
            PageRequest.of(0, size + 1))   // +1 to detect hasNext
        : productRepo.findAll(PageRequest.of(0, size + 1,
            Sort.by(Sort.Direction.DESC, "createdAt", "id"))).getContent();

    boolean hasNext = products.size() > size;
    if (hasNext) products = products.subList(0, size);

    Product last = products.isEmpty() ? null : products.getLast();
    String nextCursor = last != null
        ? last.getId() + "," + last.getCreatedAt()
        : null;

    return ResponseEntity.ok(new KeysetPage<>(
        products.stream().map(ProductResponse::from).toList(),
        hasNext,
        nextCursor));
}

public record KeysetPage<T>(List<T> content, boolean hasNext, String nextCursor) {}
```

---

## Cursor Pagination (Opaque Encoded Cursor)

Encodes the keyset as an opaque base64 string. Clients treat it as a black box.

```
GET /api/feed?cursor=eyJpZCI6IjEyMyIsInRzIjoiMjAyNC0wMS0wMVQwMDowMDowMFoifQ==&size=20
```

```java
public record PageCursor(UUID id, Instant ts) {

    public static PageCursor decode(String encoded) {
        String json = new String(Base64.getDecoder().decode(encoded));
        return objectMapper.readValue(json, PageCursor.class);
    }

    public String encode() {
        String json = objectMapper.writeValueAsString(this);
        return Base64.getEncoder().encodeToString(json.getBytes());
    }
}

@GetMapping("/api/feed")
public CursorPage<FeedItem> getFeed(
        @RequestParam(required = false) String cursor,
        @RequestParam(defaultValue = "20") int size) {

    List<FeedItem> items;

    if (cursor != null) {
        PageCursor pc = PageCursor.decode(cursor);
        items = feedRepo.findAfter(pc.ts(), pc.id(),
            PageRequest.of(0, size + 1));
    } else {
        items = feedRepo.findLatest(PageRequest.of(0, size + 1));
    }

    boolean hasNext = items.size() > size;
    if (hasNext) items = items.subList(0, size);

    FeedItem last     = items.isEmpty() ? null : items.getLast();
    String nextCursor = last != null
        ? new PageCursor(last.id(), last.createdAt()).encode()
        : null;

    return new CursorPage<>(items, nextCursor);
}

public record CursorPage<T>(List<T> content, String nextCursor) {}
```

---

## Link Headers (RFC 5988)

Expose pagination links in headers — standard for APIs:

```java
@GetMapping("/api/products")
public ResponseEntity<List<ProductResponse>> list(
        @PageableDefault(size = 20) Pageable pageable) {

    Page<Product> page = productRepo.findAll(pageable);
    List<ProductResponse> body = page.map(ProductResponse::from).getContent();

    HttpHeaders headers = new HttpHeaders();
    headers.add("X-Total-Count", String.valueOf(page.getTotalElements()));

    List<String> links = new ArrayList<>();
    if (page.hasNext()) {
        links.add(String.format("</api/products?page=%d&size=%d>; rel=\"next\"",
            pageable.getPageNumber() + 1, pageable.getPageSize()));
    }
    if (page.hasPrevious()) {
        links.add(String.format("</api/products?page=%d&size=%d>; rel=\"prev\"",
            pageable.getPageNumber() - 1, pageable.getPageSize()));
    }
    links.add(String.format("</api/products?page=%d&size=%d>; rel=\"last\"",
        page.getTotalPages() - 1, pageable.getPageSize()));
    if (!links.isEmpty()) {
        headers.add("Link", String.join(", ", links));
    }

    return ResponseEntity.ok().headers(headers).body(body);
}
```

**Response headers:**
```
X-Total-Count: 347
Link: </api/products?page=3&size=20>; rel="next",
      </api/products?page=1&size=20>; rel="prev",
      </api/products?page=17&size=20>; rel="last"
```

---

## Infinite Scroll — Minimal Response

```java
public record ScrollPage<T>(
    List<T>  content,
    boolean  hasMore,
    String   nextCursor   // null if no more pages
) {}
```

Clients check `hasMore` and use `nextCursor` to fetch the next batch when the user scrolls to the bottom.

---

## SQL Performance Tips

```sql
-- BAD: OFFSET 10000 scans 10,000 rows to discard them
SELECT * FROM products ORDER BY created_at DESC LIMIT 20 OFFSET 10000;

-- GOOD: keyset — only reads the 20 rows you need
SELECT * FROM products
WHERE created_at < :afterTs
  OR (created_at = :afterTs AND id < :afterId)
ORDER BY created_at DESC, id DESC
LIMIT 20;

-- Index required for keyset pagination
CREATE INDEX idx_products_created_id ON products (created_at DESC, id DESC);
```

---

## Pagination Strategy Summary

| Strategy | Best for | Allows random access | Stable under inserts |
|----------|---------|---------------------|---------------------|
| Offset / page | Admin UIs, small datasets | Yes | No |
| Keyset / seek | Large tables, feeds | No | Yes |
| Cursor (opaque) | Public APIs, infinite scroll | No | Yes |

| Spring Data API | Detail |
|----------------|--------|
| `Pageable` | Page number, size, and sort from query params |
| `@PageableDefault` | Defaults when client doesn't specify |
| `Page<T>` | Contains `content`, `totalElements`, `totalPages` |
| `Slice<T>` | Like `Page` but no `COUNT` query — faster |
| `List<T>` + `size + 1` | Manual keyset — fetch one extra to detect `hasNext` |

---

[← Back to README](../README.md)
