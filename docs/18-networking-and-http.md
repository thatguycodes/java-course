# Networking and HTTP

[← Back to README](../README.md)

---

Java provides networking support at two levels: **low-level sockets** (`java.net`) for raw TCP/UDP communication, and the **HttpClient** (`java.net.http`, Java 11+) for modern HTTP/1.1 and HTTP/2.

```mermaid
flowchart TD
    java_net["java.net"]
    java_net_http["java.net.http\n(Java 11+)"]

    java_net --> Socket["Socket\n(TCP client)"]
    java_net --> ServerSocket["ServerSocket\n(TCP server)"]
    java_net --> DatagramSocket["DatagramSocket\n(UDP)"]
    java_net --> URL["URL / URI"]

    java_net_http --> HttpClient["HttpClient"]
    java_net_http --> HttpRequest["HttpRequest"]
    java_net_http --> HttpResponse["HttpResponse"]
    java_net_http --> WebSocket["WebSocket"]
```

---

## URIs and URLs

A **URI** (Uniform Resource Identifier) identifies a resource. A **URL** (Uniform Resource Locator) is a URI that also specifies how to retrieve it.

```
https://api.example.com:8080/users/42?sort=name&page=1#section
│       │               │    │        │                │
scheme  host            port path     query            fragment
```

```java
import java.net.URI;

URI uri = URI.create("https://api.example.com:8080/users/42?sort=name&page=1");

System.out.println(uri.getScheme());    // https
System.out.println(uri.getHost());      // api.example.com
System.out.println(uri.getPort());      // 8080
System.out.println(uri.getPath());      // /users/42
System.out.println(uri.getQuery());     // sort=name&page=1

// build a URI programmatically
URI built = new URI("https", "api.example.com", "/users/42", "sort=name", null);

// encode special characters in a path segment
String encoded = java.net.URLEncoder.encode("hello world & more", "UTF-8");
System.out.println(encoded);  // hello+world+%26+more

String decoded = java.net.URLDecoder.decode(encoded, "UTF-8");
System.out.println(decoded);  // hello world & more
```

---

## HttpClient (Java 11+)

`HttpClient` is the modern, fluent HTTP API supporting HTTP/1.1, HTTP/2, sync and async requests, and WebSockets.

### Creating a Client

```java
import java.net.http.*;
import java.net.URI;
import java.time.Duration;

HttpClient client = HttpClient.newBuilder()
    .version(HttpClient.Version.HTTP_2)           // prefer HTTP/2, fall back to 1.1
    .connectTimeout(Duration.ofSeconds(10))
    .followRedirects(HttpClient.Redirect.NORMAL)  // follow non-HTTPS→HTTP redirects
    .build();

// or use the shared default client (Java 16+)
HttpClient client = HttpClient.newHttpClient();
```

---

### GET Request (Synchronous)

```java
HttpRequest request = HttpRequest.newBuilder()
    .uri(URI.create("https://jsonplaceholder.typicode.com/users/1"))
    .header("Accept", "application/json")
    .timeout(Duration.ofSeconds(5))
    .GET()
    .build();

HttpResponse<String> response = client.send(request, HttpResponse.BodyHandlers.ofString());

System.out.println(response.statusCode());  // 200
System.out.println(response.body());        // { "id": 1, "name": "Leanne Graham", ... }
System.out.println(response.headers().firstValue("content-type").orElse(""));
```

---

### POST Request with JSON Body

```java
String jsonBody = """
        {
          "title": "Hello",
          "body":  "World",
          "userId": 1
        }
        """;

HttpRequest request = HttpRequest.newBuilder()
    .uri(URI.create("https://jsonplaceholder.typicode.com/posts"))
    .header("Content-Type", "application/json")
    .POST(HttpRequest.BodyPublishers.ofString(jsonBody))
    .build();

HttpResponse<String> response = client.send(request, HttpResponse.BodyHandlers.ofString());
System.out.println(response.statusCode());  // 201
System.out.println(response.body());
```

---

### Other HTTP Methods

```java
// PUT
HttpRequest put = HttpRequest.newBuilder()
    .uri(URI.create("https://api.example.com/users/1"))
    .header("Content-Type", "application/json")
    .PUT(HttpRequest.BodyPublishers.ofString("{\"name\": \"Alice\"}"))
    .build();

// PATCH
HttpRequest patch = HttpRequest.newBuilder()
    .uri(URI.create("https://api.example.com/users/1"))
    .header("Content-Type", "application/json")
    .method("PATCH", HttpRequest.BodyPublishers.ofString("{\"name\": \"Alice\"}"))
    .build();

// DELETE
HttpRequest delete = HttpRequest.newBuilder()
    .uri(URI.create("https://api.example.com/users/1"))
    .DELETE()
    .build();
```

---

### Async Request

`sendAsync` returns a `CompletableFuture` — the calling thread is not blocked.

```java
HttpRequest request = HttpRequest.newBuilder()
    .uri(URI.create("https://jsonplaceholder.typicode.com/posts/1"))
    .build();

client.sendAsync(request, HttpResponse.BodyHandlers.ofString())
    .thenApply(HttpResponse::body)
    .thenAccept(System.out::println)
    .exceptionally(ex -> {
        System.err.println("Request failed: " + ex.getMessage());
        return null;
    });

// fire multiple requests in parallel
var uris = java.util.List.of(
    "https://jsonplaceholder.typicode.com/posts/1",
    "https://jsonplaceholder.typicode.com/posts/2",
    "https://jsonplaceholder.typicode.com/posts/3"
);

var futures = uris.stream()
    .map(uri -> HttpRequest.newBuilder().uri(URI.create(uri)).build())
    .map(req -> client.sendAsync(req, HttpResponse.BodyHandlers.ofString())
                      .thenApply(HttpResponse::body))
    .toList();

CompletableFuture.allOf(futures.toArray(new CompletableFuture[0])).join();
futures.forEach(f -> System.out.println(f.join()));
```

---

### Body Handlers and Publishers

| BodyHandler | Returns |
|-------------|---------|
| `ofString()` | `String` |
| `ofBytes()` | `byte[]` |
| `ofFile(path)` | `Path` (saves response to file) |
| `ofInputStream()` | `InputStream` |
| `ofLines()` | `Stream<String>` |
| `discarding()` | Discards response body |

| BodyPublisher | Sends |
|---------------|-------|
| `ofString(s)` | String as UTF-8 bytes |
| `ofByteArray(b)` | Raw byte array |
| `ofFile(path)` | Contents of a file |
| `ofInputStream(supplier)` | Stream of bytes |
| `noBody()` | Empty body (GET, DELETE) |

---

### HTTP Status Codes

```java
HttpResponse<String> response = client.send(request, HttpResponse.BodyHandlers.ofString());

int status = response.statusCode();

switch (status / 100) {
    case 2 -> System.out.println("Success: " + status);
    case 3 -> System.out.println("Redirect: " + status);
    case 4 -> System.out.println("Client error: " + status);
    case 5 -> System.out.println("Server error: " + status);
}
```

| Range | Meaning | Common codes |
|-------|---------|-------------|
| 2xx | Success | 200 OK, 201 Created, 204 No Content |
| 3xx | Redirection | 301 Moved, 302 Found, 304 Not Modified |
| 4xx | Client error | 400 Bad Request, 401 Unauthorized, 403 Forbidden, 404 Not Found |
| 5xx | Server error | 500 Internal Server Error, 502 Bad Gateway, 503 Service Unavailable |

---

### Authentication

```java
// Basic Auth — base64(username:password) in Authorization header
String credentials = java.util.Base64.getEncoder()
    .encodeToString("user:password".getBytes());

HttpRequest request = HttpRequest.newBuilder()
    .uri(URI.create("https://api.example.com/protected"))
    .header("Authorization", "Basic " + credentials)
    .build();

// Bearer token (OAuth2 / JWT)
HttpRequest request = HttpRequest.newBuilder()
    .uri(URI.create("https://api.example.com/protected"))
    .header("Authorization", "Bearer eyJhbGciOiJIUzI1NiJ9...")
    .build();
```

---

## Working with JSON

Java has no built-in JSON support — use a library. **Jackson** is the most widely used.

### Jackson (Maven)

```xml
<dependency>
    <groupId>com.fasterxml.jackson.core</groupId>
    <artifactId>jackson-databind</artifactId>
    <version>2.17.0</version>
</dependency>
```

### Object ↔ JSON

```java
import com.fasterxml.jackson.databind.ObjectMapper;

public record User(int id, String name, String email) {}

ObjectMapper mapper = new ObjectMapper();

// object → JSON string (serialization)
User user = new User(1, "Alice", "alice@example.com");
String json = mapper.writeValueAsString(user);
System.out.println(json);
// {"id":1,"name":"Alice","email":"alice@example.com"}

// pretty print
String pretty = mapper.writerWithDefaultPrettyPrinter().writeValueAsString(user);

// JSON string → object (deserialization)
String input = "{\"id\":2,\"name\":\"Bob\",\"email\":\"bob@example.com\"}";
User bob = mapper.readValue(input, User.class);
System.out.println(bob.name());  // Bob

// JSON → List
String jsonArray = "[{\"id\":1,\"name\":\"Alice\"},{\"id\":2,\"name\":\"Bob\"}]";
var users = mapper.readValue(jsonArray,
    mapper.getTypeFactory().constructCollectionType(java.util.List.class, User.class));

// JSON → Map (dynamic / unknown structure)
var map = mapper.readValue(json, java.util.Map.class);
System.out.println(map.get("name"));  // Alice
```

### Jackson with HttpClient

```java
ObjectMapper mapper = new ObjectMapper();

HttpResponse<String> response = client.send(
    HttpRequest.newBuilder()
        .uri(URI.create("https://jsonplaceholder.typicode.com/users/1"))
        .build(),
    HttpResponse.BodyHandlers.ofString()
);

if (response.statusCode() == 200) {
    User user = mapper.readValue(response.body(), User.class);
    System.out.println(user.name());
}
```

---

## TCP Sockets

For low-level TCP communication — custom protocols, game servers, IoT devices.

### TCP Client

```java
import java.net.*;
import java.io.*;

try (Socket socket = new Socket("echo.example.com", 7)) {
    var out = new PrintWriter(socket.getOutputStream(), true);
    var in  = new BufferedReader(new InputStreamReader(socket.getInputStream()));

    out.println("Hello, Server!");
    System.out.println("Echo: " + in.readLine());
}
```

### TCP Server

```java
import java.net.*;
import java.io.*;
import java.util.concurrent.Executors;

try (ServerSocket server = new ServerSocket(8080)) {
    System.out.println("Listening on port 8080");

    try (var executor = Executors.newVirtualThreadPerTaskExecutor()) {
        while (true) {
            Socket client = server.accept();  // blocks until a client connects
            executor.submit(() -> handleClient(client));
        }
    }
}

static void handleClient(Socket client) {
    try (client;
         var in  = new BufferedReader(new InputStreamReader(client.getInputStream()));
         var out = new PrintWriter(client.getOutputStream(), true)) {

        System.out.println("Connected: " + client.getInetAddress());
        String line;
        while ((line = in.readLine()) != null) {
            out.println("Echo: " + line);  // echo back
        }
    } catch (IOException e) {
        System.err.println("Client error: " + e.getMessage());
    }
}
```

---

## UDP Sockets

UDP is connectionless — faster than TCP but no delivery guarantee. Used for DNS, streaming, gaming.

```java
import java.net.*;

// UDP sender
try (DatagramSocket socket = new DatagramSocket()) {
    byte[] data = "Hello UDP".getBytes();
    InetAddress address = InetAddress.getByName("localhost");
    DatagramPacket packet = new DatagramPacket(data, data.length, address, 9090);
    socket.send(packet);
}

// UDP receiver
try (DatagramSocket socket = new DatagramSocket(9090)) {
    byte[] buffer = new byte[1024];
    DatagramPacket packet = new DatagramPacket(buffer, buffer.length);
    socket.receive(packet);  // blocks until data arrives
    String received = new String(packet.getData(), 0, packet.getLength());
    System.out.println("Received: " + received);
}
```

---

## WebSocket Client (Java 11+)

WebSockets provide full-duplex, persistent communication over a single TCP connection.

```java
import java.net.http.*;
import java.net.URI;
import java.util.concurrent.CompletableFuture;
import java.util.concurrent.CountDownLatch;

CountDownLatch latch = new CountDownLatch(3);

WebSocket ws = HttpClient.newHttpClient()
    .newWebSocketBuilder()
    .buildAsync(URI.create("wss://echo.websocket.org"), new WebSocket.Listener() {

        @Override
        public void onOpen(WebSocket webSocket) {
            System.out.println("Connected");
            webSocket.sendText("Hello!", true);
            WebSocket.Listener.super.onOpen(webSocket);
        }

        @Override
        public CompletableFuture<?> onText(WebSocket webSocket, CharSequence data, boolean last) {
            System.out.println("Received: " + data);
            latch.countDown();
            return WebSocket.Listener.super.onText(webSocket, data, last);
        }

        @Override
        public CompletableFuture<?> onClose(WebSocket webSocket, int statusCode, String reason) {
            System.out.println("Closed: " + reason);
            return WebSocket.Listener.super.onClose(webSocket, statusCode, reason);
        }

        @Override
        public void onError(WebSocket webSocket, Throwable error) {
            System.err.println("Error: " + error.getMessage());
        }
    }).join();

ws.sendText("Message 1", true);
ws.sendText("Message 2", true);

latch.await();  // wait for 3 messages
ws.sendClose(WebSocket.NORMAL_CLOSURE, "done").join();
```

---

## Network Utilities

```java
import java.net.*;

// local machine info
InetAddress local = InetAddress.getLocalHost();
System.out.println(local.getHostName());      // machine name
System.out.println(local.getHostAddress());   // IP address

// resolve a hostname
InetAddress google = InetAddress.getByName("google.com");
System.out.println(google.getHostAddress());  // e.g. 142.250.80.46

// all IPs for a hostname
InetAddress[] addrs = InetAddress.getAllByName("example.com");

// check if a host is reachable
boolean reachable = InetAddress.getByName("google.com").isReachable(3000);
System.out.println("Reachable: " + reachable);

// available port check
try (ServerSocket s = new ServerSocket(0)) {
    int availablePort = s.getLocalPort();
    System.out.println("Free port: " + availablePort);
}
```

---

## HTTP Best Practices

- **Reuse `HttpClient`** — it manages connection pooling internally. Create once, reuse everywhere.
- **Always set timeouts** — `connectTimeout` on the client and `timeout` on each request.
- **Handle all status codes** — don't assume 2xx. Check before parsing the body.
- **Use `sendAsync` for parallel requests** — avoids blocking a thread per request.
- **Encode query parameters** — use `URLEncoder.encode()` for user-provided values in query strings.
- **Set `Content-Type`** on POST/PUT requests — servers may reject requests without it.
- **Close response bodies** — if using `ofInputStream()`, always close the stream.
- **Don't log sensitive headers** — `Authorization`, `Cookie`, and API keys shouldn't appear in logs.

---

## Networking Summary

| Tool | Use |
|------|-----|
| `URI.create()` | Parse and construct resource identifiers |
| `HttpClient` | Modern HTTP/1.1 and HTTP/2 client (Java 11+) |
| `HttpRequest` | Build GET, POST, PUT, DELETE requests |
| `HttpResponse` | Access status, headers, and body |
| `BodyHandlers.ofString()` | Read response as a `String` |
| `sendAsync()` | Non-blocking HTTP request via `CompletableFuture` |
| `Socket` / `ServerSocket` | Raw TCP client and server |
| `DatagramSocket` | UDP communication |
| `WebSocket` | Persistent bi-directional messaging |
| `InetAddress` | DNS resolution and host info |
| Jackson `ObjectMapper` | Serialize/deserialize Java objects to/from JSON |

---

[← Back to README](../README.md)
