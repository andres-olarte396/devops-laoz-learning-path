# Monitoreo y Observabilidad en DevOps

## Introducción

El **monitoreo y observabilidad** son componentes críticos en DevOps que permiten entender el comportamiento de las aplicaciones y la infraestructura en tiempo real, identificar problemas antes de que afecten a los usuarios y tomar decisiones basadas en datos.

## ¿Qué es Monitoreo vs Observabilidad?

### Monitoreo

El **monitoreo** es la práctica de recopilar, procesar y actuar sobre métricas conocidas para entender el estado de un sistema.

**Características:**
- Se basa en métricas predefinidas
- Responde preguntas conocidas
- Detecta problemas conocidos
- Reactivo por naturaleza

### Observabilidad

La **observabilidad** es la capacidad de entender el estado interno de un sistema basándose en sus salidas externas.

**Características:**
- Permite explorar datos no predefinidos
- Responde preguntas desconocidas
- Ayuda a descubrir problemas nuevos
- Proactivo y exploratorio

## Los Tres Pilares de la Observabilidad

### 1. **Métricas (Metrics)**

Son medidas numéricas recolectadas a lo largo del tiempo.

#### **Tipos de Métricas**

```yaml
# Ejemplo de métricas de aplicación
application_metrics:
  # Métricas de negocio
  - user_registrations_total
  - orders_completed_total
  - revenue_dollars_total
  
  # Métricas de rendimiento
  - request_duration_seconds
  - requests_per_second
  - error_rate_percentage
  
  # Métricas de infraestructura
  - cpu_usage_percentage
  - memory_usage_bytes
  - disk_io_operations_total
```

#### **Implementación con Prometheus**

```python
# app.py - Instrumentación con Prometheus
from flask import Flask, request
from prometheus_client import Counter, Histogram, Gauge, generate_latest
import time

app = Flask(__name__)

# Definir métricas
REQUEST_COUNT = Counter(
    'http_requests_total',
    'Total HTTP requests',
    ['method', 'endpoint', 'status']
)

REQUEST_DURATION = Histogram(
    'http_request_duration_seconds',
    'HTTP request duration in seconds',
    ['method', 'endpoint']
)

ACTIVE_USERS = Gauge(
    'active_users_total',
    'Number of active users'
)

@app.before_request
def before_request():
    request.start_time = time.time()

@app.after_request
def after_request(response):
    # Registrar métricas
    REQUEST_COUNT.labels(
        method=request.method,
        endpoint=request.endpoint or 'unknown',
        status=response.status_code
    ).inc()
    
    duration = time.time() - request.start_time
    REQUEST_DURATION.labels(
        method=request.method,
        endpoint=request.endpoint or 'unknown'
    ).observe(duration)
    
    return response

@app.route('/metrics')
def metrics():
    return generate_latest()

@app.route('/api/users')
def get_users():
    # Lógica de la aplicación
    return {"users": []}

if __name__ == '__main__':
    app.run(debug=True)
```

### 2. **Logs (Registros)**

Son registros de eventos que ocurrieron en el sistema.

#### **Structured Logging**

```python
# logging_config.py
import logging
import json
from datetime import datetime

class StructuredLogger:
    def __init__(self, name):
        self.logger = logging.getLogger(name)
        self.logger.setLevel(logging.INFO)
        
        # Configurar handler
        handler = logging.StreamHandler()
        handler.setFormatter(self.JSONFormatter())
        self.logger.addHandler(handler)
    
    class JSONFormatter(logging.Formatter):
        def format(self, record):
            log_entry = {
                'timestamp': datetime.utcnow().isoformat(),
                'level': record.levelname,
                'logger': record.name,
                'message': record.getMessage(),
                'module': record.module,
                'function': record.funcName,
                'line': record.lineno
            }
            
            # Agregar campos extra
            if hasattr(record, 'user_id'):
                log_entry['user_id'] = record.user_id
            if hasattr(record, 'request_id'):
                log_entry['request_id'] = record.request_id
                
            return json.dumps(log_entry)
    
    def info(self, message, **kwargs):
        extra = kwargs
        self.logger.info(message, extra=extra)
    
    def error(self, message, **kwargs):
        extra = kwargs
        self.logger.error(message, extra=extra)
    
    def warning(self, message, **kwargs):
        extra = kwargs
        self.logger.warning(message, extra=extra)

# Uso del logger estructurado
logger = StructuredLogger(__name__)

def process_order(order_id, user_id):
    logger.info(
        "Processing order",
        user_id=user_id,
        order_id=order_id,
        action="order_processing_started"
    )
    
    try:
        # Procesar orden
        result = handle_order(order_id)
        
        logger.info(
            "Order processed successfully",
            user_id=user_id,
            order_id=order_id,
            action="order_processing_completed",
            result=result
        )
        
    except Exception as e:
        logger.error(
            "Failed to process order",
            user_id=user_id,
            order_id=order_id,
            action="order_processing_failed",
            error=str(e)
        )
        raise
```

#### **Configuración de ELK Stack**

```yaml
# docker-compose.yml - ELK Stack
version: '3.8'

services:
  elasticsearch:
    image: docker.elastic.co/elasticsearch/elasticsearch:8.8.0
    environment:
      - discovery.type=single-node
      - "ES_JAVA_OPTS=-Xms512m -Xmx512m"
      - xpack.security.enabled=false
    ports:
      - "9200:9200"
    volumes:
      - elasticsearch_data:/usr/share/elasticsearch/data

  logstash:
    image: docker.elastic.co/logstash/logstash:8.8.0
    ports:
      - "5044:5044"
      - "9600:9600"
    volumes:
      - ./logstash/pipeline:/usr/share/logstash/pipeline
      - ./logstash/config:/usr/share/logstash/config
    depends_on:
      - elasticsearch

  kibana:
    image: docker.elastic.co/kibana/kibana:8.8.0
    ports:
      - "5601:5601"
    environment:
      - ELASTICSEARCH_HOSTS=http://elasticsearch:9200
    depends_on:
      - elasticsearch

  filebeat:
    image: docker.elastic.co/beats/filebeat:8.8.0
    user: root
    volumes:
      - ./filebeat/filebeat.yml:/usr/share/filebeat/filebeat.yml:ro
      - /var/lib/docker/containers:/var/lib/docker/containers:ro
      - /var/run/docker.sock:/var/run/docker.sock:ro
    depends_on:
      - logstash

volumes:
  elasticsearch_data:
```

### 3. **Traces (Trazas)**

Son registros de la ejecución de una request a través de múltiples servicios.

#### **Distributed Tracing con OpenTelemetry**

```python
# tracing.py
from opentelemetry import trace
from opentelemetry.exporter.jaeger.thrift import JaegerExporter
from opentelemetry.sdk.trace import TracerProvider
from opentelemetry.sdk.trace.export import BatchSpanProcessor
from opentelemetry.instrumentation.flask import FlaskInstrumentor
from opentelemetry.instrumentation.requests import RequestsInstrumentor
import requests

# Configurar trazado
trace.set_tracer_provider(TracerProvider())
tracer = trace.get_tracer(__name__)

# Configurar exportador Jaeger
jaeger_exporter = JaegerExporter(
    agent_host_name="localhost",
    agent_port=6831,
)

# Agregar exportador al proveedor de trazas
span_processor = BatchSpanProcessor(jaeger_exporter)
trace.get_tracer_provider().add_span_processor(span_processor)

# Instrumentar automáticamente Flask y requests
FlaskInstrumentor().instrument()
RequestsInstrumentor().instrument()

# Aplicación de ejemplo
from flask import Flask, jsonify

app = Flask(__name__)

@app.route('/api/orders/<order_id>')
def get_order(order_id):
    with tracer.start_as_current_span("get_order") as span:
        span.set_attribute("order_id", order_id)
        
        # Simular llamada a base de datos
        order_data = fetch_order_from_db(order_id)
        
        # Simular llamada a servicio externo
        user_data = fetch_user_data(order_data['user_id'])
        
        return jsonify({
            'order': order_data,
            'user': user_data
        })

def fetch_order_from_db(order_id):
    with tracer.start_as_current_span("db_query") as span:
        span.set_attribute("query", "SELECT * FROM orders WHERE id = ?")
        span.set_attribute("order_id", order_id)
        
        # Simular consulta a DB
        import time
        time.sleep(0.1)
        
        return {
            'id': order_id,
            'user_id': '123',
            'total': 99.99,
            'status': 'completed'
        }

def fetch_user_data(user_id):
    with tracer.start_as_current_span("external_api_call") as span:
        span.set_attribute("service", "user-service")
        span.set_attribute("user_id", user_id)
        
        # Llamada a servicio externo
        response = requests.get(f'http://user-service/api/users/{user_id}')
        
        span.set_attribute("response_status", response.status_code)
        
        return response.json()
```

## Herramientas de Monitoreo

### 1. **Prometheus + Grafana**

#### **Configuración de Prometheus**

```yaml
# prometheus.yml
global:
  scrape_interval: 15s
  evaluation_interval: 15s

rule_files:
  - "alert_rules.yml"

scrape_configs:
  - job_name: 'prometheus'
    static_configs:
      - targets: ['localhost:9090']

  - job_name: 'application'
    static_configs:
      - targets: ['app:5000']
    metrics_path: '/metrics'
    scrape_interval: 5s

  - job_name: 'node-exporter'
    static_configs:
      - targets: ['node-exporter:9100']

  - job_name: 'docker'
    static_configs:
      - targets: ['cadvisor:8080']

alerting:
  alertmanagers:
    - static_configs:
        - targets:
          - alertmanager:9093
```

#### **Reglas de Alertas**

```yaml
# alert_rules.yml
groups:
  - name: application_alerts
    rules:
      - alert: HighErrorRate
        expr: rate(http_requests_total{status=~"5.."}[5m]) > 0.1
        for: 5m
        labels:
          severity: critical
        annotations:
          summary: "High error rate detected"
          description: "Error rate is {{ $value }} for {{ $labels.instance }}"

      - alert: HighResponseTime
        expr: histogram_quantile(0.95, rate(http_request_duration_seconds_bucket[5m])) > 0.5
        for: 10m
        labels:
          severity: warning
        annotations:
          summary: "High response time detected"
          description: "95th percentile response time is {{ $value }}s"

      - alert: ServiceDown
        expr: up == 0
        for: 2m
        labels:
          severity: critical
        annotations:
          summary: "Service is down"
          description: "{{ $labels.job }} has been down for more than 2 minutes"

  - name: infrastructure_alerts
    rules:
      - alert: HighCPUUsage
        expr: 100 - (avg by(instance) (rate(node_cpu_seconds_total{mode="idle"}[5m])) * 100) > 80
        for: 5m
        labels:
          severity: warning
        annotations:
          summary: "High CPU usage"
          description: "CPU usage is {{ $value }}% on {{ $labels.instance }}"

      - alert: HighMemoryUsage
        expr: (node_memory_MemTotal_bytes - node_memory_MemAvailable_bytes) / node_memory_MemTotal_bytes * 100 > 90
        for: 10m
        labels:
          severity: critical
        annotations:
          summary: "High memory usage"
          description: "Memory usage is {{ $value }}% on {{ $labels.instance }}"
```

#### **Dashboard de Grafana**

```json
{
  "dashboard": {
    "title": "Application Monitoring",
    "panels": [
      {
        "title": "Request Rate",
        "type": "graph",
        "targets": [
          {
            "expr": "rate(http_requests_total[1m])",
            "legendFormat": "{{method}} {{endpoint}}"
          }
        ]
      },
      {
        "title": "Error Rate",
        "type": "singlestat",
        "targets": [
          {
            "expr": "rate(http_requests_total{status=~\"5..\"}[5m]) / rate(http_requests_total[5m]) * 100"
          }
        ]
      },
      {
        "title": "Response Time",
        "type": "graph",
        "targets": [
          {
            "expr": "histogram_quantile(0.50, rate(http_request_duration_seconds_bucket[5m]))",
            "legendFormat": "50th percentile"
          },
          {
            "expr": "histogram_quantile(0.95, rate(http_request_duration_seconds_bucket[5m]))",
            "legendFormat": "95th percentile"
          }
        ]
      }
    ]
  }
}
```

### 2. **Health Checks**

#### **Health Check Endpoint**

```python
# health.py
from flask import Flask, jsonify
import psycopg2
import redis
import requests
from datetime import datetime

app = Flask(__name__)

class HealthChecker:
    def __init__(self):
        self.checks = {
            'database': self.check_database,
            'redis': self.check_redis,
            'external_api': self.check_external_api,
            'disk_space': self.check_disk_space
        }
    
    def check_database(self):
        try:
            conn = psycopg2.connect("postgresql://user:pass@localhost/db")
            cursor = conn.cursor()
            cursor.execute("SELECT 1")
            cursor.close()
            conn.close()
            return {'status': 'healthy', 'response_time': 0.05}
        except Exception as e:
            return {'status': 'unhealthy', 'error': str(e)}
    
    def check_redis(self):
        try:
            r = redis.Redis(host='localhost', port=6379, db=0)
            r.ping()
            return {'status': 'healthy'}
        except Exception as e:
            return {'status': 'unhealthy', 'error': str(e)}
    
    def check_external_api(self):
        try:
            response = requests.get('https://api.external.com/health', timeout=5)
            if response.status_code == 200:
                return {'status': 'healthy', 'response_time': response.elapsed.total_seconds()}
            else:
                return {'status': 'unhealthy', 'http_status': response.status_code}
        except Exception as e:
            return {'status': 'unhealthy', 'error': str(e)}
    
    def check_disk_space(self):
        import shutil
        total, used, free = shutil.disk_usage('/')
        usage_percent = (used / total) * 100
        
        if usage_percent > 90:
            return {'status': 'unhealthy', 'usage_percent': usage_percent}
        elif usage_percent > 80:
            return {'status': 'degraded', 'usage_percent': usage_percent}
        else:
            return {'status': 'healthy', 'usage_percent': usage_percent}

@app.route('/health')
def health_check():
    checker = HealthChecker()
    results = {}
    overall_status = 'healthy'
    
    for name, check_func in checker.checks.items():
        results[name] = check_func()
        if results[name]['status'] != 'healthy':
            overall_status = 'unhealthy'
    
    response = {
        'status': overall_status,
        'timestamp': datetime.utcnow().isoformat(),
        'checks': results
    }
    
    status_code = 200 if overall_status == 'healthy' else 503
    return jsonify(response), status_code

@app.route('/health/ready')
def readiness_check():
    # Verificar que la aplicación esté lista para recibir tráfico
    return jsonify({'status': 'ready'}), 200

@app.route('/health/live')
def liveness_check():
    # Verificar que la aplicación esté viva
    return jsonify({'status': 'alive'}), 200
```

### 3. **Alerting y Notificaciones**

#### **Configuración de Alertmanager**

```yaml
# alertmanager.yml
global:
  smtp_smarthost: 'smtp.gmail.com:587'
  smtp_from: 'alerts@company.com'
  smtp_auth_username: 'alerts@company.com'
  smtp_auth_password: 'password'

route:
  group_by: ['alertname', 'cluster', 'service']
  group_wait: 10s
  group_interval: 10s
  repeat_interval: 1h
  receiver: 'web.hook'
  routes:
    - match:
        severity: critical
      receiver: 'critical-alerts'
    - match:
        severity: warning
      receiver: 'warning-alerts'

receivers:
  - name: 'web.hook'
    webhook_configs:
      - url: 'http://localhost:5001/webhook'

  - name: 'critical-alerts'
    email_configs:
      - to: 'oncall@company.com'
        subject: '🚨 CRITICAL ALERT: {{ .GroupLabels.alertname }}'
        body: |
          Alert: {{ .GroupLabels.alertname }}
          Instance: {{ .CommonLabels.instance }}
          Description: {{ .CommonAnnotations.description }}
          
    slack_configs:
      - api_url: 'https://hooks.slack.com/services/YOUR/SLACK/WEBHOOK'
        channel: '#alerts'
        title: 'Critical Alert'
        text: '{{ .CommonAnnotations.description }}'

  - name: 'warning-alerts'
    email_configs:
      - to: 'team@company.com'
        subject: '⚠️ WARNING: {{ .GroupLabels.alertname }}'
```

#### **Webhook Handler para Alertas**

```python
# alert_handler.py
from flask import Flask, request, jsonify
import json
import requests

app = Flask(__name__)

@app.route('/webhook', methods=['POST'])
def handle_alert():
    data = request.json
    
    for alert in data.get('alerts', []):
        status = alert.get('status')
        labels = alert.get('labels', {})
        annotations = alert.get('annotations', {})
        
        if status == 'firing':
            handle_firing_alert(labels, annotations)
        elif status == 'resolved':
            handle_resolved_alert(labels, annotations)
    
    return jsonify({'status': 'ok'})

def handle_firing_alert(labels, annotations):
    alertname = labels.get('alertname')
    severity = labels.get('severity')
    instance = labels.get('instance')
    description = annotations.get('description')
    
    print(f"🚨 ALERT FIRING: {alertname}")
    print(f"Severity: {severity}")
    print(f"Instance: {instance}")
    print(f"Description: {description}")
    
    # Enviar a Slack
    send_slack_notification(alertname, severity, instance, description)
    
    # Crear ticket en sistema de tracking
    create_incident_ticket(alertname, severity, description)

def handle_resolved_alert(labels, annotations):
    alertname = labels.get('alertname')
    print(f"✅ ALERT RESOLVED: {alertname}")

def send_slack_notification(alertname, severity, instance, description):
    webhook_url = "https://hooks.slack.com/services/YOUR/SLACK/WEBHOOK"
    
    color = "danger" if severity == "critical" else "warning"
    
    payload = {
        "attachments": [
            {
                "color": color,
                "title": f"🚨 {alertname}",
                "fields": [
                    {
                        "title": "Severity",
                        "value": severity,
                        "short": True
                    },
                    {
                        "title": "Instance",
                        "value": instance,
                        "short": True
                    },
                    {
                        "title": "Description",
                        "value": description,
                        "short": False
                    }
                ]
            }
        ]
    }
    
    requests.post(webhook_url, json=payload)

def create_incident_ticket(alertname, severity, description):
    # Integración con sistema de tickets (JIRA, ServiceNow, etc.)
    pass
```

## Observabilidad en Microservicios

### 1. **Correlation IDs**

```python
# correlation.py
import uuid
from flask import Flask, request, g
import logging

app = Flask(__name__)

@app.before_request
def before_request():
    # Obtener o generar correlation ID
    correlation_id = request.headers.get('X-Correlation-ID', str(uuid.uuid4()))
    g.correlation_id = correlation_id
    
    # Configurar logger con correlation ID
    logger = logging.getLogger()
    logger.info(
        f"Request started: {request.method} {request.path}",
        extra={'correlation_id': correlation_id}
    )

@app.after_request
def after_request(response):
    # Agregar correlation ID al response
    response.headers['X-Correlation-ID'] = g.correlation_id
    return response

@app.route('/api/orders')
def get_orders():
    # El correlation ID está disponible en g.correlation_id
    logger = logging.getLogger()
    logger.info(
        "Fetching orders",
        extra={'correlation_id': g.correlation_id}
    )
    
    # Pasar correlation ID a servicios downstream
    headers = {'X-Correlation-ID': g.correlation_id}
    response = requests.get('http://inventory-service/api/items', headers=headers)
    
    return jsonify({'orders': []})
```

### 2. **Service Mesh Observability**

```yaml
# istio-observability.yaml
apiVersion: install.istio.io/v1alpha1
kind: IstioOperator
metadata:
  name: istio-observability
spec:
  values:
    telemetry:
      v2:
        prometheus:
          configOverride:
            metric_relabeling_configs:
              - source_labels: [__name__]
                regex: 'istio_.*'
                target_label: __tmp_istio_metric
    meshConfig:
      defaultConfig:
        tracing:
          sampling: 100.0
      extensionProviders:
        - name: jaeger
          envoyOtelAls:
            service: jaeger.istio-system.svc.cluster.local
            port: 14250
```

## Mejores Prácticas

### 1. **SLIs, SLOs y SLAs**

#### **Service Level Indicators (SLIs)**

```yaml
# slis.yml - Definición de SLIs
slis:
  availability:
    description: "Percentage of successful requests"
    query: "sum(rate(http_requests_total{status!~'5..'}[5m])) / sum(rate(http_requests_total[5m])) * 100"
    
  latency:
    description: "95th percentile response time"
    query: "histogram_quantile(0.95, rate(http_request_duration_seconds_bucket[5m]))"
    
  error_rate:
    description: "Percentage of failed requests"
    query: "sum(rate(http_requests_total{status=~'5..'}[5m])) / sum(rate(http_requests_total[5m])) * 100"
```

#### **Service Level Objectives (SLOs)**

```yaml
# slos.yml - Definición de SLOs
slos:
  availability:
    target: 99.9
    time_window: "30d"
    
  latency:
    target: 200  # ms
    time_window: "30d"
    
  error_rate:
    target: 0.1  # %
    time_window: "30d"
```

### 2. **Golden Signals**

#### **Rate, Errors, Duration, Saturation**

```python
# golden_signals.py
from prometheus_client import Counter, Histogram, Gauge

class GoldenSignals:
    def __init__(self):
        # Rate: The number of requests per second
        self.requests_total = Counter(
            'requests_total',
            'Total requests',
            ['service', 'endpoint', 'method']
        )
        
        # Errors: The number of failed requests
        self.errors_total = Counter(
            'errors_total',
            'Total errors',
            ['service', 'endpoint', 'error_type']
        )
        
        # Duration: The time it takes to service a request
        self.request_duration = Histogram(
            'request_duration_seconds',
            'Request duration',
            ['service', 'endpoint']
        )
        
        # Saturation: How full the service is
        self.cpu_usage = Gauge(
            'cpu_usage_percent',
            'CPU usage percentage'
        )
        
        self.memory_usage = Gauge(
            'memory_usage_percent',
            'Memory usage percentage'
        )
    
    def record_request(self, service, endpoint, method, duration, success=True):
        # Incrementar rate
        self.requests_total.labels(
            service=service,
            endpoint=endpoint,
            method=method
        ).inc()
        
        # Registrar duration
        self.request_duration.labels(
            service=service,
            endpoint=endpoint
        ).observe(duration)
        
        # Registrar error si aplica
        if not success:
            self.errors_total.labels(
                service=service,
                endpoint=endpoint,
                error_type='application_error'
            ).inc()
```

### 3. **Error Budget**

```python
# error_budget.py
import datetime
from dataclasses import dataclass

@dataclass
class ErrorBudget:
    slo_target: float  # 99.9% = 0.999
    time_window_days: int  # 30 days
    
    def calculate_error_budget(self):
        """Calcula el error budget permitido"""
        allowed_downtime_percent = 1 - self.slo_target
        total_minutes = self.time_window_days * 24 * 60
        allowed_downtime_minutes = total_minutes * allowed_downtime_percent
        return allowed_downtime_minutes
    
    def current_error_rate(self, total_requests, failed_requests):
        """Calcula la tasa de error actual"""
        if total_requests == 0:
            return 0
        return failed_requests / total_requests
    
    def error_budget_remaining(self, total_requests, failed_requests):
        """Calcula el error budget restante"""
        current_rate = self.current_error_rate(total_requests, failed_requests)
        allowed_rate = 1 - self.slo_target
        
        if current_rate <= allowed_rate:
            # Dentro del budget
            consumed_percent = (current_rate / allowed_rate) * 100
            remaining_percent = 100 - consumed_percent
            return remaining_percent
        else:
            # Excedió el budget
            return 0

# Ejemplo de uso
error_budget = ErrorBudget(slo_target=0.999, time_window_days=30)
print(f"Error budget permitido: {error_budget.calculate_error_budget():.2f} minutos")

# Con datos reales
total_requests = 1000000
failed_requests = 500
remaining = error_budget.error_budget_remaining(total_requests, failed_requests)
print(f"Error budget restante: {remaining:.2f}%")
```

## Herramientas del Ecosistema

### Métricas
- **Prometheus:** Recolección y almacenamiento
- **Grafana:** Visualización y dashboards
- **InfluxDB:** Base de datos de time series
- **DataDog:** Monitoreo como servicio

### Logs
- **ELK Stack:** Elasticsearch, Logstash, Kibana
- **Fluentd:** Recolector de logs
- **Splunk:** Análisis de logs empresarial
- **Loki:** Sistema de logs de Grafana

### Traces
- **Jaeger:** Distributed tracing
- **Zipkin:** Distributed tracing
- **OpenTelemetry:** Estándar de observabilidad
- **AWS X-Ray:** Tracing en AWS

## Ejercicios Prácticos

### Ejercicio 1: Implementar Métricas Básicas

```python
# Implementar métricas de Prometheus en una aplicación Flask
# 1. Request count
# 2. Request duration
# 3. Error rate
# 4. Custom business metrics
```

### Ejercicio 2: Configurar Alertas

```yaml
# Crear reglas de alertas para:
# 1. Alta latencia (>500ms)
# 2. Tasa de error alta (>5%)
# 3. Servicio caído
# 4. Alto uso de CPU (>80%)
```

### Ejercicio 3: Distributed Tracing

```python
# Implementar trazado distribuido entre 3 microservicios:
# 1. API Gateway
# 2. User Service
# 3. Order Service
```

## Recursos Adicionales

### Documentación
- [Prometheus Documentation](https://prometheus.io/docs/)
- [Grafana Documentation](https://grafana.com/docs/)
- [OpenTelemetry Documentation](https://opentelemetry.io/docs/)
- [Jaeger Documentation](https://www.jaegertracing.io/docs/)

### Libros Recomendados
- "Site Reliability Engineering" - Google
- "Observability Engineering" - Honeycomb
- "Monitoring and Observability" - O'Reilly

---

## Próximos Pasos

Una vez que domines monitoreo y observabilidad básicos, estarás listo para:

1. **SRE (Site Reliability Engineering):** Prácticas avanzadas de confiabilidad
2. **Chaos Engineering:** Testing de resiliencia
3. **Performance Engineering:** Optimización de rendimiento
4. **Advanced Analytics:** Machine learning para operaciones

¡El monitoreo y observabilidad son esenciales para mantener sistemas confiables y entender el comportamiento de tus aplicaciones en producción!