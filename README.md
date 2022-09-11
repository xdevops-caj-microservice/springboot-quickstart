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
2. Create an Repository interface with `@Repository` annotation and extends `JpaRepository<T, ID>`
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


## References

- https://mkyong.com/tutorials/spring-boot-tutorials/


## Use Lombok to simplify coding

See [Lombok](https://projectlombok.org/features/) documentation.

Other references:
- [A Complete Guide to Lombok](https://auth0.com/blog/a-complete-guide-to-lombok/)

## Customize servcie port

Configure a customized service port (default is `8080`) in `application.yaml`:
```yaml
server:
  port: 8081
```

## Build REST API with Spring Boot

References:
- https://spring.io/guides/tutorials/rest/
- https://swagger.io/resources/articles/best-practices-in-api-design/

## Error Handling

1. Define customized exception classes (subclass of `RuntimeException`) under `exception` package:
- ResourceAlreadyExistsException
- ResourceNotFoundException


**ResourceAlreadyExistsException.java**:
```java
public class ResourceAlreadyExistsException extends RuntimeException {
    public ResourceAlreadyExistsException(Long id) {
        super("Resource already exists via id: " + id);
    }
}
```

**ResourceAlreadyExistsException.java**
```java
public class ResourceNotFoundException extends RuntimeException {
    public ResourceNotFoundException(Long id) {
        super("Resource not found via id: " + id);
    }
}
```

2. Throws `ResourceAlreadyExistsException` in creation methods in Service class.
```java
    public Department saveDepartment(Department department) {
        if (department.getDepartmentId() != null &&
                departmentRepository.existsById(department.getDepartmentId())) {
            throw new ResourceAlreadyExistsException(department.getDepartmentId());
        }
        return departmentRepository.save(department);
    }
```

3. Throws `ResourceNotFoundException` in query methods in Service class.
```java
    public Department findDepartmentById(Long id) {
        return departmentRepository.findById(id)
                .orElseThrow(() -> new ResourceNotFoundException(id));
    }
```

4. Define global exception hanlder via `@ControlAdvice` to hanlde exceptions

**GlobalExceptionHandler.java**
```java
@ControllerAdvice
public class GlobalExceptionHandler extends ResponseEntityExceptionHandler {
    @ExceptionHandler(ResourceAlreadyExistsException.class)
    public void handleResourceAlreadyExists(HttpServletResponse response) throws IOException {
        response.sendError(HttpStatus.CONFLICT.value());
    }

    @ExceptionHandler(ResourceNotFoundException.class)
    public void handleResourceNotFound(HttpServletResponse response) throws IOException {
        response.sendError(HttpStatus.NOT_FOUND.value());
    }
}
```

ResourceAlreadyExists error response:
```json
{
    "timestamp": "2022-09-10T10:54:04.877+00:00",
    "status": 409,
    "error": "Conflict",
    "message": "Resource already exists via id: 1",
    "path": "/departments"
}
```

ResourceNotFound error response:
```json
{
    "timestamp": "2022-09-10T10:55:52.020+00:00",
    "status": 404,
    "error": "Not Found",
    "message": "Resource not found via id: 2",
    "path": "/departments/2"
}
```




5. Remove `trace` info in error response for security concerns if use Spring Boot dev tools

**application.yaml***
```yaml
server:
  error:
    include-stacktrace: never
```

References:
- https://mkyong.com/spring-boot/spring-rest-error-handling-example/
- https://springframework.guru/exception-handling-in-spring-boot-rest-api/
- https://stackoverflow.com/questions/54827407/remove-trace-field-from-responsestatusexception

## Data validation

### Validating a Request Body

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

Notes:
> Use `@Valid` on Complex Types
If the Input class contains a field with another complex type that should be validated, this field, too, needs to be annotated with `@Valid`.

4. Add the `MethodArgumentNotValidException` exception handler method in the `GlobalExceptionHandler` class (see previous "Error Handling")
```java
    @Override
    protected ResponseEntity<Object> handleMethodArgumentNotValid(MethodArgumentNotValidException ex,
                                 HttpHeaders headers,
                                 HttpStatus status, WebRequest request) {
        Map<String, Object> body = new LinkedHashMap<>();
        body.put("timestamp", new Date());
        body.put("status", status.value());
        body.put("error", status.getReasonPhrase());
        body.put("message", "Validation failure");

        List<String> errors = ex.getBindingResult()
                .getFieldErrors()
                .stream()
                .map(x -> x.getDefaultMessage())
                .collect(Collectors.toList());

        body.put("errors", errors);

        return new ResponseEntity<>(body, headers, status);
    }
```

Validation failure error response:
```json
{
    "timestamp": "2022-09-10T11:02:56.353+00:00",
    "status": 400,
    "error": "Bad Request",
    "message": "Validation failure",
    "errors": [
        "Department code is mandatory",
        "Department address is mandatory"
    ]
}
```

References:
- https://mkyong.com/spring-boot/spring-rest-validation-example/
- https://springframework.guru/bean-validation-in-spring-boot/
- https://springframework.guru/exception-handling-in-spring-boot-rest-api/
- https://reflectoring.io/bean-validation-with-spring-boot/


### Validating Path Variables

1. Add `@Validated` annotaion in class level for Controller class

```java
@RestController
@RequestMapping("/departments")
@Validated
@Slf4j
public class DepartmentController {
  // ...
}
```

2. Add validation annotations (e.g. `@Min`) for path variables

```java
    @GetMapping("/{id}")
    public Department findDepartmentById(@PathVariable @Min(1) Long id) {
        log.info("Call findDepartmentById() in DepartmentController...");
        return departmentService.findDepartmentById(id);
    }
```

3. Add the `ConstraintViolationException` exception handler method in the `GlobalExceptionHandler` class (see previous "Error Handling")

```java
    @ExceptionHandler(ConstraintViolationException.class)
    public void handleConstraintViolationException(HttpServletResponse response) throws IOException {
        response.sendError(HttpStatus.BAD_REQUEST.value());
    }
```

Validation failure error response:
```json
{
    "timestamp": "2022-09-10T11:05:11.534+00:00",
    "status": 400,
    "error": "Bad Request",
    "message": "findDepartmentById.id: 最小不能小于1",
    "path": "/departments/0"
}
```

### Validaing via a custom validator

TBC


## Use contuctor injection over autowired

References:
- https://reflectoring.io/constructor-injection/

## Standardlize API response

## Make Restful API with Spring Hateoas

TBC

## Change the context path

Default context path is `/`.

Change the context path in `application.yaml`:
```yaml
server:
  contextPath: /hello
```

## Multi-profiles configuration

1. `application.yaml`
2. `application-{profile}.yaml`, profile normally is for environment:
    - local
    - dev
    - test
    - prod

Spring Boot, the default profile is `default`, use `spring.profiles.active={profile}` to set the active profile.


References
- https://docs.spring.io/spring-boot/docs/current/reference/html/features.html#features.external-config

## WebFlux for reactive programing ?

## REST API doc with Swagger

## Health check via Spring Boot actuator


## Complex query in JPA





 