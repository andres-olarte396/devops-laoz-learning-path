# Módulo 04: ELK Stack - Elasticsearch, Logstash, Kibana

## 📋 Introducción

El ELK Stack (Elasticsearch, Logstash, Kibana) es la solución líder para la gestión centralizada de logs, análisis de datos y observabilidad. En este módulo aprenderás a implementar una plataforma completa de logging que permita recopilar, procesar, almacenar y visualizar logs de múltiples fuentes.

### ¿Qué es el ELK Stack?

- **Elasticsearch**: Motor de búsqueda y análisis distribuido
- **Logstash**: Pipeline de procesamiento de datos del lado del servidor
- **Kibana**: Interfaz de visualización y exploración de datos
- **Beats**: Agentes ligeros para recolección de datos (Filebeat, Metricbeat, etc.)

### ¿Por qué ELK Stack?

- **Centralización**: Logs de múltiples fuentes en un solo lugar
- **Escalabilidad**: Maneja terabytes de datos diarios
- **Flexibilidad**: Procesa cualquier tipo de log o dato estructurado
- **Visualización**: Dashboards y análisis en tiempo real
- **Búsqueda potente**: Queries complejas con Elasticsearch
- **Alerting**: Detección proactiva de problemas

---

## 🎯 Objetivos de Aprendizaje

Al completar este módulo, serás capaz de:

1. **Instalar y configurar** el stack ELK completo
2. **Configurar Beats** para recolección de logs
3. **Procesar logs** con pipelines de Logstash
4. **Gestionar índices** en Elasticsearch
5. **Crear visualizaciones** en Kibana
6. **Implementar alertas** basadas en logs
7. **Optimizar performance** y gestionar almacenamiento
8. **Configurar seguridad** y autenticación

---

## 🏗️ Instalación y Configuración

### 1. Docker Compose Setup Completo

```yaml
# docker-compose.elk.yml
version: '3.8'

services:
  elasticsearch:
    image: docker.elastic.co/elasticsearch/elasticsearch:8.11.0
    container_name: elasticsearch
    restart: unless-stopped
    environment:
      - node.name=elasticsearch
      - cluster.name=elk-cluster
      - discovery.type=single-node
      - bootstrap.memory_lock=true
      - "ES_JAVA_OPTS=-Xms2g -Xmx2g"
      - xpack.security.enabled=false
      - xpack.security.enrollment.enabled=false
    ulimits:
      memlock:
        soft: -1
        hard: -1
    volumes:
      - elasticsearch_data:/usr/share/elasticsearch/data
      - ./elasticsearch/config/elasticsearch.yml:/usr/share/elasticsearch/config/elasticsearch.yml
    ports:
      - "9200:9200"
      - "9300:9300"
    networks:
      - elk
    healthcheck:
      test: ["CMD-SHELL", "curl -f http://localhost:9200/_cluster/health || exit 1"]
      interval: 30s
      timeout: 10s
      retries: 5

  logstash:
    image: docker.elastic.co/logstash/logstash:8.11.0
    container_name: logstash
    restart: unless-stopped
    environment:
      - "LS_JAVA_OPTS=-Xmx1g -Xms1g"
    volumes:
      - ./logstash/config/logstash.yml:/usr/share/logstash/config/logstash.yml
      - ./logstash/pipeline/:/usr/share/logstash/pipeline/
      - ./logstash/patterns/:/usr/share/logstash/patterns/
    ports:
      - "5000:5000"
      - "5044:5044"
      - "9600:9600"
    networks:
      - elk
    depends_on:
      elasticsearch:
        condition: service_healthy

  kibana:
    image: docker.elastic.co/kibana/kibana:8.11.0
    container_name: kibana
    restart: unless-stopped
    environment:
      - ELASTICSEARCH_HOSTS=http://elasticsearch:9200
      - SERVER_NAME=kibana
      - SERVER_HOST=0.0.0.0
    volumes:
      - ./kibana/config/kibana.yml:/usr/share/kibana/config/kibana.yml
    ports:
      - "5601:5601"
    networks:
      - elk
    depends_on:
      elasticsearch:
        condition: service_healthy

  filebeat:
    image: docker.elastic.co/beats/filebeat:8.11.0
    container_name: filebeat
    restart: unless-stopped
    user: root
    volumes:
      - ./filebeat/config/filebeat.yml:/usr/share/filebeat/filebeat.yml:ro
      - /var/lib/docker/containers:/var/lib/docker/containers:ro
      - /var/run/docker.sock:/var/run/docker.sock:ro
      - /var/log:/var/log:ro
      - filebeat_data:/usr/share/filebeat/data
    networks:
      - elk
    depends_on:
      - logstash

  metricbeat:
    image: docker.elastic.co/beats/metricbeat:8.11.0
    container_name: metricbeat
    restart: unless-stopped
    user: root
    volumes:
      - ./metricbeat/config/metricbeat.yml:/usr/share/metricbeat/metricbeat.yml:ro
      - /var/run/docker.sock:/var/run/docker.sock:ro
      - /sys/fs/cgroup:/hostfs/sys/fs/cgroup:ro
      - /proc:/hostfs/proc:ro
      - /:/hostfs:ro
    networks:
      - elk
    depends_on:
      - elasticsearch

  # Sample application to generate logs
  nginx-sample:
    image: nginx:alpine
    container_name: nginx-sample
    ports:
      - "8080:80"
    volumes:
      - ./nginx/nginx.conf:/etc/nginx/nginx.conf
      - ./nginx/html:/usr/share/nginx/html
      - nginx_logs:/var/log/nginx
    networks:
      - elk

volumes:
  elasticsearch_data:
  filebeat_data:
  nginx_logs:

networks:
  elk:
    driver: bridge
```

### 2. Configuración de Elasticsearch

```yaml
# elasticsearch/config/elasticsearch.yml
cluster.name: "elk-cluster"
network.host: 0.0.0.0
discovery.type: single-node

# Memory settings
bootstrap.memory_lock: true

# Index settings
action.auto_create_index: .monitoring*,.watches,.triggered_watches,.watcher-history*,.ml*

# Security settings (disabled for development)
xpack.security.enabled: false

# Monitoring
xpack.monitoring.collection.enabled: true

# Performance settings
thread_pool.write.queue_size: 1000
indices.memory.index_buffer_size: 10%

# Logging
logger.org.elasticsearch.deprecation: warn
```

### 3. Configuración de Logstash

#### 3.1 Configuración Principal

```yaml
# logstash/config/logstash.yml
http.host: "0.0.0.0"
xpack.monitoring.elasticsearch.hosts: ["http://elasticsearch:9200"]

# Pipeline settings
pipeline.workers: 2
pipeline.batch.size: 125
pipeline.batch.delay: 50

# Logging
log.level: info
path.logs: /var/log/logstash
```

#### 3.2 Pipeline de Procesamiento

```ruby
# logstash/pipeline/main.conf
input {
  beats {
    port => 5044
  }
  
  tcp {
    port => 5000
    codec => json
  }
  
  http {
    port => 8080
    codec => json
  }
}

filter {
  # Procesar logs de Nginx
  if [fields][log_type] == "nginx" {
    grok {
      match => { 
        "message" => "%{COMBINEDAPACHELOG}" 
      }
    }
    
    date {
      match => [ "timestamp", "dd/MMM/yyyy:HH:mm:ss Z" ]
    }
    
    mutate {
      convert => { "response" => "integer" }
      convert => { "bytes" => "integer" }
    }
    
    if [response] >= 400 {
      mutate {
        add_tag => [ "error" ]
      }
    }
  }
  
  # Procesar logs de aplicación JSON
  if [fields][log_type] == "app" {
    json {
      source => "message"
    }
    
    date {
      match => [ "timestamp", "ISO8601" ]
    }
    
    # Extraer información útil
    mutate {
      add_field => { "log_level_numeric" => 0 }
    }
    
    if [level] == "ERROR" {
      mutate {
        update => { "log_level_numeric" => 40 }
        add_tag => [ "error" ]
      }
    } else if [level] == "WARN" {
      mutate {
        update => { "log_level_numeric" => 30 }
        add_tag => [ "warning" ]
      }
    } else if [level] == "INFO" {
      mutate {
        update => { "log_level_numeric" => 20 }
      }
    }
  }
  
  # Procesar logs de sistema
  if [fields][log_type] == "syslog" {
    grok {
      match => { 
        "message" => "%{SYSLOGTIMESTAMP:timestamp} %{IPORHOST:host} %{DATA:program}(?:\[%{POSINT:pid}\])?: %{GREEDYDATA:message}" 
      }
      overwrite => [ "message" ]
    }
    
    date {
      match => [ "timestamp", "MMM  d HH:mm:ss", "MMM dd HH:mm:ss" ]
    }
  }
  
  # Enriquecer con información geográfica
  if [clientip] {
    geoip {
      source => "clientip"
      target => "geoip"
    }
  }
  
  # Limpiar campos no necesarios
  mutate {
    remove_field => [ "host", "agent", "input", "ecs" ]
  }
}

output {
  elasticsearch {
    hosts => ["elasticsearch:9200"]
    index => "%{[@metadata][beat]}-%{[@metadata][version]}-%{+YYYY.MM.dd}"
    
    # Crear templates personalizados
    template_name => "custom_template"
    template_pattern => "*beat-*"
    template => "/usr/share/logstash/templates/custom_template.json"
  }
  
  # Debug output (opcional)
  if [tags] and "debug" in [tags] {
    stdout {
      codec => rubydebug
    }
  }
}
```

### 4. Configuración de Kibana

```yaml
# kibana/config/kibana.yml
server.name: kibana
server.host: 0.0.0.0
server.port: 5601

elasticsearch.hosts: ["http://elasticsearch:9200"]
elasticsearch.username: ""
elasticsearch.password: ""

# Monitoring
monitoring.ui.container.elasticsearch.enabled: true

# Logging
logging.appenders:
  file:
    type: file
    fileName: /var/log/kibana/kibana.log
    layout:
      type: json

logging.root:
  appenders:
    - default
    - file
  level: info

# Index patterns
kibana.defaultAppId: "discover"

# Security (disabled for development)
xpack.security.enabled: false
xpack.encryptedSavedObjects.encryptionKey: your-encryption-key-here
```

---

## 📊 Configuración de Beats

### 1. Filebeat Configuration

```yaml
# filebeat/config/filebeat.yml
filebeat.inputs:
  # Nginx access logs
  - type: log
    enabled: true
    paths:
      - /var/log/nginx/access.log
    fields:
      log_type: nginx
      service: web
    fields_under_root: true
    multiline.pattern: '^\d{4}-\d{2}-\d{2}'
    multiline.negate: true
    multiline.match: after

  # Nginx error logs
  - type: log
    enabled: true
    paths:
      - /var/log/nginx/error.log
    fields:
      log_type: nginx_error
      service: web
    fields_under_root: true

  # Application logs
  - type: log
    enabled: true
    paths:
      - /var/log/app/*.log
    fields:
      log_type: app
      service: application
    fields_under_root: true
    scan_frequency: 10s
    harvester_buffer_size: 16384
    max_bytes: 10485760

  # System logs
  - type: log
    enabled: true
    paths:
      - /var/log/syslog
      - /var/log/auth.log
    fields:
      log_type: syslog
      service: system
    fields_under_root: true

  # Docker container logs
  - type: container
    paths:
      - '/var/lib/docker/containers/*/*.log'
    stream: all
    processors:
      - add_docker_metadata:
          host: "unix:///var/run/docker.sock"

# Output configuration
output.logstash:
  hosts: ["logstash:5044"]
  compression_level: 3
  bulk_max_size: 2048

# Processors
processors:
  - add_host_metadata:
      when.not.contains.tags: forwarded
  - add_cloud_metadata: ~
  - add_docker_metadata: ~

# Logging
logging.level: info
logging.to_files: true
logging.files:
  path: /var/log/filebeat
  name: filebeat
  keepfiles: 7
  permissions: 0644

# Monitoring
monitoring.enabled: true
monitoring.elasticsearch:
  hosts: ["elasticsearch:9200"]
```

### 2. Metricbeat Configuration

```yaml
# metricbeat/config/metricbeat.yml
metricbeat.config.modules:
  path: ${path.config}/modules.d/*.yml
  reload.enabled: false

metricbeat.modules:
  # System metrics
  - module: system
    metricsets:
      - cpu
      - load
      - memory
      - network
      - process
      - process_summary
      - socket_summary
      - filesystem
      - fsstat
    enabled: true
    period: 10s
    processes: ['.*']
    cpu.metrics: ["percentages", "normalized_percentages"]
    core.metrics: ["percentages"]

  # Docker metrics
  - module: docker
    metricsets:
      - container
      - cpu
      - diskio
      - healthcheck
      - info
      - memory
      - network
    hosts: ["unix:///var/run/docker.sock"]
    period: 10s
    enabled: true

  # Nginx metrics
  - module: nginx
    metricsets: ["stubstatus"]
    enabled: true
    period: 10s
    hosts: ["http://nginx-sample/nginx_status"]

  # Elasticsearch metrics
  - module: elasticsearch
    metricsets:
      - node
      - node_stats
      - cluster_stats
    period: 10s
    hosts: ["http://elasticsearch:9200"]

# Output
output.elasticsearch:
  hosts: ["elasticsearch:9200"]
  protocol: "http"
  
# Processors
processors:
  - add_host_metadata: ~
  - add_cloud_metadata: ~
  - add_docker_metadata: ~

# Logging
logging.level: info
logging.to_files: true
logging.files:
  path: /var/log/metricbeat
  name: metricbeat
  keepfiles: 7
  permissions: 0644
```

---

## 🔍 Gestión de Índices en Elasticsearch

### 1. Index Templates

```json
{
  "index_patterns": ["logs-*"],
  "version": 1,
  "settings": {
    "number_of_shards": 1,
    "number_of_replicas": 0,
    "index.refresh_interval": "30s",
    "index.max_result_window": 50000,
    "index.mapping.total_fields.limit": 2000
  },
  "mappings": {
    "properties": {
      "@timestamp": {
        "type": "date"
      },
      "level": {
        "type": "keyword"
      },
      "message": {
        "type": "text",
        "analyzer": "standard",
        "fields": {
          "keyword": {
            "type": "keyword",
            "ignore_above": 256
          }
        }
      },
      "service": {
        "type": "keyword"
      },
      "host": {
        "type": "keyword"
      },
      "response_time": {
        "type": "float"
      },
      "status_code": {
        "type": "integer"
      },
      "user_agent": {
        "type": "text",
        "analyzer": "standard"
      },
      "ip": {
        "type": "ip"
      },
      "geoip": {
        "properties": {
          "continent_name": {"type": "keyword"},
          "country_name": {"type": "keyword"},
          "location": {"type": "geo_point"}
        }
      }
    }
  }
}
```

### 2. Index Lifecycle Management (ILM)

```json
{
  "policy": {
    "phases": {
      "hot": {
        "actions": {
          "rollover": {
            "max_size": "10GB",
            "max_age": "7d"
          },
          "set_priority": {
            "priority": 100
          }
        }
      },
      "warm": {
        "min_age": "7d",
        "actions": {
          "set_priority": {
            "priority": 50
          },
          "allocate": {
            "number_of_replicas": 0
          },
          "forcemerge": {
            "max_num_segments": 1
          }
        }
      },
      "cold": {
        "min_age": "30d",
        "actions": {
          "set_priority": {
            "priority": 0
          },
          "allocate": {
            "number_of_replicas": 0
          }
        }
      },
      "delete": {
        "min_age": "90d",
        "actions": {
          "delete": {}
        }
      }
    }
  }
}
```

### 3. Scripts de Gestión

```bash
#!/bin/bash
# elasticsearch_management.sh

ES_HOST="http://localhost:9200"

# Crear index template
create_template() {
    local template_name="$1"
    local template_file="$2"
    
    curl -X PUT "$ES_HOST/_index_template/$template_name" \
         -H "Content-Type: application/json" \
         -d @"$template_file"
}

# Crear ILM policy
create_ilm_policy() {
    local policy_name="$1"
    local policy_file="$2"
    
    curl -X PUT "$ES_HOST/_ilm/policy/$policy_name" \
         -H "Content-Type: application/json" \
         -d @"$policy_file"
}

# Listar índices
list_indices() {
    curl -X GET "$ES_HOST/_cat/indices?v&s=index"
}

# Obtener estadísticas de índice
index_stats() {
    local index_name="$1"
    curl -X GET "$ES_HOST/$index_name/_stats?pretty"
}

# Eliminar índices antiguos
delete_old_indices() {
    local days_old="$1"
    local pattern="$2"
    
    # Obtener índices más antiguos que X días
    curl -s -X GET "$ES_HOST/_cat/indices/$pattern?h=index,creation.date.string" | \
    while read index date; do
        # Lógica para calcular edad y eliminar si es necesario
        echo "Processing index: $index created on $date"
    done
}

# Reindexar datos
reindex_data() {
    local source_index="$1"
    local dest_index="$2"
    
    curl -X POST "$ES_HOST/_reindex" \
         -H "Content-Type: application/json" \
         -d "{
           \"source\": {
             \"index\": \"$source_index\"
           },
           \"dest\": {
             \"index\": \"$dest_index\"
           }
         }"
}

# Uso de funciones
case "$1" in
    "template")
        create_template "$2" "$3"
        ;;
    "policy")
        create_ilm_policy "$2" "$3"
        ;;
    "list")
        list_indices
        ;;
    "stats")
        index_stats "$2"
        ;;
    "cleanup")
        delete_old_indices "$2" "$3"
        ;;
    "reindex")
        reindex_data "$2" "$3"
        ;;
    *)
        echo "Usage: $0 {template|policy|list|stats|cleanup|reindex} [args...]"
        ;;
esac
```

---

## 📈 Visualizaciones en Kibana

### 1. Dashboard de Logs de Aplicación

#### 1.1 Configuración de Index Pattern

```bash
# Crear index pattern via API
curl -X POST "kibana:5601/api/saved_objects/index-pattern" \
     -H "Content-Type: application/json" \
     -H "kbn-xsrf: true" \
     -d '{
       "attributes": {
         "title": "logs-*",
         "timeFieldName": "@timestamp"
       }
     }'
```

#### 1.2 Visualización de Error Rate

```json
{
  "title": "Error Rate Over Time",
  "type": "line",
  "params": {
    "grid": {
      "categoryLines": false,
      "style": {
        "color": "#eee"
      }
    },
    "categoryAxes": [
      {
        "id": "CategoryAxis-1",
        "type": "category",
        "position": "bottom",
        "show": true,
        "style": {},
        "scale": {
          "type": "linear"
        },
        "labels": {
          "show": true,
          "truncate": 100
        },
        "title": {}
      }
    ],
    "valueAxes": [
      {
        "id": "ValueAxis-1",
        "name": "LeftAxis-1",
        "type": "value",
        "position": "left",
        "show": true,
        "style": {},
        "scale": {
          "type": "linear",
          "mode": "normal"
        },
        "labels": {
          "show": true,
          "rotate": 0,
          "filter": false,
          "truncate": 100
        },
        "title": {
          "text": "Error Rate %"
        }
      }
    ],
    "seriesParams": [
      {
        "show": true,
        "type": "line",
        "mode": "normal",
        "data": {
          "label": "Error Rate",
          "id": "1"
        },
        "valueAxis": "ValueAxis-1",
        "drawLinesBetweenPoints": true,
        "showCircles": true
      }
    ]
  },
  "aggs": [
    {
      "id": "1",
      "enabled": true,
      "type": "avg",
      "schema": "metric",
      "params": {
        "field": "error_rate"
      }
    },
    {
      "id": "2",
      "enabled": true,
      "type": "date_histogram",
      "schema": "segment",
      "params": {
        "field": "@timestamp",
        "interval": "auto",
        "customInterval": "2h",
        "min_doc_count": 1,
        "extended_bounds": {}
      }
    }
  ]
}
```

#### 1.3 Top Errors Table

```json
{
  "title": "Top Error Messages",
  "type": "table",
  "params": {
    "perPage": 10,
    "showPartialRows": false,
    "showMeticsAtAllLevels": false,
    "sort": {
      "columnIndex": null,
      "direction": null
    },
    "showTotal": false,
    "totalFunc": "sum"
  },
  "aggs": [
    {
      "id": "1",
      "enabled": true,
      "type": "count",
      "schema": "metric",
      "params": {}
    },
    {
      "id": "2",
      "enabled": true,
      "type": "terms",
      "schema": "bucket",
      "params": {
        "field": "message.keyword",
        "size": 10,
        "order": "desc",
        "orderBy": "1"
      }
    },
    {
      "id": "3",
      "enabled": true,
      "type": "terms",
      "schema": "bucket",
      "params": {
        "field": "service.keyword",
        "size": 5,
        "order": "desc",
        "orderBy": "1"
      }
    }
  ]
}
```

### 2. Dashboard de Performance

#### 2.1 Response Time Heatmap

```json
{
  "title": "Response Time Heatmap",
  "type": "heatmap",
  "params": {
    "addTooltip": true,
    "addLegend": true,
    "enableHover": false,
    "legendPosition": "right",
    "times": [],
    "colorsNumber": 4,
    "colorSchema": "Yellow to Red",
    "setColorRange": false,
    "colorsRange": [],
    "invertColors": false,
    "percentageMode": false,
    "valueAxes": [
      {
        "show": false,
        "id": "ValueAxis-1",
        "type": "value",
        "scale": {
          "type": "linear",
          "defaultYExtents": false
        },
        "labels": {
          "show": false,
          "rotate": 0,
          "color": "#555"
        }
      }
    ]
  },
  "aggs": [
    {
      "id": "1",
      "enabled": true,
      "type": "avg",
      "schema": "metric",
      "params": {
        "field": "response_time"
      }
    },
    {
      "id": "2",
      "enabled": true,
      "type": "date_histogram",
      "schema": "segment",
      "params": {
        "field": "@timestamp",
        "interval": "auto",
        "customInterval": "2h",
        "min_doc_count": 1,
        "extended_bounds": {}
      }
    },
    {
      "id": "3",
      "enabled": true,
      "type": "histogram",
      "schema": "group",
      "params": {
        "field": "response_time",
        "interval": 100,
        "extended_bounds": {
          "min": 0,
          "max": 2000
        }
      }
    }
  ]
}
```

### 3. Geographic Visualization

```json
{
  "title": "Requests by Country",
  "type": "region_map",
  "params": {
    "addTooltip": true,
    "legendPosition": "bottomright",
    "mapZoom": 2,
    "mapCenter": [0, 0],
    "outlineWeight": 1,
    "colorSchema": "Yellow to Red",
    "isDisplayWarning": true
  },
  "aggs": [
    {
      "id": "1",
      "enabled": true,
      "type": "count",
      "schema": "metric",
      "params": {}
    },
    {
      "id": "2",
      "enabled": true,
      "type": "terms",
      "schema": "segment",
      "params": {
        "field": "geoip.country_name.keyword",
        "size": 50,
        "order": "desc",
        "orderBy": "1"
      }
    }
  ]
}
```

---

## 🚨 Alerting y Watcher

### 1. Configuración de Watcher

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
        "indices": ["logs-*"],
        "body": {
          "query": {
            "bool": {
              "must": [
                {
                  "range": {
                    "@timestamp": {
                      "gte": "now-5m"
                    }
                  }
                },
                {
                  "term": {
                    "level": "ERROR"
                  }
                }
              ]
            }
          },
          "aggs": {
            "error_count": {
              "cardinality": {
                "field": "message.keyword"
              }
            }
          }
        }
      }
    }
  },
  "condition": {
    "compare": {
      "ctx.payload.aggregations.error_count.value": {
        "gt": 10
      }
    }
  },
  "actions": {
    "send_email": {
      "email": {
        "profile": "standard",
        "to": ["ops-team@company.com"],
        "subject": "High Error Rate Alert",
        "body": "More than 10 unique errors detected in the last 5 minutes. Please investigate.\n\nError count: {{ctx.payload.aggregations.error_count.value}}\nTime: {{ctx.execution_time}}"
      }
    },
    "log_error": {
      "logging": {
        "level": "error",
        "text": "High error rate detected: {{ctx.payload.aggregations.error_count.value}} unique errors"
      }
    }
  }
}
```

### 2. Alertas con ElastAlert

```yaml
# elastalert/rules/high_error_rate.yml
name: High Error Rate
type: frequency
index: logs-*
num_events: 50
timeframe:
  minutes: 5

filter:
- terms:
    level: ["ERROR", "FATAL"]

alert:
- "email"
- "slack"

email:
- "ops-team@company.com"

slack:
webhook_url: "https://hooks.slack.com/services/YOUR/SLACK/WEBHOOK"
slack_channel_override: "#alerts"
slack_username_override: "ElastAlert"

alert_text: |
  High error rate detected!
  
  Service: {0}
  Error Count: {1}
  Time Period: {2}
  
  Top Errors:
  {3}

alert_text_args:
  - service
  - num_matches
  - timeframe
  - top_count_keys

include:
  - "@timestamp"
  - "service"
  - "level"
  - "message"
  - "host"
```

### 3. Custom Alert Script

```python
#!/usr/bin/env python3
# elk_alerting.py

import json
import requests
import smtplib
from datetime import datetime, timedelta
from email.mime.text import MIMEText
from email.mime.multipart import MIMEMultipart

class ELKAlerting:
    def __init__(self, es_host="http://localhost:9200"):
        self.es_host = es_host
        self.alerts = []
    
    def check_error_rate(self, threshold=10, time_window=5):
        """Verifica la tasa de errores en los últimos X minutos"""
        
        query = {
            "query": {
                "bool": {
                    "must": [
                        {
                            "range": {
                                "@timestamp": {
                                    "gte": f"now-{time_window}m"
                                }
                            }
                        },
                        {
                            "terms": {
                                "level": ["ERROR", "FATAL"]
                            }
                        }
                    ]
                }
            },
            "aggs": {
                "errors_by_service": {
                    "terms": {
                        "field": "service.keyword",
                        "size": 10
                    }
                },
                "error_timeline": {
                    "date_histogram": {
                        "field": "@timestamp",
                        "interval": "1m"
                    }
                }
            }
        }
        
        response = requests.post(
            f"{self.es_host}/logs-*/_search",
            headers={"Content-Type": "application/json"},
            data=json.dumps(query)
        )
        
        if response.status_code == 200:
            data = response.json()
            total_errors = data['hits']['total']['value']
            
            if total_errors > threshold:
                alert = {
                    'type': 'high_error_rate',
                    'severity': 'critical',
                    'message': f'High error rate detected: {total_errors} errors in {time_window} minutes',
                    'details': {
                        'total_errors': total_errors,
                        'time_window': time_window,
                        'services': data['aggregations']['errors_by_service']['buckets'],
                        'timeline': data['aggregations']['error_timeline']['buckets']
                    },
                    'timestamp': datetime.now().isoformat()
                }
                self.alerts.append(alert)
                return alert
        
        return None
    
    def check_response_time(self, threshold=2000, time_window=10):
        """Verifica tiempo de respuesta promedio"""
        
        query = {
            "query": {
                "bool": {
                    "must": [
                        {
                            "range": {
                                "@timestamp": {
                                    "gte": f"now-{time_window}m"
                                }
                            }
                        },
                        {
                            "exists": {
                                "field": "response_time"
                            }
                        }
                    ]
                }
            },
            "aggs": {
                "avg_response_time": {
                    "avg": {
                        "field": "response_time"
                    }
                },
                "p95_response_time": {
                    "percentiles": {
                        "field": "response_time",
                        "percents": [95]
                    }
                }
            }
        }
        
        response = requests.post(
            f"{self.es_host}/logs-*/_search",
            headers={"Content-Type": "application/json"},
            data=json.dumps(query)
        )
        
        if response.status_code == 200:
            data = response.json()
            avg_time = data['aggregations']['avg_response_time']['value']
            p95_time = data['aggregations']['p95_response_time']['values']['95.0']
            
            if avg_time > threshold:
                alert = {
                    'type': 'slow_response_time',
                    'severity': 'warning',
                    'message': f'Slow response time detected: {avg_time:.2f}ms average',
                    'details': {
                        'avg_response_time': avg_time,
                        'p95_response_time': p95_time,
                        'threshold': threshold,
                        'time_window': time_window
                    },
                    'timestamp': datetime.now().isoformat()
                }
                self.alerts.append(alert)
                return alert
        
        return None
    
    def send_slack_alert(self, alert, webhook_url):
        """Envía alerta a Slack"""
        
        color = "danger" if alert['severity'] == 'critical' else "warning"
        
        payload = {
            "text": f"🚨 ELK Alert: {alert['type']}",
            "attachments": [
                {
                    "color": color,
                    "fields": [
                        {
                            "title": "Message",
                            "value": alert['message'],
                            "short": False
                        },
                        {
                            "title": "Severity",
                            "value": alert['severity'],
                            "short": True
                        },
                        {
                            "title": "Timestamp",
                            "value": alert['timestamp'],
                            "short": True
                        }
                    ]
                }
            ]
        }
        
        response = requests.post(webhook_url, json=payload)
        return response.status_code == 200
    
    def send_email_alert(self, alert, smtp_config, recipients):
        """Envía alerta por email"""
        
        msg = MIMEMultipart()
        msg['From'] = smtp_config['from']
        msg['To'] = ', '.join(recipients)
        msg['Subject'] = f"ELK Alert: {alert['type']} - {alert['severity']}"
        
        body = f"""
        ELK Stack Alert
        
        Type: {alert['type']}
        Severity: {alert['severity']}
        Message: {alert['message']}
        Timestamp: {alert['timestamp']}
        
        Details:
        {json.dumps(alert['details'], indent=2)}
        
        This is an automated alert from your ELK monitoring system.
        """
        
        msg.attach(MIMEText(body, 'plain'))
        
        try:
            server = smtplib.SMTP(smtp_config['server'], smtp_config['port'])
            server.starttls()
            server.login(smtp_config['username'], smtp_config['password'])
            server.send_message(msg)
            server.quit()
            return True
        except Exception as e:
            print(f"Failed to send email: {str(e)}")
            return False
    
    def run_checks(self):
        """Ejecuta todas las verificaciones"""
        self.alerts = []
        
        # Verificar tasa de errores
        error_alert = self.check_error_rate(threshold=10, time_window=5)
        if error_alert:
            print(f"Error rate alert: {error_alert['message']}")
        
        # Verificar tiempo de respuesta
        response_alert = self.check_response_time(threshold=2000, time_window=10)
        if response_alert:
            print(f"Response time alert: {response_alert['message']}")
        
        return self.alerts

if __name__ == "__main__":
    alerting = ELKAlerting()
    alerts = alerting.run_checks()
    
    if alerts:
        print(f"Found {len(alerts)} alerts")
        for alert in alerts:
            print(f"- {alert['type']}: {alert['message']}")
    else:
        print("No alerts found")
```

---

## 🔒 Seguridad y Autenticación

### 1. Configuración de X-Pack Security

```yaml
# elasticsearch.yml
xpack.security.enabled: true
xpack.security.enrollment.enabled: true

xpack.security.http.ssl:
  enabled: true
  keystore.path: certs/http.p12

xpack.security.transport.ssl:
  enabled: true
  verification_mode: certificate
  keystore.path: certs/transport.p12
  truststore.path: certs/transport.p12

cluster.initial_master_nodes: ["elasticsearch"]
```

### 2. Gestión de Usuarios

```bash
#!/bin/bash
# security_setup.sh

# Configurar passwords iniciales
docker exec elasticsearch /usr/share/elasticsearch/bin/elasticsearch-setup-passwords auto

# Crear usuarios personalizados
docker exec elasticsearch /usr/share/elasticsearch/bin/elasticsearch-users useradd kibana_user -p kibana_password -r kibana_system

docker exec elasticsearch /usr/share/elasticsearch/bin/elasticsearch-users useradd logstash_user -p logstash_password -r logstash_writer

docker exec elasticsearch /usr/share/elasticsearch/bin/elasticsearch-users useradd readonly_user -p readonly_password -r viewer

# Crear roles personalizados
curl -X POST "localhost:9200/_security/role/log_reader" \
     -u elastic:password \
     -H "Content-Type: application/json" \
     -d '{
       "indices": [
         {
           "names": [ "logs-*" ],
           "privileges": [ "read", "view_index_metadata" ]
         }
       ]
     }'

# Asignar roles a usuarios
curl -X POST "localhost:9200/_security/user/log_analyst" \
     -u elastic:password \
     -H "Content-Type: application/json" \
     -d '{
       "password": "analyst_password",
       "roles": [ "log_reader" ],
       "full_name": "Log Analyst",
       "email": "analyst@company.com"
     }'
```

### 3. Configuración de SSL/TLS

```bash
#!/bin/bash
# generate_certificates.sh

# Crear CA
docker exec elasticsearch /usr/share/elasticsearch/bin/elasticsearch-certutil ca --out config/certs/elastic-stack-ca.p12 --pass ""

# Crear certificados para nodos
docker exec elasticsearch /usr/share/elasticsearch/bin/elasticsearch-certutil cert --ca config/certs/elastic-stack-ca.p12 --ca-pass "" --out config/certs/elastic-certificates.p12 --pass ""

# Crear certificados HTTP
docker exec elasticsearch /usr/share/elasticsearch/bin/elasticsearch-certutil http
```

---

## 🔧 Optimización y Performance

### 1. Configuración de Performance

```yaml
# elasticsearch.yml - Optimizaciones
# Memory settings
bootstrap.memory_lock: true
indices.memory.index_buffer_size: 10%

# Thread pool settings
thread_pool.write.queue_size: 1000
thread_pool.search.queue_size: 1000

# Indexing settings
indices.store.throttle.max_bytes_per_sec: 100mb

# Search settings
search.max_buckets: 100000

# Circuit breaker settings
indices.breaker.total.limit: 70%
indices.breaker.fielddata.limit: 40%
indices.breaker.request.limit: 60%
```

### 2. Monitoring y Metricas

```python
#!/usr/bin/env python3
# elk_monitoring.py

import requests
import json
from datetime import datetime

class ELKMonitoring:
    def __init__(self, es_host="http://localhost:9200"):
        self.es_host = es_host
    
    def get_cluster_health(self):
        """Obtiene salud del cluster"""
        response = requests.get(f"{self.es_host}/_cluster/health")
        return response.json()
    
    def get_node_stats(self):
        """Obtiene estadísticas de nodos"""
        response = requests.get(f"{self.es_host}/_nodes/stats")
        return response.json()
    
    def get_index_stats(self):
        """Obtiene estadísticas de índices"""
        response = requests.get(f"{self.es_host}/_stats")
        return response.json()
    
    def check_performance_metrics(self):
        """Verifica métricas de performance"""
        cluster_health = self.get_cluster_health()
        node_stats = self.get_node_stats()
        index_stats = self.get_index_stats()
        
        metrics = {
            'timestamp': datetime.now().isoformat(),
            'cluster': {
                'status': cluster_health['status'],
                'active_shards': cluster_health['active_shards'],
                'relocating_shards': cluster_health['relocating_shards'],
                'unassigned_shards': cluster_health['unassigned_shards']
            },
            'nodes': {}
        }
        
        for node_id, node_data in node_stats['nodes'].items():
            node_name = node_data['name']
            metrics['nodes'][node_name] = {
                'heap_used_percent': node_data['jvm']['mem']['heap_used_percent'],
                'cpu_percent': node_data['os']['cpu']['percent'],
                'load_average': node_data['os']['cpu']['load_average'],
                'disk_available_bytes': node_data['fs']['total']['available_in_bytes'],
                'indexing_rate': node_data['indices']['indexing']['index_total'],
                'search_rate': node_data['indices']['search']['query_total']
            }
        
        return metrics
    
    def generate_health_report(self):
        """Genera reporte de salud del stack"""
        metrics = self.check_performance_metrics()
        
        report = f"""
ELK Stack Health Report
Generated: {metrics['timestamp']}

Cluster Status: {metrics['cluster']['status']}
Active Shards: {metrics['cluster']['active_shards']}
Relocating Shards: {metrics['cluster']['relocating_shards']}
Unassigned Shards: {metrics['cluster']['unassigned_shards']}

Node Details:
"""
        
        for node_name, node_metrics in metrics['nodes'].items():
            report += f"""
{node_name}:
  - Heap Usage: {node_metrics['heap_used_percent']}%
  - CPU Usage: {node_metrics['cpu_percent']}%
  - Load Average: {node_metrics['load_average']}
  - Disk Available: {node_metrics['disk_available_bytes'] / (1024**3):.2f} GB
  - Indexing Rate: {node_metrics['indexing_rate']} docs/sec
  - Search Rate: {node_metrics['search_rate']} queries/sec
"""
        
        return report

if __name__ == "__main__":
    monitoring = ELKMonitoring()
    report = monitoring.generate_health_report()
    print(report)
```

---

## 📋 Ejercicios Prácticos

### Ejercicio 1: Setup Completo de ELK Stack

**Objetivo**: Configurar un stack ELK completo para monitorear una aplicación web.

**Tareas**:

1. Configurar Elasticsearch con templates e ILM policies
2. Configurar Logstash con pipelines personalizados
3. Configurar Filebeat para múltiples tipos de logs
4. Crear dashboards en Kibana
5. Implementar alertas básicas

### Ejercicio 2: Procesamiento Avanzado de Logs

**Objetivo**: Crear pipelines de Logstash para procesar logs complejos.

**Componentes**:

- Logs de aplicación JSON
- Logs de Nginx con geoIP
- Logs de sistema con parsing personalizado
- Enriquecimiento de datos

### Ejercicio 3: Dashboard de Análisis de Seguridad

**Objetivo**: Crear un dashboard para detectar anomalías de seguridad.

**Métricas a incluir**:

- Intentos de acceso fallidos
- Patrones de tráfico anómalos
- Detección de ataques comunes
- Análisis geográfico

---

## 🧪 Laboratorio

### Lab 1: Implementación ELK Stack

1. **Configurar el entorno completo**:

   ```bash
   mkdir elk-lab
   cd elk-lab
   # Crear estructura de configuración
   # Levantar servicios con docker-compose
   ```

2. **Configurar ingesta de datos**:
   - Configurar Filebeat para logs de aplicación
   - Configurar Metricbeat para métricas de sistema
   - Crear pipelines de Logstash

3. **Crear visualizaciones**:
   - Dashboard de aplicación
   - Dashboard de infraestructura
   - Dashboard de seguridad

### Lab 2: Optimización y Scaling

1. **Configurar cluster multi-nodo**
2. **Implementar ILM policies**
3. **Configurar monitoring y alerting**
4. **Probar escenarios de alta carga**

---

## 📖 Best Practices

### 1. Diseño de Índices

```yaml
index_design_principles:
  naming_convention:
    pattern: "logs-{service}-{environment}-{YYYY.MM.dd}"
    examples:
      - "logs-webapp-prod-2024.01.15"
      - "logs-api-staging-2024.01.15"
  
  mapping_optimization:
    - "Use keyword fields for exact match searches"
    - "Use text fields for full-text search"
    - "Disable _source for high-volume metrics"
    - "Use doc_values: false for fields not used in aggregations"
  
  sharding_strategy:
    - "1 shard per 10-50GB of data"
    - "Avoid over-sharding small indices"
    - "Use rollover for time-based indices"
```

### 2. Performance Optimization

```yaml
performance_best_practices:
  indexing:
    - "Bulk index documents in batches"
    - "Use appropriate refresh intervals"
    - "Disable replicas during bulk loading"
    - "Use SSD storage for better performance"
  
  searching:
    - "Use filters instead of queries when possible"
    - "Limit result set size"
    - "Use appropriate timeout values"
    - "Cache frequently used queries"
  
  monitoring:
    - "Monitor JVM heap usage"
    - "Track indexing and search rates"
    - "Monitor disk usage and I/O"
    - "Set up alerting for key metrics"
```

---

## ✅ Resumen del Módulo

En este módulo has aprendido:

1. **Arquitectura del ELK Stack** y sus componentes principales
2. **Instalación y configuración** completa del stack
3. **Configuración de Beats** para recolección de logs y métricas
4. **Pipelines de Logstash** para procesamiento avanzado de datos
5. **Gestión de índices** en Elasticsearch con templates e ILM
6. **Visualizaciones en Kibana** y creación de dashboards
7. **Sistema de alertas** con Watcher y herramientas externas
8. **Seguridad y performance** optimization

### Próximas etapas

- **Módulo 05**: Configuraremos sistemas de alertas avanzados con Alertmanager
- **Módulo 06**: Exploraremos distributed tracing con Jaeger y Zipkin
- **Módulo 07**: Implementaremos DevSecOps y gestión de secretos

¡Continúa con el siguiente módulo para dominar los sistemas de alertas avanzados!
