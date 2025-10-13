# Módulo 11: Application Performance Monitoring (APM)

## 📋 Introducción

Application Performance Monitoring (APM) es fundamental para mantener aplicaciones distribuidas funcionando de manera óptima. APM combina métricas de infraestructura, trazas distribuidas, logs estructurados y monitoreo de experiencia de usuario para proporcionar visibilidad completa del rendimiento de aplicaciones.

### ¿Qué es APM?

APM incluye:

- **Real User Monitoring (RUM)**: Experiencia de usuarios reales
- **Synthetic Monitoring**: Pruebas automatizadas de funcionalidad
- **Infrastructure Monitoring**: Métricas de servidores y contenedores
- **Application Metrics**: Rendimiento de código y servicios
- **Error Tracking**: Detección y análisis de errores
- **Dependency Mapping**: Visualización de arquitectura

### Herramientas APM Populares

- **Elastic APM**: Stack completo de Elastic
- **New Relic**: Plataforma APM empresarial
- **Datadog**: Monitoreo unificado
- **AppDynamics**: APM inteligente
- **Dynatrace**: AI-powered APM
- **Grafana Stack**: Prometheus + Grafana + Loki + Tempo

---

## 🎯 Objetivos de Aprendizaje

Al completar este módulo, serás capaz de:

1. **Implementar** stack completo de APM con Elastic
2. **Configurar** Real User Monitoring (RUM)
3. **Diseñar** dashboards de rendimiento efectivos
4. **Implementar** alertas proactivas de performance
5. **Analizar** métricas de aplicación y correlación
6. **Configurar** synthetic monitoring
7. **Optimizar** aplicaciones basado en datos APM
8. **Establecer** SLIs/SLOs para servicios

---

## 🏗️ Arquitectura APM con Elastic Stack

### 1. Elastic APM Stack

```yaml
# docker-compose.apm.yml
version: '3.8'

services:
  # Elasticsearch
  elasticsearch:
    image: elasticsearch:8.11.0
    container_name: apm-elasticsearch
    environment:
      - discovery.type=single-node
      - xpack.security.enabled=false
      - xpack.monitoring.collection.enabled=true
      - "ES_JAVA_OPTS=-Xms2g -Xmx2g"
    volumes:
      - elasticsearch_data:/usr/share/elasticsearch/data
    ports:
      - "9200:9200"
    networks:
      - apm_network
    healthcheck:
      test: ["CMD-SHELL", "curl -f http://localhost:9200/_cluster/health || exit 1"]
      interval: 30s
      timeout: 10s
      retries: 5

  # Kibana
  kibana:
    image: kibana:8.11.0
    container_name: apm-kibana
    ports:
      - "5601:5601"
    environment:
      - ELASTICSEARCH_HOSTS=http://elasticsearch:9200
      - SERVER_NAME=kibana
      - XPACK_MONITORING_ENABLED=true
      - XPACK_APM_ENABLED=true
    volumes:
      - ./kibana/config:/usr/share/kibana/config
    networks:
      - apm_network
    depends_on:
      elasticsearch:
        condition: service_healthy

  # APM Server
  apm-server:
    image: elastic/apm-server:8.11.0
    container_name: apm-server
    ports:
      - "8200:8200"
    environment:
      - output.elasticsearch.hosts=["elasticsearch:9200"]
      - apm-server.host=0.0.0.0:8200
      - apm-server.rum.enabled=true
      - apm-server.rum.allow_origins=["*"]
      - apm-server.rum.allow_headers=["*"]
      - setup.kibana.host=kibana:5601
      - setup.template.enabled=true
      - logging.level=info
    volumes:
      - ./apm-server/config/apm-server.yml:/usr/share/apm-server/apm-server.yml:ro
    networks:
      - apm_network
    depends_on:
      elasticsearch:
        condition: service_healthy

  # Metricbeat for infrastructure metrics
  metricbeat:
    image: elastic/metricbeat:8.11.0
    container_name: apm-metricbeat
    user: root
    volumes:
      - ./metricbeat/config/metricbeat.yml:/usr/share/metricbeat/metricbeat.yml:ro
      - /var/run/docker.sock:/var/run/docker.sock:ro
      - /sys/fs/cgroup:/hostfs/sys/fs/cgroup:ro
      - /proc:/hostfs/proc:ro
      - /:/hostfs:ro
    environment:
      - ELASTICSEARCH_HOST=elasticsearch:9200
      - KIBANA_HOST=kibana:5601
    networks:
      - apm_network
    depends_on:
      elasticsearch:
        condition: service_healthy

  # Filebeat for log collection
  filebeat:
    image: elastic/filebeat:8.11.0
    container_name: apm-filebeat
    user: root
    volumes:
      - ./filebeat/config/filebeat.yml:/usr/share/filebeat/filebeat.yml:ro
      - /var/lib/docker/containers:/var/lib/docker/containers:ro
      - /var/run/docker.sock:/var/run/docker.sock:ro
    environment:
      - ELASTICSEARCH_HOST=elasticsearch:9200
      - KIBANA_HOST=kibana:5601
    networks:
      - apm_network
    depends_on:
      elasticsearch:
        condition: service_healthy

  # Heartbeat for synthetic monitoring
  heartbeat:
    image: elastic/heartbeat:8.11.0
    container_name: apm-heartbeat
    volumes:
      - ./heartbeat/config/heartbeat.yml:/usr/share/heartbeat/heartbeat.yml:ro
    environment:
      - ELASTICSEARCH_HOST=elasticsearch:9200
      - KIBANA_HOST=kibana:5601
    networks:
      - apm_network
    depends_on:
      elasticsearch:
        condition: service_healthy

  # Sample application with APM instrumentation
  sample-api:
    build:
      context: ./sample-app
      dockerfile: Dockerfile
    container_name: apm-sample-api
    ports:
      - "8080:8080"
    environment:
      - ELASTIC_APM_SERVER_URL=http://apm-server:8200
      - ELASTIC_APM_SERVICE_NAME=sample-api
      - ELASTIC_APM_SERVICE_VERSION=1.0.0
      - ELASTIC_APM_ENVIRONMENT=development
      - ELASTIC_APM_APPLICATION_PACKAGES=com.example
      - ELASTIC_APM_CAPTURE_BODY=all
      - ELASTIC_APM_TRANSACTION_SAMPLE_RATE=1.0
    networks:
      - apm_network
    depends_on:
      - apm-server
      - postgres

  # Sample frontend with RUM
  sample-frontend:
    build:
      context: ./sample-frontend
      dockerfile: Dockerfile
    container_name: apm-sample-frontend
    ports:
      - "3000:80"
    environment:
      - REACT_APP_API_URL=http://localhost:8080
      - REACT_APP_APM_SERVER_URL=http://localhost:8200
      - REACT_APP_APM_SERVICE_NAME=sample-frontend
      - REACT_APP_APM_SERVICE_VERSION=1.0.0
      - REACT_APP_APM_ENVIRONMENT=development
    networks:
      - apm_network
    depends_on:
      - sample-api

  # PostgreSQL
  postgres:
    image: postgres:15
    container_name: apm-postgres
    environment:
      - POSTGRES_DB=apm_demo
      - POSTGRES_USER=postgres
      - POSTGRES_PASSWORD=postgres123
    volumes:
      - postgres_data:/var/lib/postgresql/data
      - ./postgres/init.sql:/docker-entrypoint-initdb.d/init.sql
    ports:
      - "5432:5432"
    networks:
      - apm_network

  # Redis for caching
  redis:
    image: redis:7-alpine
    container_name: apm-redis
    ports:
      - "6379:6379"
    networks:
      - apm_network

  # Nginx as load balancer/reverse proxy
  nginx:
    image: nginx:alpine
    container_name: apm-nginx
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./nginx/config/nginx.conf:/etc/nginx/nginx.conf:ro
      - ./nginx/config/default.conf:/etc/nginx/conf.d/default.conf:ro
    networks:
      - apm_network
    depends_on:
      - sample-api

volumes:
  elasticsearch_data:
  postgres_data:

networks:
  apm_network:
    driver: bridge
```

### 2. APM Server Configuration

```yaml
# apm-server/config/apm-server.yml
apm-server:
  host: "0.0.0.0:8200"
  
  # Real User Monitoring (RUM)
  rum:
    enabled: true
    event_rate:
      limit: 10000
      lru_size: 1000
    allow_origins: ["*"]
    allow_headers: ["*"]
    source_mapping:
      enabled: true
      cache:
        expiration: 5m
      index_pattern: "apm-*-sourcemap*"
  
  # Jaeger integration
  jaeger:
    grpc:
      enabled: true
      host: "0.0.0.0:14250"
    http:
      enabled: true
      host: "0.0.0.0:14268"
  
  # OpenTelemetry integration
  otlp:
    grpc:
      enabled: true
      host: "0.0.0.0:4317"
    http:
      enabled: true
      host: "0.0.0.0:4318"

# Output configuration
output.elasticsearch:
  hosts: ["elasticsearch:9200"]
  protocol: "http"
  worker: 1
  index: "apm-%{[observer.version]}-%{[processor.event]}-%{+yyyy.MM.dd}"
  template.settings:
    index:
      number_of_shards: 1
      number_of_replicas: 0
      codec: best_compression

# Monitoring
monitoring:
  enabled: true
  elasticsearch:
    hosts: ["elasticsearch:9200"]

# Setup
setup.template.enabled: true
setup.kibana.host: "kibana:5601"

# Logging
logging.level: info
logging.to_files: true
logging.files:
  path: /var/log/apm-server
  name: apm-server
  keepfiles: 7
  permissions: 0600
```

---

## 🚀 Instrumentación de Aplicaciones

### 1. Spring Boot APM Integration

```java
// sample-app/src/main/java/com/example/SampleApiApplication.java
package com.example.sampleapi;

import co.elastic.apm.api.ElasticApm;
import co.elastic.apm.api.Transaction;
import co.elastic.apm.api.Span;
import co.elastic.apm.attach.ElasticApmAttacher;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.web.bind.annotation.*;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;
import org.springframework.stereotype.Repository;
import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.data.jpa.repository.Query;
import org.springframework.data.redis.core.RedisTemplate;
import org.springframework.cache.annotation.Cacheable;

import javax.persistence.*;
import java.util.List;
import java.util.Optional;
import java.time.LocalDateTime;
import java.util.concurrent.TimeUnit;
import java.util.Random;

@SpringBootApplication
public class SampleApiApplication {
    public static void main(String[] args) {
        // Attach Elastic APM agent programmatically
        ElasticApmAttacher.attach();
        SpringApplication.run(SampleApiApplication.class, args);
    }
}

@RestController
@RequestMapping("/api")
@CrossOrigin(origins = "*")
public class SampleController {
    
    @Autowired
    private ProductService productService;
    
    @Autowired
    private MetricsService metricsService;
    
    @GetMapping("/products")
    public ResponseEntity<List<Product>> getProducts(
            @RequestParam(defaultValue = "0") int page,
            @RequestParam(defaultValue = "10") int size) {
        
        Transaction transaction = ElasticApm.startTransaction();
        transaction.setName("GET /api/products");
        transaction.setType("request");
        transaction.addLabel("endpoint", "/api/products");
        transaction.addLabel("method", "GET");
        
        try {
            List<Product> products = productService.findProducts(page, size);
            transaction.addLabel("result_count", String.valueOf(products.size()));
            transaction.setResult("success");
            return ResponseEntity.ok(products);
        } catch (Exception e) {
            transaction.captureException(e);
            transaction.setResult("error");
            throw e;
        } finally {
            transaction.end();
        }
    }
    
    @GetMapping("/products/{id}")
    public ResponseEntity<Product> getProduct(@PathVariable Long id) {
        Transaction transaction = ElasticApm.startTransaction();
        transaction.setName("GET /api/products/{id}");
        transaction.setType("request");
        transaction.addLabel("product_id", String.valueOf(id));
        
        try {
            Optional<Product> product = productService.findById(id);
            if (product.isPresent()) {
                transaction.setResult("success");
                return ResponseEntity.ok(product.get());
            } else {
                transaction.setResult("not_found");
                return ResponseEntity.notFound().build();
            }
        } catch (Exception e) {
            transaction.captureException(e);
            transaction.setResult("error");
            throw e;
        } finally {
            transaction.end();
        }
    }
    
    @PostMapping("/products")
    public ResponseEntity<Product> createProduct(@RequestBody CreateProductRequest request) {
        Transaction transaction = ElasticApm.startTransaction();
        transaction.setName("POST /api/products");
        transaction.setType("request");
        transaction.addLabel("product_name", request.getName());
        
        try {
            Product product = productService.createProduct(request);
            transaction.addLabel("created_product_id", String.valueOf(product.getId()));
            transaction.setResult("success");
            return ResponseEntity.status(201).body(product);
        } catch (Exception e) {
            transaction.captureException(e);
            transaction.setResult("error");
            throw e;
        } finally {
            transaction.end();
        }
    }
    
    @GetMapping("/products/search")
    public ResponseEntity<List<Product>> searchProducts(
            @RequestParam String query,
            @RequestParam(defaultValue = "0") int page,
            @RequestParam(defaultValue = "10") int size) {
        
        Transaction transaction = ElasticApm.startTransaction();
        transaction.setName("GET /api/products/search");
        transaction.setType("request");
        transaction.addLabel("search_query", query);
        transaction.addLabel("query_length", String.valueOf(query.length()));
        
        try {
            List<Product> products = productService.searchProducts(query, page, size);
            transaction.addLabel("result_count", String.valueOf(products.size()));
            transaction.setResult("success");
            return ResponseEntity.ok(products);
        } catch (Exception e) {
            transaction.captureException(e);
            transaction.setResult("error");
            throw e;
        } finally {
            transaction.end();
        }
    }
    
    @GetMapping("/metrics")
    public ResponseEntity<Map<String, Object>> getMetrics() {
        return ResponseEntity.ok(metricsService.getApplicationMetrics());
    }
    
    @GetMapping("/health")
    public ResponseEntity<Map<String, String>> health() {
        Map<String, String> status = new HashMap<>();
        status.put("status", "UP");
        status.put("timestamp", LocalDateTime.now().toString());
        return ResponseEntity.ok(status);
    }
    
    @GetMapping("/slow-endpoint")
    public ResponseEntity<String> slowEndpoint() {
        Transaction transaction = ElasticApm.startTransaction();
        transaction.setName("GET /api/slow-endpoint");
        transaction.setType("request");
        
        try {
            // Simulate slow operation
            Thread.sleep(2000 + new Random().nextInt(3000)); // 2-5 seconds
            transaction.setResult("success");
            return ResponseEntity.ok("Slow operation completed");
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
            transaction.captureException(e);
            transaction.setResult("error");
            throw new RuntimeException("Operation interrupted", e);
        } finally {
            transaction.end();
        }
    }
    
    @GetMapping("/error-endpoint")
    public ResponseEntity<String> errorEndpoint() {
        Transaction transaction = ElasticApm.startTransaction();
        transaction.setName("GET /api/error-endpoint");
        transaction.setType("request");
        
        try {
            if (new Random().nextBoolean()) {
                throw new RuntimeException("Random error occurred");
            }
            transaction.setResult("success");
            return ResponseEntity.ok("Success");
        } catch (Exception e) {
            transaction.captureException(e);
            transaction.setResult("error");
            throw e;
        } finally {
            transaction.end();
        }
    }
}

@Service
public class ProductService {
    
    @Autowired
    private ProductRepository productRepository;
    
    @Autowired
    private RedisTemplate<String, Object> redisTemplate;
    
    @Cacheable(value = "products", key = "'page:' + #page + ':size:' + #size")
    public List<Product> findProducts(int page, int size) {
        Span span = ElasticApm.currentSpan().startSpan("db", "postgresql", "query");
        span.setName("ProductService.findProducts");
        span.addLabel("page", String.valueOf(page));
        span.addLabel("size", String.valueOf(size));
        
        try {
            // Simulate database latency
            Thread.sleep(100 + new Random().nextInt(200));
            
            Pageable pageable = PageRequest.of(page, size);
            Page<Product> productPage = productRepository.findAll(pageable);
            
            span.addLabel("total_elements", String.valueOf(productPage.getTotalElements()));
            span.addLabel("total_pages", String.valueOf(productPage.getTotalPages()));
            
            return productPage.getContent();
        } catch (Exception e) {
            span.captureException(e);
            throw e;
        } finally {
            span.end();
        }
    }
    
    public Optional<Product> findById(Long id) {
        Span span = ElasticApm.currentSpan().startSpan("db", "postgresql", "query");
        span.setName("ProductService.findById");
        span.addLabel("product_id", String.valueOf(id));
        
        try {
            // Check cache first
            Span cacheSpan = span.startSpan("cache", "redis", "get");
            String cacheKey = "product:" + id;
            Product cachedProduct = (Product) redisTemplate.opsForValue().get(cacheKey);
            cacheSpan.addLabel("cache_key", cacheKey);
            cacheSpan.addLabel("cache_hit", String.valueOf(cachedProduct != null));
            cacheSpan.end();
            
            if (cachedProduct != null) {
                span.addLabel("source", "cache");
                return Optional.of(cachedProduct);
            }
            
            // Query database
            Thread.sleep(50 + new Random().nextInt(100));
            Optional<Product> product = productRepository.findById(id);
            
            // Cache result
            if (product.isPresent()) {
                Span cacheSetSpan = span.startSpan("cache", "redis", "set");
                redisTemplate.opsForValue().set(cacheKey, product.get(), 5, TimeUnit.MINUTES);
                cacheSetSpan.addLabel("cache_key", cacheKey);
                cacheSetSpan.addLabel("ttl_minutes", "5");
                cacheSetSpan.end();
                
                span.addLabel("source", "database");
            }
            
            span.addLabel("found", String.valueOf(product.isPresent()));
            return product;
            
        } catch (Exception e) {
            span.captureException(e);
            throw e;
        } finally {
            span.end();
        }
    }
    
    public Product createProduct(CreateProductRequest request) {
        Span span = ElasticApm.currentSpan().startSpan("db", "postgresql", "insert");
        span.setName("ProductService.createProduct");
        span.addLabel("product_name", request.getName());
        span.addLabel("product_category", request.getCategory());
        
        try {
            // Validate product doesn't exist
            Span validationSpan = span.startSpan("validation", "business", "validate");
            boolean exists = productRepository.existsByName(request.getName());
            validationSpan.addLabel("exists", String.valueOf(exists));
            validationSpan.end();
            
            if (exists) {
                throw new ProductAlreadyExistsException("Product with name already exists");
            }
            
            // Create product
            Product product = new Product();
            product.setName(request.getName());
            product.setDescription(request.getDescription());
            product.setPrice(request.getPrice());
            product.setCategory(request.getCategory());
            product.setCreatedAt(LocalDateTime.now());
            
            // Simulate processing time
            Thread.sleep(200 + new Random().nextInt(300));
            
            Product savedProduct = productRepository.save(product);
            span.addLabel("created_product_id", String.valueOf(savedProduct.getId()));
            
            // Invalidate cache
            Span cacheSpan = span.startSpan("cache", "redis", "delete");
            redisTemplate.delete("products:*");
            cacheSpan.addLabel("operation", "invalidate_product_cache");
            cacheSpan.end();
            
            return savedProduct;
            
        } catch (Exception e) {
            span.captureException(e);
            throw e;
        } finally {
            span.end();
        }
    }
    
    public List<Product> searchProducts(String query, int page, int size) {
        Span span = ElasticApm.currentSpan().startSpan("db", "postgresql", "search");
        span.setName("ProductService.searchProducts");
        span.addLabel("search_query", query);
        span.addLabel("page", String.valueOf(page));
        span.addLabel("size", String.valueOf(size));
        
        try {
            // Simulate complex search operation
            Thread.sleep(300 + new Random().nextInt(500));
            
            Pageable pageable = PageRequest.of(page, size);
            List<Product> products = productRepository.findByNameContainingIgnoreCase(query, pageable);
            
            span.addLabel("result_count", String.valueOf(products.size()));
            span.addLabel("search_complexity", query.length() > 10 ? "high" : "low");
            
            return products;
            
        } catch (Exception e) {
            span.captureException(e);
            throw e;
        } finally {
            span.end();
        }
    }
}

@Service
public class MetricsService {
    
    @Autowired
    private MeterRegistry meterRegistry;
    
    private final Counter requestCounter;
    private final Timer responseTimer;
    private final Gauge activeUsers;
    
    public MetricsService(MeterRegistry meterRegistry) {
        this.meterRegistry = meterRegistry;
        this.requestCounter = Counter.builder("api_requests_total")
            .description("Total API requests")
            .register(meterRegistry);
        this.responseTimer = Timer.builder("api_response_time")
            .description("API response time")
            .register(meterRegistry);
        this.activeUsers = Gauge.builder("active_users")
            .description("Active users")
            .register(meterRegistry, this, MetricsService::getActiveUserCount);
    }
    
    public Map<String, Object> getApplicationMetrics() {
        Map<String, Object> metrics = new HashMap<>();
        
        // JVM metrics
        metrics.put("jvm_memory_used", meterRegistry.get("jvm.memory.used").gauge().value());
        metrics.put("jvm_memory_max", meterRegistry.get("jvm.memory.max").gauge().value());
        metrics.put("jvm_threads_live", meterRegistry.get("jvm.threads.live").gauge().value());
        
        // HTTP metrics
        metrics.put("http_requests_total", requestCounter.count());
        metrics.put("http_response_time_avg", responseTimer.mean(TimeUnit.MILLISECONDS));
        
        // Custom metrics
        metrics.put("active_users", getActiveUserCount());
        
        return metrics;
    }
    
    private double getActiveUserCount() {
        // Simulate active user count
        return 100 + new Random().nextInt(50);
    }
}

// Entity and Repository classes
@Entity
@Table(name = "products")
public class Product {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    @Column(nullable = false, unique = true)
    private String name;
    
    @Column(length = 1000)
    private String description;
    
    @Column(nullable = false)
    private BigDecimal price;
    
    @Column(nullable = false)
    private String category;
    
    @Column(nullable = false)
    private LocalDateTime createdAt;
    
    // Getters and setters
    // ... (omitted for brevity)
}

@Repository
public interface ProductRepository extends JpaRepository<Product, Long> {
    boolean existsByName(String name);
    List<Product> findByNameContainingIgnoreCase(String name, Pageable pageable);
    List<Product> findByCategory(String category, Pageable pageable);
}

public class CreateProductRequest {
    private String name;
    private String description;
    private BigDecimal price;
    private String category;
    
    // Getters and setters
    // ... (omitted for brevity)
}

public class ProductAlreadyExistsException extends RuntimeException {
    public ProductAlreadyExistsException(String message) {
        super(message);
    }
}
```

---

## 🌐 Real User Monitoring (RUM)

### 1. Frontend React Integration

```javascript
// sample-frontend/src/index.js
import React from 'react';
import ReactDOM from 'react-dom/client';
import { init as initApm } from '@elastic/apm-rum';
import { ApmRoute } from '@elastic/apm-rum-react';
import App from './App';

// Initialize Elastic APM RUM
const apm = initApm({
  serviceName: process.env.REACT_APP_APM_SERVICE_NAME || 'sample-frontend',
  serverUrl: process.env.REACT_APP_APM_SERVER_URL || 'http://localhost:8200',
  serviceVersion: process.env.REACT_APP_APM_SERVICE_VERSION || '1.0.0',
  environment: process.env.REACT_APP_APM_ENVIRONMENT || 'development',
  
  // Distributed tracing
  distributedTracingOrigins: [
    'http://localhost:8080',
    'http://sample-api:8080'
  ],
  
  // User context
  context: {
    tags: {
      region: 'us-west-1',
      datacenter: 'aws'
    }
  },
  
  // Performance monitoring
  transactionSampleRate: 1.0,
  
  // Error handling
  capturePageLoadSpan: true,
  ignoreTransactions: [
    // Ignore health check requests
    '/health',
    '/favicon.ico'
  ],
  
  // Custom configuration
  breakdownMetrics: true,
  logLevel: 'info'
});

// Set user context
apm.setUserContext({
  id: 'user123',
  username: 'john.doe',
  email: 'john.doe@example.com'
});

// Set custom context
apm.setCustomContext({
  version: '1.0.0',
  feature_flags: {
    new_checkout: true,
    beta_dashboard: false
  }
});

const root = ReactDOM.createRoot(document.getElementById('root'));
root.render(
  <React.StrictMode>
    <App />
  </React.StrictMode>
);
```

```javascript
// sample-frontend/src/App.js
import React, { useState, useEffect } from 'react';
import { BrowserRouter as Router, Routes, Route, Link } from 'react-router-dom';
import { ApmRoute } from '@elastic/apm-rum-react';
import { apm } from '@elastic/apm-rum';
import ProductList from './components/ProductList';
import ProductDetail from './components/ProductDetail';
import CreateProduct from './components/CreateProduct';
import SearchProducts from './components/SearchProducts';
import './App.css';

function App() {
  const [user, setUser] = useState(null);
  
  useEffect(() => {
    // Simulate user login
    const mockUser = {
      id: 'user123',
      name: 'John Doe',
      email: 'john.doe@example.com'
    };
    
    setUser(mockUser);
    
    // Update APM user context
    apm.setUserContext({
      id: mockUser.id,
      username: mockUser.name,
      email: mockUser.email
    });
    
    // Track page load performance
    const transaction = apm.startTransaction('app-load', 'page-load');
    transaction.addLabels({
      page: 'home',
      user_id: mockUser.id
    });
    
    setTimeout(() => {
      transaction.end();
    }, 100);
    
  }, []);
  
  const handleError = (error, errorInfo) => {
    // Capture React errors in APM
    apm.captureError(error, {
      tags: {
        component: 'App',
        source: 'react_error_boundary'
      },
      context: {
        extra: errorInfo
      }
    });
  };
  
  return (
    <Router>
      <div className="App">
        <nav className="navbar">
          <div className="nav-brand">
            <Link to="/">Product Store</Link>
          </div>
          <div className="nav-links">
            <Link to="/">Home</Link>
            <Link to="/search">Search</Link>
            <Link to="/create">Create Product</Link>
            {user && <span>Welcome, {user.name}</span>}
          </div>
        </nav>
        
        <main className="main-content">
          <Routes>
            <Route 
              path="/" 
              element={
                <ApmRoute name="home">
                  <ProductList />
                </ApmRoute>
              } 
            />
            <Route 
              path="/products/:id" 
              element={
                <ApmRoute name="product-detail">
                  <ProductDetail />
                </ApmRoute>
              } 
            />
            <Route 
              path="/search" 
              element={
                <ApmRoute name="search">
                  <SearchProducts />
                </ApmRoute>
              } 
            />
            <Route 
              path="/create" 
              element={
                <ApmRoute name="create-product">
                  <CreateProduct />
                </ApmRoute>
              } 
            />
          </Routes>
        </main>
      </div>
    </Router>
  );
}

export default App;
```

### 2. Custom RUM Instrumentation

```javascript
// sample-frontend/src/services/apiService.js
import { apm } from '@elastic/apm-rum';

const API_BASE_URL = process.env.REACT_APP_API_URL || 'http://localhost:8080';

class ApiService {
  constructor() {
    this.baseURL = API_BASE_URL;
  }
  
  async request(method, url, data = null, options = {}) {
    const fullUrl = `${this.baseURL}${url}`;
    
    // Start APM span
    const span = apm.startSpan(`${method} ${url}`, 'external.http');
    if (span) {
      span.addLabels({
        'http.method': method,
        'http.url': fullUrl,
        'service.target.type': 'http',
        'service.target.name': 'sample-api'
      });
    }
    
    try {
      const startTime = performance.now();
      
      const config = {
        method,
        headers: {
          'Content-Type': 'application/json',
          ...options.headers
        },
        ...options
      };
      
      if (data) {
        config.body = JSON.stringify(data);
      }
      
      const response = await fetch(fullUrl, config);
      const endTime = performance.now();
      const duration = endTime - startTime;
      
      if (span) {
        span.addLabels({
          'http.status_code': response.status,
          'http.response.status_code': response.status,
          'response.time.ms': Math.round(duration)
        });
        
        if (!response.ok) {
          span.addLabels({
            'error': true,
            'error.type': 'http_error'
          });
        }
      }
      
      // Custom metrics
      apm.addTransaction('api_call', {
        method,
        url,
        status: response.status,
        duration
      });
      
      if (!response.ok) {
        const error = new Error(`HTTP ${response.status}: ${response.statusText}`);
        error.status = response.status;
        error.url = fullUrl;
        throw error;
      }
      
      const result = await response.json();
      return result;
      
    } catch (error) {
      if (span) {
        span.addLabels({
          'error': true,
          'error.type': error.name || 'network_error',
          'error.message': error.message
        });
      }
      
      // Capture error in APM
      apm.captureError(error, {
        tags: {
          method,
          url,
          service: 'api_service'
        },
        context: {
          http: {
            method,
            url: fullUrl
          }
        }
      });
      
      throw error;
    } finally {
      if (span) {
        span.end();
      }
    }
  }
  
  // Product API methods
  async getProducts(page = 0, size = 10) {
    return this.request('GET', `/api/products?page=${page}&size=${size}`);
  }
  
  async getProduct(id) {
    return this.request('GET', `/api/products/${id}`);
  }
  
  async createProduct(productData) {
    return this.request('POST', '/api/products', productData);
  }
  
  async searchProducts(query, page = 0, size = 10) {
    return this.request('GET', `/api/products/search?query=${encodeURIComponent(query)}&page=${page}&size=${size}`);
  }
  
  // Test endpoints for APM demonstration
  async triggerSlowRequest() {
    return this.request('GET', '/api/slow-endpoint');
  }
  
  async triggerErrorRequest() {
    return this.request('GET', '/api/error-endpoint');
  }
}

export default new ApiService();
```

### 3. Performance Monitoring Components

```javascript
// sample-frontend/src/components/ProductList.js
import React, { useState, useEffect } from 'react';
import { Link } from 'react-router-dom';
import { apm } from '@elastic/apm-rum';
import ApiService from '../services/apiService';
import LoadingSpinner from './LoadingSpinner';

const ProductList = () => {
  const [products, setProducts] = useState([]);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);
  const [page, setPage] = useState(0);
  const [totalPages, setTotalPages] = useState(0);
  
  useEffect(() => {
    loadProducts();
  }, [page]);
  
  const loadProducts = async () => {
    const transaction = apm.startTransaction('load-products', 'user-interaction');
    transaction.addLabels({
      page: page,
      component: 'ProductList'
    });
    
    try {
      setLoading(true);
      setError(null);
      
      const startTime = performance.now();
      const response = await ApiService.getProducts(page, 10);
      const endTime = performance.now();
      
      setProducts(response.content || response);
      setTotalPages(response.totalPages || 1);
      
      transaction.addLabels({
        products_count: response.content ? response.content.length : response.length,
        load_time_ms: Math.round(endTime - startTime),
        success: true
      });
      
      // Track custom metrics
      apm.addTransaction('products_loaded', {
        count: response.content ? response.content.length : response.length,
        page: page,
        loadTime: endTime - startTime
      });
      
    } catch (err) {
      setError(err.message);
      transaction.addLabels({
        error: true,
        error_message: err.message
      });
      
      apm.captureError(err, {
        tags: {
          component: 'ProductList',
          operation: 'load_products',
          page: page
        }
      });
    } finally {
      setLoading(false);
      transaction.end();
    }
  };
  
  const handleProductClick = (productId) => {
    // Track user interaction
    apm.addTransaction('product_click', {
      product_id: productId,
      source: 'product_list'
    });
  };
  
  const handlePageChange = (newPage) => {
    setPage(newPage);
    
    // Track pagination
    apm.addTransaction('pagination', {
      from_page: page,
      to_page: newPage,
      component: 'ProductList'
    });
  };
  
  if (loading) return <LoadingSpinner />;
  if (error) return <div className="error">Error: {error}</div>;
  
  return (
    <div className="product-list">
      <h1>Products</h1>
      
      <div className="products-grid">
        {products.map(product => (
          <div key={product.id} className="product-card">
            <Link 
              to={`/products/${product.id}`}
              onClick={() => handleProductClick(product.id)}
            >
              <h3>{product.name}</h3>
              <p>{product.description}</p>
              <p className="price">${product.price}</p>
              <p className="category">{product.category}</p>
            </Link>
          </div>
        ))}
      </div>
      
      {totalPages > 1 && (
        <div className="pagination">
          <button 
            disabled={page === 0}
            onClick={() => handlePageChange(page - 1)}
          >
            Previous
          </button>
          <span>Page {page + 1} of {totalPages}</span>
          <button 
            disabled={page >= totalPages - 1}
            onClick={() => handlePageChange(page + 1)}
          >
            Next
          </button>
        </div>
      )}
      
      {/* Test buttons for APM demonstration */}
      <div className="test-buttons">
        <button onClick={async () => {
          try {
            await ApiService.triggerSlowRequest();
            alert('Slow request completed');
          } catch (error) {
            alert('Slow request failed: ' + error.message);
          }
        }}>
          Test Slow Request
        </button>
        
        <button onClick={async () => {
          try {
            await ApiService.triggerErrorRequest();
            alert('Error request succeeded');
          } catch (error) {
            alert('Error request failed: ' + error.message);
          }
        }}>
          Test Error Request
        </button>
      </div>
    </div>
  );
};

export default ProductList;
```

---

## 📊 Synthetic Monitoring

### 1. Heartbeat Configuration

```yaml
# heartbeat/config/heartbeat.yml
heartbeat.config.monitors:
  path: ${path.config}/monitors.d/*.yml
  reload.enabled: false
  reload.period: 5s

heartbeat.monitors:
# HTTP monitor for API health
- type: http
  id: api-health-check
  name: "API Health Check"
  urls: ["http://sample-api:8080/api/health"]
  schedule: '@every 30s'
  timeout: 10s
  check.request:
    method: GET
    headers:
      'User-Agent': 'Heartbeat-Monitor'
  check.response:
    status: 200
    body:
      - '"status":"UP"'
  fields:
    service: sample-api
    environment: development
  tags: ["api", "health", "critical"]

# HTTP monitor for API endpoints
- type: http
  id: api-products-list
  name: "API Products List"
  urls: ["http://sample-api:8080/api/products?page=0&size=5"]
  schedule: '@every 1m'
  timeout: 15s
  check.request:
    method: GET
    headers:
      'Content-Type': 'application/json'
  check.response:
    status: 200
    json:
      - description: "Response contains products array"
        condition:
          contains:
            content: []
  fields:
    service: sample-api
    endpoint: products-list
  tags: ["api", "products", "functional"]

# HTTP monitor for frontend
- type: http
  id: frontend-home
  name: "Frontend Home Page"
  urls: ["http://sample-frontend:80/"]
  schedule: '@every 1m'
  timeout: 10s
  check.request:
    method: GET
    headers:
      'User-Agent': 'Heartbeat-Frontend-Monitor'
  check.response:
    status: 200
    body:
      - '<title>Product Store</title>'
  fields:
    service: sample-frontend
    page: home
  tags: ["frontend", "ui", "user-facing"]

# TCP monitor for database
- type: tcp
  id: postgres-connection
  name: "PostgreSQL Connection"
  hosts: ["postgres:5432"]
  schedule: '@every 30s'
  timeout: 5s
  fields:
    service: postgres
    type: database
  tags: ["database", "infrastructure", "critical"]

# TCP monitor for Redis
- type: tcp
  id: redis-connection
  name: "Redis Connection"
  hosts: ["redis:6379"]
  schedule: '@every 30s'
  timeout: 5s
  fields:
    service: redis
    type: cache
  tags: ["cache", "infrastructure"]

# ICMP monitor for network connectivity
- type: icmp
  id: network-connectivity
  name: "Network Connectivity"
  hosts: ["8.8.8.8", "1.1.1.1"]
  schedule: '@every 5m'
  timeout: 3s
  fields:
    monitor_type: network
  tags: ["network", "connectivity", "external"]

# Browser monitor for user journeys
- type: browser
  id: user-journey-create-product
  name: "User Journey - Create Product"
  schedule: '@every 5m'
  source:
    inline:
      script: |
        step("Navigate to home page", async () => {
          await page.goto('http://sample-frontend:80/');
          await page.waitForSelector('.product-list');
        });
        
        step("Navigate to create product", async () => {
          await page.click('a[href="/create"]');
          await page.waitForSelector('.create-product-form');
        });
        
        step("Fill product form", async () => {
          await page.fill('input[name="name"]', 'Test Product');
          await page.fill('textarea[name="description"]', 'Test product description');
          await page.fill('input[name="price"]', '99.99');
          await page.selectOption('select[name="category"]', 'Electronics');
        });
        
        step("Submit form", async () => {
          await page.click('button[type="submit"]');
          await page.waitForSelector('.success-message');
        });
  fields:
    service: sample-frontend
    journey: create-product
  tags: ["browser", "user-journey", "e2e"]

# Custom scripted monitor
- type: http
  id: api-performance-test
  name: "API Performance Test"
  urls: ["http://sample-api:8080/api/products/search?query=test"]
  schedule: '@every 2m'
  timeout: 20s
  check.request:
    method: GET
  check.response:
    status: 200
  processors:
    - add_fields:
        target: monitor
        fields:
          performance_test: true
    - script:
        lang: javascript
        source: >
          function(event) {
            var responseTime = event.Get("monitor.duration.us") / 1000;
            if (responseTime > 2000) {
              event.Put("monitor.status", "degraded");
              event.Put("alert.severity", "warning");
            } else if (responseTime > 5000) {
              event.Put("monitor.status", "critical");
              event.Put("alert.severity", "critical");
            }
            event.Put("performance.response_time_ms", responseTime);
          }
  tags: ["performance", "api", "search"]

output.elasticsearch:
  hosts: ["elasticsearch:9200"]

setup.kibana:
  host: "kibana:5601"

setup.template.enabled: true
setup.dashboards.enabled: true

logging.level: info
logging.to_files: true
logging.files:
  path: /var/log/heartbeat
  name: heartbeat
  keepfiles: 7
  permissions: 0600

processors:
  - add_host_metadata:
      when.not.contains.tags: forwarded
  - add_docker_metadata: ~

xpack.monitoring.enabled: true
xpack.monitoring.elasticsearch.hosts: ["elasticsearch:9200"]
```

---

## 📈 APM Dashboards and Alerting

### 1. Custom Kibana Dashboard

```json
{
  "version": "8.11.0",
  "objects": [
    {
      "id": "apm-custom-dashboard",
      "type": "dashboard",
      "attributes": {
        "title": "Custom APM Dashboard",
        "hits": 0,
        "description": "Custom dashboard for application performance monitoring",
        "panelsJSON": "[{\"version\":\"8.11.0\",\"gridData\":{\"x\":0,\"y\":0,\"w\":24,\"h\":15,\"i\":\"1\"},\"panelIndex\":\"1\",\"embeddableConfig\":{\"enhancements\":{}},\"panelRefName\":\"panel_1\"},{\"version\":\"8.11.0\",\"gridData\":{\"x\":24,\"y\":0,\"w\":24,\"h\":15,\"i\":\"2\"},\"panelIndex\":\"2\",\"embeddableConfig\":{\"enhancements\":{}},\"panelRefName\":\"panel_2\"},{\"version\":\"8.11.0\",\"gridData\":{\"x\":0,\"y\":15,\"w\":48,\"h\":15,\"i\":\"3\"},\"panelIndex\":\"3\",\"embeddableConfig\":{\"enhancements\":{}},\"panelRefName\":\"panel_3\"}]",
        "timeRestore": false,
        "kibanaSavedObjectMeta": {
          "searchSourceJSON": "{\"query\":{\"query\":\"\",\"language\":\"kuery\"},\"filter\":[]}"
        }
      },
      "references": [
        {
          "name": "panel_1",
          "type": "visualization",
          "id": "apm-throughput-chart"
        },
        {
          "name": "panel_2", 
          "type": "visualization",
          "id": "apm-latency-chart"
        },
        {
          "name": "panel_3",
          "type": "visualization", 
          "id": "apm-error-rate-chart"
        }
      ]
    },
    {
      "id": "apm-throughput-chart",
      "type": "visualization",
      "attributes": {
        "title": "APM Throughput",
        "visState": "{\"title\":\"APM Throughput\",\"type\":\"line\",\"aggs\":[{\"id\":\"1\",\"type\":\"count\",\"params\":{}},{\"id\":\"2\",\"type\":\"date_histogram\",\"params\":{\"field\":\"@timestamp\",\"interval\":\"1m\",\"min_doc_count\":1}},{\"id\":\"3\",\"type\":\"terms\",\"params\":{\"field\":\"service.name\",\"size\":10}}]}",
        "uiStateJSON": "{}",
        "description": "",
        "kibanaSavedObjectMeta": {
          "searchSourceJSON": "{\"index\":\"apm-*\",\"query\":{\"match\":{\"processor.event\":\"transaction\"}},\"filter\":[]}"
        }
      }
    },
    {
      "id": "apm-latency-chart",
      "type": "visualization", 
      "attributes": {
        "title": "APM Latency P95",
        "visState": "{\"title\":\"APM Latency P95\",\"type\":\"line\",\"aggs\":[{\"id\":\"1\",\"type\":\"percentiles\",\"params\":{\"field\":\"transaction.duration.us\",\"percents\":[95]}},{\"id\":\"2\",\"type\":\"date_histogram\",\"params\":{\"field\":\"@timestamp\",\"interval\":\"1m\"}},{\"id\":\"3\",\"type\":\"terms\",\"params\":{\"field\":\"service.name\",\"size\":10}}]}",
        "uiStateJSON": "{}",
        "description": "",
        "kibanaSavedObjectMeta": {
          "searchSourceJSON": "{\"index\":\"apm-*\",\"query\":{\"match\":{\"processor.event\":\"transaction\"}},\"filter\":[]}"
        }
      }
    },
    {
      "id": "apm-error-rate-chart",
      "type": "visualization",
      "attributes": {
        "title": "APM Error Rate",
        "visState": "{\"title\":\"APM Error Rate\",\"type\":\"line\",\"aggs\":[{\"id\":\"1\",\"type\":\"count\",\"params\":{}},{\"id\":\"2\",\"type\":\"date_histogram\",\"params\":{\"field\":\"@timestamp\",\"interval\":\"1m\"}},{\"id\":\"3\",\"type\":\"filters\",\"params\":{\"filters\":[{\"input\":{\"query\":{\"match\":{\"transaction.result\":\"HTTP 5xx\"}}},\"label\":\"5xx Errors\"},{\"input\":{\"query\":{\"match\":{\"transaction.result\":\"HTTP 4xx\"}}},\"label\":\"4xx Errors\"}]}}]}",
        "uiStateJSON": "{}",
        "description": "",
        "kibanaSavedObjectMeta": {
          "searchSourceJSON": "{\"index\":\"apm-*\",\"query\":{\"match\":{\"processor.event\":\"transaction\"}},\"filter\":[]}"
        }
      }
    }
  ]
}
```

### 2. Watcher Alerts Configuration

```json
{
  "trigger": {
    "schedule": {
      "interval": "1m"
    }
  },
  "input": {
    "search": {
      "request": {
        "search_type": "query_then_fetch",
        "indices": ["apm-*"],
        "body": {
          "size": 0,
          "query": {
            "bool": {
              "filter": [
                {
                  "term": {
                    "processor.event": "transaction"
                  }
                },
                {
                  "range": {
                    "@timestamp": {
                      "gte": "now-5m"
                    }
                  }
                }
              ]
            }
          },
          "aggs": {
            "services": {
              "terms": {
                "field": "service.name",
                "size": 10
              },
              "aggs": {
                "avg_duration": {
                  "avg": {
                    "field": "transaction.duration.us"
                  }
                },
                "error_rate": {
                  "filter": {
                    "bool": {
                      "should": [
                        {
                          "range": {
                            "http.response.status_code": {
                              "gte": 400
                            }
                          }
                        },
                        {
                          "exists": {
                            "field": "error.id"
                          }
                        }
                      ]
                    }
                  }
                },
                "total_requests": {
                  "value_count": {
                    "field": "transaction.id"
                  }
                }
              }
            }
          }
        }
      }
    }
  },
  "condition": {
    "script": {
      "source": "def alerts = []; for (service in ctx.payload.aggregations.services.buckets) { def errorRate = service.error_rate.doc_count / service.total_requests.value * 100; def avgDuration = service.avg_duration.value / 1000; if (errorRate > 5 || avgDuration > 2000) { alerts.add(['service': service.key, 'error_rate': errorRate, 'avg_duration': avgDuration]); } } ctx.metadata.alerts = alerts; return alerts.size() > 0;"
    }
  },
  "actions": {
    "send_slack": {
      "webhook": {
        "scheme": "https",
        "host": "hooks.slack.com",
        "port": 443,
        "method": "post",
        "path": "/services/YOUR/SLACK/WEBHOOK",
        "params": {},
        "headers": {
          "Content-Type": "application/json"
        },
        "body": "{ \"text\": \"🚨 APM Alert: Performance issues detected\", \"attachments\": [ { \"color\": \"danger\", \"fields\": [ { \"title\": \"Services Affected\", \"value\": \"{{#ctx.metadata.alerts}}{{service}} (Error Rate: {{error_rate}}%, Avg Duration: {{avg_duration}}ms)\\n{{/ctx.metadata.alerts}}\", \"short\": false } ] } ] }"
      }
    },
    "create_jira_ticket": {
      "webhook": {
        "scheme": "https",
        "host": "your-company.atlassian.net",
        "port": 443,
        "method": "post",
        "path": "/rest/api/2/issue",
        "params": {},
        "headers": {
          "Content-Type": "application/json",
          "Authorization": "Basic YOUR_JIRA_AUTH"
        },
        "body": "{ \"fields\": { \"project\": { \"key\": \"PERF\" }, \"summary\": \"APM Performance Alert - {{ctx.execution_time}}\", \"description\": \"Performance issues detected:\\n{{#ctx.metadata.alerts}}Service: {{service}}\\nError Rate: {{error_rate}}%\\nAvg Duration: {{avg_duration}}ms\\n\\n{{/ctx.metadata.alerts}}\", \"issuetype\": { \"name\": \"Bug\" }, \"priority\": { \"name\": \"High\" } } }"
      }
    }
  },
  "metadata": {
    "name": "APM Performance Alert",
    "description": "Alert when services have high error rates or slow response times"
  }
}
```

---

## 📋 Ejercicios Prácticos

### Ejercicio 1: Configurar Elastic APM Stack

**Objetivo**: Desplegar y configurar stack completo de APM.

**Tareas**:

1. Desplegar Elasticsearch, Kibana, APM Server
2. Configurar Metricbeat y Filebeat
3. Instrumentar aplicación Java con Elastic APM
4. Configurar RUM para frontend React

### Ejercicio 2: Synthetic Monitoring

**Objetivo**: Implementar monitoreo sintético.

**Componentes**:

- Heartbeat configuration
- HTTP monitors para APIs
- Browser monitors para user journeys
- Alertas basadas en uptime

### Ejercicio 3: Performance Optimization

**Objetivo**: Usar APM para optimizar rendimiento.

**Tareas**:

- Identificar bottlenecks usando traces
- Analizar métricas de base de datos
- Optimizar consultas lentas
- Validar mejoras con APM

---

## 🧪 Laboratorio

### Lab 1: Complete APM Setup

1. **Desplegar infraestructura**:

   ```bash
   docker-compose -f docker-compose.apm.yml up -d
   ```

2. **Configurar instrumentación**:
   - Backend con Elastic APM agent
   - Frontend con RUM
   - Infrastructure monitoring

3. **Crear dashboards**:
   - Performance overview
   - Error tracking
   - User experience metrics

4. **Configurar alertas**:
   - High latency alerts
   - Error rate alerts
   - Synthetic monitoring alerts

### Lab 2: Performance Analysis

1. **Generate load** en aplicación
2. **Identificar problemas** usando APM
3. **Correlacionar** logs, métricas y traces
4. **Optimizar** basado en insights

---

## 📖 Best Practices

### 1. APM Implementation

```yaml
apm_best_practices:
  instrumentation:
    - "Instrumentar en todos los niveles de aplicación"
    - "Capturar contexto de negocio relevante"
    - "Evitar over-instrumentation que afecte performance"
  
  monitoring:
    - "Establecer baselines de performance"
    - "Monitorear user experience, no solo infraestructura"
    - "Correlacionar todos los tipos de telemetría"
  
  alerting:
    - "Alertar en métricas de negocio, no solo técnicas"
    - "Usar SLIs/SLOs para definir alertas"
    - "Implementar alertas proactivas y reactivas"
```

### 2. Performance Optimization

```yaml
optimization_approach:
  measurement:
    - "Medir antes de optimizar"
    - "Establecer métricas claras de éxito"
    - "Monitorear continuamente después de cambios"
  
  prioritization:
    - "Enfocar en bottlenecks que afectan usuarios"
    - "Optimizar operaciones más frecuentes primero"
    - "Considerar costo vs beneficio de optimizaciones"
```

---

## ✅ Resumen del Módulo

En este módulo has aprendido:

1. **Elastic APM Stack** - Configuración completa de APM
2. **Instrumentación** - Backend y frontend monitoring
3. **Real User Monitoring** - Experiencia de usuario real
4. **Synthetic Monitoring** - Pruebas automatizadas
5. **Performance Analysis** - Identificación de bottlenecks
6. **Dashboards** - Visualización de métricas APM
7. **Alerting** - Alertas proactivas de performance
8. **Best practices** - Principios de APM efectivo

¡Has completado con éxito la implementación de APM! El siguiente módulo cubrirá health checks y circuit breakers para completar tu stack de observabilidad.
