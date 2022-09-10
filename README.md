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

TBC


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
- 

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

### Input validation


### Error Handling

#### Resource not found


### Unfiy API response





 