# Módulo 13: Laboratorio Final y Evaluación - Fase 5

## 📋 Introducción

Este módulo final consolida todos los conocimientos adquiridos en la Fase 5: Monitoring, Observability, and Security. Implementarás un proyecto completo que integra monitoreo, observabilidad, seguridad, trazado distribuido, APM, health checks y circuit breakers en un entorno de producción simulado.

### Objetivos del Laboratorio Final

- **Integrar** todas las herramientas de monitoreo y observabilidad
- **Implementar** stack completo de seguridad DevSecOps
- **Configurar** pipelines CI/CD con security scanning
- **Establecer** SLIs/SLOs y alertas proactivas
- **Demostrar** expertise en troubleshooting y optimización
- **Evaluar** competencias en arquitectura de observabilidad

### Arquitectura del Proyecto Final

El proyecto simula una plataforma de e-commerce con:

- **Frontend**: React con RUM y synthetic monitoring
- **Backend**: Microservicios Spring Boot con APM
- **Databases**: PostgreSQL y Redis con monitoreo
- **Message Queue**: RabbitMQ con distributed tracing
- **Search**: Elasticsearch con APM integration
- **Security**: Vault, vulnerability scanning, compliance
- **Observability**: Prometheus, Grafana, Jaeger, ELK Stack

---

## 🎯 Objetivos de Evaluación

Al completar este laboratorio, habrás demostrado capacidad para:

1. **Diseñar** arquitectura completa de observabilidad
2. **Implementar** monitoring end-to-end
3. **Configurar** security scanning y compliance
4. **Establecer** SLIs/SLOs efectivos
5. **Crear** dashboards ejecutivos y técnicos
6. **Resolver** incidentes usando observabilidad
7. **Optimizar** performance basado en métricas
8. **Automatizar** respuestas a alertas

---

## 🏗️ Proyecto Final: E-Commerce Observability Platform

### 1. Arquitectura Completa

```yaml
# docker-compose.final-lab.yml
version: '3.8'

networks:
  monitoring:
    driver: bridge
  application:
    driver: bridge
  security:
    driver: bridge
  database:
    driver: bridge

volumes:
  prometheus_data:
  grafana_data:
  elasticsearch_data:
  postgres_data:
  redis_data:
  vault_data:
  jaeger_data:

services:
  # =============================================================================
  # MONITORING STACK
  # =============================================================================
  
  # Prometheus - Metrics collection
  prometheus:
    image: prom/prometheus:v2.47.0
    container_name: prometheus
    ports:
      - "9090:9090"
    volumes:
      - ./monitoring/prometheus:/etc/prometheus
      - prometheus_data:/prometheus
    command:
      - '--config.file=/etc/prometheus/prometheus.yml'
      - '--storage.tsdb.path=/prometheus'
      - '--web.console.libraries=/etc/prometheus/console_libraries'
      - '--web.console.templates=/etc/prometheus/consoles'
      - '--storage.tsdb.retention.time=30d'
      - '--web.enable-lifecycle'
      - '--web.enable-admin-api'
    networks:
      - monitoring
      - application
    restart: unless-stopped
    healthcheck:
      test: ["CMD", "wget", "-q", "--spider", "http://localhost:9090/-/healthy"]
      interval: 30s
      timeout: 10s
      retries: 3

  # Grafana - Visualization
  grafana:
    image: grafana/grafana:10.1.0
    container_name: grafana
    ports:
      - "3000:3000"
    volumes:
      - grafana_data:/var/lib/grafana
      - ./monitoring/grafana/provisioning:/etc/grafana/provisioning
      - ./monitoring/grafana/dashboards:/var/lib/grafana/dashboards
    environment:
      - GF_SECURITY_ADMIN_PASSWORD=admin123
      - GF_USERS_ALLOW_SIGN_UP=false
      - GF_INSTALL_PLUGINS=grafana-piechart-panel,grafana-worldmap-panel,grafana-clock-panel
      - GF_FEATURE_TOGGLES_ENABLE=publicDashboards
    networks:
      - monitoring
    restart: unless-stopped
    depends_on:
      - prometheus

  # AlertManager - Alert routing
  alertmanager:
    image: prom/alertmanager:v0.26.0
    container_name: alertmanager
    ports:
      - "9093:9093"
    volumes:
      - ./monitoring/alertmanager:/etc/alertmanager
    command:
      - '--config.file=/etc/alertmanager/alertmanager.yml'
      - '--storage.path=/alertmanager'
      - '--web.external-url=http://localhost:9093'
    networks:
      - monitoring
    restart: unless-stopped

  # Node Exporter - Host metrics
  node-exporter:
    image: prom/node-exporter:v1.6.1
    container_name: node-exporter
    ports:
      - "9100:9100"
    volumes:
      - /proc:/host/proc:ro
      - /sys:/host/sys:ro
      - /:/rootfs:ro
    command:
      - '--path.procfs=/host/proc'
      - '--path.rootfs=/rootfs'
      - '--path.sysfs=/host/sys'
      - '--collector.filesystem.mount-points-exclude=^/(sys|proc|dev|host|etc)($$|/)'
    networks:
      - monitoring
    restart: unless-stopped

# =============================================================================
  # OBSERVABILITY STACK
  # =============================================================================

  # Elasticsearch - Logs and APM
  elasticsearch:
    image: elasticsearch:8.11.0
    container_name: elasticsearch
    environment:
      - discovery.type=single-node
      - xpack.security.enabled=false
      - "ES_JAVA_OPTS=-Xms2g -Xmx2g"
      - xpack.monitoring.collection.enabled=true
    volumes:
      - elasticsearch_data:/usr/share/elasticsearch/data
    ports:
      - "9200:9200"
    networks:
      - monitoring
      - application
    restart: unless-stopped
    healthcheck:
      test: ["CMD-SHELL", "curl -f http://localhost:9200/_cluster/health || exit 1"]
      interval: 30s
      timeout: 10s
      retries: 5

  # Kibana - Log analysis
  kibana:
    image: kibana:8.11.0
    container_name: kibana
    ports:
      - "5601:5601"
    environment:
      - ELASTICSEARCH_HOSTS=http://elasticsearch:9200
      - XPACK_MONITORING_ENABLED=true
      - XPACK_APM_ENABLED=true
    networks:
      - monitoring
    depends_on:
      elasticsearch:
        condition: service_healthy
    restart: unless-stopped

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
    volumes:
      - ./monitoring/apm-server/apm-server.yml:/usr/share/apm-server/apm-server.yml:ro
    networks:
      - monitoring
      - application
    depends_on:
      elasticsearch:
        condition: service_healthy
    restart: unless-stopped

  # Jaeger - Distributed tracing
  jaeger:
    image: jaegertracing/all-in-one:1.50
    container_name: jaeger
    ports:
      - "16686:16686"
      - "14268:14268"
      - "6831:6831/udp"
    environment:
      - COLLECTOR_OTLP_ENABLED=true
      - SPAN_STORAGE_TYPE=elasticsearch
      - ES_SERVER_URLS=http://elasticsearch:9200
    networks:
      - monitoring
      - application
    depends_on:
      - elasticsearch
    restart: unless-stopped

  # Filebeat - Log shipper
  filebeat:
    image: elastic/filebeat:8.11.0
    container_name: filebeat
    user: root
    volumes:
      - ./monitoring/filebeat/filebeat.yml:/usr/share/filebeat/filebeat.yml:ro
      - /var/lib/docker/containers:/var/lib/docker/containers:ro
      - /var/run/docker.sock:/var/run/docker.sock:ro
    environment:
      - ELASTICSEARCH_HOST=elasticsearch:9200
    networks:
      - monitoring
    depends_on:
      - elasticsearch
    restart: unless-stopped

  # =============================================================================
  # SECURITY STACK
  # =============================================================================

  # HashiCorp Vault - Secrets management
  vault:
    image: vault:1.15.0
    container_name: vault
    ports:
      - "8201:8200"
    volumes:
      - vault_data:/vault/data
      - ./security/vault:/vault/config
    environment:
      - VAULT_DEV_ROOT_TOKEN_ID=dev-root-token
      - VAULT_DEV_LISTEN_ADDRESS=0.0.0.0:8200
    cap_add:
      - IPC_LOCK
    networks:
      - security
      - application
    restart: unless-stopped

  # SonarQube - Code quality
  sonarqube:
    image: sonarqube:10.2-community
    container_name: sonarqube
    ports:
      - "9000:9000"
    environment:
      - SONAR_JDBC_URL=jdbc:postgresql://postgres:5432/sonarqube
      - SONAR_JDBC_USERNAME=sonar
      - SONAR_JDBC_PASSWORD=sonar123
    volumes:
      - ./security/sonarqube/data:/opt/sonarqube/data
      - ./security/sonarqube/logs:/opt/sonarqube/logs
    networks:
      - security
      - database
    depends_on:
      - postgres
    restart: unless-stopped

  # OWASP ZAP - Security scanning
  zap:
    image: owasp/zap2docker-stable:2.14.0
    container_name: zap
    ports:
      - "8090:8090"
    command: zap-webswing.sh
    networks:
      - security
      - application
    restart: unless-stopped

  # =============================================================================
  # APPLICATION STACK
  # =============================================================================

  # PostgreSQL - Primary database
  postgres:
    image: postgres:15
    container_name: postgres
    environment:
      - POSTGRES_DB=ecommerce
      - POSTGRES_USER=postgres
      - POSTGRES_PASSWORD=postgres123
      - POSTGRES_MULTIPLE_DATABASES=sonarqube,vault
    volumes:
      - postgres_data:/var/lib/postgresql/data
      - ./database/init-scripts:/docker-entrypoint-initdb.d
    ports:
      - "5432:5432"
    networks:
      - database
      - application
    restart: unless-stopped
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U postgres"]
      interval: 30s
      timeout: 10s
      retries: 5

  # Redis - Cache
  redis:
    image: redis:7-alpine
    container_name: redis
    ports:
      - "6379:6379"
    volumes:
      - redis_data:/data
    command: redis-server --appendonly yes
    networks:
      - application
    restart: unless-stopped
    healthcheck:
      test: ["CMD", "redis-cli", "ping"]
      interval: 30s
      timeout: 10s
      retries: 3

  # RabbitMQ - Message broker
  rabbitmq:
    image: rabbitmq:3.12-management
    container_name: rabbitmq
    ports:
      - "5672:5672"
      - "15672:15672"
    environment:
      - RABBITMQ_DEFAULT_USER=rabbitmq
      - RABBITMQ_DEFAULT_PASS=rabbitmq123
    volumes:
      - ./messaging/rabbitmq:/etc/rabbitmq
    networks:
      - application
    restart: unless-stopped
    healthcheck:
      test: ["CMD", "rabbitmq-diagnostics", "ping"]
      interval: 30s
      timeout: 10s
      retries: 3

  # =============================================================================
  # MICROSERVICES
  # =============================================================================

  # Product Service
  product-service:
    build:
      context: ./services/product-service
      dockerfile: Dockerfile
    container_name: product-service
    ports:
      - "8081:8080"
    environment:
      - SPRING_PROFILES_ACTIVE=docker
      - DATABASE_URL=jdbc:postgresql://postgres:5432/ecommerce
      - REDIS_URL=redis://redis:6379
      - ELASTIC_APM_SERVER_URL=http://apm-server:8200
      - ELASTIC_APM_SERVICE_NAME=product-service
    networks:
      - application
      - database
      - monitoring
    depends_on:
      postgres:
        condition: service_healthy
      redis:
        condition: service_healthy
    restart: unless-stopped
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:8080/actuator/health"]
      interval: 30s
      timeout: 10s
      retries: 3

  # User Service
  user-service:
    build:
      context: ./services/user-service
      dockerfile: Dockerfile
    container_name: user-service
    ports:
      - "8082:8080"
    environment:
      - SPRING_PROFILES_ACTIVE=docker
      - DATABASE_URL=jdbc:postgresql://postgres:5432/ecommerce
      - VAULT_URL=http://vault:8200
      - VAULT_TOKEN=dev-root-token
      - ELASTIC_APM_SERVER_URL=http://apm-server:8200
      - ELASTIC_APM_SERVICE_NAME=user-service
    networks:
      - application
      - database
      - security
      - monitoring
    depends_on:
      - postgres
      - vault
    restart: unless-stopped
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:8080/actuator/health"]
      interval: 30s
      timeout: 10s
      retries: 3

  # Order Service
  order-service:
    build:
      context: ./services/order-service
      dockerfile: Dockerfile
    container_name: order-service
    ports:
      - "8083:8080"
    environment:
      - SPRING_PROFILES_ACTIVE=docker
      - DATABASE_URL=jdbc:postgresql://postgres:5432/ecommerce
      - RABBITMQ_URL=amqp://rabbitmq:rabbitmq123@rabbitmq:5672
      - PRODUCT_SERVICE_URL=http://product-service:8080
      - USER_SERVICE_URL=http://user-service:8080
      - ELASTIC_APM_SERVER_URL=http://apm-server:8200
      - ELASTIC_APM_SERVICE_NAME=order-service
    networks:
      - application
      - database
      - monitoring
    depends_on:
      - postgres
      - rabbitmq
      - product-service
      - user-service
    restart: unless-stopped
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:8080/actuator/health"]
      interval: 30s
      timeout: 10s
      retries: 3

  # API Gateway
  api-gateway:
    build:
      context: ./services/api-gateway
      dockerfile: Dockerfile
    container_name: api-gateway
    ports:
      - "8080:8080"
    environment:
      - SPRING_PROFILES_ACTIVE=docker
      - PRODUCT_SERVICE_URL=http://product-service:8080
      - USER_SERVICE_URL=http://user-service:8080
      - ORDER_SERVICE_URL=http://order-service:8080
      - ELASTIC_APM_SERVER_URL=http://apm-server:8200
      - ELASTIC_APM_SERVICE_NAME=api-gateway
    networks:
      - application
      - monitoring
    depends_on:
      - product-service
      - user-service
      - order-service
    restart: unless-stopped
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:8080/actuator/health"]
      interval: 30s
      timeout: 10s
      retries: 3

  # Frontend
  frontend:
    build:
      context: ./frontend
      dockerfile: Dockerfile
    container_name: frontend
    ports:
      - "3001:80"
    environment:
      - REACT_APP_API_URL=http://localhost:8080
      - REACT_APP_APM_SERVER_URL=http://localhost:8200
      - REACT_APP_APM_SERVICE_NAME=ecommerce-frontend
    networks:
      - application
      - monitoring
    depends_on:
      - api-gateway
    restart: unless-stopped

  # Load Generator - For testing
  load-generator:
    build:
      context: ./testing/load-generator
      dockerfile: Dockerfile
    container_name: load-generator
    environment:
      - TARGET_URL=http://api-gateway:8080
      - USERS=50
      - SPAWN_RATE=10
      - RUN_TIME=300s
    networks:
      - application
    depends_on:
      - api-gateway
    profiles:
      - testing
```

### 2. Monitoreo y Alertas Comprehensivas

```yaml
# monitoring/prometheus/prometheus.yml
global:
  scrape_interval: 15s
  evaluation_interval: 15s
  external_labels:
    cluster: 'ecommerce-final-lab'
    environment: 'production-simulation'

rule_files:
  - "rules/*.yml"

alerting:
  alertmanagers:
    - static_configs:
        - targets:
          - alertmanager:9093

scrape_configs:
  # Prometheus self-monitoring
  - job_name: 'prometheus'
    static_configs:
      - targets: ['localhost:9090']
    scrape_interval: 15s
    metrics_path: /metrics

  # Node Exporter - Infrastructure metrics
  - job_name: 'node-exporter'
    static_configs:
      - targets: ['node-exporter:9100']
    scrape_interval: 15s

  # Application services
  - job_name: 'api-gateway'
    static_configs:
      - targets: ['api-gateway:8080']
    metrics_path: /actuator/prometheus
    scrape_interval: 15s

  - job_name: 'product-service'
    static_configs:
      - targets: ['product-service:8080']
    metrics_path: /actuator/prometheus
    scrape_interval: 15s

  - job_name: 'user-service'
    static_configs:
      - targets: ['user-service:8080']
    metrics_path: /actuator/prometheus
    scrape_interval: 15s

  - job_name: 'order-service'
    static_configs:
      - targets: ['order-service:8080']
    metrics_path: /actuator/prometheus
    scrape_interval: 15s

  # Database monitoring
  - job_name: 'postgres-exporter'
    static_configs:
      - targets: ['postgres-exporter:9187']
    scrape_interval: 30s

  - job_name: 'redis-exporter'
    static_configs:
      - targets: ['redis-exporter:9121']
    scrape_interval: 30s

  # Message queue monitoring
  - job_name: 'rabbitmq'
    static_configs:
      - targets: ['rabbitmq:15692']
    scrape_interval: 30s

  # Elasticsearch monitoring
  - job_name: 'elasticsearch'
    static_configs:
      - targets: ['elasticsearch:9200']
    metrics_path: /_prometheus/metrics
    scrape_interval: 30s

  # Blackbox monitoring for synthetic checks
  - job_name: 'blackbox-http'
    metrics_path: /probe
    params:
      module: [http_2xx]
    static_configs:
      - targets:
        - http://api-gateway:8080/actuator/health
        - http://product-service:8080/actuator/health
        - http://user-service:8080/actuator/health
        - http://order-service:8080/actuator/health
        - http://frontend:80
    relabel_configs:
      - source_labels: [__address__]
        target_label: __param_target
      - source_labels: [__param_target]
        target_label: instance
      - target_label: __address__
        replacement: blackbox-exporter:9115

  # Custom business metrics
  - job_name: 'business-metrics'
    static_configs:
      - targets: ['api-gateway:8080']
    metrics_path: /actuator/prometheus
    scrape_interval: 30s
    params:
      match[]:
        - '{__name__=~"business_.*"}'
        - '{__name__=~"orders_.*"}'
        - '{__name__=~"revenue_.*"}'
        - '{__name__=~"user_.*"}'
```

### 3. Alertas de Producción

```yaml
# monitoring/prometheus/rules/application-alerts.yml
groups:
- name: ecommerce-application-alerts
  rules:
  # SLI/SLO Alerts
  - alert: HighErrorRate
    expr: |
      (
        sum(rate(http_requests_total{status=~"5.."}[5m])) by (service)
        /
        sum(rate(http_requests_total[5m])) by (service)
      ) * 100 > 1
    for: 2m
    labels:
      severity: critical
      category: availability
      slo: "99.9% availability"
    annotations:
      summary: "High error rate detected for {{ $labels.service }}"
      description: "Error rate is {{ $value | humanizePercentage }} for service {{ $labels.service }}"
      runbook_url: "https://runbooks.company.com/high-error-rate"

  - alert: HighLatency
    expr: |
      histogram_quantile(0.95, 
        sum(rate(http_request_duration_seconds_bucket[5m])) by (le, service)
      ) > 0.5
    for: 3m
    labels:
      severity: warning
      category: performance
      slo: "95th percentile < 500ms"
    annotations:
      summary: "High latency detected for {{ $labels.service }}"
      description: "95th percentile latency is {{ $value | humanizeDuration }} for {{ $labels.service }}"

  - alert: LowThroughput
    expr: |
      sum(rate(http_requests_total[5m])) by (service) < 10
    for: 5m
    labels:
      severity: warning
      category: performance
    annotations:
      summary: "Low throughput for {{ $labels.service }}"
      description: "Request rate is only {{ $value }} req/s for {{ $labels.service }}"

  # Business Metrics Alerts
  - alert: OrderProcessingDelayed
    expr: |
      avg_over_time(order_processing_duration_seconds[10m]) > 300
    for: 5m
    labels:
      severity: critical
      category: business
    annotations:
      summary: "Order processing is taking too long"
      description: "Average order processing time is {{ $value | humanizeDuration }}"

  - alert: LowRevenue
    expr: |
      increase(revenue_total[1h]) < 1000
    for: 30m
    labels:
      severity: warning
      category: business
    annotations:
      summary: "Revenue is below expected threshold"
      description: "Hourly revenue is only ${{ $value }}, expected > $1000"

  - alert: HighUserChurn
    expr: |
      rate(user_logout_total[5m]) / rate(user_login_total[5m]) > 0.3
    for: 10m
    labels:
      severity: warning
      category: business
    annotations:
      summary: "High user churn rate detected"
      description: "{{ $value | humanizePercentage }} of users are logging out quickly"

  # Infrastructure Alerts
  - alert: DatabaseConnectionsHigh
    expr: |
      pg_stat_activity_count > 80
    for: 5m
    labels:
      severity: warning
      category: database
    annotations:
      summary: "High number of database connections"
      description: "{{ $value }} active connections to PostgreSQL"

  - alert: RedisMemoryHigh
    expr: |
      redis_memory_used_bytes / redis_memory_max_bytes > 0.9
    for: 5m
    labels:
      severity: critical
      category: cache
    annotations:
      summary: "Redis memory usage is critical"
      description: "Redis memory usage is {{ $value | humanizePercentage }}"

  - alert: RabbitMQQueueBacklog
    expr: |
      rabbitmq_queue_messages > 1000
    for: 5m
    labels:
      severity: warning
      category: messaging
    annotations:
      summary: "RabbitMQ queue has high backlog"
      description: "Queue {{ $labels.queue }} has {{ $value }} messages"

  # Security Alerts
  - alert: UnauthorizedAccessAttempts
    expr: |
      increase(http_requests_total{status="401"}[5m]) > 10
    for: 2m
    labels:
      severity: warning
      category: security
    annotations:
      summary: "High number of unauthorized access attempts"
      description: "{{ $value }} unauthorized requests in the last 5 minutes"

  - alert: SuspiciousActivity
    expr: |
      increase(http_requests_total{status="403"}[5m]) > 20
    for: 1m
    labels:
      severity: critical
      category: security
    annotations:
      summary: "Suspicious activity detected"
      description: "{{ $value }} forbidden requests indicate potential attack"

- name: ecommerce-infrastructure-alerts
  rules:
  # System Resource Alerts
  - alert: HighCPUUsage
    expr: |
      100 - (avg by (instance) (irate(node_cpu_seconds_total{mode="idle"}[5m])) * 100) > 80
    for: 5m
    labels:
      severity: warning
      category: infrastructure
    annotations:
      summary: "High CPU usage on {{ $labels.instance }}"
      description: "CPU usage is {{ $value | humanizePercentage }}"

  - alert: HighMemoryUsage
    expr: |
      (1 - (node_memory_MemAvailable_bytes / node_memory_MemTotal_bytes)) * 100 > 85
    for: 5m
    labels:
      severity: warning
      category: infrastructure
    annotations:
      summary: "High memory usage on {{ $labels.instance }}"
      description: "Memory usage is {{ $value | humanizePercentage }}"

  - alert: DiskSpaceLow
    expr: |
      (1 - (node_filesystem_avail_bytes / node_filesystem_size_bytes)) * 100 > 80
    for: 5m
    labels:
      severity: warning
      category: infrastructure
    annotations:
      summary: "Low disk space on {{ $labels.instance }}"
      description: "Disk usage is {{ $value | humanizePercentage }} on {{ $labels.mountpoint }}"

  - alert: ServiceDown
    expr: up == 0
    for: 1m
    labels:
      severity: critical
      category: availability
    annotations:
      summary: "Service {{ $labels.job }} is down"
      description: "Service {{ $labels.job }} on {{ $labels.instance }} has been down for more than 1 minute"
```

### 4. Dashboard Ejecutivo

```json
{
  "dashboard": {
    "id": null,
    "title": "E-Commerce Executive Dashboard",
    "tags": ["executive", "business", "kpi"],
    "timezone": "browser",
    "refresh": "30s",
    "time": {
      "from": "now-24h",
      "to": "now"
    },
    "panels": [
      {
        "id": 1,
        "title": "Business KPIs",
        "type": "stat",
        "targets": [
          {
            "expr": "increase(orders_total[24h])",
            "legendFormat": "Orders (24h)",
            "refId": "A"
          },
          {
            "expr": "increase(revenue_total[24h])",
            "legendFormat": "Revenue (24h)",
            "refId": "B"
          },
          {
            "expr": "avg_over_time(active_users[24h])",
            "legendFormat": "Avg Active Users",
            "refId": "C"
          },
          {
            "expr": "avg_over_time(conversion_rate[24h]) * 100",
            "legendFormat": "Conversion Rate %",
            "refId": "D"
          }
        ],
        "fieldConfig": {
          "defaults": {
            "unit": "short",
            "custom": {
              "displayMode": "basic"
            }
          },
          "overrides": [
            {
              "matcher": {"id": "byName", "options": "Revenue (24h)"},
              "properties": [{"id": "unit", "value": "currencyUSD"}]
            },
            {
              "matcher": {"id": "byName", "options": "Conversion Rate %"},
              "properties": [{"id": "unit", "value": "percent"}]
            }
          ]
        },
        "gridPos": {"h": 8, "w": 24, "x": 0, "y": 0}
      },
      {
        "id": 2,
        "title": "System Health Overview",
        "type": "stat",
        "targets": [
          {
            "expr": "sum(up)",
            "legendFormat": "Services UP",
            "refId": "A"
          },
          {
            "expr": "sum(1 - up)",
            "legendFormat": "Services DOWN",
            "refId": "B"
          },
          {
            "expr": "avg(rate(http_requests_total{status=~\"2..\"}[5m])) * 100",
            "legendFormat": "Success Rate %",
            "refId": "C"
          },
          {
            "expr": "histogram_quantile(0.95, sum(rate(http_request_duration_seconds_bucket[5m])) by (le)) * 1000",
            "legendFormat": "P95 Latency (ms)",
            "refId": "D"
          }
        ],
        "fieldConfig": {
          "defaults": {
            "custom": {"displayMode": "basic"}
          },
          "overrides": [
            {
              "matcher": {"id": "byName", "options": "Services DOWN"},
              "properties": [
                {"id": "color", "value": {"mode": "fixed", "fixedColor": "red"}},
                {"id": "custom.displayMode", "value": "color-background"}
              ]
            },
            {
              "matcher": {"id": "byName", "options": "Success Rate %"},
              "properties": [{"id": "unit", "value": "percent"}]
            }
          ]
        },
        "gridPos": {"h": 8, "w": 24, "x": 0, "y": 8}
      },
      {
        "id": 3,
        "title": "Revenue Trend",
        "type": "timeseries",
        "targets": [
          {
            "expr": "increase(revenue_total[1h])",
            "legendFormat": "Hourly Revenue",
            "refId": "A"
          },
          {
            "expr": "avg_over_time(revenue_total[24h])",
            "legendFormat": "24h Average",
            "refId": "B"
          }
        ],
        "fieldConfig": {
          "defaults": {
            "unit": "currencyUSD",
            "custom": {
              "drawStyle": "line",
              "lineInterpolation": "smooth",
              "fillOpacity": 20
            }
          }
        },
        "gridPos": {"h": 8, "w": 12, "x": 0, "y": 16}
      },
      {
        "id": 4,
        "title": "User Activity",
        "type": "timeseries",
        "targets": [
          {
            "expr": "rate(user_login_total[5m]) * 60",
            "legendFormat": "Logins/min",
            "refId": "A"
          },
          {
            "expr": "rate(user_logout_total[5m]) * 60",
            "legendFormat": "Logouts/min",
            "refId": "B"
          },
          {
            "expr": "active_users",
            "legendFormat": "Active Users",
            "refId": "C"
          }
        ],
        "gridPos": {"h": 8, "w": 12, "x": 12, "y": 16}
      },
      {
        "id": 5,
        "title": "Service Performance Matrix",
        "type": "heatmap",
        "targets": [
          {
            "expr": "histogram_quantile_over_time(0.95, http_request_duration_seconds_bucket[5m])",
            "legendFormat": "{{service}}",
            "refId": "A"
          }
        ],
        "gridPos": {"h": 8, "w": 24, "x": 0, "y": 24}
      },
      {
        "id": 6,
        "title": "Alert Summary",
        "type": "table",
        "targets": [
          {
            "expr": "ALERTS{alertstate=\"firing\"}",
            "format": "table",
            "refId": "A"
          }
        ],
        "fieldConfig": {
          "overrides": [
            {
              "matcher": {"id": "byName", "options": "severity"},
              "properties": [
                {
                  "id": "custom.displayMode",
                  "value": "color-background"
                },
                {
                  "id": "mappings",
                  "value": [
                    {"options": {"critical": {"color": "red", "text": "CRITICAL"}}},
                    {"options": {"warning": {"color": "yellow", "text": "WARNING"}}},
                    {"options": {"info": {"color": "blue", "text": "INFO"}}}
                  ]
                }
              ]
            }
          ]
        },
        "gridPos": {"h": 8, "w": 24, "x": 0, "y": 32}
      }
    ]
  }
}
```

---

## 🔬 Ejercicios de Evaluación

### Ejercicio 1: Incident Response Simulation

**Escenario**: Una de las bases de datos está experimentando alta latencia.

**Tareas**:

1. **Detectar** el problema usando dashboards
2. **Investigar** la causa raíz con distributed tracing
3. **Correlacionar** logs, métricas y traces
4. **Implementar** fix temporal usando circuit breakers
5. **Documentar** el incidente y lessons learned

**Criterios de Evaluación**:

- Tiempo de detección (objetivo: < 2 minutos)
- Precisión del diagnóstico
- Efectividad de la solución temporal
- Calidad de la documentación

### Ejercicio 2: Performance Optimization

**Escenario**: El tiempo de respuesta del API Gateway está degradándose.

**Tareas**:

1. **Analizar** métricas de performance
2. **Identificar** servicios con mayor impacto
3. **Usar APM** para encontrar bottlenecks
4. **Implementar** optimizaciones
5. **Validar** mejoras con A/B testing

**Entregables**:

- Dashboard de performance analysis
- Reporte de bottlenecks identificados
- Plan de optimización implementado
- Métricas before/after

### Ejercicio 3: Security Compliance Assessment

**Escenario**: Preparar el sistema para auditoría de seguridad.

**Tareas**:

1. **Ejecutar** vulnerability scanning completo
2. **Revisar** compliance con estándares (PCI-DSS)
3. **Configurar** alertas de seguridad
4. **Implementar** data loss prevention
5. **Generar** reporte de compliance

**Entregables**:

- Vulnerability assessment report
- Compliance checklist completado
- Security dashboard configurado
- Action plan para remediation

### Ejercicio 4: Business Intelligence Dashboard

**Escenario**: El equipo ejecutivo necesita visibilidad de métricas de negocio.

**Tareas**:

1. **Definir** KPIs críticos del negocio
2. **Implementar** custom metrics collection
3. **Crear** executive dashboard
4. **Configurar** business alerts
5. **Establecer** SLOs para business metrics

**Entregables**:

- Executive dashboard funcional
- Business KPIs documentados
- SLOs establecidos y monitoreados
- Alertas de negocio configuradas

---

## 🎯 Proyecto Capstone: Chaos Engineering

### Implementación de Chaos Engineering

```python
# chaos-engineering/chaos_monkey.py
import random
import time
import docker
import requests
import logging
from datetime import datetime, timedelta
from typing import List, Dict, Any

class ChaosMonkey:
    def __init__(self, config: dict):
        self.config = config
        self.docker_client = docker.from_env()
        self.logger = self._setup_logging()
        self.experiments = []
        
    def _setup_logging(self):
        logging.basicConfig(
            level=logging.INFO,
            format='%(asctime)s - %(name)s - %(levelname)s - %(message)s',
            handlers=[
                logging.FileHandler('chaos_experiments.log'),
                logging.StreamHandler()
            ]
        )
        return logging.getLogger('ChaosMonkey')
    
    def run_experiment(self, experiment_type: str, duration: int = 300):
        """Run a chaos engineering experiment"""
        experiment_id = f"{experiment_type}_{int(time.time())}"
        self.logger.info(f"Starting chaos experiment: {experiment_id}")
        
        experiment_data = {
            'id': experiment_id,
            'type': experiment_type,
            'start_time': datetime.now(),
            'duration': duration,
            'status': 'running'
        }
        
        try:
            if experiment_type == 'service_failure':
                self._simulate_service_failure(duration)
            elif experiment_type == 'network_latency':
                self._simulate_network_latency(duration)
            elif experiment_type == 'resource_exhaustion':
                self._simulate_resource_exhaustion(duration)
            elif experiment_type == 'database_slowdown':
                self._simulate_database_slowdown(duration)
            
            experiment_data['status'] = 'completed'
            self.logger.info(f"Chaos experiment {experiment_id} completed successfully")
            
        except Exception as e:
            experiment_data['status'] = 'failed'
            experiment_data['error'] = str(e)
            self.logger.error(f"Chaos experiment {experiment_id} failed: {e}")
        
        finally:
            experiment_data['end_time'] = datetime.now()
            self.experiments.append(experiment_data)
    
    def _simulate_service_failure(self, duration: int):
        """Stop a random service for specified duration"""
        services = ['product-service', 'user-service', 'order-service']
        target_service = random.choice(services)
        
        self.logger.info(f"Stopping service: {target_service}")
        
        try:
            container = self.docker_client.containers.get(target_service)
            container.stop()
            
            # Wait for specified duration
            time.sleep(duration)
            
            self.logger.info(f"Restarting service: {target_service}")
            container.start()
            
            # Wait for service to be healthy
            self._wait_for_service_health(target_service)
            
        except docker.errors.NotFound:
            self.logger.error(f"Service {target_service} not found")
    
    def _simulate_network_latency(self, duration: int):
        """Introduce network latency using tc (traffic control)"""
        services = ['product-service', 'user-service', 'order-service']
        target_service = random.choice(services)
        latency_ms = random.randint(500, 2000)
        
        self.logger.info(f"Adding {latency_ms}ms latency to {target_service}")
        
        try:
            container = self.docker_client.containers.get(target_service)
            
            # Add latency
            container.exec_run(
                f"tc qdisc add dev eth0 root netem delay {latency_ms}ms 100ms distribution normal",
                privileged=True
            )
            
            time.sleep(duration)
            
            # Remove latency
            container.exec_run("tc qdisc del dev eth0 root", privileged=True)
            self.logger.info(f"Removed latency from {target_service}")
            
        except Exception as e:
            self.logger.error(f"Failed to simulate network latency: {e}")
    
    def _simulate_resource_exhaustion(self, duration: int):
        """Consume CPU/Memory resources"""
        services = ['product-service', 'user-service', 'order-service']
        target_service = random.choice(services)
        
        self.logger.info(f"Exhausting resources on {target_service}")
        
        try:
            container = self.docker_client.containers.get(target_service)
            
            # Start resource-intensive process
            stress_cmd = "stress --cpu 2 --vm 1 --vm-bytes 256M --timeout {}s".format(duration)
            container.exec_run(stress_cmd, detach=True)
            
            self.logger.info(f"Resource exhaustion completed on {target_service}")
            
        except Exception as e:
            self.logger.error(f"Failed to exhaust resources: {e}")
    
    def _simulate_database_slowdown(self, duration: int):
        """Simulate database performance issues"""
        self.logger.info("Simulating database slowdown")
        
        try:
            # Simulate slow queries by creating artificial load
            postgres_container = self.docker_client.containers.get('postgres')
            
            slow_query = """
            SELECT pg_sleep(0.5), * FROM (
                SELECT generate_series(1, 1000) as id,
                       md5(random()::text) as data
            ) t;
            """
            
            # Run multiple slow queries
            for _ in range(duration // 30):  # Run every 30 seconds
                postgres_container.exec_run(
                    f'psql -U postgres -d ecommerce -c "{slow_query}"',
                    detach=True
                )
                time.sleep(30)
            
            self.logger.info("Database slowdown simulation completed")
            
        except Exception as e:
            self.logger.error(f"Failed to simulate database slowdown: {e}")
    
    def _wait_for_service_health(self, service_name: str, timeout: int = 120):
        """Wait for service to become healthy"""
        start_time = time.time()
        health_url = f"http://localhost:{self._get_service_port(service_name)}/actuator/health"
        
        while time.time() - start_time < timeout:
            try:
                response = requests.get(health_url, timeout=5)
                if response.status_code == 200:
                    self.logger.info(f"Service {service_name} is healthy")
                    return True
            except requests.exceptions.RequestException:
                pass
            
            time.sleep(5)
        
        self.logger.warning(f"Service {service_name} did not become healthy within {timeout}s")
        return False
    
    def _get_service_port(self, service_name: str) -> str:
        """Get external port for service"""
        port_mapping = {
            'product-service': '8081',
            'user-service': '8082',
            'order-service': '8083',
            'api-gateway': '8080'
        }
        return port_mapping.get(service_name, '8080')
    
    def generate_experiment_report(self) -> Dict[str, Any]:
        """Generate comprehensive experiment report"""
        if not self.experiments:
            return {'message': 'No experiments conducted'}
        
        report = {
            'summary': {
                'total_experiments': len(self.experiments),
                'successful': len([e for e in self.experiments if e['status'] == 'completed']),
                'failed': len([e for e in self.experiments if e['status'] == 'failed']),
                'total_duration': sum([e.get('duration', 0) for e in self.experiments])
            },
            'experiments': self.experiments,
            'recommendations': self._generate_recommendations()
        }
        
        return report
    
    def _generate_recommendations(self) -> List[str]:
        """Generate improvement recommendations based on experiments"""
        recommendations = []
        
        failed_experiments = [e for e in self.experiments if e['status'] == 'failed']
        if failed_experiments:
            recommendations.append("Review failed experiments and improve error handling")
        
        service_failures = [e for e in self.experiments if e['type'] == 'service_failure']
        if service_failures:
            recommendations.append("Implement proper circuit breakers and fallback mechanisms")
        
        latency_experiments = [e for e in self.experiments if e['type'] == 'network_latency']
        if latency_experiments:
            recommendations.append("Consider implementing retry policies with exponential backoff")
        
        recommendations.append("Ensure all services have proper health checks configured")
        recommendations.append("Implement graceful degradation for non-critical features")
        
        return recommendations

# Usage example
if __name__ == "__main__":
    config = {
        'prometheus_url': 'http://localhost:9090',
        'grafana_url': 'http://localhost:3000',
        'experiment_duration': 300
    }
    
    chaos_monkey = ChaosMonkey(config)
    
    # Run different types of experiments
    experiments = [
        'service_failure',
        'network_latency',
        'resource_exhaustion',
        'database_slowdown'
    ]
    
    for experiment in experiments:
        chaos_monkey.run_experiment(experiment, duration=180)
        time.sleep(60)  # Wait between experiments
    
    # Generate report
    report = chaos_monkey.generate_experiment_report()
    print(json.dumps(report, indent=2, default=str))
```

---

## 📊 Evaluación Final

### Criterios de Evaluación

| Categoría | Peso | Criterios de Evaluación |
|-----------|------|------------------------|
| **Arquitectura de Observabilidad** | 25% | - Diseño completo de monitoring stack<br>- Integración entre herramientas<br>- Escalabilidad y performance |
| **Implementación Técnica** | 25% | - Calidad del código<br>- Configuraciones apropiadas<br>- Best practices aplicadas |
| **SLIs/SLOs y Alerting** | 20% | - SLIs bien definidos<br>- SLOs realistas<br>- Alertas efectivas y no ruidosas |
| **Security y Compliance** | 15% | - Vulnerability scanning implementado<br>- Compliance monitoring<br>- Security best practices |
| **Troubleshooting** | 10% | - Capacidad de diagnóstico<br>- Uso efectivo de herramientas<br>- Time to resolution |
| **Documentación** | 5% | - Runbooks claros<br>- Arquitectura documentada<br>- Procedures definidos |

### Métricas de Éxito

**SLIs Definidos**:

- API Availability: 99.9%
- Response Time P95: < 500ms
- Error Rate: < 1%
- Security Scan Coverage: 100%

**SLOs Establecidos**:

- Monthly uptime: 99.9%
- Incident response time: < 15 minutes
- Vulnerability remediation: < 7 days
- Alert noise ratio: < 5%

### Entregables Finales

1. **Arquitectura Completa** - Documentación y diagramas
2. **Stack Funcionando** - Todos los servicios operativos
3. **Dashboards Configurados** - Ejecutivo y técnico
4. **Alertas Configuradas** - Con runbooks asociados
5. **Security Compliance** - Reportes y evidencias
6. **Chaos Engineering** - Experimentos y reportes
7. **Incident Response Plan** - Procedures documentados
8. **Performance Optimization** - Before/after analysis

---

## 🏆 Certificación y Próximos Pasos

### Certificación de Competencias

Al completar exitosamente este laboratorio final, habrás demostrado dominio en:

✅ **Monitoring y Observabilidad**

- Prometheus, Grafana, Elasticsearch, Kibana
- Distributed tracing con Jaeger
- APM con Elastic Stack
- Custom metrics y business KPIs

✅ **Security y DevSecOps**

- Vulnerability scanning y management
- Secrets management con Vault
- Compliance monitoring y reporting
- Security automation en CI/CD

✅ **Reliability Engineering**

- SLIs/SLOs implementation
- Circuit breakers y resilience patterns
- Health checks y service discovery
- Chaos engineering practices

✅ **Performance Optimization**

- APM-driven optimization
- Resource monitoring y alerting
- Capacity planning
- Performance testing automation

### Camino de Aprendizaje Continuo

**Especialización Avanzada**:

1. **Site Reliability Engineering** (SRE)
2. **Platform Engineering**
3. **Cloud-Native Security**
4. **FinOps y Cost Optimization**

**Certificaciones Recomendadas**:

- Certified Kubernetes Administrator (CKA)
- AWS/Azure/GCP Professional certifications
- Prometheus Certified Associate
- Certified Ethical Hacker (CEH)

**Tecnologías Emergentes**:

- OpenTelemetry ecosystem
- eBPF-based observability
- AI/ML for AIOps
- Service mesh observability

---

## ✅ Resumen de la Fase 5

¡Felicitaciones! Has completado exitosamente la **Fase 5: Monitoring, Observability, and Security**. Durante esta fase has dominado:

### Módulos Completados

1. **Prometheus y Grafana** - Fundamentos de monitoreo
2. **Elasticsearch y Stack ELK** - Gestión centralizada de logs
3. **Métricas y Alertas** - SLIs/SLOs y alerting estratégico
4. **Logging y Análisis** - Structured logging y correlation
5. **Alertmanager** - Gestión avanzada de alertas
6. **DevSecOps y Seguridad** - Security-first development
7. **Secret Management** - HashiCorp Vault y AWS Secrets Manager
8. **Auditorías y Cumplimiento** - SOX, PCI-DSS, GDPR compliance
9. **Vulnerability Scanning** - SAST, DAST, SCA automation
10. **Distributed Tracing** - Jaeger y OpenTelemetry
11. **APM** - Application Performance Monitoring
12. **Health Checks y Circuit Breakers** - Resilience patterns
13. **Laboratorio Final** - Integración completa y evaluación

### Skills Adquiridos

🎯 **Arquitectura de Observabilidad Completa**
🔒 **Security-First Approach**
📊 **Data-Driven Decision Making**
🚨 **Proactive Incident Management**
⚡ **Performance Optimization**
🔄 **Continuous Improvement**

¡Estás listo para liderar iniciativas de observabilidad y seguridad en organizaciones de cualquier escala!

**¿Continuamos con la siguiente fase del learning path?** 🚀
