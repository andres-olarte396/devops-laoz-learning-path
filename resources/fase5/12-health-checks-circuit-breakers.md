# Módulo 12: Health Checks y Circuit Breakers

## 📋 Introducción

Los health checks y circuit breakers son patrones fundamentales para construir sistemas distribuidos resilientes. Los health checks proporcionan visibilidad del estado de servicios, mientras que los circuit breakers previenen cascadas de fallos protegiendo servicios de dependencias no disponibles.

### ¿Qué son Health Checks?

Los health checks son:

- **Readiness Probes**: ¿El servicio está listo para recibir tráfico?
- **Liveness Probes**: ¿El servicio está funcionando correctamente?
- **Startup Probes**: ¿El servicio ha iniciado completamente?
- **Deep Health Checks**: Estado de dependencias críticas
- **Graceful Degradation**: Funcionamiento parcial ante fallos

### ¿Qué son Circuit Breakers?

Los circuit breakers:

- **Previenen** cascadas de fallos
- **Detectan** servicios no disponibles
- **Permiten** recuperación automática
- **Implementan** fallback strategies
- **Protegen** recursos del sistema

### Estados del Circuit Breaker

- **CLOSED**: Funcionamiento normal
- **OPEN**: Fallo detectado, rechaza requests
- **HALF_OPEN**: Prueba si el servicio se recuperó

---

## 🎯 Objetivos de Aprendizaje

Al completar este módulo, serás capaz de:

1. **Implementar** health checks comprehensivos
2. **Configurar** circuit breakers con Resilience4j
3. **Diseñar** estrategias de fallback
4. **Integrar** health checks con Kubernetes
5. **Monitorear** estado de circuit breakers
6. **Implementar** bulk head patterns
7. **Configurar** rate limiting y retry policies
8. **Establecer** alertas basadas en health status

---

## 🏥 Health Checks Implementation

### 1. Spring Boot Health Checks

```java
// src/main/java/com/example/health/HealthCheckApplication.java
package com.example.health;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.boot.actuator.health.Health;
import org.springframework.boot.actuator.health.HealthIndicator;
import org.springframework.boot.actuator.health.Status;
import org.springframework.stereotype.Component;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.*;
import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.data.redis.core.RedisTemplate;
import org.springframework.http.ResponseEntity;

import javax.sql.DataSource;
import java.sql.Connection;
import java.sql.SQLException;
import java.time.LocalDateTime;
import java.util.Map;
import java.util.HashMap;
import java.util.concurrent.CompletableFuture;
import java.util.concurrent.TimeUnit;

@SpringBootApplication
public class HealthCheckApplication {
    public static void main(String[] args) {
        SpringApplication.run(HealthCheckApplication.class, args);
    }
}

// Custom Health Indicators
@Component
public class DatabaseHealthIndicator implements HealthIndicator {
    
    @Autowired
    private DataSource dataSource;
    
    @Override
    public Health health() {
        try (Connection connection = dataSource.getConnection()) {
            if (connection.isValid(5)) {
                return Health.up()
                    .withDetail("database", "PostgreSQL")
                    .withDetail("status", "Connected")
                    .withDetail("validationQuery", "SELECT 1")
                    .withDetail("responseTime", measureResponseTime())
                    .build();
            } else {
                return Health.down()
                    .withDetail("database", "PostgreSQL")
                    .withDetail("status", "Connection invalid")
                    .withDetail("error", "Connection validation failed")
                    .build();
            }
        } catch (SQLException e) {
            return Health.down()
                .withDetail("database", "PostgreSQL")
                .withDetail("status", "Connection failed")
                .withDetail("error", e.getMessage())
                .withException(e)
                .build();
        }
    }
    
    private long measureResponseTime() {
        long startTime = System.currentTimeMillis();
        try (Connection connection = dataSource.getConnection()) {
            connection.createStatement().execute("SELECT 1");
        } catch (SQLException e) {
            // Ignore for timing purposes
        }
        return System.currentTimeMillis() - startTime;
    }
}

@Component
public class RedisHealthIndicator implements HealthIndicator {
    
    @Autowired
    private RedisTemplate<String, Object> redisTemplate;
    
    @Override
    public Health health() {
        try {
            long startTime = System.currentTimeMillis();
            String testKey = "health:check:" + System.currentTimeMillis();
            String testValue = "test";
            
            // Test write
            redisTemplate.opsForValue().set(testKey, testValue, 10, TimeUnit.SECONDS);
            
            // Test read
            String retrievedValue = (String) redisTemplate.opsForValue().get(testKey);
            
            // Cleanup
            redisTemplate.delete(testKey);
            
            long responseTime = System.currentTimeMillis() - startTime;
            
            if (testValue.equals(retrievedValue)) {
                return Health.up()
                    .withDetail("redis", "Available")
                    .withDetail("status", "Connected")
                    .withDetail("operation", "read/write test passed")
                    .withDetail("responseTime", responseTime + "ms")
                    .build();
            } else {
                return Health.down()
                    .withDetail("redis", "Available but inconsistent")
                    .withDetail("status", "Data integrity issue")
                    .withDetail("expected", testValue)
                    .withDetail("actual", retrievedValue)
                    .build();
            }
        } catch (Exception e) {
            return Health.down()
                .withDetail("redis", "Unavailable")
                .withDetail("status", "Connection failed")
                .withDetail("error", e.getMessage())
                .withException(e)
                .build();
        }
    }
}

@Component
public class ExternalServiceHealthIndicator implements HealthIndicator {
    
    @Autowired
    private RestTemplate restTemplate;
    
    private static final String EXTERNAL_SERVICE_URL = "http://external-api:8080/health";
    
    @Override
    public Health health() {
        try {
            long startTime = System.currentTimeMillis();
            
            ResponseEntity<Map> response = restTemplate.getForEntity(
                EXTERNAL_SERVICE_URL, Map.class);
            
            long responseTime = System.currentTimeMillis() - startTime;
            
            if (response.getStatusCode().is2xxSuccessful()) {
                Map<String, Object> body = response.getBody();
                String status = (String) body.get("status");
                
                if ("UP".equals(status)) {
                    return Health.up()
                        .withDetail("externalService", "Available")
                        .withDetail("url", EXTERNAL_SERVICE_URL)
                        .withDetail("status", status)
                        .withDetail("responseTime", responseTime + "ms")
                        .withDetails(body)
                        .build();
                } else {
                    return Health.down()
                        .withDetail("externalService", "Degraded")
                        .withDetail("url", EXTERNAL_SERVICE_URL)
                        .withDetail("status", status)
                        .withDetail("responseTime", responseTime + "ms")
                        .build();
                }
            } else {
                return Health.down()
                    .withDetail("externalService", "Unavailable")
                    .withDetail("url", EXTERNAL_SERVICE_URL)
                    .withDetail("httpStatus", response.getStatusCode().value())
                    .withDetail("responseTime", responseTime + "ms")
                    .build();
            }
        } catch (Exception e) {
            return Health.down()
                .withDetail("externalService", "Unavailable")
                .withDetail("url", EXTERNAL_SERVICE_URL)
                .withDetail("error", e.getMessage())
                .withException(e)
                .build();
        }
    }
}

@Component
public class CustomBusinessHealthIndicator implements HealthIndicator {
    
    @Autowired
    private ProductRepository productRepository;
    
    @Autowired
    private OrderRepository orderRepository;
    
    @Override
    public Health health() {
        try {
            Map<String, Object> details = new HashMap<>();
            boolean isHealthy = true;
            
            // Check critical business metrics
            long productCount = productRepository.count();
            long recentOrders = orderRepository.countOrdersInLastHour();
            
            details.put("totalProducts", productCount);
            details.put("ordersLastHour", recentOrders);
            
            // Business rule: At least 10 products should exist
            if (productCount < 10) {
                isHealthy = false;
                details.put("productCountIssue", "Less than 10 products available");
            }
            
            // Check database performance
            long dbResponseTime = measureDatabasePerformance();
            details.put("dbResponseTime", dbResponseTime + "ms");
            
            if (dbResponseTime > 1000) {
                isHealthy = false;
                details.put("performanceIssue", "Database response time too high");
            }
            
            // Check system resources
            Runtime runtime = Runtime.getRuntime();
            long maxMemory = runtime.maxMemory();
            long totalMemory = runtime.totalMemory();
            long freeMemory = runtime.freeMemory();
            long usedMemory = totalMemory - freeMemory;
            
            double memoryUsagePercentage = (double) usedMemory / maxMemory * 100;
            
            details.put("memoryUsage", String.format("%.2f%%", memoryUsagePercentage));
            details.put("maxMemory", maxMemory / 1024 / 1024 + "MB");
            details.put("usedMemory", usedMemory / 1024 / 1024 + "MB");
            
            if (memoryUsagePercentage > 90) {
                isHealthy = false;
                details.put("memoryIssue", "Memory usage critical");
            }
            
            if (isHealthy) {
                return Health.up()
                    .withDetail("businessHealth", "All systems operational")
                    .withDetails(details)
                    .build();
            } else {
                return Health.down()
                    .withDetail("businessHealth", "Issues detected")
                    .withDetails(details)
                    .build();
            }
            
        } catch (Exception e) {
            return Health.down()
                .withDetail("businessHealth", "Health check failed")
                .withDetail("error", e.getMessage())
                .withException(e)
                .build();
        }
    }
    
    private long measureDatabasePerformance() {
        long startTime = System.currentTimeMillis();
        try {
            productRepository.count();
        } catch (Exception e) {
            // Return high value to indicate poor performance
            return 5000;
        }
        return System.currentTimeMillis() - startTime;
    }
}

// Advanced Health Check Controller
@RestController
@RequestMapping("/health")
public class AdvancedHealthController {
    
    @Autowired
    private DatabaseHealthIndicator databaseHealth;
    
    @Autowired
    private RedisHealthIndicator redisHealth;
    
    @Autowired
    private ExternalServiceHealthIndicator externalServiceHealth;
    
    @Autowired
    private CustomBusinessHealthIndicator businessHealth;
    
    @GetMapping("/detailed")
    public ResponseEntity<Map<String, Object>> detailedHealth() {
        Map<String, Object> healthStatus = new HashMap<>();
        
        Health dbHealth = databaseHealth.health();
        Health cacheHealth = redisHealth.health();
        Health extHealth = externalServiceHealth.health();
        Health bizHealth = businessHealth.health();
        
        healthStatus.put("database", createHealthDetail(dbHealth));
        healthStatus.put("cache", createHealthDetail(cacheHealth));
        healthStatus.put("externalServices", createHealthDetail(extHealth));
        healthStatus.put("business", createHealthDetail(bizHealth));
        
        // Overall status
        Status overallStatus = determineOverallStatus(dbHealth, cacheHealth, extHealth, bizHealth);
        healthStatus.put("status", overallStatus.getCode());
        healthStatus.put("timestamp", LocalDateTime.now());
        
        return ResponseEntity.ok(healthStatus);
    }
    
    @GetMapping("/readiness")
    public ResponseEntity<Map<String, Object>> readinessProbe() {
        Map<String, Object> readiness = new HashMap<>();
        
        // Check if service is ready to receive traffic
        Health dbHealth = databaseHealth.health();
        Health cacheHealth = redisHealth.health();
        
        boolean isReady = dbHealth.getStatus() == Status.UP && 
                         cacheHealth.getStatus() == Status.UP;
        
        readiness.put("ready", isReady);
        readiness.put("database", dbHealth.getStatus().getCode());
        readiness.put("cache", cacheHealth.getStatus().getCode());
        readiness.put("timestamp", LocalDateTime.now());
        
        return ResponseEntity.status(isReady ? 200 : 503).body(readiness);
    }
    
    @GetMapping("/liveness")
    public ResponseEntity<Map<String, Object>> livenessProbe() {
        Map<String, Object> liveness = new HashMap<>();
        
        // Basic liveness check - service is running
        boolean isAlive = true;
        
        try {
            // Check if the application can perform basic operations
            Runtime.getRuntime().totalMemory();
            System.currentTimeMillis();
            
            liveness.put("alive", isAlive);
            liveness.put("uptime", getUptime());
            liveness.put("timestamp", LocalDateTime.now());
            
            return ResponseEntity.ok(liveness);
        } catch (Exception e) {
            liveness.put("alive", false);
            liveness.put("error", e.getMessage());
            liveness.put("timestamp", LocalDateTime.now());
            
            return ResponseEntity.status(500).body(liveness);
        }
    }
    
    private Map<String, Object> createHealthDetail(Health health) {
        Map<String, Object> detail = new HashMap<>();
        detail.put("status", health.getStatus().getCode());
        detail.putAll(health.getDetails());
        return detail;
    }
    
    private Status determineOverallStatus(Health... healths) {
        for (Health health : healths) {
            if (health.getStatus() == Status.DOWN) {
                return Status.DOWN;
            }
        }
        
        for (Health health : healths) {
            if (health.getStatus() != Status.UP) {
                return Status.UNKNOWN;
            }
        }
        
        return Status.UP;
    }
    
    private long getUptime() {
        return System.currentTimeMillis() - 
               java.lang.management.ManagementFactory.getRuntimeMXBean().getStartTime();
    }
}
```

### 2. Circuit Breaker Implementation

```java
// src/main/java/com/example/circuitbreaker/CircuitBreakerConfiguration.java
package com.example.circuitbreaker;

import io.github.resilience4j.circuitbreaker.CircuitBreaker;
import io.github.resilience4j.circuitbreaker.CircuitBreakerConfig;
import io.github.resilience4j.circuitbreaker.CircuitBreakerRegistry;
import io.github.resilience4j.retry.Retry;
import io.github.resilience4j.retry.RetryConfig;
import io.github.resilience4j.retry.RetryRegistry;
import io.github.resilience4j.bulkhead.Bulkhead;
import io.github.resilience4j.bulkhead.BulkheadConfig;
import io.github.resilience4j.bulkhead.BulkheadRegistry;
import io.github.resilience4j.ratelimiter.RateLimiter;
import io.github.resilience4j.ratelimiter.RateLimiterConfig;
import io.github.resilience4j.ratelimiter.RateLimiterRegistry;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.web.client.RestTemplate;

import java.time.Duration;
import java.util.function.Predicate;

@Configuration
public class CircuitBreakerConfiguration {
    
    @Bean
    public CircuitBreakerRegistry circuitBreakerRegistry() {
        return CircuitBreakerRegistry.ofDefaults();
    }
    
    @Bean
    public CircuitBreaker externalServiceCircuitBreaker(CircuitBreakerRegistry registry) {
        CircuitBreakerConfig config = CircuitBreakerConfig.custom()
            .failureRateThreshold(50) // 50% failure rate
            .waitDurationInOpenState(Duration.ofSeconds(30)) // Wait 30s in open state
            .slidingWindowSize(10) // Consider last 10 calls
            .minimumNumberOfCalls(5) // Minimum 5 calls before calculating failure rate
            .permittedNumberOfCallsInHalfOpenState(3) // Allow 3 calls in half-open
            .recordExceptions(Exception.class) // Record all exceptions as failures
            .ignoreExceptions(IllegalArgumentException.class) // Ignore validation errors
            .recordResult(response -> {
                // Custom result recording logic
                if (response instanceof org.springframework.http.ResponseEntity) {
                    org.springframework.http.ResponseEntity<?> httpResponse = 
                        (org.springframework.http.ResponseEntity<?>) response;
                    return httpResponse.getStatusCode().is5xxServerError();
                }
                return false;
            })
            .build();
        
        return registry.circuitBreaker("externalService", config);
    }
    
    @Bean
    public CircuitBreaker databaseCircuitBreaker(CircuitBreakerRegistry registry) {
        CircuitBreakerConfig config = CircuitBreakerConfig.custom()
            .failureRateThreshold(60) // 60% failure rate for database
            .waitDurationInOpenState(Duration.ofSeconds(60)) // Wait 60s for DB recovery
            .slidingWindowSize(20) // Consider last 20 calls
            .minimumNumberOfCalls(10) // Minimum 10 calls
            .permittedNumberOfCallsInHalfOpenState(5) // Allow 5 test calls
            .recordExceptions(
                java.sql.SQLException.class,
                org.springframework.dao.DataAccessException.class,
                java.util.concurrent.TimeoutException.class
            )
            .build();
        
        return registry.circuitBreaker("database", config);
    }
    
    @Bean
    public RetryRegistry retryRegistry() {
        return RetryRegistry.ofDefaults();
    }
    
    @Bean
    public Retry externalServiceRetry(RetryRegistry registry) {
        RetryConfig config = RetryConfig.custom()
            .maxAttempts(3)
            .waitDuration(Duration.ofSeconds(2))
            .retryExceptions(
                java.net.ConnectException.class,
                java.net.SocketTimeoutException.class,
                org.springframework.web.client.ResourceAccessException.class
            )
            .ignoreExceptions(IllegalArgumentException.class)
            .retryOnResult(response -> {
                if (response instanceof org.springframework.http.ResponseEntity) {
                    org.springframework.http.ResponseEntity<?> httpResponse = 
                        (org.springframework.http.ResponseEntity<?>) response;
                    return httpResponse.getStatusCode().is5xxServerError();
                }
                return false;
            })
            .build();
        
        return registry.retry("externalService", config);
    }
    
    @Bean
    public BulkheadRegistry bulkheadRegistry() {
        return BulkheadRegistry.ofDefaults();
    }
    
    @Bean
    public Bulkhead externalServiceBulkhead(BulkheadRegistry registry) {
        BulkheadConfig config = BulkheadConfig.custom()
            .maxConcurrentCalls(10) // Maximum 10 concurrent calls
            .maxWaitDuration(Duration.ofSeconds(5)) // Wait max 5s for a permit
            .build();
        
        return registry.bulkhead("externalService", config);
    }
    
    @Bean
    public RateLimiterRegistry rateLimiterRegistry() {
        return RateLimiterRegistry.ofDefaults();
    }
    
    @Bean
    public RateLimiter externalServiceRateLimiter(RateLimiterRegistry registry) {
        RateLimiterConfig config = RateLimiterConfig.custom()
            .limitForPeriod(50) // 50 calls
            .limitRefreshPeriod(Duration.ofSeconds(60)) // per minute
            .timeoutDuration(Duration.ofSeconds(1)) // Wait max 1s for permit
            .build();
        
        return registry.rateLimiter("externalService", config);
    }
    
    @Bean
    public RestTemplate restTemplate() {
        return new RestTemplate();
    }
}

// Circuit Breaker Service Implementation
@Service
public class ExternalServiceClient {
    
    private final RestTemplate restTemplate;
    private final CircuitBreaker externalServiceCircuitBreaker;
    private final CircuitBreaker databaseCircuitBreaker;
    private final Retry externalServiceRetry;
    private final Bulkhead externalServiceBulkhead;
    private final RateLimiter externalServiceRateLimiter;
    
    public ExternalServiceClient(
            RestTemplate restTemplate,
            CircuitBreaker externalServiceCircuitBreaker,
            CircuitBreaker databaseCircuitBreaker,
            Retry externalServiceRetry,
            Bulkhead externalServiceBulkhead,
            RateLimiter externalServiceRateLimiter) {
        this.restTemplate = restTemplate;
        this.externalServiceCircuitBreaker = externalServiceCircuitBreaker;
        this.databaseCircuitBreaker = databaseCircuitBreaker;
        this.externalServiceRetry = externalServiceRetry;
        this.externalServiceBulkhead = externalServiceBulkhead;
        this.externalServiceRateLimiter = externalServiceRateLimiter;
    }
    
    public ResponseEntity<ExternalApiResponse> callExternalService(String requestData) {
        Supplier<ResponseEntity<ExternalApiResponse>> supplier = 
            CircuitBreaker.decorateSupplier(externalServiceCircuitBreaker, () -> {
                return Bulkhead.decorateSupplier(externalServiceBulkhead, () -> {
                    return RateLimiter.decorateSupplier(externalServiceRateLimiter, () -> {
                        return Retry.decorateSupplier(externalServiceRetry, () -> {
                            return performExternalCall(requestData);
                        }).get();
                    }).get();
                }).get();
            });
        
        try {
            return supplier.get();
        } catch (Exception e) {
            // Fallback mechanism
            return handleFallback(requestData, e);
        }
    }
    
    private ResponseEntity<ExternalApiResponse> performExternalCall(String requestData) {
        try {
            String url = "http://external-api:8080/api/process";
            
            HttpHeaders headers = new HttpHeaders();
            headers.setContentType(MediaType.APPLICATION_JSON);
            
            HttpEntity<String> request = new HttpEntity<>(requestData, headers);
            
            ResponseEntity<ExternalApiResponse> response = restTemplate.postForEntity(
                url, request, ExternalApiResponse.class);
            
            // Log successful call
            System.out.println("External service call successful: " + response.getStatusCode());
            
            return response;
            
        } catch (Exception e) {
            System.err.println("External service call failed: " + e.getMessage());
            throw e;
        }
    }
    
    private ResponseEntity<ExternalApiResponse> handleFallback(String requestData, Exception e) {
        System.out.println("Executing fallback for external service call. Error: " + e.getMessage());
        
        // Return cached response or default response
        ExternalApiResponse fallbackResponse = new ExternalApiResponse();
        fallbackResponse.setStatus("FALLBACK");
        fallbackResponse.setMessage("Service temporarily unavailable, using cached data");
        fallbackResponse.setData(getCachedData(requestData));
        fallbackResponse.setTimestamp(LocalDateTime.now());
        
        return ResponseEntity.ok(fallbackResponse);
    }
    
    private Object getCachedData(String requestData) {
        // Implement cache lookup logic here
        Map<String, Object> cachedData = new HashMap<>();
        cachedData.put("source", "cache");
        cachedData.put("request", requestData);
        cachedData.put("cached_at", LocalDateTime.now().minusMinutes(30));
        return cachedData;
    }
    
    // Database operations with circuit breaker
    public List<Product> getDatabaseData() {
        Supplier<List<Product>> supplier = 
            CircuitBreaker.decorateSupplier(databaseCircuitBreaker, () -> {
                return performDatabaseQuery();
            });
        
        try {
            return supplier.get();
        } catch (Exception e) {
            return handleDatabaseFallback(e);
        }
    }
    
    private List<Product> performDatabaseQuery() {
        // Simulate database call
        // In real implementation, this would call repository
        try {
            Thread.sleep(100); // Simulate DB latency
            
            // Simulate random failures for demonstration
            if (Math.random() < 0.3) { // 30% failure rate
                throw new RuntimeException("Database connection timeout");
            }
            
            return Arrays.asList(
                new Product(1L, "Product 1", "Description 1", BigDecimal.valueOf(10.00)),
                new Product(2L, "Product 2", "Description 2", BigDecimal.valueOf(20.00))
            );
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
            throw new RuntimeException("Database operation interrupted", e);
        }
    }
    
    private List<Product> handleDatabaseFallback(Exception e) {
        System.out.println("Database fallback executed. Error: " + e.getMessage());
        
        // Return empty list or cached data
        return Collections.emptyList();
    }
}

// Circuit Breaker Monitoring
@Component
public class CircuitBreakerMonitor {
    
    private final CircuitBreakerRegistry circuitBreakerRegistry;
    private final MeterRegistry meterRegistry;
    
    public CircuitBreakerMonitor(CircuitBreakerRegistry circuitBreakerRegistry, 
                                MeterRegistry meterRegistry) {
        this.circuitBreakerRegistry = circuitBreakerRegistry;
        this.meterRegistry = meterRegistry;
        
        // Register metrics for all circuit breakers
        circuitBreakerRegistry.getAllCircuitBreakers().forEach(this::registerMetrics);
    }
    
    private void registerMetrics(CircuitBreaker circuitBreaker) {
        String name = circuitBreaker.getName();
        
        // State gauge
        Gauge.builder("circuit_breaker_state")
            .tag("name", name)
            .register(meterRegistry, circuitBreaker, cb -> {
                switch (cb.getState()) {
                    case CLOSED: return 0;
                    case OPEN: return 1;
                    case HALF_OPEN: return 2;
                    default: return -1;
                }
            });
        
        // Success rate gauge
        Gauge.builder("circuit_breaker_success_rate")
            .tag("name", name)
            .register(meterRegistry, circuitBreaker, 
                cb -> cb.getMetrics().getSuccessRate());
        
        // Failure rate gauge
        Gauge.builder("circuit_breaker_failure_rate")
            .tag("name", name)
            .register(meterRegistry, circuitBreaker, 
                cb -> cb.getMetrics().getFailureRate());
        
        // Call counters
        Counter.builder("circuit_breaker_calls_total")
            .tag("name", name)
            .tag("kind", "successful")
            .register(meterRegistry);
            
        Counter.builder("circuit_breaker_calls_total")
            .tag("name", name)
            .tag("kind", "failed")
            .register(meterRegistry);
        
        // Event listeners
        circuitBreaker.getEventPublisher()
            .onSuccess(event -> 
                meterRegistry.counter("circuit_breaker_calls_total", 
                    "name", name, "kind", "successful").increment())
            .onError(event -> 
                meterRegistry.counter("circuit_breaker_calls_total", 
                    "name", name, "kind", "failed").increment())
            .onStateTransition(event -> 
                System.out.println(String.format(
                    "Circuit breaker %s transitioned from %s to %s", 
                    name, event.getStateTransition().getFromState(), 
                    event.getStateTransition().getToState())));
    }
    
    public Map<String, Object> getCircuitBreakerStatus() {
        Map<String, Object> status = new HashMap<>();
        
        circuitBreakerRegistry.getAllCircuitBreakers().forEach(cb -> {
            Map<String, Object> cbStatus = new HashMap<>();
            cbStatus.put("state", cb.getState().toString());
            cbStatus.put("successRate", cb.getMetrics().getSuccessRate());
            cbStatus.put("failureRate", cb.getMetrics().getFailureRate());
            cbStatus.put("numberOfCalls", cb.getMetrics().getNumberOfCalls());
            cbStatus.put("numberOfSuccessfulCalls", cb.getMetrics().getNumberOfSuccessfulCalls());
            cbStatus.put("numberOfFailedCalls", cb.getMetrics().getNumberOfFailedCalls());
            
            status.put(cb.getName(), cbStatus);
        });
        
        return status;
    }
}
```

---

## 🔄 Kubernetes Integration

### 1. Kubernetes Health Checks

```yaml
# k8s-health-checks.yml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: health-check-app
  namespace: default
  labels:
    app: health-check-app
    version: v1
spec:
  replicas: 3
  selector:
    matchLabels:
      app: health-check-app
  template:
    metadata:
      labels:
        app: health-check-app
        version: v1
      annotations:
        prometheus.io/scrape: "true"
        prometheus.io/port: "8080"
        prometheus.io/path: "/actuator/prometheus"
    spec:
      containers:
      - name: app
        image: health-check-app:latest
        ports:
        - containerPort: 8080
          name: http
        env:
        - name: SPRING_PROFILES_ACTIVE
          value: "kubernetes"
        - name: DATABASE_URL
          valueFrom:
            secretKeyRef:
              name: database-secret
              key: url
        - name: REDIS_URL
          valueFrom:
            configMapKeyRef:
              name: redis-config
              key: url
        
        # Startup Probe - Application startup
        startupProbe:
          httpGet:
            path: /health/liveness
            port: 8080
            scheme: HTTP
          initialDelaySeconds: 30
          periodSeconds: 10
          timeoutSeconds: 5
          failureThreshold: 30 # Allow 5 minutes for startup
          successThreshold: 1
        
        # Liveness Probe - Application is running
        livenessProbe:
          httpGet:
            path: /health/liveness
            port: 8080
            scheme: HTTP
          initialDelaySeconds: 60
          periodSeconds: 30
          timeoutSeconds: 10
          failureThreshold: 3
          successThreshold: 1
        
        # Readiness Probe - Ready to serve traffic
        readinessProbe:
          httpGet:
            path: /health/readiness
            port: 8080
            scheme: HTTP
          initialDelaySeconds: 10
          periodSeconds: 10
          timeoutSeconds: 5
          failureThreshold: 3
          successThreshold: 1
        
        resources:
          requests:
            memory: "512Mi"
            cpu: "250m"
          limits:
            memory: "1Gi"
            cpu: "500m"
        
        # Graceful shutdown
        lifecycle:
          preStop:
            exec:
              command: ["/bin/sh", "-c", "sleep 15"]
        
        # Security context
        securityContext:
          runAsNonRoot: true
          runAsUser: 1001
          allowPrivilegeEscalation: false
          readOnlyRootFilesystem: true
          capabilities:
            drop:
            - ALL
        
        # Volume mounts for temporary files
        volumeMounts:
        - name: tmp
          mountPath: /tmp
        - name: logs
          mountPath: /app/logs
      
      volumes:
      - name: tmp
        emptyDir: {}
      - name: logs
        emptyDir: {}
      
      # Service account
      serviceAccountName: health-check-app
      
      # Pod disruption budget support
      terminationGracePeriodSeconds: 60
      
      # Node selection
      nodeSelector:
        node-type: application
      
      # Anti-affinity for high availability
      affinity:
        podAntiAffinity:
          preferredDuringSchedulingIgnoredDuringExecution:
          - weight: 100
            podAffinityTerm:
              labelSelector:
                matchExpressions:
                - key: app
                  operator: In
                  values:
                  - health-check-app
              topologyKey: kubernetes.io/hostname

---
apiVersion: v1
kind: Service
metadata:
  name: health-check-service
  namespace: default
  labels:
    app: health-check-app
spec:
  type: ClusterIP
  ports:
  - port: 80
    targetPort: 8080
    protocol: TCP
    name: http
  selector:
    app: health-check-app

---
# Horizontal Pod Autoscaler
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: health-check-app-hpa
  namespace: default
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: health-check-app
  minReplicas: 3
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70
  - type: Resource
    resource:
      name: memory
      target:
        type: Utilization
        averageUtilization: 80
  behavior:
    scaleDown:
      stabilizationWindowSeconds: 300
      policies:
      - type: Percent
        value: 10
        periodSeconds: 60
    scaleUp:
      stabilizationWindowSeconds: 60
      policies:
      - type: Percent
        value: 50
        periodSeconds: 60

---
# Pod Disruption Budget
apiVersion: policy/v1
kind: PodDisruptionBudget
metadata:
  name: health-check-app-pdb
  namespace: default
spec:
  minAvailable: 2
  selector:
    matchLabels:
      app: health-check-app

---
# Network Policy
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: health-check-app-netpol
  namespace: default
spec:
  podSelector:
    matchLabels:
      app: health-check-app
  policyTypes:
  - Ingress
  - Egress
  ingress:
  - from:
    - namespaceSelector:
        matchLabels:
          name: ingress-nginx
    - podSelector:
        matchLabels:
          app: nginx-ingress
    ports:
    - protocol: TCP
      port: 8080
  egress:
  - to:
    - namespaceSelector:
        matchLabels:
          name: database
    ports:
    - protocol: TCP
      port: 5432
  - to:
    - namespaceSelector:
        matchLabels:
          name: cache
    ports:
    - protocol: TCP
      port: 6379
  - to: []
    ports:
    - protocol: TCP
      port: 53
    - protocol: UDP
      port: 53
```

### 2. Health Check Operator

```yaml
# health-check-operator.yml
apiVersion: v1
kind: ConfigMap
metadata:
  name: health-check-config
  namespace: monitoring
data:
  health-checks.yml: |
    healthChecks:
      - name: "database-health"
        type: "http"
        url: "http://health-check-service/health/detailed"
        interval: "30s"
        timeout: "10s"
        expectedStatus: 200
        jsonPath: "$.database.status"
        expectedValue: "UP"
        labels:
          service: "database"
          criticality: "high"
      
      - name: "external-service-health"
        type: "http"
        url: "http://health-check-service/health/detailed"
        interval: "1m"
        timeout: "15s"
        expectedStatus: 200
        jsonPath: "$.externalServices.status"
        expectedValue: "UP"
        labels:
          service: "external-api"
          criticality: "medium"
      
      - name: "business-health"
        type: "http"
        url: "http://health-check-service/health/detailed"
        interval: "2m"
        timeout: "20s"
        expectedStatus: 200
        jsonPath: "$.business.status"
        expectedValue: "UP"
        labels:
          service: "business-logic"
          criticality: "high"

---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: health-check-operator
  namespace: monitoring
spec:
  replicas: 1
  selector:
    matchLabels:
      app: health-check-operator
  template:
    metadata:
      labels:
        app: health-check-operator
    spec:
      serviceAccountName: health-check-operator
      containers:
      - name: operator
        image: health-check-operator:latest
        env:
        - name: CONFIG_FILE
          value: "/config/health-checks.yml"
        - name: PROMETHEUS_URL
          value: "http://prometheus:9090"
        - name: ALERTMANAGER_URL
          value: "http://alertmanager:9093"
        volumeMounts:
        - name: config
          mountPath: /config
        resources:
          requests:
            memory: "128Mi"
            cpu: "100m"
          limits:
            memory: "256Mi"
            cpu: "200m"
      volumes:
      - name: config
        configMap:
          name: health-check-config

---
apiVersion: v1
kind: ServiceAccount
metadata:
  name: health-check-operator
  namespace: monitoring

---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: health-check-operator
rules:
- apiGroups: [""]
  resources: ["pods", "services", "endpoints"]
  verbs: ["get", "list", "watch"]
- apiGroups: ["apps"]
  resources: ["deployments", "replicasets"]
  verbs: ["get", "list", "watch"]
- apiGroups: ["monitoring.coreos.com"]
  resources: ["servicemonitors", "prometheusrules"]
  verbs: ["get", "list", "watch", "create", "update", "patch"]

---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: health-check-operator
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: health-check-operator
subjects:
- kind: ServiceAccount
  name: health-check-operator
  namespace: monitoring
```

---

## 📊 Monitoring Circuit Breakers

### 1. Prometheus Metrics

```yaml
# prometheus-circuit-breaker-rules.yml
groups:
- name: circuit-breaker-rules
  rules:
  - alert: CircuitBreakerOpen
    expr: circuit_breaker_state == 1
    for: 1m
    labels:
      severity: warning
      category: availability
    annotations:
      summary: "Circuit breaker {{ $labels.name }} is OPEN"
      description: "Circuit breaker {{ $labels.name }} has been in OPEN state for more than 1 minute"
      runbook_url: "https://runbooks.example.com/circuit-breaker-open"
  
  - alert: CircuitBreakerHighFailureRate
    expr: circuit_breaker_failure_rate > 30
    for: 2m
    labels:
      severity: warning
      category: reliability
    annotations:
      summary: "High failure rate for circuit breaker {{ $labels.name }}"
      description: "Circuit breaker {{ $labels.name }} has failure rate of {{ $value }}% for more than 2 minutes"
  
  - alert: CircuitBreakerCriticalFailureRate
    expr: circuit_breaker_failure_rate > 50
    for: 1m
    labels:
      severity: critical
      category: reliability
    annotations:
      summary: "Critical failure rate for circuit breaker {{ $labels.name }}"
      description: "Circuit breaker {{ $labels.name }} has critical failure rate of {{ $value }}%"
  
  - alert: HealthCheckFailed
    expr: up{job="health-check-app"} == 0
    for: 30s
    labels:
      severity: critical
      category: availability
    annotations:
      summary: "Health check failed for {{ $labels.instance }}"
      description: "Health check endpoint is not responding for {{ $labels.instance }}"
  
  - alert: ReadinessCheckFailed
    expr: probe_success{job="kubernetes-services"} == 0
    for: 1m
    labels:
      severity: warning
      category: availability
    annotations:
      summary: "Readiness check failed for {{ $labels.instance }}"
      description: "Service {{ $labels.instance }} is not ready to serve traffic"

- name: application-health-rules
  rules:
  - alert: DatabaseHealthDegraded
    expr: |
      (
        http_request_duration_seconds{endpoint="/health/detailed",status_code="200"}
        and on(instance) 
        increase(http_requests_total{endpoint="/health/detailed",status_code="503"}[5m]) > 0
      )
    for: 2m
    labels:
      severity: warning
      category: database
    annotations:
      summary: "Database health is degraded"
      description: "Database health checks are returning degraded status"
  
  - alert: HighMemoryUsage
    expr: |
      (
        http_request_duration_seconds{endpoint="/health/detailed",status_code="200"}
        and on(instance)
        label_replace(
          probe_http_json{job="health-check-business"}, 
          "instance", "$1", "target", "([^:]+):.*"
        ) == 1
        and on(instance) 
        probe_http_json_response_memory_usage > 90
      )
    for: 5m
    labels:
      severity: critical
      category: resources
    annotations:
      summary: "High memory usage detected"
      description: "Application memory usage is above 90% for more than 5 minutes"
```

### 2. Grafana Dashboard

```json
{
  "dashboard": {
    "id": null,
    "title": "Circuit Breakers & Health Checks",
    "tags": ["circuit-breaker", "health", "resilience"],
    "timezone": "browser",
    "panels": [
      {
        "id": 1,
        "title": "Circuit Breaker States",
        "type": "stat",
        "targets": [
          {
            "expr": "circuit_breaker_state",
            "legendFormat": "{{name}}",
            "refId": "A"
          }
        ],
        "fieldConfig": {
          "defaults": {
            "mappings": [
              {"options": {"0": {"text": "CLOSED", "color": "green"}}},
              {"options": {"1": {"text": "OPEN", "color": "red"}}},
              {"options": {"2": {"text": "HALF_OPEN", "color": "yellow"}}}
            ]
          }
        },
        "gridPos": {"h": 8, "w": 12, "x": 0, "y": 0}
      },
      {
        "id": 2,
        "title": "Circuit Breaker Success Rate",
        "type": "graph",
        "targets": [
          {
            "expr": "circuit_breaker_success_rate",
            "legendFormat": "{{name}} success rate",
            "refId": "A"
          }
        ],
        "yAxes": [
          {
            "max": 100,
            "min": 0,
            "unit": "percent"
          }
        ],
        "gridPos": {"h": 8, "w": 12, "x": 12, "y": 0}
      },
      {
        "id": 3,
        "title": "Health Check Status",
        "type": "table",
        "targets": [
          {
            "expr": "probe_success",
            "format": "table",
            "refId": "A"
          }
        ],
        "transformations": [
          {
            "id": "organize",
            "options": {
              "columns": [
                {"text": "Service", "value": "instance"},
                {"text": "Status", "value": "Value"},
                {"text": "Job", "value": "job"}
              ]
            }
          }
        ],
        "fieldConfig": {
          "overrides": [
            {
              "matcher": {"id": "byName", "options": "Status"},
              "properties": [
                {
                  "id": "custom.displayMode",
                  "value": "color-background"
                },
                {
                  "id": "mappings",
                  "value": [
                    {"options": {"0": {"text": "DOWN", "color": "red"}}},
                    {"options": {"1": {"text": "UP", "color": "green"}}}
                  ]
                }
              ]
            }
          ]
        },
        "gridPos": {"h": 8, "w": 24, "x": 0, "y": 8}
      },
      {
        "id": 4,
        "title": "Request Volume & Error Rate",
        "type": "graph",
        "targets": [
          {
            "expr": "rate(http_requests_total[5m])",
            "legendFormat": "{{method}} {{endpoint}} - RPS",
            "refId": "A"
          },
          {
            "expr": "rate(http_requests_total{status_code=~'5..'}[5m]) / rate(http_requests_total[5m]) * 100",
            "legendFormat": "Error Rate %",
            "refId": "B"
          }
        ],
        "yAxes": [
          {"label": "Requests/sec", "side": "left"},
          {"label": "Error Rate %", "side": "right", "max": 100}
        ],
        "gridPos": {"h": 8, "w": 24, "x": 0, "y": 16}
      },
      {
        "id": 5,
        "title": "Response Time Percentiles",
        "type": "graph",
        "targets": [
          {
            "expr": "histogram_quantile(0.50, rate(http_request_duration_seconds_bucket[5m]))",
            "legendFormat": "50th percentile",
            "refId": "A"
          },
          {
            "expr": "histogram_quantile(0.95, rate(http_request_duration_seconds_bucket[5m]))",
            "legendFormat": "95th percentile",
            "refId": "B"
          },
          {
            "expr": "histogram_quantile(0.99, rate(http_request_duration_seconds_bucket[5m]))",
            "legendFormat": "99th percentile",
            "refId": "C"
          }
        ],
        "yAxes": [
          {"label": "Response Time", "unit": "s"}
        ],
        "gridPos": {"h": 8, "w": 24, "x": 0, "y": 24}
      }
    ],
    "time": {
      "from": "now-1h",
      "to": "now"
    },
    "refresh": "30s"
  }
}
```

---

## 🧪 Ejercicios Prácticos

### Ejercicio 1: Implementar Health Checks

**Objetivo**: Crear health checks comprehensivos para microservicio.

**Tareas**:

1. Implementar custom health indicators
2. Configurar readiness y liveness probes
3. Crear deep health checks
4. Testear escenarios de fallo

### Ejercicio 2: Circuit Breaker Configuration

**Objetivo**: Configurar circuit breakers para servicios externos.

**Componentes**:

- Resilience4j circuit breakers
- Retry policies
- Fallback mechanisms
- Métricas y monitoring

### Ejercicio 3: Kubernetes Integration

**Objetivo**: Integrar health checks con Kubernetes.

**Tareas**:

- Configurar probes en deployments
- Implementar graceful shutdown
- Configurar HPA basado en health
- Crear alertas de disponibilidad

---

## 📋 Laboratorio Final

### Lab 1: Complete Resilience Setup

1. **Desplegar aplicación** con health checks
2. **Configurar circuit breakers** para todas las dependencias
3. **Implementar monitoring** completo
4. **Testear scenarios** de fallo y recuperación

### Lab 2: Chaos Engineering

1. **Simular fallos** de dependencias
2. **Validar comportamiento** de circuit breakers
3. **Verificar fallbacks** funcionan correctamente
4. **Analizar métricas** de resilience

---

## ✅ Best Practices

### 1. Health Check Design

```yaml
health_check_principles:
  design:
    - "Separar liveness, readiness y startup checks"
    - "Incluir dependencias críticas en health checks"
    - "Implementar timeouts apropiados"
    - "Evitar health checks costosos"
  
  implementation:
    - "Usar circuit breakers para dependencias externas"
    - "Implementar graceful degradation"
    - "Cachear resultados cuando sea apropiado"
    - "Proveer detalles útiles para debugging"
```

### 2. Circuit Breaker Configuration

```yaml
circuit_breaker_guidelines:
  configuration:
    - "Ajustar thresholds basado en SLIs del servicio"
    - "Configurar timeouts apropiados"
    - "Implementar jitter en retry policies"
    - "Usar bulkheads para aislar recursos"
  
  monitoring:
    - "Alertar en transiciones de estado"
    - "Monitorear failure rates"
    - "Trackear efectividad de fallbacks"
    - "Medir impacto en user experience"
```

---

## 📖 Resumen del Módulo

En este módulo has aprendido:

1. **Health Checks** - Liveness, readiness y startup probes
2. **Circuit Breakers** - Resilience4j configuration y patterns
3. **Kubernetes Integration** - Probes y health-based scaling
4. **Monitoring** - Métricas y alertas de resilience
5. **Fallback Strategies** - Graceful degradation patterns
6. **Best Practices** - Principios de sistemas resilientes

¡Has completado exitosamente la implementación de health checks y circuit breakers! El siguiente módulo será el laboratorio final y evaluación para completar la Fase 5.
