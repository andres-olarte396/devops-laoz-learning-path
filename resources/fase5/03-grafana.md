# Módulo 03: Grafana - Visualización y Dashboards

## 📋 Introducción

Grafana es la plataforma de visualización y observabilidad líder en el mercado, que transforma métricas en dashboards interactivos y comprensibles. En este módulo aprenderás a crear visualizaciones profesionales, configurar alertas avanzadas y diseñar dashboards que proporcionen insights accionables.

### ¿Por qué Grafana?

Grafana es esencial en el ecosistema de monitoreo porque:

- **Múltiples fuentes de datos** - Prometheus, InfluxDB, Elasticsearch, etc.
- **Visualizaciones ricas** - Gráficos, mapas de calor, tablas, gauges
- **Alerting avanzado** - Notificaciones multi-canal con lógica compleja
- **Templating** - Dashboards dinámicos y reutilizables
- **Plugins ecosystem** - Extensibilidad y personalización
- **Enterprise features** - RBAC, auditoría, reporting

---

## 🎯 Objetivos de Aprendizaje

Al completar este módulo, serás capaz de:

1. **Instalar y configurar** Grafana en diferentes entornos
2. **Conectar múltiples datasources** incluyendo Prometheus
3. **Crear paneles** con diferentes tipos de visualizaciones
4. **Diseñar dashboards** profesionales y efectivos
5. **Implementar templating** para dashboards dinámicos
6. **Configurar alertas** avanzadas con múltiples canales
7. **Gestionar usuarios** y permisos
8. **Implementar best practices** de diseño y UX

---

## 🏗️ Instalación y Configuración

### 1. Instalación con Docker

#### 1.1 Docker Compose Completo

```yaml
# docker-compose.grafana.yml
version: '3.8'

services:
  grafana:
    image: grafana/grafana:latest
    container_name: grafana
    restart: unless-stopped
    ports:
      - "3000:3000"
    environment:
      # Configuración básica
      - GF_SECURITY_ADMIN_USER=admin
      - GF_SECURITY_ADMIN_PASSWORD=admin123
      - GF_USERS_ALLOW_SIGN_UP=false
      - GF_USERS_ALLOW_ORG_CREATE=false
      
      # Base de datos
      - GF_DATABASE_TYPE=postgres
      - GF_DATABASE_HOST=postgres:5432
      - GF_DATABASE_NAME=grafana
      - GF_DATABASE_USER=grafana
      - GF_DATABASE_PASSWORD=grafana_password
      
      # SMTP para alertas
      - GF_SMTP_ENABLED=true
      - GF_SMTP_HOST=smtp.gmail.com:587
      - GF_SMTP_USER=your-email@gmail.com
      - GF_SMTP_PASSWORD=your-app-password
      - GF_SMTP_FROM_ADDRESS=your-email@gmail.com
      - GF_SMTP_FROM_NAME=Grafana
      
      # Security
      - GF_SERVER_ROOT_URL=http://localhost:3000
      - GF_SECURITY_COOKIE_SECURE=false
      - GF_SECURITY_COOKIE_SAMESITE=lax
      
    volumes:
      - grafana_data:/var/lib/grafana
      - ./grafana/provisioning:/etc/grafana/provisioning
      - ./grafana/dashboards:/var/lib/grafana/dashboards
    depends_on:
      - postgres
      - prometheus
    networks:
      - monitoring

  postgres:
    image: postgres:13
    container_name: grafana-postgres
    restart: unless-stopped
    environment:
      - POSTGRES_DB=grafana
      - POSTGRES_USER=grafana
      - POSTGRES_PASSWORD=grafana_password
    volumes:
      - postgres_data:/var/lib/postgresql/data
    networks:
      - monitoring

  prometheus:
    image: prom/prometheus:latest
    container_name: prometheus
    restart: unless-stopped
    ports:
      - "9090:9090"
    volumes:
      - ./prometheus.yml:/etc/prometheus/prometheus.yml
      - prometheus_data:/prometheus
    command:
      - '--config.file=/etc/prometheus/prometheus.yml'
      - '--storage.tsdb.path=/prometheus'
      - '--web.console.libraries=/etc/prometheus/console_libraries'
      - '--web.console.templates=/etc/prometheus/consoles'
      - '--storage.tsdb.retention.time=15d'
      - '--web.enable-lifecycle'
    networks:
      - monitoring

  # Aplicación de ejemplo para monitorear
  sample-app:
    build: ./sample-app
    container_name: sample-app
    restart: unless-stopped
    ports:
      - "5000:5000"
    networks:
      - monitoring

volumes:
  grafana_data:
  postgres_data:
  prometheus_data:

networks:
  monitoring:
    driver: bridge
```

#### 1.2 Configuración de Provisioning

```yaml
# grafana/provisioning/datasources/prometheus.yml
apiVersion: 1

datasources:
  - name: Prometheus
    type: prometheus
    access: proxy
    url: http://prometheus:9090
    isDefault: true
    editable: true
    jsonData:
      timeInterval: "15s"
      queryTimeout: "60s"
      httpMethod: POST
    secureJsonData:
      # Si Prometheus requiere autenticación
      # basicAuthPassword: "password"

  - name: Loki
    type: loki
    access: proxy
    url: http://loki:3100
    editable: true
    jsonData:
      maxLines: 1000

  - name: Jaeger
    type: jaeger
    access: proxy
    url: http://jaeger:16686
    editable: true

  - name: TestData
    type: testdata
    access: proxy
    editable: true
```

```yaml
# grafana/provisioning/dashboards/dashboards.yml
apiVersion: 1

providers:
  - name: 'default'
    orgId: 1
    folder: ''
    type: file
    disableDeletion: false
    updateIntervalSeconds: 10
    allowUiUpdates: true
    options:
      path: /var/lib/grafana/dashboards

  - name: 'system'
    orgId: 1
    folder: 'System Monitoring'
    type: file
    disableDeletion: false
    updateIntervalSeconds: 10
    allowUiUpdates: true
    options:
      path: /var/lib/grafana/dashboards/system

  - name: 'applications'
    orgId: 1
    folder: 'Application Monitoring'
    type: file
    disableDeletion: false
    updateIntervalSeconds: 10
    allowUiUpdates: true
    options:
      path: /var/lib/grafana/dashboards/applications
```

### 2. Instalación Nativa

#### 2.1 Script de Instalación (Ubuntu)

```bash
#!/bin/bash
# install_grafana.sh

# Agregar repositorio de Grafana
sudo apt-get install -y software-properties-common
wget -q -O - https://packages.grafana.com/gpg.key | sudo apt-key add -
echo "deb https://packages.grafana.com/oss/deb stable main" | sudo tee -a /etc/apt/sources.list.d/grafana.list

# Actualizar e instalar
sudo apt-get update
sudo apt-get install -y grafana

# Habilitar y iniciar servicio
sudo systemctl daemon-reload
sudo systemctl enable grafana-server
sudo systemctl start grafana-server

# Verificar estado
sudo systemctl status grafana-server

echo "Grafana installed successfully!"
echo "Access at: http://localhost:3000"
echo "Default credentials: admin/admin"
```

#### 2.2 Configuración Personalizada

```ini
# /etc/grafana/grafana.ini
[default]
instance_name = production-grafana

[paths]
data = /var/lib/grafana
logs = /var/log/grafana
plugins = /var/lib/grafana/plugins
provisioning = /etc/grafana/provisioning

[server]
protocol = http
http_addr = 0.0.0.0
http_port = 3000
domain = localhost
enforce_domain = false
root_url = http://localhost:3000
serve_from_sub_path = false

[database]
type = postgres
host = localhost:5432
name = grafana
user = grafana
password = your_password
ssl_mode = require

[session]
provider = postgres
provider_config = user=grafana password=your_password host=localhost port=5432 dbname=grafana sslmode=require

[security]
admin_user = admin
admin_password = your_secure_password
secret_key = your_secret_key
disable_gravatar = true
cookie_secure = false
cookie_samesite = lax

[users]
allow_sign_up = false
allow_org_create = false
auto_assign_org = true
auto_assign_org_id = 1
auto_assign_org_role = Viewer

[auth]
disable_login_form = false
disable_signout_menu = false

[auth.anonymous]
enabled = false

[smtp]
enabled = true
host = smtp.gmail.com:587
user = your-email@gmail.com
password = your-app-password
from_address = your-email@gmail.com
from_name = Grafana
```

---

## 📊 Creación de Dashboards

### 1. Dashboard Básico de Sistema

```json
{
  "dashboard": {
    "id": null,
    "title": "System Overview",
    "tags": ["system", "monitoring"],
    "timezone": "browser",
    "panels": [
      {
        "id": 1,
        "title": "CPU Usage",
        "type": "stat",
        "targets": [
          {
            "expr": "100 - (avg(rate(node_cpu_seconds_total{mode=\"idle\"}[5m])) * 100)",
            "legendFormat": "CPU Usage %",
            "refId": "A"
          }
        ],
        "fieldConfig": {
          "defaults": {
            "color": {
              "mode": "thresholds"
            },
            "thresholds": {
              "steps": [
                {
                  "color": "green",
                  "value": null
                },
                {
                  "color": "yellow",
                  "value": 70
                },
                {
                  "color": "red",
                  "value": 85
                }
              ]
            },
            "unit": "percent"
          }
        },
        "gridPos": {
          "h": 8,
          "w": 6,
          "x": 0,
          "y": 0
        }
      },
      {
        "id": 2,
        "title": "Memory Usage",
        "type": "gauge",
        "targets": [
          {
            "expr": "(1 - (node_memory_MemAvailable_bytes / node_memory_MemTotal_bytes)) * 100",
            "legendFormat": "Memory Usage %",
            "refId": "A"
          }
        ],
        "fieldConfig": {
          "defaults": {
            "color": {
              "mode": "thresholds"
            },
            "thresholds": {
              "steps": [
                {
                  "color": "green",
                  "value": null
                },
                {
                  "color": "yellow",
                  "value": 70
                },
                {
                  "color": "red",
                  "value": 85
                }
              ]
            },
            "max": 100,
            "min": 0,
            "unit": "percent"
          }
        },
        "gridPos": {
          "h": 8,
          "w": 6,
          "x": 6,
          "y": 0
        }
      },
      {
        "id": 3,
        "title": "Disk Usage",
        "type": "bargauge",
        "targets": [
          {
            "expr": "100 - ((node_filesystem_avail_bytes * 100) / node_filesystem_size_bytes)",
            "legendFormat": "{{mountpoint}}",
            "refId": "A"
          }
        ],
        "fieldConfig": {
          "defaults": {
            "color": {
              "mode": "thresholds"
            },
            "thresholds": {
              "steps": [
                {
                  "color": "green",
                  "value": null
                },
                {
                  "color": "yellow",
                  "value": 70
                },
                {
                  "color": "red",
                  "value": 90
                }
              ]
            },
            "max": 100,
            "min": 0,
            "unit": "percent"
          }
        },
        "gridPos": {
          "h": 8,
          "w": 12,
          "x": 12,
          "y": 0
        }
      },
      {
        "id": 4,
        "title": "CPU Usage Over Time",
        "type": "timeseries",
        "targets": [
          {
            "expr": "100 - (avg(rate(node_cpu_seconds_total{mode=\"idle\"}[5m])) * 100)",
            "legendFormat": "CPU Usage %",
            "refId": "A"
          }
        ],
        "fieldConfig": {
          "defaults": {
            "color": {
              "mode": "palette-classic"
            },
            "custom": {
              "axisLabel": "",
              "axisPlacement": "auto",
              "barAlignment": 0,
              "drawStyle": "line",
              "fillOpacity": 10,
              "gradientMode": "none",
              "hideFrom": {
                "graph": false,
                "legend": false,
                "tooltip": false
              },
              "lineInterpolation": "linear",
              "lineWidth": 1,
              "pointSize": 5,
              "scaleDistribution": {
                "type": "linear"
              },
              "showPoints": "never",
              "spanNulls": true
            },
            "unit": "percent"
          }
        },
        "options": {
          "legend": {
            "calcs": [],
            "displayMode": "list",
            "placement": "bottom"
          },
          "tooltip": {
            "mode": "single"
          }
        },
        "gridPos": {
          "h": 8,
          "w": 24,
          "x": 0,
          "y": 8
        }
      }
    ],
    "time": {
      "from": "now-1h",
      "to": "now"
    },
    "timepicker": {},
    "timezone": "",
    "title": "System Overview",
    "version": 1
  }
}
```

### 2. Dashboard de Aplicación Web

#### 2.1 Panel de Métricas RED

```json
{
  "panels": [
    {
      "id": 10,
      "title": "Request Rate",
      "type": "stat",
      "targets": [
        {
          "expr": "sum(rate(http_requests_total[5m]))",
          "legendFormat": "Requests/sec",
          "refId": "A"
        }
      ],
      "fieldConfig": {
        "defaults": {
          "color": {
            "mode": "thresholds"
          },
          "thresholds": {
            "steps": [
              {
                "color": "green",
                "value": null
              },
              {
                "color": "yellow",
                "value": 100
              },
              {
                "color": "red",
                "value": 1000
              }
            ]
          },
          "unit": "reqps"
        }
      }
    },
    {
      "id": 11,
      "title": "Error Rate",
      "type": "stat",
      "targets": [
        {
          "expr": "(sum(rate(http_requests_total{status=~\"5..\"}[5m])) / sum(rate(http_requests_total[5m]))) * 100",
          "legendFormat": "Error Rate %",
          "refId": "A"
        }
      ],
      "fieldConfig": {
        "defaults": {
          "color": {
            "mode": "thresholds"
          },
          "thresholds": {
            "steps": [
              {
                "color": "green",
                "value": null
              },
              {
                "color": "yellow",
                "value": 1
              },
              {
                "color": "red",
                "value": 5
              }
            ]
          },
          "unit": "percent"
        }
      }
    },
    {
      "id": 12,
      "title": "Response Time",
      "type": "stat",
      "targets": [
        {
          "expr": "histogram_quantile(0.95, sum(rate(http_request_duration_seconds_bucket[5m])) by (le))",
          "legendFormat": "95th Percentile",
          "refId": "A"
        }
      ],
      "fieldConfig": {
        "defaults": {
          "color": {
            "mode": "thresholds"
          },
          "thresholds": {
            "steps": [
              {
                "color": "green",
                "value": null
              },
              {
                "color": "yellow",
                "value": 0.5
              },
              {
                "color": "red",
                "value": 2
              }
            ]
          },
          "unit": "s"
        }
      }
    }
  ]
}
```

#### 2.2 Panel de Heatmap para Latencia

```json
{
  "id": 13,
  "title": "Response Time Heatmap",
  "type": "heatmap",
  "targets": [
    {
      "expr": "sum(rate(http_request_duration_seconds_bucket[5m])) by (le)",
      "format": "heatmap",
      "legendFormat": "{{le}}",
      "refId": "A"
    }
  ],
  "heatmap": {
    "xBucketSize": null,
    "yBucketSize": null,
    "yBucketBound": "auto"
  },
  "color": {
    "cardColor": "#b4ff00",
    "colorScale": "sqrt",
    "colorScheme": "interpolateOranges",
    "exponent": 0.5,
    "mode": "spectrum"
  },
  "dataFormat": "tsbuckets",
  "gridPos": {
    "h": 8,
    "w": 24,
    "x": 0,
    "y": 16
  }
}
```

### 3. Dashboard con Variables/Templates

#### 3.1 Definición de Variables

```json
{
  "templating": {
    "list": [
      {
        "allValue": null,
        "current": {
          "selected": false,
          "text": "All",
          "value": "$__all"
        },
        "datasource": "Prometheus",
        "definition": "label_values(up, instance)",
        "hide": 0,
        "includeAll": true,
        "label": "Instance",
        "multi": true,
        "name": "instance",
        "options": [],
        "query": "label_values(up, instance)",
        "refresh": 1,
        "regex": "",
        "skipUrlSync": false,
        "sort": 1,
        "tagValuesQuery": "",
        "tags": [],
        "tagsQuery": "",
        "type": "query",
        "useTags": false
      },
      {
        "allValue": null,
        "current": {
          "selected": false,
          "text": "5m",
          "value": "5m"
        },
        "hide": 0,
        "includeAll": false,
        "label": "Interval",
        "multi": false,
        "name": "interval",
        "options": [
          {
            "selected": false,
            "text": "1m",
            "value": "1m"
          },
          {
            "selected": true,
            "text": "5m",
            "value": "5m"
          },
          {
            "selected": false,
            "text": "10m",
            "value": "10m"
          },
          {
            "selected": false,
            "text": "30m",
            "value": "30m"
          }
        ],
        "query": "1m,5m,10m,30m",
        "skipUrlSync": false,
        "type": "custom"
      }
    ]
  }
}
```

#### 3.2 Uso de Variables en Consultas

```json
{
  "targets": [
    {
      "expr": "100 - (avg(rate(node_cpu_seconds_total{mode=\"idle\", instance=~\"$instance\"}[$interval])) * 100)",
      "legendFormat": "CPU Usage - {{instance}}",
      "refId": "A"
    }
  ]
}
```

---

## 🎨 Tipos de Visualizaciones

### 1. Time Series (Gráfico de Series Temporales)

```json
{
  "type": "timeseries",
  "title": "HTTP Requests Over Time",
  "targets": [
    {
      "expr": "sum(rate(http_requests_total[5m])) by (method)",
      "legendFormat": "{{method}}"
    }
  ],
  "fieldConfig": {
    "defaults": {
      "custom": {
        "drawStyle": "line",
        "lineInterpolation": "linear",
        "barAlignment": 0,
        "lineWidth": 1,
        "fillOpacity": 10,
        "gradientMode": "none",
        "spanNulls": false,
        "insertNulls": false,
        "showPoints": "auto",
        "pointSize": 5,
        "stackingConfig": {
          "mode": "none",
          "group": "A"
        },
        "axisPlacement": "auto",
        "axisLabel": "",
        "scaleDistribution": {
          "type": "linear"
        }
      },
      "color": {
        "mode": "palette-classic"
      },
      "unit": "reqps"
    }
  }
}
```

### 2. Stat Panel (Panel de Estadística)

```json
{
  "type": "stat",
  "title": "Current Active Users",
  "targets": [
    {
      "expr": "active_users_total",
      "legendFormat": "Active Users"
    }
  ],
  "fieldConfig": {
    "defaults": {
      "color": {
        "mode": "value"
      },
      "mappings": [],
      "thresholds": {
        "mode": "absolute",
        "steps": [
          {
            "color": "green",
            "value": null
          },
          {
            "color": "red",
            "value": 80
          }
        ]
      }
    }
  },
  "options": {
    "orientation": "auto",
    "reduceOptions": {
      "values": false,
      "calcs": ["lastNotNull"],
      "fields": ""
    },
    "textMode": "auto",
    "wideLayout": true,
    "colorMode": "value",
    "graphMode": "area",
    "justifyMode": "auto"
  }
}
```

### 3. Table Panel (Tabla)

```json
{
  "type": "table",
  "title": "Top Endpoints by Request Count",
  "targets": [
    {
      "expr": "topk(10, sum(rate(http_requests_total[5m])) by (endpoint))",
      "format": "table",
      "instant": true
    }
  ],
  "fieldConfig": {
    "defaults": {
      "custom": {
        "align": "auto",
        "cellOptions": {
          "type": "auto"
        },
        "filterable": false,
        "inspect": false
      },
      "mappings": [],
      "thresholds": {
        "mode": "absolute",
        "steps": [
          {
            "color": "green",
            "value": null
          },
          {
            "color": "red",
            "value": 80
          }
        ]
      }
    },
    "overrides": [
      {
        "matcher": {
          "id": "byName",
          "options": "Value"
        },
        "properties": [
          {
            "id": "unit",
            "value": "reqps"
          },
          {
            "id": "custom.cellOptions",
            "value": {
              "type": "color-background"
            }
          }
        ]
      }
    ]
  }
}
```

### 4. Pie Chart (Gráfico de Torta)

```json
{
  "type": "piechart",
  "title": "Requests by Status Code",
  "targets": [
    {
      "expr": "sum(rate(http_requests_total[5m])) by (status)",
      "legendFormat": "{{status}}"
    }
  ],
  "options": {
    "reduceOptions": {
      "values": false,
      "calcs": ["lastNotNull"],
      "fields": ""
    },
    "pieType": "pie",
    "tooltip": {
      "mode": "single",
      "sort": "none"
    },
    "legend": {
      "displayMode": "list",
      "placement": "bottom",
      "values": []
    },
    "displayLabels": ["name", "percent"]
  }
}
```

### 5. Worldmap Panel (Plugin)

```json
{
  "type": "grafana-worldmap-panel",
  "title": "Requests by Country",
  "targets": [
    {
      "expr": "sum(rate(http_requests_total[5m])) by (country)",
      "legendFormat": "{{country}}"
    }
  ],
  "worldmapSettings": {
    "showTooltip": true,
    "showLegend": true,
    "enableMouseWheelZoom": true,
    "mouseWheelZoomEnabled": true
  },
  "colors": ["#C4162A", "#8AB8FF", "#40E0D0"],
  "decimals": 0,
  "valueName": "current"
}
```

---

## 🚨 Configuración de Alertas

### 1. Alerta Básica de CPU

```json
{
  "alert": {
    "id": 1,
    "version": 1,
    "uid": "cpu_alert",
    "title": "High CPU Usage",
    "condition": "B",
    "data": [
      {
        "refId": "A",
        "queryType": "",
        "relativeTimeRange": {
          "from": 600,
          "to": 0
        },
        "model": {
          "expr": "100 - (avg(rate(node_cpu_seconds_total{mode=\"idle\"}[5m])) * 100)",
          "interval": "",
          "refId": "A"
        }
      },
      {
        "refId": "B",
        "queryType": "",
        "relativeTimeRange": {
          "from": 0,
          "to": 0
        },
        "model": {
          "conditions": [
            {
              "evaluator": {
                "params": [80],
                "type": "gt"
              },
              "operator": {
                "type": "and"
              },
              "query": {
                "params": ["A"]
              },
              "reducer": {
                "params": [],
                "type": "last"
              },
              "type": "query"
            }
          ],
          "datasource": {
            "type": "__expr__",
            "uid": "__expr__"
          },
          "expression": "A",
          "hide": false,
          "intervalMs": 1000,
          "maxDataPoints": 43200,
          "reducer": "last",
          "refId": "B",
          "type": "reduce"
        }
      }
    ],
    "intervalSeconds": 60,
    "maxDataPoints": 43200,
    "noDataState": "NoData",
    "execErrState": "Alerting"
  },
  "annotations": {
    "description": "CPU usage is above 80% for more than 5 minutes",
    "runbook_url": "https://wiki.company.com/runbooks/high-cpu",
    "summary": "High CPU usage detected on {{ $labels.instance }}"
  },
  "labels": {
    "severity": "warning",
    "team": "infrastructure"
  }
}
```

### 2. Notification Channels

#### 2.1 Slack Notification

```json
{
  "id": 1,
  "name": "slack-alerts",
  "type": "slack",
  "settings": {
    "url": "https://hooks.slack.com/services/YOUR/SLACK/WEBHOOK",
    "username": "Grafana",
    "channel": "#alerts",
    "iconEmoji": ":grafana:",
    "iconUrl": "",
    "title": "{{ range .Alerts }}{{ .Annotations.summary }}{{ end }}",
    "text": "{{ range .Alerts }}{{ .Annotations.description }}{{ end }}",
    "color": "{{ if eq .Status \"firing\" }}danger{{ else }}good{{ end }}"
  }
}
```

#### 2.2 Email Notification

```json
{
  "id": 2,
  "name": "email-alerts",
  "type": "email",
  "settings": {
    "addresses": "ops-team@company.com;sre-team@company.com",
    "subject": "[Grafana Alert] {{ .GroupLabels.alertname }}",
    "body": "{{ range .Alerts }}\n**Alert:** {{ .Annotations.summary }}\n**Description:** {{ .Annotations.description }}\n**Severity:** {{ .Labels.severity }}\n**Runbook:** {{ .Annotations.runbook_url }}\n{{ end }}"
  }
}
```

#### 2.3 PagerDuty Integration

```json
{
  "id": 3,
  "name": "pagerduty-critical",
  "type": "pagerduty",
  "settings": {
    "integrationKey": "YOUR_PAGERDUTY_INTEGRATION_KEY",
    "severity": "critical",
    "customDetails": {
      "firing": "{{ range .Alerts }}{{ .Annotations.description }}{{ end }}",
      "num_firing": "{{ .Alerts.Firing | len }}",
      "num_resolved": "{{ .Alerts.Resolved | len }}"
    }
  }
}
```

### 3. Alert Rules Avanzadas

#### 3.1 Regla Multi-Condición

```json
{
  "alert": {
    "title": "Service Health Check",
    "message": "Multiple health indicators are failing",
    "frequency": "30s",
    "conditions": [
      {
        "query": {
          "params": ["A", "5m", "now"]
        },
        "reducer": {
          "type": "avg",
          "params": []
        },
        "evaluator": {
          "params": [0.8],
          "type": "lt"
        }
      }
    ]
  },
  "targets": [
    {
      "expr": "(\n  (rate(http_requests_total{status=\"200\"}[5m]) / rate(http_requests_total[5m])) * \n  (1 - (histogram_quantile(0.95, http_request_duration_seconds_bucket) > 2)) * \n  (up == 1)\n)",
      "refId": "A"
    }
  ]
}
```

#### 3.2 Alerta Basada en Predicción

```json
{
  "targets": [
    {
      "expr": "predict_linear(node_filesystem_free_bytes[1h], 4 * 3600) < 0",
      "refId": "A"
    }
  ],
  "alert": {
    "title": "Disk Will Fill Soon",
    "message": "Disk is predicted to fill within 4 hours based on current trend"
  }
}
```

---

## 🎛️ Funciones Avanzadas

### 1. Annotations (Anotaciones)

```json
{
  "annotations": {
    "list": [
      {
        "name": "Deployments",
        "datasource": "Prometheus",
        "enable": true,
        "hide": false,
        "iconColor": "rgba(0, 211, 255, 1)",
        "query": "changes(deployment_version[1m]) > 0",
        "showIn": 0,
        "tags": ["deployment"],
        "textFormat": "Deployment: {{version}}",
        "titleFormat": "New Deployment"
      },
      {
        "name": "Incidents",
        "datasource": "-- Grafana --",
        "enable": true,
        "hide": false,
        "iconColor": "rgba(255, 96, 96, 1)",
        "tags": ["incident"],
        "type": "tags"
      }
    ]
  }
}
```

### 2. Links y Navigation

```json
{
  "links": [
    {
      "asDropdown": false,
      "icon": "external link",
      "includeVars": true,
      "keepTime": true,
      "tags": [],
      "targetBlank": true,
      "title": "Prometheus",
      "tooltip": "Open Prometheus UI",
      "type": "link",
      "url": "http://prometheus:9090"
    },
    {
      "asDropdown": false,
      "icon": "dashboard",
      "includeVars": true,
      "keepTime": true,
      "tags": ["system"],
      "targetBlank": false,
      "title": "System Dashboards",
      "type": "dashboards"
    }
  ]
}
```

### 3. Transformations

#### 3.1 Merge Series

```json
{
  "transformations": [
    {
      "id": "merge",
      "options": {}
    },
    {
      "id": "organize",
      "options": {
        "excludeByName": {},
        "indexByName": {
          "Time": 0,
          "cpu_usage": 1,
          "memory_usage": 2,
          "disk_usage": 3
        },
        "renameByName": {
          "cpu_usage": "CPU %",
          "memory_usage": "Memory %",
          "disk_usage": "Disk %"
        }
      }
    }
  ]
}
```

#### 3.2 Calculate Field

```json
{
  "transformations": [
    {
      "id": "calculateField",
      "options": {
        "mode": "binary",
        "reduce": {
          "reducer": "sum"
        },
        "binary": {
          "left": "requests_success",
          "operator": "/",
          "right": "requests_total"
        },
        "alias": "Success Rate"
      }
    }
  ]
}
```

---

## 🏢 Gestión de Usuarios y Organizaciones

### 1. Configuración de Usuarios

#### 1.1 Script de Gestión de Usuarios

```bash
#!/bin/bash
# manage_users.sh

GRAFANA_URL="http://localhost:3000"
ADMIN_USER="admin"
ADMIN_PASSWORD="admin123"

# Función para crear usuario
create_user() {
    local name="$1"
    local email="$2"
    local login="$3"
    local password="$4"
    local role="$5"

    curl -X POST \
         -H "Content-Type: application/json" \
         -d "{
           \"name\": \"$name\",
           \"email\": \"$email\",
           \"login\": \"$login\",
           \"password\": \"$password\",
           \"OrgId\": 1
         }" \
         -u "$ADMIN_USER:$ADMIN_PASSWORD" \
         "$GRAFANA_URL/api/admin/users"

    # Asignar rol
    local user_id=$(get_user_id "$login")
    curl -X PATCH \
         -H "Content-Type: application/json" \
         -d "{\"role\": \"$role\"}" \
         -u "$ADMIN_USER:$ADMIN_PASSWORD" \
         "$GRAFANA_URL/api/orgs/1/users/$user_id"
}

# Función para obtener ID de usuario
get_user_id() {
    local login="$1"
    curl -s -u "$ADMIN_USER:$ADMIN_PASSWORD" \
         "$GRAFANA_URL/api/users/lookup?loginOrEmail=$login" | \
         jq -r '.id'
}

# Crear usuarios de ejemplo
create_user "John Doe" "john@company.com" "john" "password123" "Editor"
create_user "Jane Smith" "jane@company.com" "jane" "password123" "Viewer"
create_user "Bob Admin" "bob@company.com" "bob" "password123" "Admin"

echo "Users created successfully!"
```

#### 1.2 Configuración LDAP

```ini
# /etc/grafana/ldap.toml
[[servers]]
host = "ldap.company.com"
port = 389
use_ssl = false
start_tls = false
ssl_skip_verify = false

bind_dn = "cn=admin,dc=company,dc=com"
bind_password = 'grafana'

search_filter = "(cn=%s)"
search_base_dns = ["dc=company,dc=com"]

[servers.attributes]
name = "givenName"
surname = "sn"
username = "cn"
member_of = "memberOf"
email =  "email"

[[servers.group_mappings]]
group_dn = "cn=admins,dc=company,dc=com"
org_role = "Admin"

[[servers.group_mappings]]
group_dn = "cn=editors,dc=company,dc=com"
org_role = "Editor"

[[servers.group_mappings]]
group_dn = "*"
org_role = "Viewer"
```

### 2. Roles y Permisos

#### 2.1 Configuración de Folder Permissions

```bash
# Script para configurar permisos de folders
#!/bin/bash

GRAFANA_URL="http://localhost:3000"
ADMIN_USER="admin"
ADMIN_PASSWORD="admin123"

# Crear folder
create_folder() {
    local title="$1"
    local uid="$2"
    
    curl -X POST \
         -H "Content-Type: application/json" \
         -d "{
           \"uid\": \"$uid\",
           \"title\": \"$title\"
         }" \
         -u "$ADMIN_USER:$ADMIN_PASSWORD" \
         "$GRAFANA_URL/api/folders"
}

# Configurar permisos de folder
set_folder_permissions() {
    local folder_uid="$1"
    local team_id="$2"
    local permission="$3"
    
    curl -X POST \
         -H "Content-Type: application/json" \
         -d "{
           \"items\": [
             {
               \"teamId\": $team_id,
               \"permission\": $permission
             }
           ]
         }" \
         -u "$ADMIN_USER:$ADMIN_PASSWORD" \
         "$GRAFANA_URL/api/folders/$folder_uid/permissions"
}

# Crear folders y configurar permisos
create_folder "Infrastructure" "infrastructure"
create_folder "Applications" "applications"
create_folder "Business" "business"

# Configurar permisos (requiere IDs de teams)
# set_folder_permissions "infrastructure" 1 2  # Edit permission
# set_folder_permissions "applications" 2 1    # View permission
```

---

## 📊 Dashboards Avanzados

### 1. Dashboard de SLI/SLO

```json
{
  "dashboard": {
    "title": "SLI/SLO Dashboard",
    "panels": [
      {
        "title": "Availability SLO (99.9%)",
        "type": "stat",
        "targets": [
          {
            "expr": "(\n  sum(rate(http_requests_total{status!~\"5..\"}[30d])) /\n  sum(rate(http_requests_total[30d]))\n) * 100",
            "legendFormat": "Availability %"
          }
        ],
        "fieldConfig": {
          "defaults": {
            "thresholds": {
              "steps": [
                {"color": "red", "value": 0},
                {"color": "yellow", "value": 99.5},
                {"color": "green", "value": 99.9}
              ]
            },
            "min": 99,
            "max": 100,
            "unit": "percent"
          }
        }
      },
      {
        "title": "Error Budget Remaining",
        "type": "bargauge",
        "targets": [
          {
            "expr": "(\n  (0.999 - (\n    sum(rate(http_requests_total{status!~\"5..\"}[30d])) /\n    sum(rate(http_requests_total[30d]))\n  )) / 0.001\n) * 100",
            "legendFormat": "Error Budget %"
          }
        ],
        "fieldConfig": {
          "defaults": {
            "thresholds": {
              "steps": [
                {"color": "red", "value": 0},
                {"color": "yellow", "value": 25},
                {"color": "green", "value": 50}
              ]
            },
            "min": 0,
            "max": 100,
            "unit": "percent"
          }
        }
      }
    ]
  }
}
```

### 2. Dashboard de Capacity Planning

```json
{
  "panels": [
    {
      "title": "CPU Capacity Forecast",
      "type": "timeseries",
      "targets": [
        {
          "expr": "predict_linear(avg(cpu_usage_percent)[7d:1h], 30 * 24 * 3600)",
          "legendFormat": "Predicted CPU Usage (30 days)"
        },
        {
          "expr": "avg(cpu_usage_percent)",
          "legendFormat": "Current CPU Usage"
        }
      ],
      "fieldConfig": {
        "defaults": {
          "custom": {
            "drawStyle": "line",
            "lineWidth": 2
          },
          "color": {
            "mode": "palette-classic"
          },
          "unit": "percent"
        },
        "overrides": [
          {
            "matcher": {"id": "byRegexp", "options": "/Predicted/"},
            "properties": [
              {"id": "custom.lineStyle", "value": {"dash": [10, 10]}}
            ]
          }
        ]
      }
    }
  ]
}
```

---

## 🔧 Plugins y Extensiones

### 1. Instalación de Plugins

```bash
# Instalar plugins via CLI
grafana-cli plugins install grafana-worldmap-panel
grafana-cli plugins install grafana-piechart-panel
grafana-cli plugins install grafana-polystat-panel

# Reiniciar Grafana
sudo systemctl restart grafana-server
```

### 2. Configuración de Plugins en Docker

```yaml
# docker-compose.yml
services:
  grafana:
    image: grafana/grafana:latest
    environment:
      - GF_INSTALL_PLUGINS=grafana-worldmap-panel,grafana-piechart-panel,grafana-polystat-panel
```

### 3. Plugin Personalizado Básico

```javascript
// plugin.json
{
  "type": "panel",
  "name": "Custom Status Panel",
  "id": "custom-status-panel",
  "info": {
    "description": "Custom status panel for monitoring",
    "author": {
      "name": "Your Name"
    },
    "version": "1.0.0"
  },
  "dependencies": {
    "grafana": ">=7.0.0"
  }
}
```

```typescript
// module.ts
import { PanelPlugin } from '@grafana/data';
import { SimpleOptions } from './types';
import { SimplePanel } from './SimplePanel';

export const plugin = new PanelPlugin<SimpleOptions>(SimplePanel).setPanelOptions(builder => {
  return builder
    .addTextInput({
      path: 'text',
      name: 'Simple text option',
      description: 'Description of panel option',
      defaultValue: 'Default value of text input option',
    });
});
```

---

## 📋 Ejercicios Prácticos

### Ejercicio 1: Dashboard Completo de Aplicación

**Objetivo**: Crear un dashboard completo para monitorear una aplicación web.

**Requisitos**:

1. Paneles para métricas RED (Rate, Errors, Duration)
2. Tabla con top endpoints por tráfico
3. Heatmap de latencia
4. Alertas para alta tasa de errores y latencia
5. Variables para filtrar por instancia y intervalo de tiempo

### Ejercicio 2: Dashboard de Infraestructura

**Objetivo**: Crear un dashboard para monitoreo de infraestructura.

**Componentes**:

- Overview de múltiples servidores
- Métricas detalladas por servidor
- Alertas por recursos críticos
- Predicciones de capacidad

### Ejercicio 3: Sistema de Alertas Multi-Canal

**Objetivo**: Configurar alertas que se envíen a diferentes canales según severidad.

**Configuración**:

- Slack para alertas de warning
- Email para alertas críticas
- PagerDuty para alertas de producción

---

## 📖 Best Practices

### 1. Diseño de Dashboards

**Principios de Diseño**:

```yaml
dashboard_design_principles:
  visual_hierarchy:
    - "Most important metrics at the top"
    - "Use size and color to indicate priority"
    - "Group related metrics together"
  
  color_usage:
    - "Consistent color scheme across dashboards"
    - "Red for errors/critical states"
    - "Green for healthy states"
    - "Yellow/orange for warnings"
  
  layout:
    - "Use the inverted pyramid: overview -> details"
    - "Keep related panels close"
    - "Use whitespace effectively"
  
  performance:
    - "Limit number of panels per dashboard"
    - "Use appropriate time ranges"
    - "Optimize queries for speed"
```

### 2. Naming Conventions

```yaml
naming_conventions:
  dashboards:
    format: "[Category] - [Service/Component] - [Environment]"
    examples:
      - "Infrastructure - Node Monitoring - Production"
      - "Application - User Service - Staging"
      - "Business - E-commerce Metrics - Production"
  
  panels:
    format: "Clear, descriptive titles"
    examples:
      - "HTTP Request Rate (req/s)"
      - "95th Percentile Response Time"
      - "Error Rate by Endpoint"
  
  alerts:
    format: "[Severity] [Component] [Metric] [Condition]"
    examples:
      - "Critical API HighErrorRate >5%"
      - "Warning Database HighConnections >80%"
```

### 3. Performance Optimization

```yaml
performance_tips:
  queries:
    - "Use recording rules for complex calculations"
    - "Limit time ranges for heavy queries"
    - "Use appropriate step intervals"
  
  dashboards:
    - "Limit panels per dashboard (max 20-25)"
    - "Use template variables to reduce query load"
    - "Cache frequently accessed dashboards"
  
  alerting:
    - "Use appropriate evaluation intervals"
    - "Group related alerts"
    - "Implement alert dependencies"
```

---

## ✅ Resumen del Módulo

En este módulo has aprendido:

1. **Instalación y configuración** de Grafana en múltiples entornos
2. **Creación de dashboards** profesionales con diferentes tipos de paneles
3. **Configuración de datasources** incluyendo Prometheus
4. **Templating y variables** para dashboards dinámicos
5. **Sistema de alertas** avanzado con múltiples canales de notificación
6. **Gestión de usuarios** y organizaciones
7. **Plugins y extensiones** para funcionalidad adicional
8. **Best practices** para diseño y performance

### Próximas etapas

- **Módulo 04**: Implementaremos el stack ELK para gestión centralizada de logs
- **Módulo 05**: Configuraremos Alertmanager para gestión avanzada de alertas
- **Módulo 06**: Exploraremos distributed tracing con Jaeger

¡Continúa con el siguiente módulo para dominar la gestión de logs con ELK Stack!
