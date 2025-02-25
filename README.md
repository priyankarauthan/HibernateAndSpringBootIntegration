# Hibernate & Spring Boot Integration

## 📌 1. What is Hibernate & Why Use It?
Hibernate maps Java objects to database tables, allowing interaction without SQL.

- ✅ Reduces repetitive JDBC code
- ✅ Supports caching for faster performance
- ✅ Works across databases

## 📌 2. How is Hibernate Different from JDBC?
JDBC requires SQL queries and manual database handling.
Hibernate automates mapping and caching for better performance.

## 📌 3. Key Hibernate Components
- ✔ **SessionFactory** – Manages connections
- ✔ **Session** – Executes queries
- ✔ **Transaction** – Ensures commit/rollback

## 📌 4. What is HQL?
HQL works on Java objects instead of database tables.

- 🔹 **SQL Example**: `SELECT * FROM employees;`
- 🔹 **HQL Example**: `FROM Employee`

## 📌 5. What is Caching in Hibernate?
Caching improves query performance.

- ⚡ **First-Level Cache** – Default, session-specific.
- ⚡ **Second-Level Cache** – Configurable, shared across sessions.

## 📌 6. Lazy vs. Eager Loading
- 👉 **Lazy Loading** – Fetches data only when accessed.
- 👉 **Eager Loading** – Fetches all data immediately.

### 💡 Example:
```java
@OneToMany(fetch = FetchType.LAZY)
@OneToMany(fetch = FetchType.EAGER)
```

## 📌 7. How Does Hibernate Handle Transactions?
Hibernate ensures ACID transactions for consistency.

### ✅ Example:
```java
Session session = sessionFactory.openSession();
Transaction tx = session.beginTransaction();
session.save(employee);
tx.commit();
session.close();
```

## 📌 8. Common Hibernate Annotations
- `@Entity` – Marks a class as a database entity
- `@Table` – Maps a class to a table
- `@Id` – Specifies the primary key

## 📌 9. Hibernate Relationships
- ✔ `@OneToOne` – One-to-one
- ✔ `@OneToMany` – One-to-many
- ✔ `@ManyToMany` – Many-to-many

## 📌 10. Hibernate Integration with Spring Boot
💡 Add the following to `application.properties` to configure Hibernate and enable seamless interaction via Spring Data JPA.

```properties
spring.datasource.url=jdbc:mysql://localhost:3306/db
spring.datasource.username=root
spring.datasource.password=root
spring.jpa.hibernate.ddl-auto=update
spring.jpa.show-sql=true
```

---

# Global Exception Handling in Spring Boot

## 📌 1. Using `@ControllerAdvice`
The `@ControllerAdvice` annotation is used to create a global exception handler for all controllers. It centralizes exception handling logic, making the code cleaner and more maintainable.

### Steps:
1. Create a class annotated with `@ControllerAdvice`.
2. Define methods annotated with `@ExceptionHandler` to handle specific exceptions.
3. Customize the response using `ResponseEntity`.

### Example:
```java
@ControllerAdvice
public class GlobalExceptionHandler {

    @ExceptionHandler(ResourceNotFoundException.class)
    public ResponseEntity<String> handleResourceNotFoundException(ResourceNotFoundException ex) {
        return new ResponseEntity<>(ex.getMessage(), HttpStatus.NOT_FOUND);
    }

    @ExceptionHandler(Exception.class)
    public ResponseEntity<String> handleGenericException(Exception ex) {
        return new ResponseEntity<>("An error occurred: " + ex.getMessage(), HttpStatus.INTERNAL_SERVER_ERROR);
    }
}
```

### Benefits:
- Centralized exception handling.
- Reusable and maintainable.

## 📌 2. Using `@ExceptionHandler` in Controllers
For more localized exception handling, you can use the `@ExceptionHandler` annotation directly inside a controller class.

### Example:
```java
@RestController
public class ProductController {

    @GetMapping("/products/{id}")
    public Product getProductById(@PathVariable Long id) {
        return productService.findById(id).orElseThrow(() -> new ResourceNotFoundException("Product not found"));
    }

    @ExceptionHandler(ResourceNotFoundException.class)
    public ResponseEntity<String> handleResourceNotFoundException(ResourceNotFoundException ex) {
        return new ResponseEntity<>(ex.getMessage(), HttpStatus.NOT_FOUND);
    }
}
```

## 📌 3. Using `@ResponseStatus`
The `@ResponseStatus` annotation is used to mark exceptions with specific HTTP status codes.

### Example:
```java
@ResponseStatus(HttpStatus.NOT_FOUND)
public class ResourceNotFoundException extends RuntimeException {
    public ResourceNotFoundException(String message) {
        super(message);
    }
}
```

## 📌 4. Returning Custom Error Responses
You can define a custom error response structure for better clarity.

### Custom Error Response Class:
```java
public class ErrorResponse {
    private String message;
    private String details;

    public ErrorResponse(String message, String details) {
        this.message = message;
        this.details = details;
    }
    // Getters and setters
}
```

### Example with Global Exception Handler:
```java
@ControllerAdvice
public class GlobalExceptionHandler {

    @ExceptionHandler(ResourceNotFoundException.class)
    public ResponseEntity<ErrorResponse> handleResourceNotFound(ResourceNotFoundException ex) {
        ErrorResponse error = new ErrorResponse(ex.getMessage(), "Resource not found in the system");
        return new ResponseEntity<>(error, HttpStatus.NOT_FOUND);
    }

    @ExceptionHandler(Exception.class)
    public ResponseEntity<ErrorResponse> handleGenericException(Exception ex) {
        ErrorResponse error = new ErrorResponse("Internal Server Error", ex.getMessage());
        return new ResponseEntity<>(error, HttpStatus.INTERNAL_SERVER_ERROR);
    }
}
```

## 📌 5. Using `ResponseEntityExceptionHandler`
Spring provides a base class, `ResponseEntityExceptionHandler`, which you can extend to handle common exceptions.

### Example:
```java
@ControllerAdvice
public class CustomExceptionHandler extends ResponseEntityExceptionHandler {

    @Override
    protected ResponseEntity<Object> handleMethodArgumentNotValid(MethodArgumentNotValidException ex,
                                                                  HttpHeaders headers,
                                                                  HttpStatus status,
                                                                  WebRequest request) {
        String errors = ex.getBindingResult()
                          .getFieldErrors()
                          .stream()
                          .map(err -> err.getField() + ": " + err.getDefaultMessage())
                          .collect(Collectors.joining(", "));

        return new ResponseEntity<>(new ErrorResponse("Validation failed", errors), HttpStatus.BAD_REQUEST);
    }
}
```

## 📌 6. Logging Exceptions
Always log exceptions for debugging and monitoring purposes. Use frameworks like SLF4J with Logback or Log4j.

### Example:
```java
@ExceptionHandler(Exception.class)
public ResponseEntity<String> handleGenericException(Exception ex) {
    logger.error("An error occurred", ex);
    return new ResponseEntity<>("An error occurred: " + ex.getMessage(), HttpStatus.INTERNAL_SERVER_ERROR);
}
```

## 📌 7. Best Practices for Exception Handling
- Use **specific exceptions** for better clarity.
- Avoid leaking **internal details** in error messages.
- Standardize **error responses** across the application.
- Always **log exceptions** for monitoring and debugging.
- Implement **fallback mechanisms** for resilience.

##  How SAGA Design Pattern Works?
The SAGA Design Pattern manages long-running distributed transactions by breaking them into smaller steps, each with its own compensating action in case of failure. Here’s how it works:

Breaking Down the Transaction: A big transaction is divided into smaller, independent sub-transactions (steps), each handled by different services. For example, reserving a product, charging the customer, and shipping the item.
Independent Execution: Each step runs independently without waiting for the others to finish. If a step succeeds, the next step proceeds.
SAGA Execution Coordinator: The SAGA Execution Coordinator manages and coordinate the flow of these steps, triggering each one in sequence.
Compensating Actions: If any step fails, the system doesn’t roll back everything. Instead, it executes compensating actions to undo the work done in previous successful steps, like refunding a payment or canceling a reservation.
SAGA Log: Helps manage and track the state of a long-running distributed transaction, ensuring that all steps are completed successfully or properly compensated in case of failure.


### How JWT Works? 🔐


JWT (JSON Web Token) is a compact, self-contained token format used for securely transmitting information between parties as a JSON object. It is commonly used for authentication and authorization.

1️⃣ JWT Structure
A JWT consists of three parts, separated by dots (.):

css
Copy
Edit
header.payload.signature
Example:

eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiJ1c2VyMTIiLCJyb2xlIjoiVVNFUiIsImlhdCI6MTY5MjY0MzAwMCwiZXhwIjoxNjkyNjQ2NjAwfQ.8f5a9cS9Tgqz5Pz4Qcn4J4pTURZJh3wT_2Z
## 📌 Breakdown of JWT Parts

# 1️⃣ Header (Base64 Encoded JSON)

Contains metadata about the token, including the algorithm used for signing.
```
{
  "alg": "HS256",
  "typ": "JWT"
}
```
# 2️⃣ Payload (Base64 Encoded JSON)

Contains claims (user data, roles, and expiration time).
```
{
  "sub": "user12",
  "role": "USER",
  "iat": 1692643000,  // Issued at (Unix timestamp)
  "exp": 1692646600   // Expiration time
}
```
# 3️⃣ Signature

Ensures data integrity and prevents token tampering.
Generated using the header, payload, and secret key.
```
HMACSHA256(
    base64UrlEncode(header) + "." + base64UrlEncode(payload),
    secret_key
)
```

### 2️⃣ How JWT Authentication Works? 🚀

🔹 Step 1: User Logs In
User sends username & password to the authentication server.

🔹 Step 2: Server Generates JWT
Server verifies the credentials.
If valid, it creates a JWT and sends it to the user.

🔹 Step 3: User Sends JWT in API Requests
The user includes the JWT in the HTTP Authorization header.


Authorization: Bearer <JWT_TOKEN>

🔹 Step 4: Server Validates JWT
The server decodes and verifies the token using the secret key.
If valid, it allows access; otherwise, it denies the request.

### JWT Authentication Flow in Spring Boot

1️⃣ User Logs In → Server issues JWT
2️⃣ User includes JWT in API Requests
3️⃣ Server verifies JWT → Grants access
4️⃣ JWT expires → User must log in again or refresh token

###  Advantages of JWT
✅ Stateless & Scalable – No need to store session data.
✅ Secure & Tamper-proof – The signature prevents token modification.
✅ Efficient – Compact and can be used in URL, headers, or cookies.
✅ Cross-platform – Works with any language (Java, Python, Node.js, etc.).

###  JWT vs. Traditional Session-Based Authentication
Feature	JWT Authentication	Session-Based Authentication
State	Stateless (No DB storage)	Stateful (Session stored in DB)
Scalability	Highly Scalable	Limited Scalability
Security	Tamper-proof signature	Session hijacking risk
Storage	Stored on the client	Stored on the server
🔹 JWT in Spring Boot Security
Spring Boot uses JWT for authentication in APIs. Here’s an example of how to validate a JWT in Spring Boot:


http
    .authorizeHttpRequests(auth -> auth
        .requestMatchers("/api/admin/**").hasRole("ADMIN")
        .requestMatchers("/api/user/**").authenticated()
        .anyRequest().permitAll()
    )
    .oauth2ResourceServer(oauth2 -> oauth2.jwt());
### JWT Expiration & Refresh Tokens
JWT should expire to prevent misuse. You can use Refresh Tokens to get a new access token without logging in again.

1️⃣ Access Token (Short-lived, e.g., 15 mins)
2️⃣ Refresh Token (Long-lived, e.g., 7 days)

### Example: JWT Token Generation in Java

Here’s how you generate a JWT token in Java using io.jsonwebtoken (JJWT):

```
import io.jsonwebtoken.Jwts;
import io.jsonwebtoken.SignatureAlgorithm;

import java.util.Date;

public class JwtUtil {
    private static final String SECRET_KEY = "mySecretKey";

    public static String generateToken(String username) {
        return Jwts.builder()
                .setSubject(username)
                .setIssuedAt(new Date())
                .setExpiration(new Date(System.currentTimeMillis() + 1000 * 60 * 15)) // 15 min expiry
                .signWith(SignatureAlgorithm.HS256, SECRET_KEY)
                .compact();
    }
}
```

### 🔹 JWT Token Validation
```
import io.jsonwebtoken.Claims;
import io.jsonwebtoken.Jwts;

public class JwtValidator {
    private static final String SECRET_KEY = "mySecretKey";

    public static Claims validateToken(String token) {
        return Jwts.parser()
                .setSigningKey(SECRET_KEY)
                .parseClaimsJws(token)
                .getBody();
    }
}
```

### 💡 Summary
JWT is a stateless authentication mechanism that contains user claims in a self-contained token.
The server generates JWT upon successful login.
The user sends JWT in API requests (Authorization: Bearer <token>).
The server verifies JWT using a secret key and grants access.
JWT can expire, requiring token renewal using a refresh token.


### Difference between JpaRepository and CrudRepository

Use CrudRepository when you only need basic CRUD operations.
Use JpaRepository when you need additional features like pagination, sorting, batch operations, and flushing.

JpaRepository is a powerful extension of CrudRepository, offering: ✅ Advanced CRUD operations
✅ Pagination & Sorting
✅ Batch processing
✅ Immediate flushing
✅ Custom JPQL and Native Queries
✅ Dynamic queries with JpaSpecificationExecutor
✅ Entity graph support for optimized queries


### LOMBOK

Lombok is a Java library that helps reduce boilerplate code by automatically generating common methods like getters, setters, constructors, equals, hashCode, and toString at compile time using annotations.

## 📌 Complete List of Lombok Annotations
1️⃣ Getter & Setter
@Getter
@Setter
2️⃣ Constructors
@NoArgsConstructor
@AllArgsConstructor
@RequiredArgsConstructor
3️⃣ Object Methods
@ToString
@EqualsAndHashCode
@Data (Shortcut for @Getter, @Setter, @ToString, @EqualsAndHashCode, @RequiredArgsConstructor)
4️⃣ Builder Pattern
@Builder
@Singular (For collections in @Builder)
5️⃣ Logging
@Slf4j (SLF4J Logger)
@Log (Java Util Logging)
@CommonsLog (Apache Commons Logging)
@Log4j (Log4j)
@Log4j2 (Log4j2)
@XSlf4j (Extended SLF4J)
6️⃣ Exception Handling
@SneakyThrows (Sneaky exception handling)
7️⃣ Lazy Initialization
@LazyGetter
@LazySetter
8️⃣ Utility Class
@UtilityClass
9️⃣ Thread Safety
@Synchronized
🔟 Miscellaneous
@NonNull
@Cleanup (Auto-closing resources)
@Value (Immutable class with final fields)
@With (Creates a copy with modified fields)
@Accessors (Fluent setters)
@FieldDefaults (Set default field access modifiers)

## CIRCUIT BREAKER DESIGN PATTERN

How It Works:
Closed State: The consumer (client) calls the remote service normally.

Open State: If the remote service fails too many times (due to timeouts, errors, etc.), the circuit breaker opens and stops further calls to the failing service.

Half-Open State: After a cool-down period, a few test requests are allowed to check if the service has recovered.

Back to Closed State: If the service responds successfully, the circuit closes, and normal operations resume.



