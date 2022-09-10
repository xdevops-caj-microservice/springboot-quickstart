# Spring Boot Quickstart

## IDE

- [IntelliJ IDEA](https://www.jetbrains.com/idea/)
- [Spring Tool Suite](https://spring.io/tools)

## Create a new Spring Boot project

- [Spring Intializr](https://start.spring.io/)
- Spring Initializr when create a new project on IntelliJ IDEA


## Project dependencies

Click "Add Dependencies" in [Spring Intializr](https://start.spring.io/)

Common dependencies:
- Developer Tools
    - Spring Boot DevTools
    - Lombok
- Web
    - Spring Web
- SQL
    - Spring Data JPA
    - H2 Database
    - MySQL Driver
- Messaging
    - Spring for RabbitMQ

## Application configuration

Support both yaml and properties.

## Project structure

### Simple CRUD project structure

```bash
├── Application.java # Spring Boot Application
├── controller # Rest Controller, API layer
├── entity # JPA Entity
├── repository # JPA Repository
└── service # Service
```

### DDD project structure

See [ddd-code-structure](https://github.com/xdevops-caj-ddd/ddd-building-blocks/tree/main/ddd-code-structure)


## Development flow

### Simple CRUD project development flow

1. Create an Entity class with `@Entity` annotation
2. Create an Repository interface with `@Repository` annotation and extends `JpaRepository<T, Long>`
3. Create a Service class with `@Service` annotation
4. Create a Controller class with `@RestController` annotation
5. Implement APIs in the Controller class for each API endpoint
6. Implement business methods in the Service class

Request call path:
```bash
Controller -> Service -> Repository
```

Demp project:
- [department-service](https://github.com/xdevops-caj-microservice/dcb-sb-ms-example/tree/main/department-service)

## Run Spring Boot application

- Run the `main` method in Application class
- Run `mvn spring-boot:run`


## Development tips


## Use Lombok to simplify coding

See [Lombok](https://projectlombok.org/features/) documentation.

Other references:
- [A Complete Guide to Lombok](https://auth0.com/blog/a-complete-guide-to-lombok/)

### Customize servcie port

Configure a customized service port (default is `8080`) in `application.yaml`:
```yaml
server:
  port: 8081
```

### Error Handling

1. Define customized exception classes (subclass of `RuntimeException`) under `exception` package:
- ResourceAlreadyExistsException
- ResourceNotFoundException


**ResourceAlreadyExistsException.java**:
```java
public class ResourceNotFoundException extends RuntimeException {
    private String message;

    public ResourceNotFoundException() {
    }

    public ResourceNotFoundException(String message) {
        super(message);
        this.message = message;
    }
}
```

**ResourceAlreadyExistsException.java**
```java
public class ResourceNotFoundException extends RuntimeException {
    private String message;

    public ResourceNotFoundException() {
    }

    public ResourceNotFoundException(String message) {
        super(message);
        this.message = message;
    }
}
```

2. Throws ResourceAlreadyExistsException in creation methods in Service class.
```java
    public Department saveDepartment(Department department) {
        if (departmentRepository.existsById(department.getDepartmentId())) {
            throw new ResourceAlreadyExistsException();
        }
        return departmentRepository.save(department);
    }
```

3. Throws ResourceNotFoundException in query methods in Service class.
```java
    public Department findDepartmentById(Long id) {
        if (departmentRepository.findById(id).isEmpty()) {
            throw new ResourceNotFoundException();
        }
        return departmentRepository.findById(id).get();
    }
```

4. Define global exception hanlder via `@ControlAdvice` to hanlde exceptions
**GlobalExceptionHandler.java**
```java
@ControllerAdvice
public class GlobalExceptionHandler {
    @ExceptionHandler(ResourceAlreadyExistsException.class)
    public ResponseEntity handleResourceAlreadyExistsException(ResourceAlreadyExistsException e) {
        return new ResponseEntity("Resource already exits", HttpStatus.CONFLICT);
    }

    @ExceptionHandler(ResourceNotFoundException.class)
    public ResponseEntity handleResourceNotFoundException(ResourceNotFoundException e) {
        return new ResponseEntity("Resource not found", HttpStatus.NOT_FOUND);
    }
}
```


References:
- https://springframework.guru/exception-handling-in-spring-boot-rest-api/

### Bean validation

1. Import `spring-boot-starter-validation` dependency
```xml
  <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-validation</artifactId>
  </dependency>
```

2. Add validation annotations for the fields in the API resource class (for demo, API resource is same as the Entity class)
```java
    @NotBlank(message = "Department name is mandatory")
    private String departmentName;
```

3. Add `@Valid` annotion for method arguments in the Controller class
```java
    @PostMapping("")
    public Department saveDepartment(@RequestBody @Valid Department department) {
        log.info("Call saveDepartment() in DepartmentController...");
        return departmentService.saveDepartment(department);
    }
```

4. Add the `MethodArgumentNotValidException` exception handler in the `GlobalExceptionHandler` class (see previous "Error Handling")
```java
    @ExceptionHandler(MethodArgumentNotValidException.class)
    public ResponseEntity handleMethodArgumentNotValidException(MethodArgumentNotValidException e) {
        Map<String, String> errors = new HashMap<>();
        e.getBindingResult().getFieldErrors().forEach(error ->
                errors.put(error.getField(), error.getDefaultMessage()));
        return new ResponseEntity(errors, HttpStatus.BAD_REQUEST);
    }
```

References:
- https://springframework.guru/bean-validation-in-spring-boot/
- https://springframework.guru/exception-handling-in-spring-boot-rest-api/
- https://mkyong.com/spring-boot/spring-rest-validation-example/



#### Resource not found


### Unfiy API response





 