# Módulo 01: Fundamentos de Monitoreo

## 📋 Introducción

El monitoreo es uno de los pilares fundamentales de DevOps moderno. En este módulo aprenderás los conceptos básicos del monitoreo de sistemas, aplicaciones e infraestructura, estableciendo las bases para implementar una estrategia de observabilidad completa.

### ¿Por qué es importante el monitoreo?

El monitoreo efectivo permite:

- **Detección temprana de problemas** antes de que afecten a los usuarios
- **Optimización de rendimiento** basada en datos reales
- **Cumplimiento de SLAs** y objetivos de negocio
- **Toma de decisiones informada** sobre escalabilidad y recursos
- **Reducción del MTTR** (Mean Time To Recovery)

---

## 🎯 Objetivos de Aprendizaje

Al completar este módulo, serás capaz de:

1. **Comprender** los conceptos fundamentales del monitoreo
2. **Identificar** los diferentes tipos de métricas y su importancia
3. **Diseñar** una estrategia básica de monitoreo
4. **Implementar** monitoreo básico con herramientas nativas
5. **Establecer** alertas efectivas y umbrales apropiados

---

## 📚 Conceptos Fundamentales

### 1. ¿Qué es el Monitoreo?

El **monitoreo** es el proceso continuo de observar y recopilar datos sobre el comportamiento de sistemas, aplicaciones e infraestructura para:

- Detectar anomalías y problemas
- Medir el rendimiento y la disponibilidad
- Validar el cumplimiento de objetivos de servicio
- Proporcionar visibilidad operacional

### 2. Tipos de Monitoreo

#### 2.1 Monitoreo de Infraestructura

```bash
# Ejemplos de métricas de infraestructura
- CPU utilization: 85%
- Memory usage: 12GB/16GB (75%)
- Disk I/O: 150 IOPS
- Network throughput: 100 Mbps
- Storage utilization: 80%
```

#### 2.2 Monitoreo de Aplicaciones

```javascript
// Ejemplos de métricas de aplicación
const metrics = {
  responseTime: "250ms",
  throughput: "1000 requests/min",
  errorRate: "0.1%",
  concurrentUsers: 150,
  databaseConnections: 25
};
```

#### 2.3 Monitoreo de Negocio

```yaml
# Métricas de negocio
business_metrics:
  - conversion_rate: "2.5%"
  - revenue_per_hour: "$1,250"
  - user_engagement: "15 min average session"
  - feature_adoption: "65%"
```

### 3. Los Cuatro Pilares de la Observabilidad

#### 3.1 Métricas (Metrics)

**Definición**: Datos numéricos que se recopilan a lo largo del tiempo.

```prometheus
# Ejemplo de métricas en formato Prometheus
http_requests_total{method="GET", status="200"} 1027
cpu_usage_percent{instance="web-01"} 45.2
memory_usage_bytes{instance="web-01"} 8589934592
```

**Características**:

- Agregables y analizables estadísticamente
- Eficientes en almacenamiento
- Ideales para dashboards y alertas
- Permiten identificar tendencias

#### 3.2 Logs (Registros)

**Definición**: Registros textuales de eventos discretos que ocurren en el sistema.

```json
{
  "timestamp": "2024-01-15T10:30:45Z",
  "level": "ERROR",
  "service": "user-api",
  "message": "Failed to connect to database",
  "user_id": "12345",
  "correlation_id": "abc-def-123",
  "stack_trace": "DatabaseConnectionException..."
}
```

**Características**:

- Proporcionan contexto detallado
- Útiles para debugging y análisis forense
- Pueden contener información sensible
- Requieren más almacenamiento

#### 3.3 Trazas (Traces)

**Definición**: Representan el camino de una solicitud a través de múltiples servicios.

```yaml
# Ejemplo de traza distribuida
trace_id: "5b8aa5a2d2c872e8321cf37308d69df2"
spans:
  - span_id: "051581bf3cb55c13"
    service: "api-gateway"
    operation: "http_request"
    duration: "245ms"
    
  - span_id: "5fb15cc4bf9aa8a2"
    service: "user-service"
    operation: "get_user"
    duration: "123ms"
    parent_span: "051581bf3cb55c13"
```

#### 3.4 Eventos (Events)

**Definición**: Ocurrencias discretas que pueden afectar el sistema.

```yaml
# Ejemplos de eventos
events:
  - type: "deployment"
    timestamp: "2024-01-15T09:00:00Z"
    service: "payment-api"
    version: "v2.1.0"
    
  - type: "alert"
    timestamp: "2024-01-15T10:15:30Z"
    severity: "critical"
    message: "High error rate detected"
```

---

## 🔧 Implementación Práctica

### 1. Configuración de Monitoreo Básico con Scripts

#### 1.1 Script de Monitoreo de Sistema (Bash)

```bash
#!/bin/bash
# system_monitor.sh - Monitor básico de sistema

# Configuración
LOG_FILE="/var/log/system_monitor.log"
ALERT_THRESHOLD_CPU=80
ALERT_THRESHOLD_MEMORY=85
ALERT_THRESHOLD_DISK=90

# Función para logging
log_message() {
    echo "[$(date '+%Y-%m-%d %H:%M:%S')] $1" | tee -a "$LOG_FILE"
}

# Monitoreo de CPU
check_cpu() {
    local cpu_usage=$(top -bn1 | grep "Cpu(s)" | awk '{print $2}' | cut -d'%' -f1)
    cpu_usage=${cpu_usage%.*}  # Remover decimales
    
    log_message "CPU Usage: ${cpu_usage}%"
    
    if [ "$cpu_usage" -gt "$ALERT_THRESHOLD_CPU" ]; then
        log_message "ALERT: High CPU usage detected: ${cpu_usage}%"
        # Enviar alerta (webhook, email, etc.)
        send_alert "High CPU" "$cpu_usage%"
    fi
}

# Monitoreo de Memoria
check_memory() {
    local memory_info=$(free | grep Mem)
    local total=$(echo $memory_info | awk '{print $2}')
    local used=$(echo $memory_info | awk '{print $3}')
    local memory_percent=$((used * 100 / total))
    
    log_message "Memory Usage: ${memory_percent}%"
    
    if [ "$memory_percent" -gt "$ALERT_THRESHOLD_MEMORY" ]; then
        log_message "ALERT: High memory usage detected: ${memory_percent}%"
        send_alert "High Memory" "${memory_percent}%"
    fi
}

# Monitoreo de Disco
check_disk() {
    local disk_usage=$(df -h / | awk 'NR==2 {print $5}' | cut -d'%' -f1)
    
    log_message "Disk Usage: ${disk_usage}%"
    
    if [ "$disk_usage" -gt "$ALERT_THRESHOLD_DISK" ]; then
        log_message "ALERT: High disk usage detected: ${disk_usage}%"
        send_alert "High Disk Usage" "${disk_usage}%"
    fi
}

# Función para enviar alertas
send_alert() {
    local alert_type="$1"
    local value="$2"
    
    # Ejemplo con webhook
    curl -X POST "https://hooks.slack.com/your/webhook/url" \
         -H 'Content-type: application/json' \
         --data "{\"text\":\"🚨 ALERT: $alert_type - $value\"}" 2>/dev/null
}

# Ejecutar monitoreo
main() {
    log_message "Starting system monitoring..."
    check_cpu
    check_memory
    check_disk
    log_message "Monitoring cycle completed"
}

# Ejecutar si es llamado directamente
if [[ "${BASH_SOURCE[0]}" == "${0}" ]]; then
    main
fi
```

#### 1.2 Script de Monitoreo de Aplicación (Python)

```python
#!/usr/bin/env python3
# app_monitor.py - Monitor de aplicación web

import requests
import time
import json
import logging
from datetime import datetime
from typing import Dict, Any

# Configuración de logging
logging.basicConfig(
    level=logging.INFO,
    format='%(asctime)s - %(levelname)s - %(message)s',
    handlers=[
        logging.FileHandler('/var/log/app_monitor.log'),
        logging.StreamHandler()
    ]
)

class ApplicationMonitor:
    def __init__(self, config_file: str = 'monitor_config.json'):
        self.config = self.load_config(config_file)
        self.logger = logging.getLogger(__name__)
        
    def load_config(self, config_file: str) -> Dict[str, Any]:
        """Carga la configuración desde archivo JSON"""
        try:
            with open(config_file, 'r') as f:
                return json.load(f)
        except FileNotFoundError:
            # Configuración por defecto
            return {
                "endpoints": [
                    {"url": "http://localhost:8080/health", "name": "API Health"},
                    {"url": "http://localhost:8080/api/users", "name": "User API"}
                ],
                "thresholds": {
                    "response_time": 2000,  # ms
                    "error_rate": 5.0  # %
                },
                "check_interval": 60  # segundos
            }
    
    def check_endpoint(self, endpoint: Dict[str, str]) -> Dict[str, Any]:
        """Verifica el estado de un endpoint"""
        start_time = time.time()
        
        try:
            response = requests.get(
                endpoint['url'], 
                timeout=10,
                headers={'User-Agent': 'AppMonitor/1.0'}
            )
            
            response_time = (time.time() - start_time) * 1000  # Convert to ms
            
            result = {
                'name': endpoint['name'],
                'url': endpoint['url'],
                'status_code': response.status_code,
                'response_time': round(response_time, 2),
                'is_healthy': response.status_code == 200,
                'timestamp': datetime.now().isoformat()
            }
            
            # Verificar umbral de tiempo de respuesta
            if response_time > self.config['thresholds']['response_time']:
                self.send_alert(
                    f"Slow response from {endpoint['name']}", 
                    f"Response time: {response_time:.2f}ms"
                )
            
            return result
            
        except requests.exceptions.RequestException as e:
            self.logger.error(f"Error checking {endpoint['name']}: {str(e)}")
            self.send_alert(
                f"Endpoint {endpoint['name']} is down", 
                str(e)
            )
            
            return {
                'name': endpoint['name'],
                'url': endpoint['url'],
                'status_code': 0,
                'response_time': 0,
                'is_healthy': False,
                'error': str(e),
                'timestamp': datetime.now().isoformat()
            }
    
    def send_alert(self, title: str, message: str):
        """Envía una alerta"""
        alert = {
            'title': title,
            'message': message,
            'timestamp': datetime.now().isoformat(),
            'severity': 'critical'
        }
        
        self.logger.warning(f"ALERT: {title} - {message}")
        
        # Aquí podrías integrar con servicios como:
        # - Slack
        # - PagerDuty
        # - Email
        # - Discord
        # - Teams
    
    def generate_metrics_report(self, results: list) -> Dict[str, Any]:
        """Genera un reporte de métricas"""
        if not results:
            return {}
        
        healthy_count = sum(1 for r in results if r.get('is_healthy', False))
        total_count = len(results)
        availability = (healthy_count / total_count) * 100
        
        avg_response_time = sum(
            r.get('response_time', 0) for r in results if r.get('is_healthy', False)
        ) / max(healthy_count, 1)
        
        report = {
            'timestamp': datetime.now().isoformat(),
            'total_endpoints': total_count,
            'healthy_endpoints': healthy_count,
            'availability_percent': round(availability, 2),
            'average_response_time': round(avg_response_time, 2),
            'endpoints_detail': results
        }
        
        return report
    
    def run_monitoring_cycle(self):
        """Ejecuta un ciclo completo de monitoreo"""
        self.logger.info("Starting monitoring cycle...")
        
        results = []
        for endpoint in self.config['endpoints']:
            result = self.check_endpoint(endpoint)
            results.append(result)
            self.logger.info(
                f"Checked {result['name']}: "
                f"Status={result.get('status_code', 'ERROR')}, "
                f"Response={result.get('response_time', 0)}ms"
            )
        
        # Generar reporte
        report = self.generate_metrics_report(results)
        self.logger.info(f"Monitoring cycle completed. Availability: {report['availability_percent']}%")
        
        return report
    
    def start_continuous_monitoring(self):
        """Inicia el monitoreo continuo"""
        self.logger.info("Starting continuous monitoring...")
        
        try:
            while True:
                self.run_monitoring_cycle()
                time.sleep(self.config['check_interval'])
                
        except KeyboardInterrupt:
            self.logger.info("Monitoring stopped by user")
        except Exception as e:
            self.logger.error(f"Monitoring error: {str(e)}")
            self.send_alert("Monitoring System Error", str(e))

if __name__ == "__main__":
    monitor = ApplicationMonitor()
    monitor.start_continuous_monitoring()
```

#### 1.3 Configuración del Monitor (JSON)

```json
{
  "endpoints": [
    {
      "url": "http://localhost:8080/health",
      "name": "API Health Check"
    },
    {
      "url": "http://localhost:8080/api/v1/users",
      "name": "Users API"
    },
    {
      "url": "http://localhost:3000",
      "name": "Frontend App"
    },
    {
      "url": "http://db.example.com:5432",
      "name": "Database Connection"
    }
  ],
  "thresholds": {
    "response_time": 2000,
    "error_rate": 5.0,
    "availability": 99.0
  },
  "check_interval": 60,
  "alert_settings": {
    "slack_webhook": "https://hooks.slack.com/your/webhook/url",
    "email_recipients": ["ops@company.com"],
    "pagerduty_key": "your-pagerduty-integration-key"
  },
  "retention": {
    "logs_days": 30,
    "metrics_days": 90
  }
}
```

### 2. Implementación con Docker y Docker Compose

#### 2.1 Dockerfile para Monitor

```dockerfile
# Dockerfile
FROM python:3.9-slim

WORKDIR /app

# Instalar dependencias del sistema
RUN apt-get update && apt-get install -y \
    curl \
    && rm -rf /var/lib/apt/lists/*

# Copiar requirements
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# Copiar código de la aplicación
COPY app_monitor.py .
COPY monitor_config.json .

# Crear directorio para logs
RUN mkdir -p /var/log

# Exponer puerto para métricas (opcional)
EXPOSE 9090

# Comando por defecto
CMD ["python", "app_monitor.py"]
```

#### 2.2 Docker Compose Setup

```yaml
# docker-compose.monitoring.yml
version: '3.8'

services:
  app-monitor:
    build: .
    container_name: app-monitor
    restart: unless-stopped
    volumes:
      - ./logs:/var/log
      - ./config:/app/config
    environment:
      - MONITOR_CONFIG=/app/config/monitor_config.json
      - LOG_LEVEL=INFO
    networks:
      - monitoring
    depends_on:
      - web-app
      - database

  web-app:
    image: nginx:alpine
    container_name: web-app
    ports:
      - "8080:80"
    volumes:
      - ./html:/usr/share/nginx/html
    networks:
      - monitoring

  database:
    image: postgres:13
    container_name: database
    environment:
      - POSTGRES_DB=myapp
      - POSTGRES_USER=user
      - POSTGRES_PASSWORD=password
    volumes:
      - postgres_data:/var/lib/postgresql/data
    networks:
      - monitoring

  # Monitor de sistema básico
  system-monitor:
    image: prom/node-exporter:latest
    container_name: system-monitor
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

volumes:
  postgres_data:

networks:
  monitoring:
    driver: bridge
```

### 3. Configuración de Alertas Básicas

#### 3.1 Sistema de Alertas Simple

```python
# alert_system.py
import smtplib
import json
from email.mime.text import MIMEText
from email.mime.multipart import MIMEMultipart
from typing import Dict, List
import requests

class AlertManager:
    def __init__(self, config: Dict):
        self.config = config
        
    def send_email_alert(self, subject: str, message: str, recipients: List[str]):
        """Envía alerta por email"""
        try:
            smtp_config = self.config.get('smtp', {})
            
            msg = MIMEMultipart()
            msg['From'] = smtp_config['from']
            msg['To'] = ', '.join(recipients)
            msg['Subject'] = f"[ALERT] {subject}"
            
            body = f"""
            ALERT NOTIFICATION
            
            Subject: {subject}
            Message: {message}
            Timestamp: {datetime.now().isoformat()}
            
            This is an automated alert from your monitoring system.
            """
            
            msg.attach(MIMEText(body, 'plain'))
            
            server = smtplib.SMTP(smtp_config['server'], smtp_config['port'])
            server.starttls()
            server.login(smtp_config['username'], smtp_config['password'])
            server.send_message(msg)
            server.quit()
            
            print(f"Email alert sent to {recipients}")
            
        except Exception as e:
            print(f"Failed to send email alert: {str(e)}")
    
    def send_slack_alert(self, title: str, message: str):
        """Envía alerta a Slack"""
        try:
            webhook_url = self.config.get('slack', {}).get('webhook_url')
            if not webhook_url:
                return
            
            payload = {
                "text": f"🚨 *{title}*",
                "attachments": [
                    {
                        "color": "danger",
                        "fields": [
                            {
                                "title": "Message",
                                "value": message,
                                "short": False
                            },
                            {
                                "title": "Timestamp",
                                "value": datetime.now().isoformat(),
                                "short": True
                            }
                        ]
                    }
                ]
            }
            
            response = requests.post(webhook_url, json=payload)
            response.raise_for_status()
            
            print("Slack alert sent successfully")
            
        except Exception as e:
            print(f"Failed to send Slack alert: {str(e)}")
    
    def send_webhook_alert(self, title: str, message: str, webhook_url: str):
        """Envía alerta via webhook genérico"""
        try:
            payload = {
                "alert": {
                    "title": title,
                    "message": message,
                    "timestamp": datetime.now().isoformat(),
                    "severity": "critical"
                }
            }
            
            response = requests.post(
                webhook_url, 
                json=payload,
                headers={'Content-Type': 'application/json'}
            )
            response.raise_for_status()
            
            print(f"Webhook alert sent to {webhook_url}")
            
        except Exception as e:
            print(f"Failed to send webhook alert: {str(e)}")
```

---

## 📊 Métricas Fundamentales

### 1. Métricas de Infraestructura

#### 1.1 USE Method (Utilization, Saturation, Errors)

```bash
# Utilización
cpu_utilization = (cpu_busy_time / total_time) * 100
memory_utilization = (used_memory / total_memory) * 100
disk_utilization = (disk_busy_time / total_time) * 100

# Saturación
cpu_queue_length = number_of_processes_waiting_for_cpu
memory_swap_usage = swap_used / swap_total
disk_queue_length = average_disk_queue_length

# Errores
network_packet_errors = dropped_packets + corrupted_packets
disk_errors = read_errors + write_errors
memory_errors = correctable_ecc_errors + uncorrectable_ecc_errors
```

#### 1.2 Script de Métricas USE

```bash
#!/bin/bash
# use_metrics.sh - Implementación del método USE

collect_cpu_metrics() {
    echo "=== CPU Metrics ==="
    
    # Utilización
    cpu_util=$(top -bn1 | grep "Cpu(s)" | awk '{print $2}' | cut -d'%' -f1)
    echo "CPU Utilization: ${cpu_util}%"
    
    # Saturación (load average)
    load_avg=$(uptime | awk -F'load average:' '{print $2}' | awk '{print $1}' | tr -d ',')
    echo "Load Average (1min): ${load_avg}"
    
    # Errores (context switches como proxy)
    context_switches=$(vmstat 1 2 | tail -1 | awk '{print $12}')
    echo "Context Switches/sec: ${context_switches}"
}

collect_memory_metrics() {
    echo "=== Memory Metrics ==="
    
    # Utilización
    memory_info=$(free -m)
    total=$(echo "$memory_info" | grep Mem | awk '{print $2}')
    used=$(echo "$memory_info" | grep Mem | awk '{print $3}')
    util_percent=$((used * 100 / total))
    echo "Memory Utilization: ${util_percent}%"
    
    # Saturación (swap usage)
    swap_total=$(echo "$memory_info" | grep Swap | awk '{print $2}')
    swap_used=$(echo "$memory_info" | grep Swap | awk '{print $3}')
    if [ "$swap_total" -gt 0 ]; then
        swap_percent=$((swap_used * 100 / swap_total))
        echo "Swap Usage: ${swap_percent}%"
    else
        echo "Swap Usage: 0% (no swap configured)"
    fi
}

collect_disk_metrics() {
    echo "=== Disk Metrics ==="
    
    # Utilización
    disk_util=$(iostat -x 1 2 | tail -n +4 | head -1 | awk '{print $10}')
    echo "Disk Utilization: ${disk_util}%"
    
    # Saturación (average queue size)
    avg_queue=$(iostat -x 1 2 | tail -n +4 | head -1 | awk '{print $9}')
    echo "Average Queue Size: ${avg_queue}"
    
    # Errores de espacio en disco
    disk_usage=$(df -h / | awk 'NR==2 {print $5}' | cut -d'%' -f1)
    echo "Disk Space Usage: ${disk_usage}%"
}

collect_network_metrics() {
    echo "=== Network Metrics ==="
    
    # Utilización (throughput)
    interface="eth0"  # Ajustar según tu interfaz
    rx_bytes=$(cat /sys/class/net/$interface/statistics/rx_bytes 2>/dev/null || echo "0")
    tx_bytes=$(cat /sys/class/net/$interface/statistics/tx_bytes 2>/dev/null || echo "0")
    echo "RX Bytes: $rx_bytes"
    echo "TX Bytes: $tx_bytes"
    
    # Errores
    rx_errors=$(cat /sys/class/net/$interface/statistics/rx_errors 2>/dev/null || echo "0")
    tx_errors=$(cat /sys/class/net/$interface/statistics/tx_errors 2>/dev/null || echo "0")
    echo "RX Errors: $rx_errors"
    echo "TX Errors: $tx_errors"
}

# Ejecutar todas las métricas
main() {
    echo "=== System Metrics Collection (USE Method) ==="
    echo "Timestamp: $(date)"
    echo ""
    
    collect_cpu_metrics
    echo ""
    collect_memory_metrics
    echo ""
    collect_disk_metrics
    echo ""
    collect_network_metrics
    echo ""
    echo "=== End of Metrics Collection ==="
}

main
```

### 2. Métricas de Aplicación

#### 2.1 RED Method (Rate, Errors, Duration)

```python
# red_metrics.py - Implementación del método RED
import time
import statistics
from collections import defaultdict, deque
from typing import Dict, List
import threading

class REDMetrics:
    def __init__(self, window_size: int = 300):  # 5 minutos
        self.window_size = window_size
        self.requests = deque()
        self.errors = deque()
        self.durations = deque()
        self.lock = threading.Lock()
        
    def record_request(self, duration: float, is_error: bool = False):
        """Registra una solicitud con su duración y estado de error"""
        now = time.time()
        
        with self.lock:
            # Limpiar datos antiguos
            self._clean_old_data(now)
            
            # Registrar nueva solicitud
            self.requests.append(now)
            self.durations.append(duration)
            
            if is_error:
                self.errors.append(now)
    
    def _clean_old_data(self, current_time: float):
        """Limpia datos más antiguos que la ventana de tiempo"""
        cutoff = current_time - self.window_size
        
        while self.requests and self.requests[0] < cutoff:
            self.requests.popleft()
            self.durations.popleft()
            
        while self.errors and self.errors[0] < cutoff:
            self.errors.popleft()
    
    def get_metrics(self) -> Dict[str, float]:
        """Obtiene las métricas RED actuales"""
        now = time.time()
        
        with self.lock:
            self._clean_old_data(now)
            
            # Rate (requests per second)
            rate = len(self.requests) / self.window_size if self.requests else 0
            
            # Errors (error rate percentage)
            error_rate = (len(self.errors) / len(self.requests) * 100) if self.requests else 0
            
            # Duration (response time statistics)
            if self.durations:
                avg_duration = statistics.mean(self.durations)
                p50_duration = statistics.median(self.durations)
                p95_duration = statistics.quantiles(self.durations, n=20)[18]  # 95th percentile
                p99_duration = statistics.quantiles(self.durations, n=100)[98]  # 99th percentile
            else:
                avg_duration = p50_duration = p95_duration = p99_duration = 0
            
            return {
                'rate_rps': round(rate, 2),
                'error_rate_percent': round(error_rate, 2),
                'duration_avg_ms': round(avg_duration * 1000, 2),
                'duration_p50_ms': round(p50_duration * 1000, 2),
                'duration_p95_ms': round(p95_duration * 1000, 2),
                'duration_p99_ms': round(p99_duration * 1000, 2),
                'total_requests': len(self.requests)
            }

# Ejemplo de uso con Flask
from flask import Flask, request, jsonify
import random

app = Flask(__name__)
metrics = REDMetrics()

@app.before_request
def before_request():
    request.start_time = time.time()

@app.after_request
def after_request(response):
    duration = time.time() - request.start_time
    is_error = response.status_code >= 400
    metrics.record_request(duration, is_error)
    return response

@app.route('/api/users')
def get_users():
    # Simular trabajo
    time.sleep(random.uniform(0.1, 0.5))
    
    # Simular errores ocasionales
    if random.random() < 0.1:  # 10% error rate
        return jsonify({'error': 'Service unavailable'}), 503
    
    return jsonify({'users': ['user1', 'user2', 'user3']})

@app.route('/metrics')
def get_metrics():
    return jsonify(metrics.get_metrics())

if __name__ == '__main__':
    app.run(debug=True, port=5000)
```

---

## 🚨 Configuración de Alertas

### 1. Tipos de Alertas

#### 1.1 Alertas por Umbral

```yaml
# alerts_config.yaml
alerts:
  cpu_high:
    metric: "cpu_utilization"
    threshold: 80
    comparison: ">"
    duration: "5m"
    severity: "warning"
    message: "CPU utilization is above 80%"
    
  memory_critical:
    metric: "memory_utilization"
    threshold: 95
    comparison: ">"
    duration: "2m"
    severity: "critical"
    message: "Memory utilization is critically high"
    
  disk_space:
    metric: "disk_usage_percent"
    threshold: 90
    comparison: ">"
    duration: "1m"
    severity: "critical"
    message: "Disk space is running low"
    
  response_time:
    metric: "avg_response_time_ms"
    threshold: 2000
    comparison: ">"
    duration: "3m"
    severity: "warning"
    message: "Average response time is too high"
    
  error_rate:
    metric: "error_rate_percent"
    threshold: 5
    comparison: ">"
    duration: "5m"
    severity: "critical"
    message: "Error rate is above acceptable threshold"
```

#### 1.2 Motor de Alertas

```python
# alert_engine.py
import yaml
import time
from typing import Dict, List, Any
from dataclasses import dataclass
from datetime import datetime, timedelta

@dataclass
class Alert:
    name: str
    metric: str
    threshold: float
    comparison: str
    duration: str
    severity: str
    message: str
    triggered_at: datetime = None
    is_active: bool = False

class AlertEngine:
    def __init__(self, config_file: str):
        self.alerts = self.load_alerts_config(config_file)
        self.alert_history = []
        
    def load_alerts_config(self, config_file: str) -> List[Alert]:
        """Carga configuración de alertas desde YAML"""
        with open(config_file, 'r') as f:
            config = yaml.safe_load(f)
        
        alerts = []
        for name, alert_config in config.get('alerts', {}).items():
            alert = Alert(
                name=name,
                metric=alert_config['metric'],
                threshold=alert_config['threshold'],
                comparison=alert_config['comparison'],
                duration=alert_config['duration'],
                severity=alert_config['severity'],
                message=alert_config['message']
            )
            alerts.append(alert)
        
        return alerts
    
    def parse_duration(self, duration_str: str) -> int:
        """Convierte duración string a segundos"""
        if duration_str.endswith('s'):
            return int(duration_str[:-1])
        elif duration_str.endswith('m'):
            return int(duration_str[:-1]) * 60
        elif duration_str.endswith('h'):
            return int(duration_str[:-1]) * 3600
        else:
            return int(duration_str)
    
    def evaluate_condition(self, value: float, threshold: float, comparison: str) -> bool:
        """Evalúa si se cumple la condición de alerta"""
        if comparison == '>':
            return value > threshold
        elif comparison == '>=':
            return value >= threshold
        elif comparison == '<':
            return value < threshold
        elif comparison == '<=':
            return value <= threshold
        elif comparison == '==':
            return value == threshold
        elif comparison == '!=':
            return value != threshold
        return False
    
    def check_alerts(self, current_metrics: Dict[str, float]):
        """Verifica todas las alertas contra las métricas actuales"""
        now = datetime.now()
        
        for alert in self.alerts:
            if alert.metric not in current_metrics:
                continue
            
            current_value = current_metrics[alert.metric]
            condition_met = self.evaluate_condition(
                current_value, 
                alert.threshold, 
                alert.comparison
            )
            
            if condition_met:
                if not alert.is_active:
                    # Primera vez que se activa la condición
                    alert.triggered_at = now
                    alert.is_active = True
                elif alert.triggered_at:
                    # Verificar si ha pasado suficiente tiempo
                    duration_seconds = self.parse_duration(alert.duration)
                    if (now - alert.triggered_at).total_seconds() >= duration_seconds:
                        # Disparar alerta
                        self.fire_alert(alert, current_value)
                        alert.triggered_at = now  # Reset para evitar spam
            else:
                # Condición no se cumple, resetear alerta
                if alert.is_active:
                    self.resolve_alert(alert)
                alert.is_active = False
                alert.triggered_at = None
    
    def fire_alert(self, alert: Alert, current_value: float):
        """Dispara una alerta"""
        alert_data = {
            'name': alert.name,
            'severity': alert.severity,
            'message': alert.message,
            'metric': alert.metric,
            'current_value': current_value,
            'threshold': alert.threshold,
            'timestamp': datetime.now().isoformat()
        }
        
        print(f"🚨 ALERT FIRED: {alert.name}")
        print(f"   Severity: {alert.severity}")
        print(f"   Message: {alert.message}")
        print(f"   Current Value: {current_value}")
        print(f"   Threshold: {alert.threshold}")
        
        # Aquí integrarías con sistemas de notificación
        self.send_notification(alert_data)
        self.alert_history.append(alert_data)
    
    def resolve_alert(self, alert: Alert):
        """Resuelve una alerta"""
        print(f"✅ ALERT RESOLVED: {alert.name}")
        
        resolve_data = {
            'name': alert.name,
            'status': 'resolved',
            'timestamp': datetime.now().isoformat()
        }
        
        self.send_notification(resolve_data)
    
    def send_notification(self, alert_data: Dict[str, Any]):
        """Envía notificación de alerta"""
        # Implementar integración con:
        # - Slack
        # - Email
        # - PagerDuty
        # - Discord
        # - Teams
        pass
    
    def get_active_alerts(self) -> List[Dict[str, Any]]:
        """Obtiene alertas activas"""
        return [
            {
                'name': alert.name,
                'severity': alert.severity,
                'message': alert.message,
                'triggered_at': alert.triggered_at.isoformat() if alert.triggered_at else None
            }
            for alert in self.alerts if alert.is_active
        ]

# Ejemplo de uso
if __name__ == "__main__":
    engine = AlertEngine('alerts_config.yaml')
    
    # Simular métricas
    sample_metrics = {
        'cpu_utilization': 85.5,
        'memory_utilization': 78.2,
        'disk_usage_percent': 92.1,
        'avg_response_time_ms': 2500,
        'error_rate_percent': 8.3
    }
    
    engine.check_alerts(sample_metrics)
    print("\nActive Alerts:", engine.get_active_alerts())
```

### 2. Dashboard Básico con HTML/JavaScript

```html
<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Sistema de Monitoreo</title>
    <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
    <style>
        body { 
            font-family: Arial, sans-serif; 
            margin: 0; 
            padding: 20px; 
            background-color: #f5f5f5; 
        }
        .dashboard { 
            display: grid; 
            grid-template-columns: repeat(auto-fit, minmax(300px, 1fr)); 
            gap: 20px; 
        }
        .card { 
            background: white; 
            padding: 20px; 
            border-radius: 8px; 
            box-shadow: 0 2px 4px rgba(0,0,0,0.1); 
        }
        .metric { 
            display: flex; 
            justify-content: space-between; 
            align-items: center; 
            margin: 10px 0; 
        }
        .metric-value { 
            font-size: 24px; 
            font-weight: bold; 
        }
        .status-good { color: #28a745; }
        .status-warning { color: #ffc107; }
        .status-critical { color: #dc3545; }
        .alert { 
            padding: 10px; 
            margin: 5px 0; 
            border-radius: 4px; 
            border-left: 4px solid; 
        }
        .alert-warning { 
            background-color: #fff3cd; 
            border-color: #ffc107; 
        }
        .alert-critical { 
            background-color: #f8d7da; 
            border-color: #dc3545; 
        }
    </style>
</head>
<body>
    <h1>🖥️ Dashboard de Monitoreo</h1>
    
    <div class="dashboard">
        <!-- Métricas de Sistema -->
        <div class="card">
            <h3>📊 Métricas de Sistema</h3>
            <div class="metric">
                <span>CPU:</span>
                <span id="cpu-value" class="metric-value status-good">--</span>
            </div>
            <div class="metric">
                <span>Memoria:</span>
                <span id="memory-value" class="metric-value status-good">--</span>
            </div>
            <div class="metric">
                <span>Disco:</span>
                <span id="disk-value" class="metric-value status-good">--</span>
            </div>
        </div>
        
        <!-- Métricas de Aplicación -->
        <div class="card">
            <h3>🚀 Métricas de Aplicación</h3>
            <div class="metric">
                <span>Requests/sec:</span>
                <span id="rps-value" class="metric-value status-good">--</span>
            </div>
            <div class="metric">
                <span>Tiempo Respuesta:</span>
                <span id="response-time-value" class="metric-value status-good">--</span>
            </div>
            <div class="metric">
                <span>Tasa de Error:</span>
                <span id="error-rate-value" class="metric-value status-good">--</span>
            </div>
        </div>
        
        <!-- Alertas Activas -->
        <div class="card">
            <h3>🚨 Alertas Activas</h3>
            <div id="alerts-container">
                <p>No hay alertas activas</p>
            </div>
        </div>
        
        <!-- Gráfico de CPU -->
        <div class="card">
            <h3>📈 CPU Utilización</h3>
            <canvas id="cpu-chart"></canvas>
        </div>
    </div>

    <script>
        // Configuración del gráfico
        const ctx = document.getElementById('cpu-chart').getContext('2d');
        const cpuChart = new Chart(ctx, {
            type: 'line',
            data: {
                labels: [],
                datasets: [{
                    label: 'CPU %',
                    data: [],
                    borderColor: 'rgb(75, 192, 192)',
                    backgroundColor: 'rgba(75, 192, 192, 0.2)',
                    tension: 0.1
                }]
            },
            options: {
                responsive: true,
                scales: {
                    y: {
                        beginAtZero: true,
                        max: 100
                    }
                },
                animation: {
                    duration: 0
                }
            }
        });
        
        // Función para actualizar métricas
        async function updateMetrics() {
            try {
                // Simular datos (en producción, estos vendrían de tu API)
                const metrics = {
                    cpu: Math.random() * 100,
                    memory: Math.random() * 100,
                    disk: Math.random() * 100,
                    rps: Math.random() * 1000,
                    responseTime: Math.random() * 500,
                    errorRate: Math.random() * 10
                };
                
                // Actualizar valores en la UI
                updateMetricValue('cpu-value', metrics.cpu, '%');
                updateMetricValue('memory-value', metrics.memory, '%');
                updateMetricValue('disk-value', metrics.disk, '%');
                updateMetricValue('rps-value', metrics.rps.toFixed(0), '');
                updateMetricValue('response-time-value', metrics.responseTime.toFixed(0), 'ms');
                updateMetricValue('error-rate-value', metrics.errorRate.toFixed(1), '%');
                
                // Actualizar gráfico
                const now = new Date().toLocaleTimeString();
                cpuChart.data.labels.push(now);
                cpuChart.data.datasets[0].data.push(metrics.cpu);
                
                // Mantener solo los últimos 20 puntos
                if (cpuChart.data.labels.length > 20) {
                    cpuChart.data.labels.shift();
                    cpuChart.data.datasets[0].data.shift();
                }
                
                cpuChart.update();
                
                // Simular alertas
                updateAlerts(metrics);
                
            } catch (error) {
                console.error('Error updating metrics:', error);
            }
        }
        
        function updateMetricValue(elementId, value, unit) {
            const element = document.getElementById(elementId);
            element.textContent = value.toFixed(1) + unit;
            
            // Cambiar color basado en el valor
            element.className = 'metric-value';
            if (value > 80) {
                element.classList.add('status-critical');
            } else if (value > 60) {
                element.classList.add('status-warning');
            } else {
                element.classList.add('status-good');
            }
        }
        
        function updateAlerts(metrics) {
            const alertsContainer = document.getElementById('alerts-container');
            let alerts = [];
            
            if (metrics.cpu > 80) {
                alerts.push({
                    severity: 'critical',
                    message: `CPU utilization is high: ${metrics.cpu.toFixed(1)}%`
                });
            }
            
            if (metrics.memory > 85) {
                alerts.push({
                    severity: 'critical',
                    message: `Memory utilization is high: ${metrics.memory.toFixed(1)}%`
                });
            }
            
            if (metrics.responseTime > 300) {
                alerts.push({
                    severity: 'warning',
                    message: `Response time is slow: ${metrics.responseTime.toFixed(0)}ms`
                });
            }
            
            if (alerts.length === 0) {
                alertsContainer.innerHTML = '<p>No hay alertas activas</p>';
            } else {
                alertsContainer.innerHTML = alerts.map(alert => 
                    `<div class="alert alert-${alert.severity}">
                        <strong>${alert.severity.toUpperCase()}:</strong> ${alert.message}
                    </div>`
                ).join('');
            }
        }
        
        // Actualizar métricas cada 5 segundos
        setInterval(updateMetrics, 5000);
        
        // Actualización inicial
        updateMetrics();
    </script>
</body>
</html>
```

---

## 🔄 Automatización con Cron

### 1. Configuración de Tareas Cron

```bash
# crontab -e
# Ejecutar monitoreo cada minuto
* * * * * /opt/monitoring/system_monitor.sh >> /var/log/monitoring.log 2>&1

# Ejecutar reporte diario a las 6 AM
0 6 * * * /opt/monitoring/daily_report.sh

# Limpiar logs antiguos cada domingo a las 2 AM
0 2 * * 0 find /var/log/monitoring* -name "*.log" -mtime +30 -delete

# Backup de métricas cada día a las 1 AM
0 1 * * * /opt/monitoring/backup_metrics.sh
```

### 2. Script de Reporte Diario

```bash
#!/bin/bash
# daily_report.sh

REPORT_DATE=$(date +%Y-%m-%d)
REPORT_FILE="/var/log/daily_report_${REPORT_DATE}.txt"

echo "Daily System Report - $REPORT_DATE" > "$REPORT_FILE"
echo "=====================================" >> "$REPORT_FILE"
echo "" >> "$REPORT_FILE"

# Resumen de sistema
echo "System Summary:" >> "$REPORT_FILE"
echo "- Uptime: $(uptime -p)" >> "$REPORT_FILE"
echo "- Load Average: $(uptime | awk -F'load average:' '{print $2}')" >> "$REPORT_FILE"
echo "- Memory Usage: $(free -h | grep Mem | awk '{print $3 "/" $2}')" >> "$REPORT_FILE"
echo "- Disk Usage: $(df -h / | awk 'NR==2 {print $5 " used"}')" >> "$REPORT_FILE"
echo "" >> "$REPORT_FILE"

# Análisis de logs
echo "Log Analysis:" >> "$REPORT_FILE"
echo "- Total log entries: $(wc -l /var/log/monitoring.log | awk '{print $1}')" >> "$REPORT_FILE"
echo "- Alerts fired: $(grep -c 'ALERT:' /var/log/monitoring.log)" >> "$REPORT_FILE"
echo "- Errors detected: $(grep -c 'ERROR' /var/log/monitoring.log)" >> "$REPORT_FILE"
echo "" >> "$REPORT_FILE"

# Top processes by CPU
echo "Top CPU Processes:" >> "$REPORT_FILE"
ps aux --sort=-%cpu | head -10 >> "$REPORT_FILE"
echo "" >> "$REPORT_FILE"

# Enviar reporte por email (opcional)
# mail -s "Daily System Report - $REPORT_DATE" admin@company.com < "$REPORT_FILE"
```

---

## 📋 Ejercicios Prácticos

### Ejercicio 1: Monitor Básico de Sistema

**Objetivo**: Crear un script que monitoree CPU, memoria y disco cada 30 segundos.

**Requisitos**:

1. Mostrar métricas en formato JSON
2. Alertar cuando CPU > 80%, Memoria > 85%, Disco > 90%
3. Mantener historial de últimas 10 mediciones
4. Enviar alertas a un webhook

**Solución**:

```bash
#!/bin/bash
# Implementar el monitor aquí
```

### Ejercicio 2: API de Métricas con Flask

**Objetivo**: Crear una API REST que exponga métricas del sistema.

**Endpoints requeridos**:

- `GET /metrics` - Métricas actuales
- `GET /metrics/history` - Historial de métricas
- `GET /health` - Estado de salud del sistema
- `POST /alerts/test` - Probar sistema de alertas

### Ejercicio 3: Dashboard en Tiempo Real

**Objetivo**: Crear un dashboard web que se actualice automáticamente.

**Características**:

- Gráficos en tiempo real
- Sistema de alertas visual
- Responsive design
- Actualización cada 5 segundos

---

## 🧪 Laboratorio

### Lab 1: Implementación Completa de Monitoreo

1. **Configurar entorno**:

   ```bash
   mkdir monitoring-lab
   cd monitoring-lab
   
   # Crear estructura
   mkdir -p {scripts,config,logs,web}
   ```

2. **Implementar scripts de monitoreo**
3. **Configurar alertas**
4. **Crear dashboard básico**
5. **Probar escenarios de falla**

### Lab 2: Integración con Docker

1. **Dockerizar el sistema de monitoreo**
2. **Configurar docker-compose**
3. **Implementar health checks**
4. **Monitorear contenedores**

---

## 📖 Recursos Adicionales

### Documentación

- [Prometheus Documentation](https://prometheus.io/docs/)
- [Grafana Documentation](https://grafana.com/docs/)
- [ELK Stack Guide](https://www.elastic.co/guide/)

### Herramientas

- **htop** - Monitor interactivo de procesos
- **iotop** - Monitor de I/O de disco
- **netstat** - Estadísticas de red
- **vmstat** - Estadísticas de memoria virtual

### Best Practices

1. **Monitorear lo que importa** - No todo necesita ser monitoreado
2. **Establecer baselines** - Conocer el comportamiento normal
3. **Alertas inteligentes** - Evitar la fatiga de alertas
4. **Documentar métricas** - Cada métrica debe tener propósito claro
5. **Review regular** - Ajustar umbrales y métricas periódicamente

---

## ✅ Resumen del Módulo

En este módulo has aprendido:

1. **Conceptos fundamentales** del monitoreo y observabilidad
2. **Tipos de métricas** (infraestructura, aplicación, negocio)
3. **Los cuatro pilares** de la observabilidad (métricas, logs, trazas, eventos)
4. **Implementación práctica** de sistemas de monitoreo básicos
5. **Configuración de alertas** efectivas y no intrusivas
6. **Métricas USE y RED** como metodologías de monitoreo
7. **Herramientas básicas** para recopilación y visualización

### Próximas etapas

- **Módulo 02**: Profundizaremos en Prometheus para recolección avanzada de métricas
- **Módulo 03**: Exploraremos Grafana para visualización profesional
- **Módulo 04**: Implementaremos el stack ELK para gestión de logs

¡Continúa con el siguiente módulo para dominar las herramientas profesionales de monitoreo!
