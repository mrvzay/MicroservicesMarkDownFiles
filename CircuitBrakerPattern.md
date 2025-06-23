# The Circuit Breaker Pattern

The Circuit Breaker pattern is a resilience design pattern used in distributed systems to prevent cascading failures when a service or component becomes unavailable or starts failing.

## How It Works

The pattern is inspired by electrical circuit breakers, which trip to stop current flow when a fault is detected. Similarly, in software:

1. **Closed State**: Normal operation - requests flow to the service
2. **Open State**: When failures exceed a threshold, the circuit "trips" and fails fast without attempting the operation
3. **Half-Open State**: After a timeout, limited requests are allowed to test if the service has recovered

## Key Benefits

- **Prevents cascading failures**: Stops one failing service from bringing down the whole system
- **Fails fast**: Avoids long timeouts waiting for unresponsive services
- **Graceful degradation**: Allows systems to provide reduced functionality rather than complete failure
- **Automatic recovery**: Can automatically retry connections when the remote service recovers

## Implementation Components

1. **Failure Threshold**: Number of failures needed to trip the circuit
2. **Timeout Duration**: How long to stay open before testing recovery
3. **Fallback Mechanism**: Alternative path when circuit is open (cached data, default response, etc.)
4. **Monitoring**: Track circuit state changes for observability

## Example Use Cases

- Database access during outages
- External API calls (payment processors, weather services)
- Microservice-to-microservice communication
- Any remote resource that might become unavailable

## Sample Implementation (Pseudocode)

```
class CircuitBreaker {
  constructor(failureThreshold, recoveryTimeout) {
    this.state = 'CLOSED';
    this.failureCount = 0;
    this.failureThreshold = failureThreshold;
    this.recoveryTimeout = recoveryTimeout;
  }

  async execute(requestFn) {
    if (this.state === 'OPEN') {
      return this.fallback();
    }
    
    try {
      const result = await requestFn();
      this.reset();
      return result;
    } catch (error) {
      this.failureCount++;
      if (this.failureCount >= this.failureThreshold) {
        this.trip();
      }
      throw error;
    }
  }

  trip() {
    this.state = 'OPEN';
    setTimeout(() => {
      this.state = 'HALF_OPEN';
    }, this.recoveryTimeout);
  }

  reset() {
    this.state = 'CLOSED';
    this.failureCount = 0;
  }

  fallback() {
    // Return cached data or default response
  }
}
```

## Popular Libraries

- **Hystrix** (Netflix, now in maintenance)
- **Resilience4j** (Java)
- **Polly** (.NET)
- **Opossum** (Node.js)
- **go-breaker** (Go)

The Circuit Breaker pattern is particularly valuable in microservices architectures where service dependencies can create complex failure modes. It's often combined with other patterns like Retry, Timeout, and Bulkhead for comprehensive resilience.


---

The **Circuit Breaker Pattern** is a **resilience pattern** used in microservices and distributed systems to **prevent cascading failures** when a service is down, slow, or unreliable.

---

## 🔌 What is the Circuit Breaker Pattern?

It works like an **electrical circuit breaker**:

* If a service call **fails repeatedly**, the circuit "opens" to **stop further calls**, giving the downstream service time to recover.
* After a certain **cool-down time**, the circuit allows a few trial requests ("half-open").
* If the service is healthy again, the circuit closes; otherwise, it remains open.

---

## 🔄 Circuit Breaker States

| State         | Description                                                      |
| ------------- | ---------------------------------------------------------------- |
| **Closed**    | All requests are allowed through. Failures are counted.          |
| **Open**      | Requests are blocked immediately. Fallback logic is executed.    |
| **Half-Open** | A limited number of requests are allowed to test service health. |

---

## 📦 Real-World Libraries

* **Resilience4j** (Java)
* **Hystrix** (Netflix, now in maintenance mode)
* **Sentinel** (Alibaba)
* **Istio / Envoy** (Service Mesh)

---

## 🛠 Example (Spring Boot + Resilience4j)

```java
@CircuitBreaker(name = "employeeService", fallbackMethod = "fallbackGetEmployee")
public Employee getEmployeeDetails(String id) {
    return restTemplate.getForObject("http://employee-service/employees/" + id, Employee.class);
}

public Employee fallbackGetEmployee(String id, Throwable ex) {
    // fallback logic
    return new Employee(id, "Unavailable", "Fallback");
}
```

### 🧾 `application.yml`

```yaml
resilience4j.circuitbreaker:
  instances:
    employeeService:
      registerHealthIndicator: true
      slidingWindowSize: 10
      failureRateThreshold: 50
      waitDurationInOpenState: 10s
      permittedNumberOfCallsInHalfOpenState: 3
```

---

## ✅ Why Use Circuit Breaker?

| Benefit                      | Explanation                                           |
| ---------------------------- | ----------------------------------------------------- |
| 🚫 Avoid cascading failures  | Prevents one failure from taking down other services. |
| ⚡ Fast failure responses     | Returns fallback immediately when service is down.    |
| ♻️ Allows self-recovery      | Retries after some time, doesn't hammer the service.  |
| 📉 Improves system stability | Ensures high availability and fault tolerance.        |

---

## 🧠 When to Use It?

* External service calls (e.g., REST APIs, DB calls)
* Long-running or failure-prone services
* Network-sensitive dependencies

---

## 🔄 Related Patterns

* **Retry**: Retry failed requests before giving up.
* **Timeout**: Limit how long a request can wait.
* **Bulkhead**: Isolate services so one failure doesn’t affect others.
* **Fallback**: Provide a default response or cache.

---

Would you like a **complete Spring Boot project** using Resilience4j and Circuit Breaker with fallback and monitoring (e.g., via actuator or Prometheus)?
