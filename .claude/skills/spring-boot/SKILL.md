---
name: spring-boot
description: Spring Boot 3.x development - REST APIs, JPA, Security, Testing, and Cloud-native patterns. Use for building enterprise Java applications with Spring Boot.
metadata:
  version: "2.0.0"
  domain: backend
  triggers: Spring Boot, Spring Framework, Spring Security, Spring Data JPA, Spring WebFlux, Java REST API, Microservices Java
  role: specialist
  scope: implementation
  output-format: code
---

# Spring Boot Skill

Enterprise Spring Boot 3.x development with focus on clean architecture and production-ready code.

## Done means

Run the owning repo's test command (`./mvnw test`, or `mvn test` where there is no wrapper) and `spotless:apply`, and confirm the tests are green before reporting the work done.

## Quick Start Templates

### Entity
```java
@Entity
@Table(name = "products")
@Getter
@Setter
@NoArgsConstructor
public class Product {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @NotBlank
    private String name;

    @DecimalMin("0.0")
    private BigDecimal price;
}
```

### Repository
```java
public interface ProductRepository extends JpaRepository<Product, Long> {
    List<Product> findByNameContainingIgnoreCase(String name);
}
```

### Service
```java
@Service
@Transactional(readOnly = true)
public class ProductService {
    private final ProductRepository repo;

    public ProductService(ProductRepository repo) {
        this.repo = repo;
    }

    public List<Product> search(String name) {
        return repo.findByNameContainingIgnoreCase(name);
    }

    @Transactional
    public Product create(ProductRequest request) {
        var product = new Product();
        product.setName(request.name());
        product.setPrice(request.price());
        return repo.save(product);
    }
}
```

### REST Controller
```java
@RestController
@RequestMapping("/api/v1/products")
@Validated
public class ProductController {
    private final ProductService service;

    public ProductController(ProductService service) {
        this.service = service;
    }

    @GetMapping
    public List<Product> search(@RequestParam(defaultValue = "") String name) {
        return service.search(name);
    }

    @PostMapping
    @ResponseStatus(HttpStatus.CREATED)
    public Product create(@Valid @RequestBody ProductRequest request) {
        return service.create(request);
    }
}
```

### DTO (Record)
```java
public record ProductRequest(
    @NotBlank String name,
    @DecimalMin("0.0") BigDecimal price
) {}
```

### Global Exception Handler
```java
@RestControllerAdvice
public class GlobalExceptionHandler {
    @ExceptionHandler(MethodArgumentNotValidException.class)
    @ResponseStatus(HttpStatus.BAD_REQUEST)
    public Map<String, String> handleValidation(MethodArgumentNotValidException ex) {
        return ex.getBindingResult().getFieldErrors().stream()
            .collect(Collectors.toMap(FieldError::getField,
                    error -> error.getDefaultMessage() != null ? error.getDefaultMessage() : "Invalid"));
    }

    @ExceptionHandler(EntityNotFoundException.class)
    @ResponseStatus(HttpStatus.NOT_FOUND)
    public Map<String, String> handleNotFound(EntityNotFoundException ex) {
        return Map.of("error", ex.getMessage());
    }
}
```

### Test Slice
```java
@WebMvcTest(ProductController.class)
class ProductControllerTest {
    @Autowired MockMvc mockMvc;
    @MockitoBean ProductService service; // org.springframework.test.context.bean.override.mockito.MockitoBean

    @Test
    void createProduct_validRequest_returns201() throws Exception {
        var product = new Product();
        product.setName("Widget");
        when(service.create(any())).thenReturn(product);

        mockMvc.perform(post("/api/v1/products")
                .contentType(MediaType.APPLICATION_JSON)
                .content("""
                    {"name":"Widget","price":10.0}
                    """))
            .andExpect(status().isCreated())
            .andExpect(jsonPath("$.name").value("Widget"));
    }
}
```

## Reference Guide

Load detailed patterns based on context:

| Topic | Reference | When to Load |
|-------|-----------|-------------|
| Web/REST | `references/web.md` | Controllers, validation, exception handling |
| Data Access | `references/data.md` | JPA, repositories, transactions, queries |
| Security | `references/security.md` | Spring Security 6, OAuth2, JWT, auth |
| Cloud/Config | `references/cloud.md` | Config server, discovery, resilience |
| Testing | `references/testing.md` | Unit, integration, slice tests |

## Constraints

- Constructor injection only; no field `@Autowired` in beans. This matches the repos' CLAUDE.md files.
- Validate request bodies with `@Valid`, and handle errors in one `@RestControllerAdvice`.
- Read configuration through `@ConfigurationProperties`. Keep secrets and environment-specific URLs out of `application.*`; they come from env vars or Consul.

## Architecture Patterns

**Project Structure:** packages are feature-first, with layers inside each feature. scan-manager, for example, uses `com.getvisibility.scanmanager.<domain>.<layer>`. Follow the owning repo's CLAUDE.md for its exact layout.
```
src/main/java/com/getvisibility/<service>/
├── <domain>/          # one package per feature (e.g. scan, connector, tagging)
│   ├── controller/    # implements the OpenAPI-generated interface
│   ├── service/
│   └── repository/
└── configs/           # cross-cutting Spring config beans
```

**Clean Architecture Principles:**
- Domain models independent of frameworks
- Use case driven design
- Dependency inversion (interfaces)
- Clear boundaries between layers

## Common Annotations

| Annotation | Purpose |
|------------|---------|
| `@RestController` | REST controller (combines @Controller + @ResponseBody) |
| `@Service` | Business logic component |
| `@Repository` | Data access component |
| `@Transactional` | Transaction management |
| `@Valid` | Trigger validation |
| `@ConfigurationProperties` | Bind properties to class |
| `@EnableMethodSecurity` | Enable method security |

## Reactive WebFlux Endpoint

```java
@RestController
@RequestMapping("/api/v1/orders")
public class OrderController {
    private final OrderService orderService;

    public OrderController(OrderService orderService) {
        this.orderService = orderService;
    }

    @GetMapping("/{id}")
    public Mono<ResponseEntity<OrderDto>> getOrder(@PathVariable UUID id) {
        return orderService.findById(id)
                .map(ResponseEntity::ok)
                .defaultIfEmpty(ResponseEntity.notFound().build());
    }

    @PostMapping
    @ResponseStatus(HttpStatus.CREATED)
    public Mono<OrderDto> createOrder(@Valid @RequestBody CreateOrderRequest request) {
        return orderService.create(request);
    }
}
```

## Spring Security JWT

```java
@Configuration
@EnableMethodSecurity
public class SecurityConfig {
    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        return http
                .csrf(AbstractHttpConfigurer::disable)
                .sessionManagement(s -> s.sessionCreationPolicy(STATELESS))
                .authorizeHttpRequests(auth -> auth
                        .requestMatchers("/actuator/health").permitAll()
                        .anyRequest().authenticated())
                .oauth2ResourceServer(oauth2 -> oauth2.jwt(Customizer.withDefaults()))
                .build();
    }
}
```

## Knowledge Base

Spring Boot 3.5, Java 25, Spring Kafka, Spring Data Redis, Spring Security 6, OAuth2/JWT (Keycloak), Resilience4j, Failsafe, Micrometer, JUnit 5, Testcontainers, Mockito, AssertJ, Instancio, Maven
