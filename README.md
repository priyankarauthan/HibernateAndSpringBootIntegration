1. Global Exception Handling using @ControllerAdvice
The @ControllerAdvice annotation is used to create a global exception handler for all controllers. It centralizes exception handling logic, making the code cleaner and more maintainable.

Steps:

Create a class annotated with @ControllerAdvice.
Define methods annotated with @ExceptionHandler to handle specific exceptions.
Customize the response using ResponseEntity.
Example:

java
Copy
Edit
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
Benefits:

Centralized exception handling.
Reusable and maintainable.
2. Using @ExceptionHandler in Controllers
For more localized exception handling, you can use the @ExceptionHandler annotation directly inside a controller class.

Example:

java
Copy
Edit
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
3. Using @ResponseStatus
The @ResponseStatus annotation is used to mark exceptions with specific HTTP status codes. This is useful for simple cases where you don’t need a custom response body.

Example:

java
Copy
Edit
@ResponseStatus(HttpStatus.NOT_FOUND)
public class ResourceNotFoundException extends RuntimeException {
    public ResourceNotFoundException(String message) {
        super(message);
    }
}
4. Returning Custom Error Responses
You can define a custom error response structure for better clarity.

Custom Error Response Class:

java
Copy
Edit
public class ErrorResponse {
    private String message;
    private String details;

    public ErrorResponse(String message, String details) {
        this.message = message;
        this.details = details;
    }

    // Getters and setters
}
Example with Global Exception Handler:

java
Copy
Edit
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
5. Using ResponseEntityExceptionHandler
Spring provides a base class, ResponseEntityExceptionHandler, which you can extend to handle common exceptions like MethodArgumentNotValidException or HttpMessageNotReadableException.

Example:

java
Copy
Edit
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
6. Logging Exceptions
Always log exceptions for debugging and monitoring purposes. Use frameworks like SLF4J with Logback or Log4j.

Example:

java
Copy
Edit
@ExceptionHandler(Exception.class)
public ResponseEntity<String> handleGenericException(Exception ex) {
    logger.error("An error occurred", ex);
    return new ResponseEntity<>("An error occurred: " + ex.getMessage(), HttpStatus.INTERNAL_SERVER_ERROR);
}
7. Handling Exceptions in RESTful APIs
When building REST APIs, it's important to return structured error responses. Use JSON or XML for error representation with clear fields like timestamp, message, and details.

Example Response:

json
Copy
Edit
{
    "timestamp": "2025-01-28T10:30:00Z",
    "message": "Resource not found",
    "details": "Product ID 123 does not exist"
}
Best Practices
Use Specific Exceptions: Create custom exceptions for better clarity.
Avoid Leaking Internal Details: Don't expose sensitive information in error messages.
Standardize Responses: Ensure consistency in error responses across the application.
Log Exceptions: Always log exceptions for monitoring and debugging.
Fallback Mechanisms: Use fallback mechanisms (like Circuit Breakers) for resilience.
