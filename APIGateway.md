### **API Gateway in Microservices: Routing, Configuration, and Predicates**  
An **API Gateway** acts as a single entry point for clients, handling routing, load balancing, security, and more. Popular tools include **Spring Cloud Gateway**, **Kong**, and **AWS API Gateway**.  

---

## **1. Key Roles of an API Gateway**  
| Function           | Description                                                                 |
|--------------------|-----------------------------------------------------------------------------|
| **Routing**        | Forward requests to appropriate microservices (`/orders` → OrderService).   |
| **Load Balancing** | Distribute traffic across service instances.                                |
| **Security**       | Authentication (JWT/OAuth2), rate limiting, IP whitelisting.               |
| **Aggregation**    | Combine responses from multiple services (GraphQL, BFF pattern).            |
| **Caching**        | Cache frequent responses (e.g., product listings).                          |

---

## **2. Routing Configuration**  
### **Option 1: Programmatic (Java/Code)**  
Example using **Spring Cloud Gateway**:  
```java
@Bean
public RouteLocator customRouteLocator(RouteLocatorBuilder builder) {
    return builder.routes()
        .route("order-service", r -> r.path("/orders/**")
            .uri("lb://order-service"))  // "lb" = Load-balanced via Eureka
        .route("user-service", r -> r.path("/users/**")
            .filters(f -> f.addRequestHeader("X-User-Id", "123"))
            .uri("lb://user-service"))
        .build();
}
```

### **Option 2: Declarative (YAML)**  
Example in `application.yml`:  
```yaml
spring:
  cloud:
    gateway:
      routes:
        - id: order-service
          uri: lb://order-service
          predicates:
            - Path=/orders/**
          filters:
            - AddRequestHeader=X-User-Id, 123
        - id: user-service
          uri: lb://user-service
          predicates:
            - Path=/users/**
```

---

## **3. Predicates (When to Route)**  
Predicates define conditions for routing. Common options:  

| Predicate          | Example                          | Description                              |
|--------------------|----------------------------------|------------------------------------------|
| **Path**          | `Path=/api/orders/**`           | Matches URL paths.                       |
| **Method**        | `Method=GET,POST`               | Filters by HTTP method.                  |
| **Header**        | `Header=X-Request-Id, \\d+`     | Checks request headers.                  |
| **Query**         | `Query=category,electronics`    | Matches query parameters.                |
| **Host**          | `Host=**.example.com`           | Routes by `Host` header.                 |
| **Weight**        | `Weight=group-name, 80`         | Splits traffic (A/B testing).            |

**Example**:  
```yaml
predicates:
  - Path=/products/**
  - Method=GET
  - Header=Authorization, Bearer.*
  - Query=sortBy,price
```

---

## **4. Filters (Request/Response Manipulation)**  
Filters modify requests/responses before/after routing.  

| Filter               | Example                                      | Purpose                                  |
|----------------------|----------------------------------------------|------------------------------------------|
| **AddRequestHeader** | `AddRequestHeader=X-User-Id, 123`           | Adds a header before forwarding.         |
| **RewritePath**      | `RewritePath=/old/(?<segment>.*), /new/\$\{segment}` | Rewrites URL paths.       |
| **Retry**            | `Retry=3,INTERNAL_SERVER_ERROR`              | Retries failed requests.                 |
| **CircuitBreaker**   | `CircuitBreaker=myCircuitBreaker`            | Integrates with Resilience4j.            |

**Example**:  
```yaml
filters:
  - AddRequestHeader=X-User-Id, 123
  - RewritePath=/api/v1/(?<segment>.*), /$\{segment}
  - Retry=3,INTERNAL_SERVER_ERROR
```

---

## **5. Dynamic Routing with Service Discovery**  
Integrate with **Eureka** or **Consul** to avoid hardcoding URIs:  
```yaml
spring:
  cloud:
    gateway:
      discovery:
        locator:
          enabled: true  # Auto-route via service IDs (e.g., /service-name/**)
          lower-case-service-id: true
```

**Result**:  
- `/order-service/**` → Routes to `order-service` (registered in Eureka).  
- `/user-service/**` → Routes to `user-service`.  

---

## **6. Programmatic vs. YAML Configuration**  
| **Aspect**          | **Programmatic (Java)**                          | **YAML/Properties**                          |
|----------------------|--------------------------------------------------|-----------------------------------------------|
| **Flexibility**      | High (dynamic routes, custom logic).             | Limited to static configuration.              |
| **Maintainability**  | Harder to manage for complex setups.             | Easier for simple, declarative routing.       |
| **Use Case**         | Conditional routing, custom filters.             | Standard routing, quick setup.                |

**Best Practice**: Use YAML for static routes and Java for dynamic logic.  

---

## **7. Example: Full API Gateway Setup**  
### **`application.yml`**  
```yaml
spring:
  cloud:
    gateway:
      routes:
        - id: order-service
          uri: lb://order-service
          predicates:
            - Path=/orders/**
          filters:
            - AddRequestHeader=X-Request-From, gateway
            - CircuitBreaker=orderServiceCB
        - id: user-service
          uri: lb://user-service
          predicates:
            - Path=/users/**
            - Method=GET,POST
```

### **With Eureka Integration**  
```yaml
eureka:
  client:
    service-url:
      defaultZone: http://localhost:8761/eureka
```

---

## **8. Advanced Features**  
- **Rate Limiting**: Use Redis with `RequestRateLimiter`.  
- **OAuth2**: Secure routes with JWT.  
- **gRPC Proxying**: Route gRPC traffic (requires additional setup).  

---

## **9. Tools Comparison**  
| **Tool**            | **Pros**                                      | **Cons**                          |
|----------------------|-----------------------------------------------|------------------------------------|
| **Spring Cloud Gateway** | Java-native, lightweight.               | Limited to Spring ecosystem.       |
| **Kong**            | Plugins (logging, auth), scalable.           | Requires DB (PostgreSQL).          |
| **AWS API Gateway**  | Serverless, integrates with AWS services.     | Vendor lock-in, costly at scale.   |

---

## **Summary**  
1. **Routing**: Map paths to microservices (YAML or Java).  
2. **Predicates**: Define conditions for routing (`Path`, `Header`, etc.).  
3. **Filters**: Modify requests/responses (`AddHeader`, `Retry`).  
4. **Service Discovery**: Auto-route via Eureka/Consul.  
5. **Use YAML** for simplicity, **Java** for dynamic logic.  

**Next Steps**:  
- Try [Spring Cloud Gateway](https://spring.io/projects/spring-cloud-gateway) with Eureka.  
- Explore [Kong](https://konghq.com/) for advanced use cases.  

Would you like a deep dive on securing the gateway with JWT?


-----------------------------

### ✅ **API Gateway in Microservices (Spring Cloud Gateway)**

---

## 🔷 What is an API Gateway?

An **API Gateway** acts as a **reverse proxy** that routes client requests to appropriate **microservices**. It’s a **single entry point** for all external requests in a microservices architecture.

---

## 🔧 **Why Use API Gateway?**

| Feature              | Purpose                                      |
| -------------------- | -------------------------------------------- |
| Routing              | Forward requests to appropriate microservice |
| Load balancing       | Distribute traffic among instances           |
| Security             | Handle authentication, authorization         |
| Rate limiting        | Protect backend services                     |
| Monitoring / Logging | Centralized request/response logging         |
| Protocol translation | E.g., HTTP → WebSocket, etc.                 |

---

## ✅ **Spring Cloud Gateway**

Spring Cloud Gateway is the **official gateway** developed by Spring for modern microservices.

---

## 🔹 Routing in Spring Cloud Gateway

There are two ways to configure routing:

---

### 📝 1. **YAML-based (Declarative) Configuration**

```yaml
spring:
  application:
    name: api-gateway
  cloud:
    gateway:
      routes:
        - id: user-service
          uri: http://localhost:8081
          predicates:
            - Path=/user/**
        - id: order-service
          uri: http://localhost:8082
          predicates:
            - Path=/order/**
```

---

### 🔧 2. **Java-based (Programmatic) Configuration**

```java
@Bean
public RouteLocator customRoutes(RouteLocatorBuilder builder) {
    return builder.routes()
        .route("user-service", r -> r
            .path("/user/**")
            .uri("http://localhost:8081"))
        .route("order-service", r -> r
            .path("/order/**")
            .uri("http://localhost:8082"))
        .build();
}
```

---

## 🎯 **Predicates (Routing Conditions)**

Predicates define **conditions** that determine which route should be selected. Common ones:

| Predicate  | Usage Example                       |
| ---------- | ----------------------------------- |
| Path       | `/user/**`                          |
| Method     | `GET`, `POST`, etc.                 |
| Header     | `Header=X-Request-Type, Value=json` |
| Host       | `*.example.com`                     |
| Cookie     | `name=value`                        |
| Query      | `?type=mobile`                      |
| RemoteAddr | IP-based filtering                  |
| Between    | Time-based routing                  |

```yaml
predicates:
  - Path=/api/**
  - Method=GET
  - Header=X-App-Version,1.0
```

---

## 🔄 **Filters (Modify Requests/Responses)**

Filters are used to:

* Add/remove headers
* Modify request path
* Retry
* Apply rate-limiting, logging, etc.

**Example:**

```yaml
filters:
  - AddRequestHeader=X-Request-ID, demo-id
  - RewritePath=/old-path/(?<segment>.*), /new-path/${segment}
```

---

## ⚙️ **Important Properties**

```yaml
server:
  port: 8080

spring:
  cloud:
    gateway:
      default-filters:
        - AddResponseHeader=X-Gateway-Global, true
      routes:
        - id: product-service
          uri: lb://PRODUCT-SERVICE
          predicates:
            - Path=/products/**
          filters:
            - name: Retry
              args:
                retries: 3
                statuses: BAD_GATEWAY
```

* `lb://` = Use **Eureka service discovery** + load balancer
* `filters:` = Apply logic before or after routing

---

## ✅ Example Flow

```
[Client]
   |
[API Gateway] --- routes via Path/Predicate --> [Product Service]
                          |
                          +-- Authentication filter
                          +-- Rate Limiting
                          +-- Monitoring
```

---

## 🧠 Summary Table

| Concept      | Description                                    |
| ------------ | ---------------------------------------------- |
| Routing      | Forwards request to microservices              |
| YAML Config  | Simple declarative config in `application.yml` |
| Programmatic | More dynamic control via `RouteLocator` bean   |
| Predicate    | Conditions like Path, Method, Header, etc.     |
| Filter       | Modify request/response, apply custom logic    |
| `lb://`      | Load-balanced URI for service discovery        |

---

Would you like a **complete Spring Boot API Gateway project** integrated with Eureka and secured with JWT filters and route-based logic?
