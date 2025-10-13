# Módulo 02: Prometheus - Recolección y Almacenamiento de Métricas

## 📋 Introducción

Prometheus es un sistema de monitoreo y alerta de código abierto, diseñado específicamente para la recolección y almacenamiento de métricas de series temporales. En este módulo aprenderás a implementar, configurar y utilizar Prometheus para crear un sistema de monitoreo robusto y escalable.

### ¿Por qué Prometheus?

Prometheus se ha convertido en el estándar de facto para el monitoreo en entornos cloud-native por:

- **Modelo de datos dimensional** - Métricas con etiquetas flexibles
- **Pull-based model** - Prometheus extrae métricas de los targets
- **Lenguaje de consulta potente** - PromQL para análisis avanzado
- **Integración nativa con Kubernetes** - Service discovery automático
- **Ecosistema rico** - Amplia gama de exporters disponibles

---

## 🎯 Objetivos de Aprendizaje

Al completar este módulo, serás capaz de:

1. **Instalar y configurar** Prometheus en diferentes entornos
2. **Entender el modelo de datos** y tipos de métricas de Prometheus
3. **Configurar targets** y service discovery
4. **Crear métricas personalizadas** en aplicaciones
5. **Utilizar PromQL** para consultas y análisis
6. **Implementar exporters** para sistemas externos
7. **Configurar retention** y almacenamiento

---

## 📚 Arquitectura de Prometheus

### 1. Componentes Principales

```mermaid
graph TB
    A[Prometheus Server] --> B[Time Series Database]
    A --> C[HTTP Server]
    A --> D[Retrieval Component]
    
    D --> E[Service Discovery]
    D --> F[Scrape Targets]
    
    F --> G[Application /metrics]
    F --> H[Node Exporter]
    F --> I[Custom Exporters]
    
    C --> J[PromQL API]
    C --> K[Grafana]
    C --> L[Alertmanager]
    
    E --> M[Kubernetes API]
    E --> N[Consul]
    E --> O[Static Config]
```

### 2. Modelo Pull vs Push

```yaml
# Modelo Pull (Prometheus)
prometheus_config:
  scrape_configs:
    - job_name: 'web-app'
      static_configs:
        - targets: ['localhost:8080']
      scrape_interval: 15s
      metrics_path: /metrics

# Versus Modelo Push (tradicional)
# Las aplicaciones envían métricas al servidor
# Prometheus extrae (pull) métricas de las aplicaciones
```

---

## 🔧 Instalación y Configuración

### 1. Instalación con Docker

#### 1.1 Docker Compose Setup

```yaml
# docker-compose.prometheus.yml
version: '3.8'

services:
  prometheus:
    image: prom/prometheus:latest
    container_name: prometheus
    ports:
      - "9090:9090"
    volumes:
      - ./prometheus.yml:/etc/prometheus/prometheus.yml
      - ./rules:/etc/prometheus/rules
      - prometheus_data:/prometheus
    command:
      - '--config.file=/etc/prometheus/prometheus.yml'
      - '--storage.tsdb.path=/prometheus'
      - '--web.console.libraries=/etc/prometheus/console_libraries'
      - '--web.console.templates=/etc/prometheus/consoles'
      - '--storage.tsdb.retention.time=15d'
      - '--web.enable-lifecycle'
      - '--web.enable-admin-api'
    restart: unless-stopped
    networks:
      - monitoring

  node-exporter:
    image: prom/node-exporter:latest
    container_name: node-exporter
    restart: unless-stopped
    volumes:
      - /proc:/host/proc:ro
      - /sys:/host/sys:ro
      - /:/rootfs:ro
    command:
      - '--path.procfs=/host/proc'
      - '--path.rootfs=/rootfs'
      - '--path.sysfs=/host/sys'
      - '--collector.filesystem.mount-points-exclude=^/(sys|proc|dev|host|etc)($$|/)'
    ports:
      - "9100:9100"
    networks:
      - monitoring

  cadvisor:
    image: gcr.io/cadvisor/cadvisor:latest
    container_name: cadvisor
    restart: unless-stopped
    ports:
      - "8080:8080"
    volumes:
      - /:/rootfs:ro
      - /var/run:/var/run:ro
      - /sys:/sys:ro
      - /var/lib/docker/:/var/lib/docker:ro
      - /dev/disk/:/dev/disk:ro
    devices:
      - /dev/kmsg:/dev/kmsg
    networks:
      - monitoring

  # Aplicación de ejemplo para monitorear
  sample-app:
    image: nginx:alpine
    container_name: sample-app
    ports:
      - "8081:80"
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf
    networks:
      - monitoring

volumes:
  prometheus_data:

networks:
  monitoring:
    driver: bridge
```

#### 1.2 Configuración Principal de Prometheus

```yaml
# prometheus.yml
global:
  scrape_interval: 15s
  evaluation_interval: 15s
  external_labels:
    cluster: 'production'
    region: 'us-east-1'

# Reglas de alertas
rule_files:
  - "/etc/prometheus/rules/*.yml"

# Configuración de scraping
scrape_configs:
  # Prometheus se monitorea a sí mismo
  - job_name: 'prometheus'
    static_configs:
      - targets: ['localhost:9090']
    scrape_interval: 5s
    metrics_path: /metrics

  # Node Exporter para métricas del sistema
  - job_name: 'node-exporter'
    static_configs:
      - targets: ['node-exporter:9100']
    scrape_interval: 10s
    scrape_timeout: 5s

  # cAdvisor para métricas de contenedores
  - job_name: 'cadvisor'
    static_configs:
      - targets: ['cadvisor:8080']
    scrape_interval: 10s
    metrics_path: /metrics

  # Aplicación personalizada
  - job_name: 'web-app'
    static_configs:
      - targets: ['sample-app:80']
    scrape_interval: 15s
    metrics_path: /metrics
    scheme: http

  # Ejemplo con service discovery (Kubernetes)
  - job_name: 'kubernetes-pods'
    kubernetes_sd_configs:
      - role: pod
    relabel_configs:
      - source_labels: [__meta_kubernetes_pod_annotation_prometheus_io_scrape]
        action: keep
        regex: true
      - source_labels: [__meta_kubernetes_pod_annotation_prometheus_io_path]
        action: replace
        target_label: __metrics_path__
        regex: (.+)

# Configuración de Alertmanager
alerting:
  alertmanagers:
    - static_configs:
        - targets:
          - alertmanager:9093

# Configuración de almacenamiento remoto (opcional)
remote_write:
  - url: "https://prometheus-remote-write.example.com/api/v1/write"
    basic_auth:
      username: admin
      password: secret

remote_read:
  - url: "https://prometheus-remote-read.example.com/api/v1/read"
    basic_auth:
      username: admin
      password: secret
```

### 2. Instalación Nativa

#### 2.1 Script de Instalación (Linux)

```bash
#!/bin/bash
# install_prometheus.sh

PROMETHEUS_VERSION="2.45.0"
PROMETHEUS_USER="prometheus"
PROMETHEUS_GROUP="prometheus"

# Crear usuario del sistema
sudo useradd --no-create-home --shell /bin/false $PROMETHEUS_USER

# Crear directorios
sudo mkdir -p /etc/prometheus
sudo mkdir -p /var/lib/prometheus
sudo mkdir -p /var/log/prometheus

# Descargar Prometheus
cd /tmp
wget https://github.com/prometheus/prometheus/releases/download/v${PROMETHEUS_VERSION}/prometheus-${PROMETHEUS_VERSION}.linux-amd64.tar.gz

# Extraer
tar xvf prometheus-${PROMETHEUS_VERSION}.linux-amd64.tar.gz
cd prometheus-${PROMETHEUS_VERSION}.linux-amd64

# Instalar binarios
sudo cp prometheus /usr/local/bin/
sudo cp promtool /usr/local/bin/

# Instalar archivos de configuración
sudo cp -r consoles /etc/prometheus
sudo cp -r console_libraries /etc/prometheus

# Configurar permisos
sudo chown -R $PROMETHEUS_USER:$PROMETHEUS_GROUP /etc/prometheus
sudo chown -R $PROMETHEUS_USER:$PROMETHEUS_GROUP /var/lib/prometheus
sudo chown -R $PROMETHEUS_USER:$PROMETHEUS_GROUP /var/log/prometheus
sudo chown $PROMETHEUS_USER:$PROMETHEUS_GROUP /usr/local/bin/prometheus
sudo chown $PROMETHEUS_USER:$PROMETHEUS_GROUP /usr/local/bin/promtool

# Limpiar
cd /
rm -rf /tmp/prometheus-${PROMETHEUS_VERSION}.linux-amd64*

echo "Prometheus installed successfully!"
echo "Next steps:"
echo "1. Create /etc/prometheus/prometheus.yml"
echo "2. Create systemd service"
echo "3. Start prometheus service"
```

#### 2.2 Servicio Systemd

```ini
# /etc/systemd/system/prometheus.service
[Unit]
Description=Prometheus
Wants=network-online.target
After=network-online.target

[Service]
User=prometheus
Group=prometheus
Type=simple
Restart=always
RestartSec=5
ExecStart=/usr/local/bin/prometheus \
    --config.file /etc/prometheus/prometheus.yml \
    --storage.tsdb.path /var/lib/prometheus/ \
    --web.console.templates=/etc/prometheus/consoles \
    --web.console.libraries=/etc/prometheus/console_libraries \
    --web.listen-address=0.0.0.0:9090 \
    --web.enable-lifecycle \
    --storage.tsdb.retention.time=15d

StandardOutput=journal
StandardError=journal
SyslogIdentifier=prometheus

[Install]
WantedBy=multi-user.target
```

---

## 📊 Tipos de Métricas en Prometheus

### 1. Counter (Contador)

**Definición**: Métrica que solo puede incrementar o resetearse a cero.

```python
# Ejemplo en Python con prometheus_client
from prometheus_client import Counter, start_http_server

# Definir contador
http_requests_total = Counter(
    'http_requests_total',
    'Total HTTP requests',
    ['method', 'endpoint', 'status']
)

# Incrementar contador
def handle_request(method, endpoint, status):
    http_requests_total.labels(
        method=method,
        endpoint=endpoint,
        status=status
    ).inc()

# Uso
handle_request('GET', '/api/users', '200')
handle_request('POST', '/api/users', '201')
handle_request('GET', '/api/users/123', '404')

# Iniciar servidor de métricas
start_http_server(8000)
```

```promql
# Consultas PromQL para counters
# Tasa de requests por segundo
rate(http_requests_total[5m])

# Total de requests en los últimos 5 minutos
increase(http_requests_total[5m])

# Requests por método
sum(rate(http_requests_total[5m])) by (method)
```

### 2. Gauge (Medidor)

**Definición**: Métrica que puede subir y bajar arbitrariamente.

```python
from prometheus_client import Gauge
import psutil
import threading
import time

# Definir gauges
cpu_usage = Gauge('cpu_usage_percent', 'CPU usage percentage')
memory_usage = Gauge('memory_usage_bytes', 'Memory usage in bytes')
active_connections = Gauge(
    'active_connections', 
    'Active database connections',
    ['database']
)

def collect_system_metrics():
    while True:
        # CPU usage
        cpu_percent = psutil.cpu_percent()
        cpu_usage.set(cpu_percent)
        
        # Memory usage
        memory = psutil.virtual_memory()
        memory_usage.set(memory.used)
        
        # Simular conexiones de base de datos
        active_connections.labels(database='primary').set(25)
        active_connections.labels(database='replica').set(12)
        
        time.sleep(10)

# Iniciar recolección en background
threading.Thread(target=collect_system_metrics, daemon=True).start()
```

```promql
# Consultas PromQL para gauges
# CPU usage actual
cpu_usage_percent

# Promedio de CPU en los últimos 5 minutos
avg_over_time(cpu_usage_percent[5m])

# Máximo uso de memoria en la última hora
max_over_time(memory_usage_bytes[1h])

# Conexiones activas por base de datos
sum(active_connections) by (database)
```

### 3. Histogram (Histograma)

**Definición**: Métrica que muestrea observaciones y las cuenta en buckets configurables.

```python
from prometheus_client import Histogram
import time
import random

# Definir histogram
request_duration = Histogram(
    'http_request_duration_seconds',
    'HTTP request duration',
    ['method', 'endpoint'],
    buckets=(0.1, 0.25, 0.5, 1.0, 2.5, 5.0, 10.0)
)

def process_request(method, endpoint):
    # Simular tiempo de procesamiento
    start = time.time()
    
    # Simular trabajo
    time.sleep(random.uniform(0.1, 2.0))
    
    # Registrar duración
    duration = time.time() - start
    request_duration.labels(
        method=method,
        endpoint=endpoint
    ).observe(duration)
    
    return f"Processed {method} {endpoint} in {duration:.2f}s"

# Ejemplos de uso
process_request('GET', '/api/users')
process_request('POST', '/api/orders')
```

```promql
# Consultas PromQL para histograms
# Percentil 95 de duración de requests
histogram_quantile(0.95, http_request_duration_seconds_bucket)

# Duración promedio
rate(http_request_duration_seconds_sum[5m]) / rate(http_request_duration_seconds_count[5m])

# Requests que toman más de 1 segundo
sum(rate(http_request_duration_seconds_bucket{le="1.0"}[5m]))
```

### 4. Summary (Resumen)

**Definición**: Similar al histogram pero calcula percentiles en el cliente.

```python
from prometheus_client import Summary

# Definir summary
request_latency = Summary(
    'http_request_latency_seconds',
    'HTTP request latency',
    ['method']
)

@request_latency.labels(method='GET').time()
def get_handler():
    time.sleep(random.uniform(0.1, 1.0))
    return "GET response"

# O manualmente
def post_handler():
    with request_latency.labels(method='POST').time():
        time.sleep(random.uniform(0.2, 1.5))
        return "POST response"
```

---

## 🔍 PromQL - Lenguaje de Consulta

### 1. Sintaxis Básica

```promql
# Selector básico
http_requests_total

# Con filtros de etiquetas
http_requests_total{method="GET"}
http_requests_total{method="GET", status="200"}

# Operadores de comparación
http_requests_total{status!="200"}
http_requests_total{endpoint=~"/api/.*"}
http_requests_total{status=~"2..|3.."}

# Rangos de tiempo
http_requests_total[5m]  # Últimos 5 minutos
http_requests_total[1h]  # Última hora
http_requests_total[1d]  # Último día
```

### 2. Funciones de Agregación

```promql
# Suma
sum(http_requests_total)
sum(http_requests_total) by (method)
sum(http_requests_total) without (instance)

# Promedio
avg(cpu_usage_percent)
avg(cpu_usage_percent) by (instance)

# Máximo y mínimo
max(memory_usage_bytes)
min(disk_available_bytes)

# Contar elementos
count(up == 1)  # Instancias activas
count(http_requests_total) by (status)
```

### 3. Funciones de Tiempo

```promql
# Rate - tasa por segundo
rate(http_requests_total[5m])

# Increase - incremento total
increase(http_requests_total[1h])

# irate - tasa instantánea
irate(http_requests_total[5m])

# delta - diferencia
delta(cpu_usage_percent[10m])

# deriv - derivada
deriv(memory_usage_bytes[5m])
```

### 4. Consultas Avanzadas

```promql
# Top 5 endpoints con más tráfico
topk(5, sum(rate(http_requests_total[5m])) by (endpoint))

# Percentiles de latencia
histogram_quantile(0.95,
  sum(rate(http_request_duration_seconds_bucket[5m])) by (le)
)

# Comparar con hace una hora
cpu_usage_percent - cpu_usage_percent offset 1h

# Predicción lineal
predict_linear(cpu_usage_percent[1h], 3600)  # Predicción para 1 hora

# Alertas complejas
(
  100 * (1 - avg(rate(node_cpu_seconds_total{mode="idle"}[5m])))
) > 80
```

---

## 🏗️ Instrumentación de Aplicaciones

### 1. Aplicación Python con Flask

```python
# app.py - Aplicación Flask instrumentada
from flask import Flask, request, jsonify
from prometheus_client import Counter, Histogram, Gauge, generate_latest
import time
import random
import threading

app = Flask(__name__)

# Métricas personalizadas
REQUEST_COUNT = Counter(
    'http_requests_total',
    'Total HTTP requests',
    ['method', 'endpoint', 'status']
)

REQUEST_LATENCY = Histogram(
    'http_request_duration_seconds',
    'HTTP request latency',
    ['method', 'endpoint'],
    buckets=(0.01, 0.05, 0.1, 0.25, 0.5, 1.0, 2.5, 5.0)
)

ACTIVE_REQUESTS = Gauge(
    'http_requests_active',
    'Active HTTP requests'
)

DATABASE_CONNECTIONS = Gauge(
    'database_connections_active',
    'Active database connections',
    ['pool']
)

ERROR_COUNT = Counter(
    'application_errors_total',
    'Total application errors',
    ['error_type']
)

# Middleware para instrumentación
@app.before_request
def before_request():
    request.start_time = time.time()
    ACTIVE_REQUESTS.inc()

@app.after_request
def after_request(response):
    request_latency = time.time() - request.start_time
    
    REQUEST_LATENCY.labels(
        method=request.method,
        endpoint=request.endpoint or 'unknown'
    ).observe(request_latency)
    
    REQUEST_COUNT.labels(
        method=request.method,
        endpoint=request.endpoint or 'unknown',
        status=response.status_code
    ).inc()
    
    ACTIVE_REQUESTS.dec()
    return response

# Rutas de la aplicación
@app.route('/')
def home():
    return jsonify({'message': 'Hello from monitored app!'})

@app.route('/api/users')
def get_users():
    # Simular trabajo y ocasionales errores
    processing_time = random.uniform(0.1, 1.0)
    time.sleep(processing_time)
    
    if random.random() < 0.05:  # 5% error rate
        ERROR_COUNT.labels(error_type='database_error').inc()
        return jsonify({'error': 'Database connection failed'}), 500
    
    return jsonify({
        'users': [
            {'id': 1, 'name': 'Alice'},
            {'id': 2, 'name': 'Bob'},
            {'id': 3, 'name': 'Charlie'}
        ]
    })

@app.route('/api/orders', methods=['POST'])
def create_order():
    time.sleep(random.uniform(0.2, 1.5))
    
    if random.random() < 0.02:  # 2% error rate
        ERROR_COUNT.labels(error_type='validation_error').inc()
        return jsonify({'error': 'Invalid order data'}), 400
    
    return jsonify({'order_id': random.randint(1000, 9999)}), 201

@app.route('/health')
def health():
    return jsonify({'status': 'healthy', 'timestamp': time.time()})

# Endpoint de métricas para Prometheus
@app.route('/metrics')
def metrics():
    return generate_latest()

# Simulador de métricas de fondo
def background_metrics():
    """Simula métricas que cambian en el tiempo"""
    while True:
        # Simular conexiones de base de datos
        DATABASE_CONNECTIONS.labels(pool='primary').set(
            random.randint(10, 50)
        )
        DATABASE_CONNECTIONS.labels(pool='readonly').set(
            random.randint(5, 25)
        )
        
        time.sleep(30)

# Iniciar thread de fondo
threading.Thread(target=background_metrics, daemon=True).start()

if __name__ == '__main__':
    print("Starting monitored application...")
    print("Metrics available at: http://localhost:5000/metrics")
    app.run(host='0.0.0.0', port=5000, debug=True)
```

### 2. Aplicación Node.js

```javascript
// app.js - Aplicación Express instrumentada
const express = require('express');
const promClient = require('prom-client');

const app = express();
app.use(express.json());

// Configurar métricas por defecto
promClient.collectDefaultMetrics({ timeout: 5000 });

// Métricas personalizadas
const httpRequestsTotal = new promClient.Counter({
  name: 'http_requests_total',
  help: 'Total number of HTTP requests',
  labelNames: ['method', 'route', 'status_code']
});

const httpRequestDuration = new promClient.Histogram({
  name: 'http_request_duration_seconds',
  help: 'Duration of HTTP requests in seconds',
  labelNames: ['method', 'route'],
  buckets: [0.01, 0.05, 0.1, 0.25, 0.5, 1, 2.5, 5, 10]
});

const activeRequests = new promClient.Gauge({
  name: 'http_requests_active',
  help: 'Number of active HTTP requests'
});

// Middleware de instrumentación
app.use((req, res, next) => {
  const start = Date.now();
  activeRequests.inc();
  
  res.on('finish', () => {
    const duration = (Date.now() - start) / 1000;
    
    httpRequestDuration
      .labels(req.method, req.route?.path || req.path)
      .observe(duration);
    
    httpRequestsTotal
      .labels(req.method, req.route?.path || req.path, res.statusCode)
      .inc();
    
    activeRequests.dec();
  });
  
  next();
});

// Rutas
app.get('/', (req, res) => {
  res.json({ message: 'Hello from Node.js monitored app!' });
});

app.get('/api/users', async (req, res) => {
  // Simular async work
  await new Promise(resolve => setTimeout(resolve, Math.random() * 500));
  
  if (Math.random() < 0.05) {
    return res.status(500).json({ error: 'Internal server error' });
  }
  
  res.json({
    users: [
      { id: 1, name: 'Alice' },
      { id: 2, name: 'Bob' }
    ]
  });
});

// Endpoint de métricas
app.get('/metrics', (req, res) => {
  res.set('Content-Type', promClient.register.contentType);
  res.end(promClient.register.metrics());
});

const port = process.env.PORT || 3000;
app.listen(port, () => {
  console.log(`App running on port ${port}`);
  console.log(`Metrics available at: http://localhost:${port}/metrics`);
});
```

### 3. Configuración de Nginx con módulo de métricas

```nginx
# nginx.conf
events {
    worker_connections 1024;
}

http {
    # Configurar logging para métricas
    log_format prometheus '$remote_addr - $remote_user [$time_local] '
                         '"$request" $status $body_bytes_sent '
                         '"$http_referer" "$http_user_agent" '
                         'rt=$request_time uct="$upstream_connect_time" '
                         'uht="$upstream_header_time" urt="$upstream_response_time"';

    access_log /var/log/nginx/access.log prometheus;
    
    # Servidor principal
    server {
        listen 80;
        server_name localhost;
        
        location / {
            proxy_pass http://sample-app:5000;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
        }
        
        # Métricas básicas de Nginx
        location /nginx_status {
            stub_status on;
            access_log off;
            allow 127.0.0.1;
            allow 172.0.0.0/8;  # Docker networks
            deny all;
        }
    }
}
```

---

## 📈 Exporters Populares

### 1. Node Exporter (Métricas del Sistema)

```bash
# Instalación
docker run -d \
  --name node-exporter \
  --restart unless-stopped \
  -p 9100:9100 \
  -v "/proc:/host/proc:ro" \
  -v "/sys:/host/sys:ro" \
  -v "/:/rootfs:ro" \
  prom/node-exporter:latest \
  --path.procfs=/host/proc \
  --path.rootfs=/rootfs \
  --path.sysfs=/host/sys \
  --collector.filesystem.mount-points-exclude='^/(sys|proc|dev|host|etc)($$|/)'
```

**Métricas principales**:

```promql
# CPU usage
100 - (avg(rate(node_cpu_seconds_total{mode="idle"}[5m])) * 100)

# Memory usage
(1 - (node_memory_MemAvailable_bytes / node_memory_MemTotal_bytes)) * 100

# Disk usage
100 - ((node_filesystem_avail_bytes * 100) / node_filesystem_size_bytes)

# Load average
node_load1
node_load5
node_load15

# Network traffic
rate(node_network_receive_bytes_total[5m])
rate(node_network_transmit_bytes_total[5m])
```

### 2. PostgreSQL Exporter

```yaml
# docker-compose.yml excerpt
postgres-exporter:
  image: prometheuscommunity/postgres-exporter
  environment:
    DATA_SOURCE_NAME: "postgresql://username:password@postgres:5432/database?sslmode=disable"
  ports:
    - "9187:9187"
  depends_on:
    - postgres
```

**Métricas principales**:

```promql
# Conexiones activas
pg_stat_activity_count

# Tamaño de base de datos
pg_database_size_bytes

# Transacciones por segundo
rate(pg_stat_database_xact_commit_total[5m]) + rate(pg_stat_database_xact_rollback_total[5m])

# Bloqueos activos
pg_locks_count

# Cache hit ratio
pg_stat_database_blks_hit / (pg_stat_database_blks_hit + pg_stat_database_blks_read) * 100
```

### 3. Redis Exporter

```bash
# Instalación
docker run -d \
  --name redis-exporter \
  -p 9121:9121 \
  oliver006/redis_exporter \
  --redis.addr=redis://redis:6379
```

### 4. Exporter Personalizado en Python

```python
#!/usr/bin/env python3
# custom_exporter.py
import time
import requests
import psutil
from prometheus_client import start_http_server, Gauge, Counter
import threading

# Métricas personalizadas
CUSTOM_GAUGE = Gauge('custom_system_metric', 'Custom system metric', ['type'])
API_HEALTH = Gauge('external_api_health', 'External API health status', ['api'])
PROCESS_COUNT = Gauge('custom_process_count', 'Number of specific processes')

class CustomExporter:
    def __init__(self):
        self.apis_to_check = [
            {'name': 'github', 'url': 'https://api.github.com'},
            {'name': 'httpbin', 'url': 'https://httpbin.org/status/200'}
        ]
    
    def collect_system_metrics(self):
        """Recolecta métricas personalizadas del sistema"""
        while True:
            try:
                # CPU por core
                cpu_percents = psutil.cpu_percent(percpu=True)
                for i, cpu_percent in enumerate(cpu_percents):
                    CUSTOM_GAUGE.labels(type=f'cpu_core_{i}').set(cpu_percent)
                
                # Procesos por estado
                process_states = {}
                for proc in psutil.process_iter(['status']):
                    try:
                        status = proc.info['status']
                        process_states[status] = process_states.get(status, 0) + 1
                    except (psutil.NoSuchProcess, psutil.AccessDenied):
                        continue
                
                for state, count in process_states.items():
                    CUSTOM_GAUGE.labels(type=f'process_state_{state}').set(count)
                
                # Procesos específicos (ejemplo: nginx)
                nginx_count = len([p for p in psutil.process_iter(['name']) 
                                 if p.info['name'] == 'nginx'])
                PROCESS_COUNT.set(nginx_count)
                
            except Exception as e:
                print(f"Error collecting system metrics: {e}")
            
            time.sleep(30)
    
    def check_external_apis(self):
        """Verifica salud de APIs externas"""
        while True:
            for api in self.apis_to_check:
                try:
                    response = requests.get(api['url'], timeout=5)
                    status = 1 if response.status_code == 200 else 0
                    API_HEALTH.labels(api=api['name']).set(status)
                except Exception:
                    API_HEALTH.labels(api=api['name']).set(0)
            
            time.sleep(60)
    
    def start(self):
        """Inicia el exporter"""
        # Iniciar threads de recolección
        threading.Thread(target=self.collect_system_metrics, daemon=True).start()
        threading.Thread(target=self.check_external_apis, daemon=True).start()
        
        # Iniciar servidor HTTP
        start_http_server(9999)
        print("Custom exporter started on port 9999")
        print("Metrics available at: http://localhost:9999/metrics")
        
        # Mantener el programa corriendo
        try:
            while True:
                time.sleep(60)
        except KeyboardInterrupt:
            print("Exporter stopped")

if __name__ == '__main__':
    exporter = CustomExporter()
    exporter.start()
```

---

## 🔧 Configuración Avanzada

### 1. Service Discovery con Kubernetes

```yaml
# prometheus-k8s.yml
scrape_configs:
  # Descubrimiento de pods
  - job_name: 'kubernetes-pods'
    kubernetes_sd_configs:
      - role: pod
    relabel_configs:
      # Solo scrape pods con anotación prometheus.io/scrape=true
      - source_labels: [__meta_kubernetes_pod_annotation_prometheus_io_scrape]
        action: keep
        regex: true
      # Usar puerto de anotación si existe
      - source_labels: [__meta_kubernetes_pod_annotation_prometheus_io_port]
        action: replace
        target_label: __address__
        regex: ([^:]+)(?::\d+)?;(\d+)
        replacement: $1:$2
      # Usar path de anotación si existe
      - source_labels: [__meta_kubernetes_pod_annotation_prometheus_io_path]
        action: replace
        target_label: __metrics_path__
        regex: (.+)
      # Agregar labels útiles
      - source_labels: [__meta_kubernetes_namespace]
        action: replace
        target_label: kubernetes_namespace
      - source_labels: [__meta_kubernetes_pod_name]
        action: replace
        target_label: kubernetes_pod_name

  # Descubrimiento de services
  - job_name: 'kubernetes-services'
    kubernetes_sd_configs:
      - role: service
    relabel_configs:
      - source_labels: [__meta_kubernetes_service_annotation_prometheus_io_scrape]
        action: keep
        regex: true
      - source_labels: [__meta_kubernetes_service_annotation_prometheus_io_path]
        action: replace
        target_label: __metrics_path__
        regex: (.+)
```

### 2. Configuración de Alertas

```yaml
# rules/alerts.yml
groups:
  - name: system_alerts
    rules:
      - alert: HighCPUUsage
        expr: 100 - (avg(rate(node_cpu_seconds_total{mode="idle"}[5m])) * 100) > 80
        for: 5m
        labels:
          severity: warning
          team: infrastructure
        annotations:
          summary: "High CPU usage detected"
          description: "CPU usage is above 80% for more than 5 minutes on {{ $labels.instance }}"

      - alert: HighMemoryUsage
        expr: (1 - (node_memory_MemAvailable_bytes / node_memory_MemTotal_bytes)) * 100 > 85
        for: 10m
        labels:
          severity: critical
          team: infrastructure
        annotations:
          summary: "High memory usage detected"
          description: "Memory usage is above 85% for more than 10 minutes on {{ $labels.instance }}"

      - alert: DiskSpaceLow
        expr: (node_filesystem_avail_bytes / node_filesystem_size_bytes) * 100 < 10
        for: 1m
        labels:
          severity: critical
          team: infrastructure
        annotations:
          summary: "Disk space is running low"
          description: "Disk space is below 10% on {{ $labels.instance }}"

  - name: application_alerts
    rules:
      - alert: HighErrorRate
        expr: |
          (
            sum(rate(http_requests_total{status=~"5.."}[5m])) /
            sum(rate(http_requests_total[5m]))
          ) * 100 > 5
        for: 5m
        labels:
          severity: critical
          team: backend
        annotations:
          summary: "High error rate detected"
          description: "Error rate is above 5% for the last 5 minutes"

      - alert: SlowResponseTime
        expr: histogram_quantile(0.95, http_request_duration_seconds_bucket) > 2
        for: 10m
        labels:
          severity: warning
          team: backend
        annotations:
          summary: "Slow response time detected"
          description: "95th percentile response time is above 2 seconds"

      - alert: ServiceDown
        expr: up == 0
        for: 1m
        labels:
          severity: critical
          team: infrastructure
        annotations:
          summary: "Service is down"
          description: "{{ $labels.job }} on {{ $labels.instance }} has been down for more than 1 minute"
```

### 3. Configuración de Retention y Storage

```yaml
# prometheus.yml - Configuración de almacenamiento
global:
  scrape_interval: 15s
  evaluation_interval: 15s

# Configuración de almacenamiento local
storage:
  tsdb:
    path: /prometheus
    retention.time: 15d
    retention.size: "10GB"
    # Compactación más agresiva para ahorrar espacio
    min-block-duration: 2h
    max-block-duration: 25h

# Configuración de almacenamiento remoto
remote_write:
  - url: "https://prometheus-remote-storage.example.com/api/v1/write"
    queue_config:
      capacity: 2500
      max_shards: 200
      min_shards: 1
      max_samples_per_send: 500
      batch_send_deadline: 5s
      min_backoff: 30ms
      max_backoff: 100ms
    write_relabel_configs:
      # Solo enviar métricas específicas al almacenamiento remoto
      - source_labels: [__name__]
        regex: 'http_requests_total|cpu_usage_percent|memory_usage_bytes'
        action: keep

remote_read:
  - url: "https://prometheus-remote-storage.example.com/api/v1/read"
    read_recent: true
```

---

## 📋 Ejercicios Prácticos

### Ejercicio 1: Instrumentar una Aplicación Web

**Objetivo**: Agregar métricas personalizadas a una aplicación web existente.

**Tareas**:

1. Crear una aplicación Flask/Express simple
2. Agregar métricas de requests, latencia y errores
3. Configurar endpoint `/metrics`
4. Configurar Prometheus para scrape
5. Crear consultas PromQL básicas

### Ejercicio 2: Configurar Múltiples Exporters

**Objetivo**: Configurar un stack completo de monitoreo.

**Componentes**:

- Prometheus
- Node Exporter
- PostgreSQL Exporter
- Custom Exporter
- cAdvisor

### Ejercicio 3: Crear Dashboards de Métricas

**Objetivo**: Visualizar métricas usando la consola de Prometheus.

**Métricas a mostrar**:

- CPU, Memory, Disk usage
- Application response times
- Error rates
- Database performance

---

## 🧪 Laboratorio

### Lab 1: Setup Completo de Prometheus

1. **Configurar el entorno**:

   ```bash
   mkdir prometheus-lab
   cd prometheus-lab
   # Crear docker-compose.yml
   # Crear prometheus.yml
   # Crear aplicación de ejemplo
   ```

2. **Levantar servicios**:

   ```bash
   docker-compose up -d
   ```

3. **Verificar métricas**:
   - Acceder a Prometheus UI (<http://localhost:9090>)
   - Revisar targets en Status > Targets
   - Ejecutar consultas PromQL básicas

4. **Crear alertas básicas**
5. **Probar escenarios de carga**

### Lab 2: Exporters Personalizados

1. **Crear exporter para base de datos MongoDB**
2. **Implementar métricas de negocio**
3. **Configurar service discovery**
4. **Optimizar performance y retention**

---

## 📖 Recursos Adicionales

### Documentación Oficial

- [Prometheus Documentation](https://prometheus.io/docs/)
- [PromQL Cheat Sheet](https://promlabs.com/promql-cheat-sheet/)
- [Exporters List](https://prometheus.io/docs/instrumenting/exporters/)

### Herramientas Útiles

- **promtool** - Validación de configuración y consultas
- **prometheus_client** - Librerías para instrumentación
- **node_exporter** - Métricas del sistema
- **blackbox_exporter** - Probes externos

### Best Practices

1. **Naming conventions** - Usar nombres descriptivos y consistentes
2. **Label discipline** - No usar labels con alta cardinalidad
3. **Efficient queries** - Evitar consultas costosas en producción
4. **Proper retention** - Balance entre utilidad y costo de almacenamiento
5. **Monitoring monitoring** - Monitorear el propio Prometheus

---

## ✅ Resumen del Módulo

En este módulo has aprendido:

1. **Arquitectura de Prometheus** y su modelo pull-based
2. **Tipos de métricas** (Counter, Gauge, Histogram, Summary)
3. **PromQL** para consultas y análisis avanzado
4. **Instrumentación** de aplicaciones en múltiples lenguajes
5. **Configuración avanzada** incluyendo service discovery
6. **Exporters** populares y creación de exporters personalizados
7. **Gestión de storage** y retention políticas
8. **Alerting básico** con reglas y umbrales

### Próximas etapas

- **Módulo 03**: Exploraremos Grafana para visualización avanzada de métricas
- **Módulo 04**: Implementaremos el stack ELK para gestión centralizada de logs
- **Módulo 05**: Configuraremos sistemas de alertas con Alertmanager

¡Continúa con el siguiente módulo para dominar la visualización de métricas con Grafana!
