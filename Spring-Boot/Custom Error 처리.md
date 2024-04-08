Spring Boot 는 기본적으로 throw에 따라서 적절한 response를 보내도록 설계되어 있다.
ResponseEntityExceptionHandler 는 추상 클래스로 상속받아서 사용하면 된다.

> 에러응답의 body를 어떻게 보내줄지 정해주었다.
```JAVA  
public class ErrorDetails {  
    private LocalDateTime timestamp;  
    private String message;  
    private String details;  
  
    public ErrorDetails(LocalDateTime timestamp, String message, String details) {  
        this.timestamp = timestamp;  
        this.message = message;  
        this.details = details;  
    }  
  
    public LocalDateTime getTimestamp() {  
        return timestamp;  
    }  
  
    public String getMessage() {  
        return message;  
    }  
  
    public String getDetails() {  
        return details;  
    }  
}
```

```JAVA
@ControllerAdvice  
public class CustomizedResponseEntityExceptionHandler extends ResponseEntityExceptionHandler {  
  
    @ExceptionHandler(Exception.class)  
    public final ResponseEntity<ErrorDetails> handleAllException(Exception ex, WebRequest request) {  
        ErrorDetails errorDetails = new ErrorDetails(LocalDateTime.now(),  
                ex.getMessage(), request.getDescription(false));  
  
        return new ResponseEntity<ErrorDetails>(errorDetails, HttpStatus.INTERNAL_SERVER_ERROR);  
    }  
  
    @ExceptionHandler(UserNotFoundException.class)  
    public final ResponseEntity<ErrorDetails> handleUserNotFoundException(Exception ex, WebRequest request) {  
        ErrorDetails errorDetails = new ErrorDetails(LocalDateTime.now(),  
                ex.getMessage(), request.getDescription(false));  
        System.out.println("okay");  
        return new ResponseEntity<ErrorDetails>(errorDetails, HttpStatus.NOT_FOUND);  
    }  
}
```
