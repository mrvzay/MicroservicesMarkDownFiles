### **Eureka Server: Service Discovery & Load Balancing in Microservices**  
**Eureka** is a Netflix OSS tool for **service discovery** and **client-side load balancing** in microservices architectures. It helps services dynamically find and communicate with each other without hardcoding IPs/ports.  

---

## **1. What is Eureka Server?**  
- **Service Registry**: Central server where microservices register themselves.  
- **Health Monitoring**: Tracks service availability (heartbeats).  
- **Client-Side Load Balancing**: Integrates with Ribbon to distribute requests.  

#### **Key Components**  
| Component       | Role                                                                 |
|----------------|----------------------------------------------------------------------|
| **Eureka Server** | Registry holding service metadata (host, port, health status).       |
| **Eureka Client** | Services that register with Eureka and fetch registry data.          |
| **Ribbon**       | Client-side load balancer (deprecated but still used in legacy systems; newer alternatives include Spring Cloud LoadBalancer). |

---

## **2. How to Set Up a Eureka Server**  
### **Step 1: Add Dependencies (Spring Boot)**  
```xml
<!-- Eureka Server -->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>
</dependency>
```

### **Step 2: Enable Eureka Server**  
Annotate your main class with `@EnableEurekaServer`:  
```java
@SpringBootApplication
@EnableEurekaServer
public class EurekaServerApplication {
    public static void main(String[] args) {
        SpringApplication.run(EurekaServerApplication.class, args);
    }
}
```

### **Step 3: Configure Eureka (`application.yml`)**  
```yaml
server:
  port: 8761  # Default Eureka port

eureka:
  client:
    register-with-eureka: false  # This server doesn't register itself
    fetch-registry: false        # No need to fetch other registries
  instance:
    hostname: localhost
```

### **Step 4: Run and Access the Dashboard**  
- Start the server and visit:  
  **`http://localhost:8761`**  
  ![Eureka Dashboard](https://miro.medium.com/v2/resize:fit:1400/1*_6MfwckxNfQTz6-Lh7Kv4A.png)

---

## **3. Registering Services with Eureka (Eureka Client)**  
### **Step 1: Add Eureka Client Dependency**  
```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
</dependency>
```

### **Step 2: Enable Eureka Client**  
Annotate your service’s main class:  
```java
@SpringBootApplication
@EnableEurekaClient  // or @EnableDiscoveryClient (generic)
public class OrderServiceApplication { ... }
```

### **Step 3: Configure Service Registration**  
```yaml
# order-service/src/main/resources/application.yml
spring:
  application:
    name: order-service  # Service ID in Eureka

eureka:
  client:
    service-url:
      defaultZone: http://localhost:8761/eureka  # Eureka server URL
  instance:
    prefer-ip-address: true
```

### **Step 4: Verify Registration**  
- Restart the service and check the Eureka dashboard:  
  ![Registered Services](https://www.baeldung.com/wp-content/uploads/2018/08/Screenshot-2018-08-05-at-2.03.30-PM.png)

---

## **4. Load Balancing with Eureka + Ribbon**  
Eureka integrates with **Ribbon** (client-side LB) to distribute requests among service instances.  

### **Step 1: Add Ribbon (Legacy)**  
```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-ribbon</artifactId>
</dependency>
```

### **Step 2: Use `@LoadBalanced` with RestTemplate**  
```java
@Bean
@LoadBalanced  // Enables Ribbon LB
public RestTemplate restTemplate() {
    return new RestTemplate();
}
```

### **Step 3: Call Other Services Using Service ID**  
```java
@Service
public class PaymentService {
    @Autowired
    private RestTemplate restTemplate;

    public String callOrderService() {
        // Ribbon resolves "order-service" to an available instance
        return restTemplate.getForObject(
            "http://order-service/orders",  // Service ID (not URL)
            String.class
        );
    }
}
```

### **How Ribbon Works**  
1. Fetches all instances of `order-service` from Eureka.  
2. Applies LB algorithm (round-robin by default).  
3. Sends the request to a healthy instance.  

---

## **5. Key Features & Best Practices**  
### **Features**  
- **Self-Preservation Mode**: Protects the registry during network issues.  
- **Zone Awareness**: Prioritizes services in the same AWS/AZ zone.  
- **Health Checks**: Integrates with Spring Boot Actuator.  

### **Best Practices**  
1. **High Availability**: Deploy multiple Eureka servers (peer-aware mode).  
   ```yaml
   # Eureka Server 1 config
   eureka:
     client:
       service-url:
         defaultZone: http://eureka-server2:8762/eureka
   ```
2. **Service Heartbeats**: Default is 30 seconds; adjust if needed:  
   ```yaml
   eureka:
     instance:
       lease-renewal-interval-in-seconds: 10
   ```
3. **Use Spring Cloud LoadBalancer** (modern alternative to Ribbon).  

---

## **6. Alternatives to Eureka**  
| Tool               | Pros                                      | Cons                          |
|--------------------|------------------------------------------|-------------------------------|
| **Consul**         | Multi-DC support, KV store               | Complex setup                 |
| **Zookeeper**      | Strong consistency                       | No built-in LB                |
| **Kubernetes DNS** | Native in K8s                            | Limited to K8s environments   |
| **Nginx**          | Simple LB                                | No service discovery          |

---

## **Summary**  
1. **Eureka Server** acts as a service registry.  
2. **Services register** themselves on startup.  
3. **Ribbon** (or Spring Cloud LB) distributes requests.  
4. **Dashboard** at `http://localhost:8761` shows live services.  

For new projects, consider **Spring Cloud LoadBalancer** (replacement for Ribbon) or **Consul** for advanced features.  

Would you like a tutorial on setting up Eureka in AWS or Kubernetes?

--------------------------------

### ✅ **Eureka Server in Microservices (with Spring Boot)**

---

## 🔹 What is Eureka?

**Eureka** is a **Service Discovery Server** provided by **Netflix OSS** and integrated into **Spring Cloud**.

It allows microservices to:

* **Register themselves** (service registry)
* **Discover other services** dynamically (service discovery)
* Enable **client-side load balancing** with **Ribbon** or **Spring Cloud LoadBalancer**

---

## 🔧 Why Do We Need Eureka?

Without service discovery:

* Services must know the **IP and port** of every other service (hardcoded or config-driven)

With Eureka:

* Services only need the **logical service name**
* Eureka handles **dynamic discovery**, **health checks**, and **load balancing**

---

## ✅ Step-by-Step: Setting up Eureka Server

### 1️⃣ **Create a Eureka Server (Spring Boot App)**

**Dependencies** (via Spring Initializr):

* `spring-cloud-starter-netflix-eureka-server`
* `spring-boot-starter-web`

**Main Class:**

```java
@SpringBootApplication
@EnableEurekaServer
public class EurekaServerApplication {
    public static void main(String[] args) {
        SpringApplication.run(EurekaServerApplication.class, args);
    }
}
```

**application.yml:**

```yaml
server:
  port: 8761

eureka:
  client:
    register-with-eureka: false
    fetch-registry: false
```

---

## ✅ Registering Microservices to Eureka

Each microservice must:

### 2️⃣ **Add Eureka Client Dependency**

```xml
<dependency>
  <groupId>org.springframework.cloud</groupId>
  <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
</dependency>
```

### 3️⃣ **Enable Eureka Client**

```java
@SpringBootApplication
@EnableEurekaClient // Optional with recent Spring Cloud
public class MyServiceApplication {
    public static void main(String[] args) {
        SpringApplication.run(MyServiceApplication.class, args);
    }
}
```

### 4️⃣ **Configure application.yml**

```yaml
spring:
  application:
    name: product-service

eureka:
  client:
    service-url:
      defaultZone: http://localhost:8761/eureka/
```

✅ Now your service will appear in **[http://localhost:8761](http://localhost:8761)** Eureka Dashboard!

---

## ⚖️ Load Balancing with Eureka Clients

Spring Boot supports **client-side load balancing** using:

### ✅ 1. **Spring Cloud LoadBalancer (default in recent versions)**

**Service-to-service call:**

```java
@Autowired
private WebClient.Builder webClientBuilder;

public String getFromOtherService() {
    return webClientBuilder.build()
            .get()
            .uri("http://order-service/api/orders") // Logical name from Eureka
            .retrieve()
            .bodyToMono(String.class)
            .block();
}
```

### ✅ 2. **RestTemplate with LoadBalancer (older style)**

```java
@Bean
@LoadBalanced
public RestTemplate restTemplate() {
    return new RestTemplate();
}
```

```java
restTemplate.getForObject("http://order-service/api/orders", String.class);
```

---

## 🔄 How Load Balancing Works

* Eureka returns **all healthy instances** of the target service.
* Spring Cloud LoadBalancer chooses one using **round-robin** or **custom strategy**.
* No need for external load balancer.

---

## 💡 Extra Features of Eureka

| Feature           | Description                                    |
| ----------------- | ---------------------------------------------- |
| Self-registration | Services register and renew themselves         |
| Heartbeat         | Periodic health check (default every 30s)      |
| Zone-awareness    | Choose services from the same zone/data center |
| Dashboard         | Web UI to view registered services             |

---

## 🧠 Summary

| Topic               | Description                                        |
| ------------------- | -------------------------------------------------- |
| Eureka              | Service registry for microservices                 |
| @EnableEurekaServer | Used in Eureka Server                              |
| @EnableEurekaClient | Used in Eureka Clients                             |
| Load balancing      | Achieved via RestTemplate/WebClient + service name |
| Service Discovery   | Dynamic — no need for hardcoded URLs               |

---

Would you like a **complete end-to-end code project** with Eureka Server, two microservices, and inter-service calls using load balancing?
