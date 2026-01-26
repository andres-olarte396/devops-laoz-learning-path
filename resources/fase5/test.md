# Test Fase 5: Monitoring, Observability, and Security

## 📋 Información del Test

- **Fase**: 5 - Monitoring, Observability, and Security
- **Duración estimada**: 90-120 minutos
- **Modalidad**: Teórico-Práctico
- **Puntuación total**: 100 puntos
- **Puntuación mínima para aprobar**: 70 puntos

### Distribución de Puntos por Módulo

| Módulo | Descripción                                                                   | Puntos |
| ------ | ----------------------------------------------------------------------------- | ------ |
| 01-03  | Monitoring Fundamentals (Prometheus, Grafana, ELK)                            | 15     |
| 04-05  | Logging & Alerting (Structured Logs, Alertmanager)                            | 15     |
| 06-09  | DevSecOps & Security (DevSecOps, Secrets, Compliance, Vulnerability Scanning) | 25     |
| 10-11  | Observability (Distributed Tracing, APM)                                      | 20     |
| 12-13  | Resilience & Integration (Health Checks, Circuit Breakers, Lab Final)         | 25     |

---

## 📝 SECCIÓN 1: MONITORING FUNDAMENTALS (15 puntos)

### 1.1 Prometheus y Métricas (5 puntos)

**Pregunta 1** (2 puntos): Explica los 4 tipos de métricas en Prometheus y proporciona un ejemplo de uso para cada uno.

```
Respuesta:
1. Counter: ____________________
   Ejemplo: ____________________

2. Gauge: ______________________
   Ejemplo: ____________________

3. Histogram: __________________
   Ejemplo: ____________________

4. Summary: ____________________
   Ejemplo: ____________________
```

**Pregunta 2** (3 puntos): Escribe una consulta PromQL para cada escenario:

a) Tasa de errores HTTP 5xx en los últimos 5 minutos:

```promql
# Tu respuesta aquí
```

b) Percentil 95 de latencia de requests:

```promql
# Tu respuesta aquí
```

c) Uso de CPU promedio por servicio en la última hora:

```promql
# Tu respuesta aquí
```

### 1.2 Grafana y Visualización (5 puntos)

**Pregunta 3** (3 puntos): Describe las diferencias entre los siguientes tipos de panel en Grafana y cuándo usar cada uno:

- **Time Series Panel**:
- **Stat Panel**:
- **Table Panel**:
- **Heatmap Panel**:

**Pregunta 4** (2 puntos): ¿Qué son los "template variables" en Grafana y cómo mejoran la reutilización de dashboards?

```
Respuesta:
```

### 1.3 ELK Stack (5 puntos)

**Pregunta 5** (3 puntos): Completa la configuración de Logstash para procesar logs de aplicación web:

```ruby
input {
  beats {
    port => 5044
  }
}

filter {
  if [fields][log_type] == "webapp" {
    grok {
      match => {
        "message" => "%{TIMESTAMP_ISO8601:timestamp} %{LOGLEVEL:level} %{DATA:logger} - %{GREEDYDATA:message}"
      }
    }

    # Tu configuración aquí para:
    # 1. Parsear el timestamp
    # 2. Agregar campo "environment"
    # 3. Filtrar logs de nivel DEBUG
  }
}

output {
  # Tu configuración de output
}
```

**Pregunta 6** (2 puntos): ¿Cuál es la diferencia entre Elasticsearch, Logstash y Kibana en el stack ELK?

```
Elasticsearch: _______________
Logstash: ___________________
Kibana: ____________________
```

---

## 🚨 SECCIÓN 2: LOGGING & ALERTING (15 puntos)

### 2.1 Structured Logging (7 puntos)

**Pregunta 7** (4 puntos): Convierte este log no estructurado a formato JSON estructurado:

```
Log original: "2023-10-13 14:30:25 ERROR UserService - Failed to authenticate user john.doe@example.com from IP 192.168.1.100 - Invalid password attempt 3/3"
```

```json
{
  // Tu respuesta JSON aquí
}
```

**Pregunta 8** (3 puntos): Lista 5 campos que siempre deberían incluirse en logs estructurados para microservicios:

1. ***
2. ***
3. ***
4. ***
5. ***

### 2.2 Alertmanager y Alerting (8 puntos)

**Pregunta 9** (4 puntos): Configura una regla de alerta en Prometheus para detectar cuando un servicio tiene más de 5% de error rate durante 3 minutos consecutivos:

```yaml
groups:
  - name: service-alerts
    rules:
      - alert: # Tu configuración aquí
```

**Pregunta 10** (4 puntos): Diseña una estrategia de routing de alertas en Alertmanager:

- **Alertas críticas** → Slack + PagerDuty + Email
- **Alertas de warning** → Slack + Email
- **Alertas de info** → Solo Email

```yaml
route:
  # Tu configuración aquí
```

---

## 🔒 SECCIÓN 3: DEVSECOPS & SECURITY (25 puntos)

### 3.1 DevSecOps Fundamentals (6 puntos)

**Pregunta 11** (3 puntos): Explica qué significa "Shift Left Security" y proporciona 3 ejemplos prácticos:

```
Definición: ____________________

Ejemplos:
1. ___________________________
2. ___________________________
3. ___________________________
```

**Pregunta 12** (3 puntos): Identifica el tipo de security testing para cada herramienta:

- **SonarQube**: ******\_\_\_\_******
- **OWASP ZAP**: ******\_\_\_\_******
- **Dependency-Track**: ****\_\_****
- **Trivy**: ********\_\_\_\_********
- **Snyk**: ********\_\_\_\_********
- **Bandit**: ********\_\_\_********

### 3.2 Secret Management (6 puntos)

**Pregunta 13** (4 puntos): Completa esta configuración de HashiCorp Vault para una aplicación Spring Boot:

```yaml
# Vault policy
path "secret/data/myapp/*" {
  capabilities = ["read"]
}

path "database/creds/myapp-role" {
  capabilities = ["_______"]  # Tu respuesta
}
```

```yaml
# Spring Boot configuration
spring:
  cloud:
    vault:
      host: vault.company.com
      port: 8200
      scheme: https
      authentication: _______ # Tu respuesta
      generic:
        enabled: true
        backend: _______ # Tu respuesta
```

**Pregunta 14** (2 puntos): ¿Cuáles son las ventajas de usar dynamic secrets vs static secrets?

```
Dynamic Secrets:
- Ventaja 1: ____________________
- Ventaja 2: ____________________

Static Secrets:
- Desventaja 1: _________________
- Desventaja 2: _________________
```

### 3.3 Compliance y Auditoría (7 puntos)

**Pregunta 15** (4 puntos): Mapea estos controles de SOX a implementaciones técnicas:

| Control SOX       | Implementación Técnica |
| ----------------- | ---------------------- |
| Access Control    | ********\_********     |
| Change Management | ******\_******         |
| Data Integrity    | ******\_\_\_******     |
| Audit Trail       | ********\_********     |

**Pregunta 16** (3 puntos): Diseña un proceso de compliance automation para PCI-DSS:

```
1. Scanning automatizado: _________________
2. Remediation tracking: __________________
3. Reporting: ____________________________
```

### 3.4 Vulnerability Scanning (6 puntos)

**Pregunta 17** (3 puntos): Configura un pipeline de CI/CD con security gates:

```yaml
stages:
  - build
  - sast # Tu herramienta: _________
  - dependency # Tu herramienta: _________
  - deploy-staging
  - dast # Tu herramienta: _________
  - deploy-prod
# Criterios para pasar security gates:
# SAST: _____________________________
# Dependency: _______________________
# DAST: _____________________________
```

**Pregunta 18** (3 puntos): ¿Cuál es la diferencia entre SAST, DAST y SCA? Proporciona un ejemplo de vulnerabilidad que cada uno detectaría:

```
SAST: _____________________________
Ejemplo: __________________________

DAST: _____________________________
Ejemplo: __________________________

SCA: ______________________________
Ejemplo: __________________________
```

---

## 📊 SECCIÓN 4: OBSERVABILITY (20 puntos)

### 4.1 Distributed Tracing (10 puntos)

**Pregunta 19** (4 puntos): Explica los conceptos de Distributed Tracing:

- **Trace**: **********\_\_\_\_**********
- **Span**: ************\_************
- **Baggage**: **********\_\_**********
- **Sampling**: **********\_**********

**Pregunta 20** (3 puntos): Configura OpenTelemetry para una aplicación Java:

```java
@Configuration
public class TracingConfiguration {

    @Bean
    public OpenTelemetry openTelemetry() {
        return OpenTelemetrySdk.builder()
            .setTracerProvider(
                SdkTracerProvider.builder()
                    .addSpanProcessor(/* Tu configuración */)
                    .setResource(/* Tu configuración */)
                    .setSampler(/* Tu configuración */)
                    .build())
            .build();
    }
}
```

**Pregunta 21** (3 puntos): ¿Cómo ayuda el distributed tracing a resolver estos problemas?

- **Latencia alta en microservicios**: ********\_********
- **Error rate aumentando**: **********\_\_\_**********
- **Dependency mapping**: ************\_\_************

### 4.2 Application Performance Monitoring (10 puntos)

**Pregunta 22** (4 puntos): Configura Elastic APM para una aplicación React:

```javascript
import { init as initApm } from "@elastic/apm-rum";

const apm = initApm({
  serviceName: "my-react-app",
  serverUrl: "http://localhost:8200",
  serviceVersion: "1.0.0",
  environment: "production",

  // Tu configuración para:
  // 1. Distributed tracing origins
  // 2. Transaction sample rate
  // 3. Custom context
  // 4. User context
});
```

**Pregunta 23** (3 puntos): ¿Qué métricas de APM son más importantes para cada stakeholder?

- **Developers**: ********\_\_\_********
- **Operations**: ********\_\_********
- **Business**: ********\_\_\_\_********

**Pregunta 24** (3 puntos): Define estos conceptos de APM:

- **Real User Monitoring (RUM)**: ********\_\_\_********
- **Synthetic Monitoring**: **********\_\_\_**********
- **Error Tracking**: ************\_\_\_\_************

---

## 🏥 SECCIÓN 5: RESILIENCE & INTEGRATION (25 puntos)

### 5.1 Health Checks (8 puntos)

**Pregunta 25** (4 puntos): Implementa un health check comprehensivo para Spring Boot:

```java
@Component
public class DatabaseHealthIndicator implements HealthIndicator {

    @Autowired
    private DataSource dataSource;

    @Override
    public Health health() {
        try {
            // Tu implementación aquí
            // Debe verificar:
            // 1. Conexión a base de datos
            // 2. Tiempo de respuesta
            // 3. Estado de conexiones

            return Health.up()
                .withDetail("database", "PostgreSQL")
                .withDetail("status", "Connected")
                // Más detalles...
                .build();
        } catch (Exception e) {
            // Tu manejo de errores
        }
    }
}
```

**Pregunta 26** (4 puntos): Configura probes de Kubernetes:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app
spec:
  template:
    spec:
      containers:
        - name: app
          image: my-app:latest

          # Tu configuración de probes:
          startupProbe:
            # Configuración startup probe

          livenessProbe:
            # Configuración liveness probe

          readinessProbe:
            # Configuración readiness probe
```

### 5.2 Circuit Breakers (8 puntos)

**Pregunta 27** (4 puntos): Configura Resilience4j Circuit Breaker:

```java
@Configuration
public class CircuitBreakerConfig {

    @Bean
    public CircuitBreaker externalServiceCircuitBreaker() {
        CircuitBreakerConfig config = CircuitBreakerConfig.custom()
            .failureRateThreshold(/* Tu configuración */)
            .waitDurationInOpenState(/* Tu configuración */)
            .slidingWindowSize(/* Tu configuración */)
            // Más configuración...
            .build();

        return CircuitBreaker.of("externalService", config);
    }
}
```

**Pregunta 28** (4 puntos): Explica los estados del Circuit Breaker y cuándo transiciona entre ellos:

```
CLOSED → OPEN: _____________________
OPEN → HALF_OPEN: __________________
HALF_OPEN → CLOSED: ________________
HALF_OPEN → OPEN: __________________
```

### 5.3 SLIs/SLOs y Monitoring Strategy (9 puntos)

**Pregunta 29** (4 puntos): Define SLIs y SLOs para un e-commerce:

```
SLI 1 - Availability:
Métrica: ___________________________
SLO: ______________________________

SLI 2 - Latency:
Métrica: ___________________________
SLO: ______________________________

SLI 3 - Error Rate:
Métrica: ___________________________
SLO: ______________________________

SLI 4 - Throughput:
Métrica: ___________________________
SLO: ______________________________
```

**Pregunta 30** (5 puntos): Diseña una estrategia de monitoring completa:

```
1. Golden Signals a monitorear:
   - ______________________________
   - ______________________________
   - ______________________________
   - ______________________________

2. Herramientas por capa:
   - Infrastructure: ________________
   - Application: __________________
   - Business: ____________________
   - Security: ____________________

3. Estrategia de alertas:
   - Críticas: _____________________
   - Warning: _____________________
   - Info: ________________________

4. Dashboards necesarios:
   - Executive: ____________________
   - Operations: __________________
   - Development: _________________
```

---

## 🧪 SECCIÓN PRÁCTICA: TROUBLESHOOTING SCENARIO (Bonus: 10 puntos)

### Escenario de Incident Response

Tu aplicación e-commerce está experimentando los siguientes síntomas:

- Latencia P95 aumentó de 200ms a 2000ms
- Error rate subió de 0.1% a 5%
- Usuarios reportan timeouts en el checkout
- CPU y memoria de los pods están normales

**Pregunta 31** (10 puntos): Describe tu proceso de troubleshooting paso a paso:

```
1. Herramientas que usarías para investigar:
   - ________________________________
   - ________________________________
   - ________________________________

2. Métricas específicas a revisar:
   - ________________________________
   - ________________________________
   - ________________________________

3. Dashboards/queries que ejecutarías:
   - ________________________________
   - ________________________________
   - ________________________________

4. Posibles causas raíz y cómo validarlas:
   - Causa 1: _______________________
     Validación: ____________________

   - Causa 2: _______________________
     Validación: ____________________

   - Causa 3: _______________________
     Validación: ____________________

5. Acciones de mitigación inmediata:
   - ________________________________
   - ________________________________
   - ________________________________

6. Prevención de futuros incidentes:
   - ________________________________
   - ________________________________
   - ________________________________
```

---

## 📊 CRITERIOS DE EVALUACIÓN

### Distribución de Puntos

| Sección                      | Puntos Máximos | Criterios                                                        |
| ---------------------------- | -------------- | ---------------------------------------------------------------- |
| **Monitoring Fundamentals**  | 15             | Conocimiento de Prometheus, PromQL, Grafana, ELK                 |
| **Logging & Alerting**       | 15             | Structured logging, reglas de alerta, routing                    |
| **DevSecOps & Security**     | 25             | Security scanning, secrets, compliance, vulnerability management |
| **Observability**            | 20             | Distributed tracing, APM, OpenTelemetry                          |
| **Resilience & Integration** | 25             | Health checks, circuit breakers, SLIs/SLOs                       |
| **Troubleshooting (Bonus)**  | 10             | Incident response, problem-solving approach                      |

### Escala de Calificación

- **90-100 puntos**: Excelente - Expertise avanzado
- **80-89 puntos**: Muy Bueno - Conocimiento sólido
- **70-79 puntos**: Bueno - Conocimiento básico suficiente
- **60-69 puntos**: Regular - Necesita refuerzo
- **< 60 puntos**: Insuficiente - Requiere estudio adicional

---

## ✅ RESPUESTAS MODELO

### Monitoring Fundamentals

**1.1**: Counter (requests_total), Gauge (memory_usage), Histogram (request_duration), Summary (request_size)

**1.2**:

```promql
# a) rate(http_requests_total{status=~"5.."}[5m])
# b) histogram_quantile(0.95, rate(http_request_duration_seconds_bucket[5m]))
# c) avg_over_time(cpu_usage_percent[1h]) by (service)
```

### DevSecOps & Security

**3.1**: Shift Left = integrar security desde desarrollo temprano

**3.2**: Vault authentication debe ser kubernetes/token, backend debe ser secret

### Observability

**4.1**: Trace = request completo, Span = operación individual, Baggage = metadata propagado

### Resilience

**5.1**: CLOSED→OPEN cuando failure rate > threshold, OPEN→HALF_OPEN después de wait duration

---

## 📝 INSTRUCCIONES PARA EL EXAMINADO

1. **Tiempo**: Máximo 120 minutos
2. **Recursos permitidos**: Documentación oficial de herramientas
3. **Formato**: Completar directamente en este documento
4. **Entrega**: Enviar archivo completado
5. **Consultas**: Solo para aclaraciones de enunciados

### Antes de comenzar

- [ ] Leer todas las preguntas
- [ ] Planificar el tiempo por sección
- [ ] Verificar entendimiento de cada pregunta

### Durante el examen

- [ ] Responder preguntas conocidas primero
- [ ] Dejar tiempo para revisión final
- [ ] Verificar que todas las respuestas estén completas

¡Buena suerte! 🚀

---

**Nombre del examinado**: **********\_\_\_\_**********

**Fecha**: ******************\_\_******************

**Puntuación obtenida**: **\_** / 100 puntos

**Calificación**: **************\_\_\_**************
