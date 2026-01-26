# Módulo 10: Distributed Tracing con Jaeger

## 📋 Introducción

El distributed tracing es una técnica esencial para monitorear y diagnosticar aplicaciones distribuidas modernas. En arquitecturas de microservicios, una sola operación puede involucrar múltiples servicios, lo que hace difícil rastrear el flujo completo de una request. Jaeger es una herramienta de distributed tracing de código abierto que ayuda a visualizar y analizar estos flujos.

### ¿Qué es Distributed Tracing?

Distributed tracing permite:

- **Rastrear** requests a través de múltiples servicios
- **Visualizar** la topología de servicios y dependencias
- **Identificar** cuellos de botella y problemas de rendimiento
- **Monitorear** la latencia end-to-end
- **Detectar** errores y fallos en sistemas distribuidos

### Conceptos Clave

- **Trace**: Representación completa de una operación distribuida
- **Span**: Unidad de trabalho individual dentro de un trace
- **Context Propagation**: Transmisión de información de tracing entre servicios
- **Sampling**: Estrategia para controlar el volumen de traces capturados
- **Baggage**: Datos adicionales propagados con el trace

---

## 🎯 Objetivos de Aprendizaje

Al completar este módulo, serás capaz de:

1. **Implementar** Jaeger en arquitecturas de microservicios
2. **Configurar** instrumentación de aplicaciones
3. **Diseñar** estrategias de sampling efectivas
4. **Analizar** traces para detectar problemas
5. **Correlacionar** logs y métricas con traces
6. **Optimizar** el rendimiento basado en insights de tracing
7. **Configurar** alertas basadas en tracing data
8. **Implementar** custom instrumentation

---

## 🏗️ Arquitectura de Jaeger

### 1. Componentes de Jaeger

```yaml
# docker-compose.jaeger.yml
version: '3.8'

services:
  # Jaeger All-in-One (development)
  jaeger-all-in-one:
    image: jaegertracing/all-in-one:1.51
    container_name: tracing-jaeger
    ports:
      - "16686:16686"  # Jaeger UI
      - "14250:14250"  # gRPC
      - "14268:14268"  # HTTP
      - "14269:14269"  # Admin HTTP
      - "6831:6831/udp"  # Compact thrift protocol
      - "6832:6832/udp"  # Binary thrift protocol
      - "5778:5778"   # Config server HTTP
      - "5775:5775/udp"  # Zipkin compact thrift
    environment:
      - COLLECTOR_OTLP_ENABLED=true
      - COLLECTOR_ZIPKIN_HOST_PORT=:9411
    networks:
      - tracing_network

  # Elasticsearch para storage (production)
  elasticsearch:
    image: elasticsearch:8.11.0
    container_name: tracing-elasticsearch
    environment:
      - discovery.type=single-node
      - xpack.security.enabled=false
      - "ES_JAVA_OPTS=-Xms2g -Xmx2g"
    volumes:
      - es_data:/usr/share/elasticsearch/data
    ports:
      - "9200:9200"
    networks:
      - tracing_network

  # Jaeger Collector (production)
  jaeger-collector:
    image: jaegertracing/jaeger-collector:1.51
    container_name: tracing-collector
    ports:
      - "14269:14269"
      - "14268:14268"
      - "14250:14250"
      - "9411:9411"
    environment:
      - SPAN_STORAGE_TYPE=elasticsearch
      - ES_SERVER_URLS=http://elasticsearch:9200
      - ES_NUM_SHARDS=1
      - ES_NUM_REPLICAS=0
      - COLLECTOR_OTLP_ENABLED=true
    depends_on:
      - elasticsearch
    networks:
      - tracing_network
    profiles:
      - production

  # Jaeger Query (production)
  jaeger-query:
    image: jaegertracing/jaeger-query:1.51
    container_name: tracing-query
    ports:
      - "16686:16686"
      - "16687:16687"
    environment:
      - SPAN_STORAGE_TYPE=elasticsearch
      - ES_SERVER_URLS=http://elasticsearch:9200
      - QUERY_BASE_PATH=/jaeger
    depends_on:
      - elasticsearch
    networks:
      - tracing_network
    profiles:
      - production

  # Jaeger Agent (production)
  jaeger-agent:
    image: jaegertracing/jaeger-agent:1.51
    container_name: tracing-agent
    ports:
      - "6831:6831/udp"
      - "6832:6832/udp"
      - "5778:5778"
    environment:
      - REPORTER_GRPC_HOST_PORT=jaeger-collector:14250
    depends_on:
      - jaeger-collector
    networks:
      - tracing_network
    profiles:
      - production

  # OpenTelemetry Collector
  otel-collector:
    image: otel/opentelemetry-collector-contrib:0.89.0
    container_name: tracing-otel-collector
    command: ["--config=/etc/otel-collector-config.yaml"]
    volumes:
      - ./otel-collector-config.yaml:/etc/otel-collector-config.yaml
    ports:
      - "4317:4317"   # OTLP gRPC receiver
      - "4318:4318"   # OTLP HTTP receiver
      - "8888:8888"   # Prometheus metrics
      - "8889:8889"   # Prometheus exporter metrics
    depends_on:
      - jaeger-all-in-one
    networks:
      - tracing_network

  # Sample microservice: User Service
  user-service:
    build:
      context: ./services/user-service
      dockerfile: Dockerfile
    container_name: tracing-user-service
    ports:
      - "8081:8080"
    environment:
      - JAEGER_AGENT_HOST=jaeger-all-in-one
      - JAEGER_AGENT_PORT=6831
      - JAEGER_SAMPLER_TYPE=const
      - JAEGER_SAMPLER_PARAM=1
      - OTEL_EXPORTER_JAEGER_ENDPOINT=http://jaeger-all-in-one:14268/api/traces
    depends_on:
      - jaeger-all-in-one
      - postgres
    networks:
      - tracing_network

  # Sample microservice: Order Service
  order-service:
    build:
      context: ./services/order-service
      dockerfile: Dockerfile
    container_name: tracing-order-service
    ports:
      - "8082:8080"
    environment:
      - JAEGER_AGENT_HOST=jaeger-all-in-one
      - JAEGER_AGENT_PORT=6831
      - JAEGER_SAMPLER_TYPE=const
      - JAEGER_SAMPLER_PARAM=1
      - USER_SERVICE_URL=http://user-service:8080
    depends_on:
      - jaeger-all-in-one
      - user-service
      - postgres
    networks:
      - tracing_network

  # Sample microservice: Notification Service
  notification-service:
    build:
      context: ./services/notification-service
      dockerfile: Dockerfile
    container_name: tracing-notification-service
    ports:
      - "8083:8080"
    environment:
      - JAEGER_AGENT_HOST=jaeger-all-in-one
      - JAEGER_AGENT_PORT=6831
      - JAEGER_SAMPLER_TYPE=const
      - JAEGER_SAMPLER_PARAM=1
    depends_on:
      - jaeger-all-in-one
      - rabbitmq
    networks:
      - tracing_network

  # PostgreSQL Database
  postgres:
    image: postgres:15
    container_name: tracing-postgres
    environment:
      - POSTGRES_DB=tracing_demo
      - POSTGRES_USER=postgres
      - POSTGRES_PASSWORD=postgres123
    volumes:
      - postgres_data:/var/lib/postgresql/data
      - ./init-db.sql:/docker-entrypoint-initdb.d/init-db.sql
    ports:
      - "5432:5432"
    networks:
      - tracing_network

  # RabbitMQ for async messaging
  rabbitmq:
    image: rabbitmq:3.12-management
    container_name: tracing-rabbitmq
    ports:
      - "5672:5672"
      - "15672:15672"
    environment:
      - RABBITMQ_DEFAULT_USER=admin
      - RABBITMQ_DEFAULT_PASS=admin123
    volumes:
      - rabbitmq_data:/var/lib/rabbitmq
    networks:
      - tracing_network

  # Prometheus for metrics
  prometheus:
    image: prom/prometheus:latest
    container_name: tracing-prometheus
    ports:
      - "9090:9090"
    volumes:
      - ./prometheus.yml:/etc/prometheus/prometheus.yml
      - prometheus_data:/prometheus
    networks:
      - tracing_network

  # Grafana for visualization
  grafana:
    image: grafana/grafana:10.2.0
    container_name: tracing-grafana
    ports:
      - "3000:3000"
    environment:
      - GF_SECURITY_ADMIN_PASSWORD=admin123
    volumes:
      - grafana_data:/var/lib/grafana
      - ./grafana/provisioning:/etc/grafana/provisioning
    networks:
      - tracing_network

volumes:
  es_data:
  postgres_data:
  rabbitmq_data:
  prometheus_data:
  grafana_data:

networks:
  tracing_network:
    driver: bridge
```

### 2. OpenTelemetry Collector Configuration

```yaml
# otel-collector-config.yaml
receivers:
  otlp:
    protocols:
      grpc:
        endpoint: 0.0.0.0:4317
      http:
        endpoint: 0.0.0.0:4318
  
  jaeger:
    protocols:
      grpc:
        endpoint: 0.0.0.0:14250
      thrift_http:
        endpoint: 0.0.0.0:14268
      thrift_compact:
        endpoint: 0.0.0.0:6831
      thrift_binary:
        endpoint: 0.0.0.0:6832
  
  zipkin:
    endpoint: 0.0.0.0:9411

processors:
  batch:
    timeout: 1s
    send_batch_size: 1024
    send_batch_max_size: 2048
  
  memory_limiter:
    limit_mib: 512
    spike_limit_mib: 128
    check_interval: 5s
  
  resource:
    attributes:
      - key: environment
        value: production
        action: upsert
      - key: service.namespace
        value: microservices-demo
        action: upsert
  
  probabilistic_sampler:
    sampling_percentage: 10
  
  span:
    name:
      to_attributes:
        rules:
          - ^\/api\/v(?P<version>\d+)\/(?P<endpoint>.*)$
          - ^(?P<method>\w+)\s+(?P<path>.*)$
    
  attributes:
    actions:
      - key: http.user_agent
        action: delete
      - key: http.request.header.authorization
        action: delete
      - key: sensitive_data
        action: hash

exporters:
  jaeger:
    endpoint: jaeger-all-in-one:14250
    tls:
      insecure: true
  
  prometheus:
    endpoint: "0.0.0.0:8889"
    const_labels:
      environment: production
  
  logging:
    loglevel: debug
  
  elasticsearch:
    endpoints: ["http://elasticsearch:9200"]
    index: traces
    mapping:
      mode: ecs

service:
  telemetry:
    logs:
      level: info
    metrics:
      address: 0.0.0.0:8888
  
  pipelines:
    traces:
      receivers: [otlp, jaeger, zipkin]
      processors: [memory_limiter, resource, batch, probabilistic_sampler, span]
      exporters: [jaeger, logging]
    
    metrics:
      receivers: [otlp]
      processors: [memory_limiter, resource, batch]
      exporters: [prometheus, logging]

  extensions: [health_check, pprof, zpages]
```

---

## 🔧 Instrumentación de Aplicaciones

### 1. Spring Boot con OpenTelemetry

```java
// services/user-service/src/main/java/com/example/UserServiceApplication.java
package com.example.userservice;

import io.opentelemetry.api.OpenTelemetry;
import io.opentelemetry.api.trace.Tracer;
import io.opentelemetry.context.Scope;
import io.opentelemetry.api.trace.Span;
import io.opentelemetry.api.trace.StatusCode;
import io.opentelemetry.api.common.Attributes;
import io.opentelemetry.api.common.AttributeKey;
import io.opentelemetry.semconv.SemanticAttributes;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.web.bind.annotation.*;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;
import org.springframework.stereotype.Repository;
import org.springframework.data.jpa.repository.JpaRepository;

import javax.persistence.*;
import java.util.List;
import java.util.Optional;
import java.time.LocalDateTime;

@SpringBootApplication
public class UserServiceApplication {
    public static void main(String[] args) {
        SpringApplication.run(UserServiceApplication.class, args);
    }
}

@RestController
@RequestMapping("/api/users")
public class UserController {
    
    @Autowired
    private UserService userService;
    
    @Autowired
    private Tracer tracer;
    
    @GetMapping("/{id}")
    public ResponseEntity<User> getUser(@PathVariable Long id) {
        Span span = tracer.spanBuilder("get-user")
            .setSpanKind(SpanKind.SERVER)
            .setAttribute(SemanticAttributes.HTTP_METHOD, "GET")
            .setAttribute(SemanticAttributes.HTTP_URL, "/api/users/" + id)
            .setAttribute("user.id", id)
            .startSpan();
        
        try (Scope scope = span.makeCurrent()) {
            User user = userService.findById(id);
            if (user != null) {
                span.setAttribute("user.found", true);
                span.setAttribute("user.email", user.getEmail());
                return ResponseEntity.ok(user);
            } else {
                span.setAttribute("user.found", false);
                span.setStatus(StatusCode.ERROR, "User not found");
                return ResponseEntity.notFound().build();
            }
        } catch (Exception e) {
            span.setStatus(StatusCode.ERROR, e.getMessage());
            span.recordException(e);
            throw e;
        } finally {
            span.end();
        }
    }
    
    @PostMapping
    public ResponseEntity<User> createUser(@RequestBody CreateUserRequest request) {
        Span span = tracer.spanBuilder("create-user")
            .setSpanKind(SpanKind.SERVER)
            .setAttribute(SemanticAttributes.HTTP_METHOD, "POST")
            .setAttribute(SemanticAttributes.HTTP_URL, "/api/users")
            .setAttribute("user.email", request.getEmail())
            .startSpan();
        
        try (Scope scope = span.makeCurrent()) {
            // Add custom baggage
            Baggage baggage = Baggage.current()
                .toBuilder()
                .put("user.creation.source", "api")
                .put("user.creation.timestamp", LocalDateTime.now().toString())
                .build();
            
            try (Scope baggageScope = baggage.makeCurrent()) {
                User user = userService.createUser(request);
                span.setAttribute("user.created.id", user.getId());
                span.addEvent("user-created", 
                    Attributes.of(
                        AttributeKey.stringKey("user.id"), user.getId().toString(),
                        AttributeKey.stringKey("user.email"), user.getEmail()
                    ));
                return ResponseEntity.status(201).body(user);
            }
        } catch (Exception e) {
            span.setStatus(StatusCode.ERROR, e.getMessage());
            span.recordException(e);
            throw e;
        } finally {
            span.end();
        }
    }
    
    @GetMapping
    public ResponseEntity<List<User>> getAllUsers(
            @RequestParam(defaultValue = "0") int page,
            @RequestParam(defaultValue = "10") int size) {
        
        Span span = tracer.spanBuilder("get-all-users")
            .setSpanKind(SpanKind.SERVER)
            .setAttribute(SemanticAttributes.HTTP_METHOD, "GET")
            .setAttribute(SemanticAttributes.HTTP_URL, "/api/users")
            .setAttribute("pagination.page", page)
            .setAttribute("pagination.size", size)
            .startSpan();
        
        try (Scope scope = span.makeCurrent()) {
            List<User> users = userService.findAll(page, size);
            span.setAttribute("users.count", users.size());
            return ResponseEntity.ok(users);
        } catch (Exception e) {
            span.setStatus(StatusCode.ERROR, e.getMessage());
            span.recordException(e);
            throw e;
        } finally {
            span.end();
        }
    }
}

@Service
public class UserService {
    
    @Autowired
    private UserRepository userRepository;
    
    @Autowired
    private Tracer tracer;
    
    public User findById(Long id) {
        Span span = tracer.spanBuilder("user-service.find-by-id")
            .setSpanKind(SpanKind.INTERNAL)
            .setAttribute("user.id", id)
            .startSpan();
        
        try (Scope scope = span.makeCurrent()) {
            span.addEvent("database-query-start");
            Optional<User> user = userRepository.findById(id);
            span.addEvent("database-query-end");
            
            if (user.isPresent()) {
                span.setAttribute("query.result", "found");
                return user.get();
            } else {
                span.setAttribute("query.result", "not-found");
                return null;
            }
        } finally {
            span.end();
        }
    }
    
    public User createUser(CreateUserRequest request) {
        Span span = tracer.spanBuilder("user-service.create")
            .setSpanKind(SpanKind.INTERNAL)
            .setAttribute("user.email", request.getEmail())
            .startSpan();
        
        try (Scope scope = span.makeCurrent()) {
            // Validate user doesn't exist
            Span validationSpan = tracer.spanBuilder("user-validation")
                .setParent(Context.current())
                .startSpan();
            
            try (Scope validationScope = validationSpan.makeCurrent()) {
                if (userRepository.existsByEmail(request.getEmail())) {
                    validationSpan.setStatus(StatusCode.ERROR, "User already exists");
                    throw new UserAlreadyExistsException("User with email already exists");
                }
                validationSpan.addEvent("validation-passed");
            } finally {
                validationSpan.end();
            }
            
            // Create user
            User user = new User();
            user.setEmail(request.getEmail());
            user.setFirstName(request.getFirstName());
            user.setLastName(request.getLastName());
            user.setCreatedAt(LocalDateTime.now());
            
            span.addEvent("user-entity-created");
            
            // Save to database
            Span dbSpan = tracer.spanBuilder("database.save-user")
                .setSpanKind(SpanKind.CLIENT)
                .setAttribute(SemanticAttributes.DB_SYSTEM, "postgresql")
                .setAttribute(SemanticAttributes.DB_NAME, "tracing_demo")
                .setAttribute(SemanticAttributes.DB_OPERATION, "INSERT")
                .setAttribute(SemanticAttributes.DB_SQL_TABLE, "users")
                .startSpan();
            
            try (Scope dbScope = dbSpan.makeCurrent()) {
                User savedUser = userRepository.save(user);
                dbSpan.setAttribute("db.rows_affected", 1);
                span.setAttribute("user.created.id", savedUser.getId());
                return savedUser;
            } finally {
                dbSpan.end();
            }
            
        } finally {
            span.end();
        }
    }
    
    public List<User> findAll(int page, int size) {
        Span span = tracer.spanBuilder("user-service.find-all")
            .setSpanKind(SpanKind.INTERNAL)
            .setAttribute("pagination.page", page)
            .setAttribute("pagination.size", size)
            .startSpan();
        
        try (Scope scope = span.makeCurrent()) {
            Pageable pageable = PageRequest.of(page, size);
            Page<User> userPage = userRepository.findAll(pageable);
            
            span.setAttribute("results.total", userPage.getTotalElements());
            span.setAttribute("results.pages", userPage.getTotalPages());
            span.setAttribute("results.current_page_size", userPage.getContent().size());
            
            return userPage.getContent();
        } finally {
            span.end();
        }
    }
}

@Repository
public interface UserRepository extends JpaRepository<User, Long> {
    boolean existsByEmail(String email);
}

@Entity
@Table(name = "users")
public class User {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    @Column(unique = true, nullable = false)
    private String email;
    
    @Column(nullable = false)
    private String firstName;
    
    @Column(nullable = false)
    private String lastName;
    
    @Column(nullable = false)
    private LocalDateTime createdAt;
    
    // Getters and setters
    // ... (omitted for brevity)
}

public class CreateUserRequest {
    private String email;
    private String firstName;
    private String lastName;
    
    // Getters and setters
    // ... (omitted for brevity)
}

public class UserAlreadyExistsException extends RuntimeException {
    public UserAlreadyExistsException(String message) {
        super(message);
    }
}
```

### 2. Configuración de OpenTelemetry

```java
// services/user-service/src/main/java/com/example/config/TracingConfiguration.java
package com.example.userservice.config;

import io.opentelemetry.api.OpenTelemetry;
import io.opentelemetry.api.trace.Tracer;
import io.opentelemetry.context.propagation.ContextPropagators;
import io.opentelemetry.extension.trace.propagation.JaegerPropagator;
import io.opentelemetry.exporter.jaeger.JaegerGrpcSpanExporter;
import io.opentelemetry.exporter.otlp.trace.OtlpGrpcSpanExporter;
import io.opentelemetry.instrumentation.spring.webmvc.v6_0.SpringWebMvcTelemetry;
import io.opentelemetry.instrumentation.jdbc.OpenTelemetryDriver;
import io.opentelemetry.sdk.OpenTelemetrySdk;
import io.opentelemetry.sdk.trace.SdkTracerProvider;
import io.opentelemetry.sdk.trace.export.BatchSpanProcessor;
import io.opentelemetry.sdk.trace.samplers.Sampler;
import io.opentelemetry.sdk.resources.Resource;
import io.opentelemetry.semconv.ResourceAttributes;

import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.web.servlet.config.annotation.InterceptorRegistry;
import org.springframework.web.servlet.config.annotation.WebMvcConfigurer;

import java.time.Duration;

@Configuration
public class TracingConfiguration implements WebMvcConfigurer {
    
    @Value("${spring.application.name:user-service}")
    private String serviceName;
    
    @Value("${jaeger.endpoint:http://localhost:14268/api/traces}")
    private String jaegerEndpoint;
    
    @Value("${otel.exporter.otlp.endpoint:http://localhost:4317}")
    private String otlpEndpoint;
    
    @Value("${tracing.sampling.ratio:0.1}")
    private double samplingRatio;
    
    @Bean
    public OpenTelemetry openTelemetry() {
        Resource resource = Resource.getDefault()
            .merge(Resource.builder()
                .put(ResourceAttributes.SERVICE_NAME, serviceName)
                .put(ResourceAttributes.SERVICE_VERSION, "1.0.0")
                .put(ResourceAttributes.DEPLOYMENT_ENVIRONMENT, "development")
                .build());
        
        // Configure OTLP exporter
        OtlpGrpcSpanExporter otlpExporter = OtlpGrpcSpanExporter.builder()
            .setEndpoint(otlpEndpoint)
            .setTimeout(Duration.ofSeconds(30))
            .build();
        
        // Configure Jaeger exporter as fallback
        JaegerGrpcSpanExporter jaegerExporter = JaegerGrpcSpanExporter.builder()
            .setEndpoint(jaegerEndpoint)
            .setTimeout(Duration.ofSeconds(30))
            .build();
        
        // Configure tracer provider
        SdkTracerProvider tracerProvider = SdkTracerProvider.builder()
            .addSpanProcessor(BatchSpanProcessor.builder(otlpExporter)
                .setMaxExportBatchSize(512)
                .setExportTimeout(Duration.ofSeconds(30))
                .setScheduleDelay(Duration.ofSeconds(5))
                .build())
            .addSpanProcessor(BatchSpanProcessor.builder(jaegerExporter)
                .setMaxExportBatchSize(512)
                .setExportTimeout(Duration.ofSeconds(30))
                .setScheduleDelay(Duration.ofSeconds(5))
                .build())
            .setResource(resource)
            .setSampler(Sampler.traceIdRatioBased(samplingRatio))
            .build();
        
        // Configure context propagation
        ContextPropagators contextPropagators = ContextPropagators.create(
            TextMapPropagator.composite(
                W3CTraceContextPropagator.getInstance(),
                W3CBaggagePropagator.getInstance(),
                B3Propagator.injectingSingleHeader(),
                JaegerPropagator.getInstance()
            )
        );
        
        return OpenTelemetrySdk.builder()
            .setTracerProvider(tracerProvider)
            .setPropagators(contextPropagators)
            .buildAndRegisterGlobal();
    }
    
    @Bean
    public Tracer tracer(OpenTelemetry openTelemetry) {
        return openTelemetry.getTracer(serviceName, "1.0.0");
    }
    
    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        registry.addInterceptor(
            SpringWebMvcTelemetry.create(openTelemetry()).newInterceptor()
        );
    }
    
    @Bean
    public TracingFilter tracingFilter() {
        return new TracingFilter(openTelemetry());
    }
}

// Custom filter for additional tracing context
@Component
public class TracingFilter implements Filter {
    
    private final OpenTelemetry openTelemetry;
    
    public TracingFilter(OpenTelemetry openTelemetry) {
        this.openTelemetry = openTelemetry;
    }
    
    @Override
    public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain)
            throws IOException, ServletException {
        
        HttpServletRequest httpRequest = (HttpServletRequest) request;
        HttpServletResponse httpResponse = (HttpServletResponse) response;
        
        // Extract trace context from headers
        Context extractedContext = openTelemetry.getPropagators()
            .getTextMapPropagator()
            .extract(Context.current(), httpRequest, HttpServletRequestGetter.INSTANCE);
        
        // Set additional span attributes
        Span currentSpan = Span.current();
        if (currentSpan != null) {
            currentSpan.setAttribute("http.request.method", httpRequest.getMethod());
            currentSpan.setAttribute("http.request.url", httpRequest.getRequestURL().toString());
            currentSpan.setAttribute("http.request.user_agent", httpRequest.getHeader("User-Agent"));
            currentSpan.setAttribute("http.request.remote_addr", httpRequest.getRemoteAddr());
            
            // Add correlation ID if present
            String correlationId = httpRequest.getHeader("X-Correlation-ID");
            if (correlationId != null) {
                currentSpan.setAttribute("correlation.id", correlationId);
            }
        }
        
        try (Scope scope = extractedContext.makeCurrent()) {
            chain.doFilter(request, response);
            
            // Set response attributes
            if (currentSpan != null) {
                currentSpan.setAttribute("http.response.status_code", httpResponse.getStatus());
                currentSpan.setAttribute("http.response.size", httpResponse.getBufferSize());
            }
        }
    }
    
    private enum HttpServletRequestGetter implements TextMapGetter<HttpServletRequest> {
        INSTANCE;
        
        @Override
        public Iterable<String> keys(HttpServletRequest carrier) {
            return Collections.list(carrier.getHeaderNames());
        }
        
        @Override
        public String get(HttpServletRequest carrier, String key) {
            return carrier.getHeader(key);
        }
    }
}
```

---

## 🔄 Service-to-Service Communication

### 1. HTTP Client Instrumentation

```java
// services/order-service/src/main/java/com/example/config/HttpClientConfiguration.java
package com.example.orderservice.config;

import io.opentelemetry.api.OpenTelemetry;
import io.opentelemetry.instrumentation.httpclient.HttpClientTelemetry;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.web.client.RestTemplate;
import org.springframework.http.client.ClientHttpRequestInterceptor;

import java.net.http.HttpClient;
import java.util.List;
import java.util.ArrayList;

@Configuration
public class HttpClientConfiguration {
    
    private final OpenTelemetry openTelemetry;
    
    public HttpClientConfiguration(OpenTelemetry openTelemetry) {
        this.openTelemetry = openTelemetry;
    }
    
    @Bean
    public RestTemplate restTemplate() {
        RestTemplate restTemplate = new RestTemplate();
        
        List<ClientHttpRequestInterceptor> interceptors = new ArrayList<>();
        interceptors.add(new TracingClientHttpRequestInterceptor(openTelemetry));
        restTemplate.setInterceptors(interceptors);
        
        return restTemplate;
    }
    
    @Bean
    public HttpClient httpClient() {
        return HttpClientTelemetry.create(openTelemetry)
            .newHttpClientBuilder()
            .build();
    }
}

// Custom interceptor for RestTemplate
public class TracingClientHttpRequestInterceptor implements ClientHttpRequestInterceptor {
    
    private final OpenTelemetry openTelemetry;
    private final TextMapSetter<HttpHeaders> httpHeadersSetter;
    
    public TracingClientHttpRequestInterceptor(OpenTelemetry openTelemetry) {
        this.openTelemetry = openTelemetry;
        this.httpHeadersSetter = (headers, key, value) -> headers.add(key, value);
    }
    
    @Override
    public ClientHttpResponse intercept(
            HttpRequest request, 
            byte[] body, 
            ClientHttpRequestExecution execution) throws IOException {
        
        // Inject trace context into headers
        openTelemetry.getPropagators()
            .getTextMapPropagator()
            .inject(Context.current(), request.getHeaders(), httpHeadersSetter);
        
        // Add correlation ID
        String correlationId = MDC.get("correlationId");
        if (correlationId != null) {
            request.getHeaders().add("X-Correlation-ID", correlationId);
        }
        
        return execution.execute(request, body);
    }
}
```

### 2. Order Service Implementation

```java
// services/order-service/src/main/java/com/example/service/OrderService.java
package com.example.orderservice.service;

import io.opentelemetry.api.trace.Tracer;
import io.opentelemetry.api.trace.Span;
import io.opentelemetry.api.trace.SpanKind;
import io.opentelemetry.context.Scope;
import io.opentelemetry.api.trace.StatusCode;
import io.opentelemetry.semconv.SemanticAttributes;

import org.springframework.stereotype.Service;
import org.springframework.web.client.RestTemplate;
import org.springframework.amqp.rabbit.core.RabbitTemplate;
import org.springframework.beans.factory.annotation.Value;

@Service
public class OrderService {
    
    private final RestTemplate restTemplate;
    private final RabbitTemplate rabbitTemplate;
    private final Tracer tracer;
    
    @Value("${user.service.url:http://user-service:8080}")
    private String userServiceUrl;
    
    public OrderService(RestTemplate restTemplate, RabbitTemplate rabbitTemplate, Tracer tracer) {
        this.restTemplate = restTemplate;
        this.rabbitTemplate = rabbitTemplate;
        this.tracer = tracer;
    }
    
    public Order createOrder(CreateOrderRequest request) {
        Span span = tracer.spanBuilder("order-service.create-order")
            .setSpanKind(SpanKind.INTERNAL)
            .setAttribute("order.user_id", request.getUserId())
            .setAttribute("order.items_count", request.getItems().size())
            .startSpan();
        
        try (Scope scope = span.makeCurrent()) {
            // 1. Validate user exists
            User user = validateUser(request.getUserId());
            span.setAttribute("user.validated", true);
            
            // 2. Calculate total
            BigDecimal total = calculateOrderTotal(request.getItems());
            span.setAttribute("order.total", total.doubleValue());
            
            // 3. Create order
            Order order = new Order();
            order.setUserId(request.getUserId());
            order.setItems(request.getItems());
            order.setTotal(total);
            order.setStatus(OrderStatus.PENDING);
            order.setCreatedAt(LocalDateTime.now());
            
            // 4. Save order
            Order savedOrder = saveOrder(order);
            span.setAttribute("order.id", savedOrder.getId());
            
            // 5. Send notification asynchronously
            sendOrderNotification(savedOrder);
            
            span.addEvent("order-created-successfully");
            return savedOrder;
            
        } catch (Exception e) {
            span.setStatus(StatusCode.ERROR, e.getMessage());
            span.recordException(e);
            throw e;
        } finally {
            span.end();
        }
    }
    
    private User validateUser(Long userId) {
        Span span = tracer.spanBuilder("order-service.validate-user")
            .setSpanKind(SpanKind.CLIENT)
            .setAttribute(SemanticAttributes.HTTP_METHOD, "GET")
            .setAttribute(SemanticAttributes.HTTP_URL, userServiceUrl + "/api/users/" + userId)
            .setAttribute("user.id", userId)
            .startSpan();
        
        try (Scope scope = span.makeCurrent()) {
            String url = userServiceUrl + "/api/users/" + userId;
            User user = restTemplate.getForObject(url, User.class);
            
            if (user == null) {
                span.setStatus(StatusCode.ERROR, "User not found");
                throw new UserNotFoundException("User with ID " + userId + " not found");
            }
            
            span.setAttribute("user.email", user.getEmail());
            span.addEvent("user-validation-successful");
            return user;
            
        } catch (Exception e) {
            span.setStatus(StatusCode.ERROR, e.getMessage());
            span.recordException(e);
            throw e;
        } finally {
            span.end();
        }
    }
    
    private BigDecimal calculateOrderTotal(List<OrderItem> items) {
        Span span = tracer.spanBuilder("order-service.calculate-total")
            .setSpanKind(SpanKind.INTERNAL)
            .setAttribute("items.count", items.size())
            .startSpan();
        
        try (Scope scope = span.makeCurrent()) {
            BigDecimal total = items.stream()
                .map(item -> item.getPrice().multiply(new BigDecimal(item.getQuantity())))
                .reduce(BigDecimal.ZERO, BigDecimal::add);
            
            span.setAttribute("calculation.total", total.doubleValue());
            span.addEvent("total-calculated");
            return total;
            
        } finally {
            span.end();
        }
    }
    
    private Order saveOrder(Order order) {
        Span span = tracer.spanBuilder("order-service.save-order")
            .setSpanKind(SpanKind.CLIENT)
            .setAttribute(SemanticAttributes.DB_SYSTEM, "postgresql")
            .setAttribute(SemanticAttributes.DB_NAME, "tracing_demo")
            .setAttribute(SemanticAttributes.DB_OPERATION, "INSERT")
            .setAttribute(SemanticAttributes.DB_SQL_TABLE, "orders")
            .startSpan();
        
        try (Scope scope = span.makeCurrent()) {
            // Simulate database save
            Thread.sleep(100); // Simulate DB latency
            order.setId(System.currentTimeMillis()); // Mock ID generation
            
            span.setAttribute("db.rows_affected", 1);
            span.addEvent("order-persisted");
            return order;
            
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
            span.setStatus(StatusCode.ERROR, "Database operation interrupted");
            throw new RuntimeException("Database save interrupted", e);
        } finally {
            span.end();
        }
    }
    
    private void sendOrderNotification(Order order) {
        Span span = tracer.spanBuilder("order-service.send-notification")
            .setSpanKind(SpanKind.PRODUCER)
            .setAttribute("messaging.system", "rabbitmq")
            .setAttribute("messaging.destination", "order.notifications")
            .setAttribute("messaging.destination_kind", "queue")
            .setAttribute("order.id", order.getId())
            .startSpan();
        
        try (Scope scope = span.makeCurrent()) {
            OrderNotificationEvent event = new OrderNotificationEvent();
            event.setOrderId(order.getId());
            event.setUserId(order.getUserId());
            event.setTotal(order.getTotal());
            event.setTimestamp(LocalDateTime.now());
            
            // Inject trace context into message
            Map<String, Object> headers = new HashMap<>();
            openTelemetry.getPropagators()
                .getTextMapPropagator()
                .inject(Context.current(), headers, 
                    (carrier, key, value) -> carrier.put(key, value));
            
            rabbitTemplate.convertAndSend("order.notifications", event, message -> {
                headers.forEach((key, value) -> 
                    message.getMessageProperties().setHeader(key, value.toString()));
                return message;
            });
            
            span.addEvent("notification-sent");
            
        } catch (Exception e) {
            span.setStatus(StatusCode.ERROR, e.getMessage());
            span.recordException(e);
            // Don't throw - notification failure shouldn't fail order creation
        } finally {
            span.end();
        }
    }
}
```

---

## 📊 Sampling Strategies

### 1. Advanced Sampling Configuration

```java
// src/main/java/com/example/config/SamplingConfiguration.java
package com.example.config;

import io.opentelemetry.sdk.trace.samplers.Sampler;
import io.opentelemetry.sdk.trace.samplers.SamplingResult;
import io.opentelemetry.sdk.trace.samplers.SamplingDecision;
import io.opentelemetry.api.trace.SpanKind;
import io.opentelemetry.api.common.Attributes;
import io.opentelemetry.context.Context;
import io.opentelemetry.sdk.trace.data.LinkData;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

import java.util.List;
import java.util.Map;
import java.util.concurrent.ConcurrentHashMap;
import java.util.concurrent.atomic.AtomicLong;

@Configuration
public class SamplingConfiguration {
    
    @Bean
    public Sampler customSampler() {
        return new AdaptiveSampler();
    }
}

public class AdaptiveSampler implements Sampler {
    
    private final Map<String, ServiceSamplingState> serviceSamplingStates = new ConcurrentHashMap<>();
    private final double baseSamplingRate = 0.1; // 10% base rate
    private final double maxSamplingRate = 1.0;  // 100% max rate
    private final long windowSizeMs = 60000; // 1 minute window
    
    @Override
    public SamplingResult shouldSample(
            Context parentContext,
            String traceId,
            String name,
            SpanKind spanKind,
            Attributes attributes,
            List<LinkData> parentLinks) {
        
        String serviceName = attributes.get(SemanticAttributes.SERVICE_NAME);
        if (serviceName == null) {
            serviceName = "unknown";
        }
        
        ServiceSamplingState state = serviceSamplingStates.computeIfAbsent(
            serviceName, 
            k -> new ServiceSamplingState()
        );
        
        // Check for error conditions that should always be sampled
        if (shouldAlwaysSample(attributes, name)) {
            state.incrementSampled();
            return SamplingResult.create(SamplingDecision.RECORD_AND_SAMPLE);
        }
        
        // Check for high-value operations
        if (isHighValueOperation(name, attributes)) {
            double highValueRate = Math.min(baseSamplingRate * 5, maxSamplingRate);
            if (Math.random() < highValueRate) {
                state.incrementSampled();
                return SamplingResult.create(SamplingDecision.RECORD_AND_SAMPLE);
            }
        }
        
        // Adaptive sampling based on service load
        double adaptiveRate = calculateAdaptiveRate(state);
        if (Math.random() < adaptiveRate) {
            state.incrementSampled();
            return SamplingResult.create(SamplingDecision.RECORD_AND_SAMPLE);
        }
        
        state.incrementNotSampled();
        return SamplingResult.create(SamplingDecision.NOT_RECORD);
    }
    
    private boolean shouldAlwaysSample(Attributes attributes, String operationName) {
        // Always sample errors
        String httpStatusCode = attributes.get(SemanticAttributes.HTTP_STATUS_CODE);
        if (httpStatusCode != null) {
            int statusCode = Integer.parseInt(httpStatusCode);
            if (statusCode >= 400) {
                return true;
            }
        }
        
        // Always sample slow operations (if duration is available)
        Long durationMs = attributes.get(AttributeKey.longKey("duration.ms"));
        if (durationMs != null && durationMs > 5000) { // > 5 seconds
            return true;
        }
        
        // Always sample critical operations
        if (operationName.contains("payment") || 
            operationName.contains("checkout") ||
            operationName.contains("login")) {
            return true;
        }
        
        return false;
    }
    
    private boolean isHighValueOperation(String operationName, Attributes attributes) {
        // Database operations
        if (attributes.get(SemanticAttributes.DB_SYSTEM) != null) {
            return true;
        }
        
        // External HTTP calls
        if (attributes.get(SemanticAttributes.HTTP_URL) != null && 
            !attributes.get(SemanticAttributes.HTTP_URL).contains("localhost")) {
            return true;
        }
        
        // Message queue operations
        if (attributes.get(SemanticAttributes.MESSAGING_SYSTEM) != null) {
            return true;
        }
        
        return false;
    }
    
    private double calculateAdaptiveRate(ServiceSamplingState state) {
        long currentTime = System.currentTimeMillis();
        state.cleanupOldEntries(currentTime, windowSizeMs);
        
        double currentLoad = state.getCurrentLoad();
        
        // Increase sampling rate for low-traffic services
        if (currentLoad < 10) { // < 10 requests per minute
            return Math.min(baseSamplingRate * 3, maxSamplingRate);
        }
        
        // Decrease sampling rate for high-traffic services
        if (currentLoad > 1000) { // > 1000 requests per minute
            return baseSamplingRate * 0.1;
        }
        
        return baseSamplingRate;
    }
    
    @Override
    public String getDescription() {
        return "AdaptiveSampler{baseSamplingRate=" + baseSamplingRate + "}";
    }
    
    private static class ServiceSamplingState {
        private final AtomicLong sampledCount = new AtomicLong();
        private final AtomicLong notSampledCount = new AtomicLong();
        private final Map<Long, Long> requestCounts = new ConcurrentHashMap<>();
        
        void incrementSampled() {
            sampledCount.incrementAndGet();
            recordRequest();
        }
        
        void incrementNotSampled() {
            notSampledCount.incrementAndGet();
            recordRequest();
        }
        
        private void recordRequest() {
            long currentMinute = System.currentTimeMillis() / 60000;
            requestCounts.merge(currentMinute, 1L, Long::sum);
        }
        
        double getCurrentLoad() {
            long currentMinute = System.currentTimeMillis() / 60000;
            return requestCounts.getOrDefault(currentMinute, 0L);
        }
        
        void cleanupOldEntries(long currentTime, long windowSizeMs) {
            long cutoffMinute = (currentTime - windowSizeMs) / 60000;
            requestCounts.entrySet().removeIf(entry -> entry.getKey() < cutoffMinute);
        }
    }
}
```

---

## 📊 Análisis y Visualización

### 1. Custom Queries en Jaeger

```javascript
// jaeger-queries.js - Custom queries for Jaeger analysis

const JaegerQueries = {
    // Find slow operations
    findSlowOperations: {
        service: "*",
        operation: "*",
        tags: {},
        minDuration: "5s",
        maxDuration: "",
        limit: 100
    },
    
    // Find error traces
    findErrorTraces: {
        service: "*",
        operation: "*",
        tags: {
            "error": "true",
            "http.status_code": "5xx"
        },
        limit: 50
    },
    
    // Database performance analysis
    findDatabaseIssues: {
        service: "*",
        operation: "*",
        tags: {
            "db.system": "*",
            "span.kind": "client"
        },
        minDuration: "1s",
        limit: 100
    },
    
    // Service dependency analysis
    findServiceDependencies: {
        service: "order-service",
        operation: "*",
        tags: {
            "span.kind": "client"
        },
        limit: 200
    }
};

// Custom analysis functions
function analyzeServiceLatency(traces) {
    const latencyStats = {};
    
    traces.forEach(trace => {
        trace.spans.forEach(span => {
            const service = span.process.serviceName;
            const duration = span.duration / 1000; // Convert to ms
            
            if (!latencyStats[service]) {
                latencyStats[service] = {
                    count: 0,
                    totalDuration: 0,
                    minDuration: Infinity,
                    maxDuration: 0,
                    durations: []
                };
            }
            
            const stats = latencyStats[service];
            stats.count++;
            stats.totalDuration += duration;
            stats.minDuration = Math.min(stats.minDuration, duration);
            stats.maxDuration = Math.max(stats.maxDuration, duration);
            stats.durations.push(duration);
        });
    });
    
    // Calculate percentiles
    Object.keys(latencyStats).forEach(service => {
        const stats = latencyStats[service];
        stats.avgDuration = stats.totalDuration / stats.count;
        
        const sortedDurations = stats.durations.sort((a, b) => a - b);
        stats.p50 = getPercentile(sortedDurations, 50);
        stats.p95 = getPercentile(sortedDurations, 95);
        stats.p99 = getPercentile(sortedDurations, 99);
    });
    
    return latencyStats;
}

function getPercentile(sortedArray, percentile) {
    const index = Math.ceil((percentile / 100) * sortedArray.length) - 1;
    return sortedArray[index];
}

function findBottlenecks(traces) {
    const bottlenecks = [];
    
    traces.forEach(trace => {
        const totalDuration = trace.spans[0].duration;
        const serviceDurations = {};
        
        trace.spans.forEach(span => {
            const service = span.process.serviceName;
            if (!serviceDurations[service]) {
                serviceDurations[service] = 0;
            }
            serviceDurations[service] += span.duration;
        });
        
        // Find services that take >30% of total trace time
        Object.keys(serviceDurations).forEach(service => {
            const percentage = (serviceDurations[service] / totalDuration) * 100;
            if (percentage > 30) {
                bottlenecks.push({
                    traceId: trace.traceID,
                    service: service,
                    percentage: percentage,
                    duration: serviceDurations[service] / 1000
                });
            }
        });
    });
    
    return bottlenecks.sort((a, b) => b.percentage - a.percentage);
}
```

### 2. Grafana Dashboard para Tracing

```json
{
  "dashboard": {
    "id": null,
    "title": "Distributed Tracing Dashboard",
    "tags": ["tracing", "jaeger", "performance"],
    "timezone": "browser",
    "refresh": "30s",
    "panels": [
      {
        "id": 1,
        "title": "Request Rate by Service",
        "type": "timeseries",
        "targets": [
          {
            "expr": "sum by (service_name) (rate(traces_total[5m]))",
            "legendFormat": "{{service_name}}"
          }
        ]
      },
      {
        "id": 2,
        "title": "Average Latency by Service",
        "type": "timeseries",
        "targets": [
          {
            "expr": "histogram_quantile(0.50, sum by (service_name, le) (rate(trace_duration_bucket[5m])))",
            "legendFormat": "p50 - {{service_name}}"
          },
          {
            "expr": "histogram_quantile(0.95, sum by (service_name, le) (rate(trace_duration_bucket[5m])))",
            "legendFormat": "p95 - {{service_name}}"
          }
        ]
      },
      {
        "id": 3,
        "title": "Error Rate by Service",
        "type": "timeseries",
        "targets": [
          {
            "expr": "sum by (service_name) (rate(traces_total{status=\"error\"}[5m])) / sum by (service_name) (rate(traces_total[5m])) * 100",
            "legendFormat": "{{service_name}}"
          }
        ],
        "fieldConfig": {
          "defaults": {
            "unit": "percent"
          }
        }
      },
      {
        "id": 4,
        "title": "Service Dependencies",
        "type": "nodeGraph",
        "targets": [
          {
            "expr": "sum by (source_service, target_service) (rate(service_calls_total[5m]))",
            "format": "table"
          }
        ]
      },
      {
        "id": 5,
        "title": "Top Slow Operations",
        "type": "table",
        "targets": [
          {
            "expr": "topk(10, histogram_quantile(0.95, sum by (operation_name, service_name, le) (rate(operation_duration_bucket[5m]))))",
            "format": "table",
            "instant": true
          }
        ]
      },
      {
        "id": 6,
        "title": "Sampling Rate",
        "type": "gauge",
        "targets": [
          {
            "expr": "sum(rate(traces_sampled_total[5m])) / sum(rate(traces_received_total[5m])) * 100",
            "legendFormat": "Sampling Rate"
          }
        ],
        "fieldConfig": {
          "defaults": {
            "unit": "percent",
            "thresholds": {
              "steps": [
                {"color": "red", "value": 0},
                {"color": "yellow", "value": 5},
                {"color": "green", "value": 10}
              ]
            }
          }
        }
      }
    ]
  }
}
```

---

## 📋 Ejercicios Prácticos

### Ejercicio 1: Local Jaeger Setup

**Objetivo**: Configurar Jaeger localmente con microservicios.

**Tareas**:

1. Desplegar Jaeger All-in-One
2. Instrumentar aplicación Spring Boot
3. Generar traces de prueba
4. Analizar traces en Jaeger UI

### Ejercicio 2: Production Jaeger Setup

**Objetivo**: Configurar Jaeger para producción.

**Componentes**:

- Elasticsearch backend
- Jaeger Collector cluster
- Jaeger Query instances
- OpenTelemetry Collector

### Ejercicio 3: Custom Instrumentation

**Objetivo**: Implementar instrumentación personalizada.

**Tareas**:

- Custom spans para operaciones de negocio
- Baggage propagation
- Custom sampling strategies
- Error tracking y alertas

---

## 🧪 Laboratorio

### Lab 1: Microservices Tracing

1. **Desplegar arquitectura completa**:

   ```bash
   docker-compose -f docker-compose.jaeger.yml up -d
   ```

2. **Instrumentar servicios**:
   - User Service
   - Order Service
   - Notification Service

3. **Generar traffic de prueba**:
   - Crear usuarios
   - Procesar órdenes
   - Enviar notificaciones

4. **Analizar traces**:
   - Identificar bottlenecks
   - Analizar dependencias
   - Detectar errores

### Lab 2: Performance Optimization

1. **Identificar problemas de rendimiento**
2. **Optimizar consultas lentas**
3. **Reducir latencia de servicios**
4. **Validar mejoras con tracing**

---

## 📖 Best Practices

### 1. Principios de Distributed Tracing

```yaml
tracing_principles:
  instrumentation:
    - "Instrumentar en boundaries de servicios"
    - "Capturar contexto de negocio relevante"
    - "Evitar instrumentación excesiva"
  
  sampling:
    - "Usar sampling adaptativo"
    - "Siempre capturar errores y operaciones lentas"
    - "Considerar el volumen de datos"
  
  analysis:
    - "Correlacionar con logs y métricas"
    - "Enfocar en user journeys completos"
    - "Automatizar detección de anomalías"
```

### 2. Context Propagation

```yaml
context_propagation:
  headers:
    - "Usar estándares como W3C Trace Context"
    - "Propagar correlation IDs"
    - "Mantener consistencia entre servicios"
  
  async_operations:
    - "Propagar contexto en operaciones async"
    - "Usar baggage para datos de negocio"
    - "Mantener trazabilidad en message queues"
```

---

## ✅ Resumen del Módulo

En este módulo has aprendido:

1. **Arquitectura de Jaeger** - Componentes y configuración
2. **Instrumentación de aplicaciones** - OpenTelemetry y Spring Boot
3. **Service-to-service communication** - Context propagation
4. **Sampling strategies** - Estrategias adaptativas y personalizadas
5. **Análisis de traces** - Identificación de bottlenecks y problemas
6. **Visualización** - Grafana dashboards y Jaeger UI
7. **Best practices** - Principios de distributed tracing efectivo
8. **Optimización** - Performance tuning basado en insights

### Próximos pasos

- **Módulo 11**: Application Performance Monitoring (APM)
- **Módulo 12**: Health checks y circuit breakers
- **Módulo 13**: Laboratorio final y evaluación

¡Continúa con el siguiente módulo para completar tu conocimiento en APM!
