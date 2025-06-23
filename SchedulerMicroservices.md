# Creating a Scheduler Microservice

Here's a comprehensive guide to building a scheduler as a microservice:

## Architecture Options

### 1. Standalone Scheduler Service
**Components**:
- **Scheduler Core**: Manages job scheduling/execution
- **Job Store**: Persistent storage for jobs (database)
- **API Endpoints**: REST/gRPC for job management
- **Worker Pool**: Executes scheduled tasks

### 2. Distributed Task Scheduler
**Components**:
- **Coordination Service** (Zookeeper/etcd)
- **Leader Election**: For master node
- **Worker Nodes**: Execute tasks
- **Distributed Locking**: For job consistency

## Implementation Approaches

### Option 1: Using Spring Boot + Quartz (Java)

```java
// 1. Add dependencies (pom.xml)
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-quartz</artifactId>
</dependency>

// 2. Configure Quartz
@Configuration
public class QuartzConfig {
    @Bean
    public JobDetail sampleJobDetail() {
        return JobBuilder.newJob(SampleJob.class)
            .withIdentity("sampleJob")
            .storeDurably()
            .build();
    }

    @Bean
    public Trigger sampleJobTrigger() {
        SimpleScheduleBuilder schedule = 
            SimpleScheduleBuilder.simpleSchedule()
                .withIntervalInSeconds(10)
                .repeatForever();

        return TriggerBuilder.newTrigger()
            .forJob(sampleJobDetail())
            .withIdentity("sampleTrigger")
            .withSchedule(schedule)
            .build();
    }
}

// 3. Define Job
public class SampleJob implements Job {
    @Override
    public void execute(JobExecutionContext context) {
        // Job logic here
    }
}
```

### Option 2: Using Celery + Redis (Python)

```python
# app.py
from celery import Celery

app = Celery('scheduler',
             broker='redis://localhost:6379/0',
             backend='redis://localhost:6379/0')

@app.task
def scheduled_task(param):
    # Task implementation
    return f"Processed {param}"

# To schedule:
from datetime import datetime, timedelta
scheduled_task.apply_async(eta=datetime.now() + timedelta(minutes=5))
```

## Key Considerations

### 1. Persistence Layer
- **Database Schema**:
  ```sql
  CREATE TABLE scheduled_jobs (
    id VARCHAR(36) PRIMARY KEY,
    job_name VARCHAR(255) NOT NULL,
    cron_expression VARCHAR(50),
    next_run TIMESTAMP,
    last_run TIMESTAMP,
    status ENUM('ACTIVE','PAUSED','COMPLETED'),
    payload JSON
  );
  ```

### 2. Fault Tolerance
- **Recovery Mechanism**:
  ```java
  @Scheduled(fixedDelay = 30000)
  public void recoverMissedJobs() {
      List<Job> missedJobs = jobRepository
          .findByNextRunBeforeAndStatus(LocalDateTime.now(), ACTIVE);
      
      missedJobs.forEach(job -> {
          jobExecutor.execute(job);
          job.setNextRun(calculateNextRun(job));
          jobRepository.save(job);
      });
  }
  ```

### 3. Scaling Approach

**Vertical Partitioning**:
```yaml
# docker-compose.yml
services:
  scheduler-master:
    image: scheduler-service
    environment:
      - ROLE=master
  scheduler-worker1:
    image: scheduler-service 
    environment:
      - ROLE=worker
  scheduler-worker2:
    image: scheduler-service
    environment:
      - ROLE=worker
```

### 4. API Design (REST Example)

**Endpoints**:
```
POST   /api/jobs       - Create new job
GET    /api/jobs       - List all jobs
GET    /api/jobs/{id}  - Get job details
PUT    /api/jobs/{id}  - Update job
DELETE /api/jobs/{id}  - Delete job
POST   /api/jobs/{id}/execute - Manual trigger
```

### 5. Monitoring

**Prometheus Metrics**:
```java
@Bean
public MeterRegistryCustomizer<PrometheusMeterRegistry> metrics() {
    return registry -> {
        registry.config().commonTags("application", "scheduler-service");
        Gauge.builder("scheduler.jobs.queued", 
                     jobQueue, 
                     q -> q.size())
            .register(registry);
    };
}
```

## Advanced Features

1. **Dynamic Scheduling**:
```java
public void addDynamicJob(String jobName, String cronExpression) {
    JobDetail job = newJob(DynamicJob.class)
        .withIdentity(jobName)
        .build();
    
    Trigger trigger = newTrigger()
        .withIdentity(jobName + "-trigger")
        .withSchedule(cronSchedule(cronExpression))
        .build();
    
    scheduler.scheduleJob(job, trigger);
}
```

2. **Cross-Service Communication**:
```java
@FeignClient(name = "target-service")
public interface TargetServiceClient {
    @PostMapping("/api/process")
    void triggerProcessing(@RequestBody JobPayload payload);
}

// In job execution:
targetServiceClient.triggerProcessing(jobData);
```

3. **Event-Driven Architecture**:
```java
@EventListener
public void handleJobCompletion(JobCompletedEvent event) {
    eventPublisher.publishEvent(
        new NotificationEvent("Job completed: " + event.getJobId()));
}
```

## Deployment Considerations

1. **High Availability**:
   - Run multiple instances with leader election
   - Use distributed locks for job coordination

2. **Database**:
   - Cluster configuration for job store
   - Regular backups of job definitions

3. **Containerization**:
   ```dockerfile
   FROM openjdk:17-jdk-slim
   COPY target/scheduler-service.jar /app.jar
   ENTRYPOINT ["java","-jar","/app.jar"]
   ```

   -------


   Creating a **scheduler as a microservice** involves building a standalone microservice responsible for **running scheduled tasks** (e.g., sending emails, cleaning up old data, retrying failed jobs). This approach is **scalable, isolated, and fault-tolerant**.

---

## 🛠️ Steps to Create a Scheduler Microservice (Spring Boot Example)

---

### ✅ 1. **Create a Spring Boot Project**

Include these dependencies in `pom.xml`:

```xml
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter</artifactId>
</dependency>
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```

Optional:

* `spring-cloud-starter-netflix-eureka-client` (if using service discovery)
* `spring-boot-starter-quartz` (if using Quartz Scheduler)

---

### ✅ 2. **Enable Scheduling**

Add `@EnableScheduling` to your main class:

```java
@SpringBootApplication
@EnableScheduling
public class SchedulerServiceApplication {
    public static void main(String[] args) {
        SpringApplication.run(SchedulerServiceApplication.class, args);
    }
}
```

---

### ✅ 3. **Define a Scheduled Task**

```java
@Component
public class EmailRetryScheduler {

    @Scheduled(fixedRate = 60000) // Every 60 seconds
    public void retryFailedEmails() {
        System.out.println("Running job: Retry failed emails...");
        // logic to fetch and retry failed emails from DB
    }
}
```

Other options:

* `@Scheduled(cron = "0 0 * * * *")` — cron expression
* `@Scheduled(fixedDelay = 10000)` — delay after last completion

---

### ✅ 4. **Externalize Schedule Timing with Config**

In `application.yml`:

```yaml
scheduler:
  retry-delay: 60000
```

In code:

```java
@Scheduled(fixedRateString = "${scheduler.retry-delay}")
public void runTask() {
    // logic
}
```

---

## 🧱 Architecture: How It Works in Microservices

```
+--------------------------+
|  Scheduler Microservice  |
|--------------------------|
| - Runs background jobs   |
| - Talks to DB / APIs     |
| - Scales independently   |
+-----------+--------------+
            |
            v
  Other Services / Resources
```

---

## 💡 Use Cases for Scheduler Microservice

| Use Case                       | Description                          |
| ------------------------------ | ------------------------------------ |
| Retry failed messages/emails   | Job pulls failed records and retries |
| Clean old/inactive sessions    | Delete expired sessions periodically |
| Generate daily/weekly reports  | Automate reporting jobs              |
| Sync data between services     | Poll and sync remote APIs or DBs     |
| Billing or Subscription checks | Periodically evaluate billing status |

---

## 🚦 Scheduling Options

| Approach     | Tool/Tech                     | Notes                                  |
| ------------ | ----------------------------- | -------------------------------------- |
| Simple       | `@Scheduled` (Spring)         | Good for basic jobs, in-memory         |
| Distributed  | **Quartz Scheduler**          | Supports clustering, cron, persistence |
| Scalable     | **Kubernetes CronJob**        | Externalize scheduling using infra     |
| Asynchronous | **Event-driven (e.g. Kafka)** | Produce scheduled events to a queue    |

---

## ☁️ Deploying on Cloud

If using **Kubernetes**:

```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: email-retry-job
spec:
  schedule: "*/5 * * * *"
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: scheduler
            image: my-scheduler-service
          restartPolicy: OnFailure
```

---

## ✅ Best Practices

* Externalize all timings via config.
* Make scheduled jobs **idempotent** (safe to run multiple times).
* Track job runs in DB/logs for auditing.
* Avoid long-running jobs in `@Scheduled` (consider async or Quartz).
* Use **distributed locks** (e.g., Redis, DB) if running multiple instances to avoid duplicate execution.

---

Would you like a **complete Spring Boot project** using `@Scheduled` with Quartz or Redis-based locking (for distributed scheduling)?


This architecture provides a resilient, scalable scheduler microservice that can integrate with other services while maintaining fault tolerance and operational visibility.
