# Módulo 05: Alertmanager - Gestión Avanzada de Alertas

## 📋 Introducción

Alertmanager es el componente central del ecosistema Prometheus para la gestión de alertas. Maneja las alertas enviadas por aplicaciones cliente como el servidor Prometheus, se encarga de desduplicar, agrupar y enrutar las alertas a los receptores correctos como email, PagerDuty, Slack, y más.

### ¿Qué es Alertmanager?

Alertmanager es un sistema de gestión de alertas que:

- **Agrupa** alertas similares en una sola notificación
- **Desduplicar** alertas idénticas
- **Enruta** alertas a diferentes receptores según reglas configuradas
- **Silencia** alertas temporalmente
- **Inhibe** alertas de menor prioridad cuando hay alertas críticas
- **Gestiona escalación** de alertas no resueltas

### Características Principales

- **Agrupación inteligente**: Combina alertas relacionadas
- **Routing flexible**: Enrutamiento basado en etiquetas
- **Múltiples receptores**: Email, Slack, PagerDuty, webhooks
- **Silenciamiento**: Suprimir alertas temporalmente
- **Inhibición**: Prevenir spam de alertas relacionadas
- **High Availability**: Clustering para redundancia

---

## 🎯 Objetivos de Aprendizaje

Al completar este módulo, serás capaz de:

1. **Instalar y configurar** Alertmanager
2. **Configurar routing** de alertas basado en etiquetas
3. **Integrar múltiples receptores** (Slack, Email, PagerDuty)
4. **Implementar agrupación** y deduplicación
5. **Configurar silenciamientos** e inhibiciones
6. **Gestionar escalación** de alertas
7. **Configurar clustering** para alta disponibilidad
8. **Crear plantillas personalizadas** para notificaciones

---

## 🏗️ Instalación y Configuración Básica

### 1. Docker Compose Setup

```yaml
# docker-compose.alertmanager.yml
version: '3.8'

services:
  alertmanager:
    image: prom/alertmanager:v0.26.0
    container_name: alertmanager
    restart: unless-stopped
    ports:
      - "9093:9093"
    volumes:
      - ./alertmanager/config:/etc/alertmanager
      - alertmanager_data:/alertmanager
    command:
      - '--config.file=/etc/alertmanager/alertmanager.yml'
      - '--storage.path=/alertmanager'
      - '--web.external-url=http://localhost:9093'
      - '--cluster.listen-address=0.0.0.0:9094'
      - '--log.level=info'
    networks:
      - monitoring

  # Alertmanager secundario para clustering
  alertmanager-2:
    image: prom/alertmanager:v0.26.0
    container_name: alertmanager-2
    restart: unless-stopped
    ports:
      - "9094:9093"
    volumes:
      - ./alertmanager/config:/etc/alertmanager
      - alertmanager2_data:/alertmanager
    command:
      - '--config.file=/etc/alertmanager/alertmanager.yml'
      - '--storage.path=/alertmanager'
      - '--web.external-url=http://localhost:9094'
      - '--cluster.listen-address=0.0.0.0:9094'
      - '--cluster.peer=alertmanager:9094'
      - '--log.level=info'
    networks:
      - monitoring
    depends_on:
      - alertmanager

  # Webhook receiver para testing
  webhook-receiver:
    image: jrhunt/webhook:latest
    container_name: webhook-receiver
    ports:
      - "8080:8080"
    networks:
      - monitoring

volumes:
  alertmanager_data:
  alertmanager2_data:

networks:
  monitoring:
    external: true
```

### 2. Configuración Principal de Alertmanager

```yaml
# alertmanager/config/alertmanager.yml
global:
  # Configuración SMTP para email
  smtp_smarthost: 'localhost:587'
  smtp_from: 'alertmanager@company.com'
  smtp_auth_username: 'alertmanager@company.com'
  smtp_auth_password: 'your-smtp-password'
  
  # URLs de plantillas personalizadas
  http_config:
    follow_redirects: true
  
  # Configuraciones globales de resolución
  resolve_timeout: 5m

# Configuración de clustering
cluster:
  # Configuración de gossip protocol
  settle_timeout: 30s
  gossip_interval: 200ms
  peer_timeout: 15s
  push_pull_interval: 60s

# Plantillas personalizadas
templates:
  - '/etc/alertmanager/templates/*.tmpl'

# Definición de rutas de alertas
route:
  # Receptor por defecto
  receiver: 'default-receiver'
  
  # Tiempo de agrupación inicial
  group_wait: 30s
  
  # Tiempo entre alertas del mismo grupo
  group_interval: 5m
  
  # Tiempo antes de reenviar una alerta no resuelta
  repeat_interval: 4h
  
  # Criterios de agrupación
  group_by: ['alertname', 'cluster', 'service']

  # Rutas específicas
  routes:
    # Alertas críticas - escalación inmediata
    - match:
        severity: critical
      receiver: 'critical-alerts'
      group_wait: 10s
      group_interval: 1m
      repeat_interval: 30m
      routes:
        # Alertas críticas de base de datos
        - match:
            service: database
          receiver: 'database-critical'
          group_by: ['alertname', 'instance']
        
        # Alertas críticas de infraestructura
        - match:
            component: infrastructure
          receiver: 'infrastructure-critical'

    # Alertas de warning
    - match:
        severity: warning
      receiver: 'warning-alerts'
      group_wait: 5m
      group_interval: 10m
      repeat_interval: 2h

    # Alertas específicas por equipo
    - match:
        team: backend
      receiver: 'backend-team'
      group_by: ['alertname', 'service']
      
    - match:
        team: frontend
      receiver: 'frontend-team'
      group_by: ['alertname', 'service']

    # Alertas de desarrollo (solo durante horas laborales)
    - match:
        environment: development
      receiver: 'dev-alerts'
      group_wait: 10m
      repeat_interval: 6h
      active_time_intervals:
        - 'business-hours'

# Definición de receptores
receivers:
  # Receptor por defecto
  - name: 'default-receiver'
    webhook_configs:
      - url: 'http://webhook-receiver:8080/webhook'
        send_resolved: true

  # Alertas críticas - múltiples canales
  - name: 'critical-alerts'
    email_configs:
      - to: 'oncall@company.com'
        subject: '🚨 CRITICAL: {{ .GroupLabels.alertname }}'
        body: |
          {{ range .Alerts }}
          Alert: {{ .Annotations.summary }}
          Description: {{ .Annotations.description }}
          Labels: {{ range .Labels.SortedPairs }}{{ .Name }}={{ .Value }} {{ end }}
          {{ end }}
        headers:
          Priority: 'high'
    
    slack_configs:
      - api_url: 'https://hooks.slack.com/services/YOUR/SLACK/WEBHOOK'
        channel: '#critical-alerts'
        title: '🚨 Critical Alert'
        text: '{{ range .Alerts }}{{ .Annotations.summary }}{{ end }}'
        color: 'danger'
        send_resolved: true
    
    pagerduty_configs:
      - routing_key: 'your-pagerduty-integration-key'
        description: '{{ range .Alerts }}{{ .Annotations.summary }}{{ end }}'
        severity: 'critical'

  # Alertas críticas de base de datos
  - name: 'database-critical'
    email_configs:
      - to: 'dba@company.com, oncall@company.com'
        subject: '🔥 DATABASE CRITICAL: {{ .GroupLabels.alertname }}'
        body: |
          Database Critical Alert!
          
          {{ range .Alerts }}
          Instance: {{ .Labels.instance }}
          Alert: {{ .Annotations.summary }}
          Description: {{ .Annotations.description }}
          Runbook: {{ .Annotations.runbook_url }}
          {{ end }}
    
    slack_configs:
      - api_url: 'https://hooks.slack.com/services/YOUR/SLACK/WEBHOOK'
        channel: '#database-alerts'
        title: '🔥 Database Critical'
        text: |
          {{ range .Alerts }}
          *Instance:* {{ .Labels.instance }}
          *Alert:* {{ .Annotations.summary }}
          *Description:* {{ .Annotations.description }}
          {{ if .Annotations.runbook_url }}*Runbook:* {{ .Annotations.runbook_url }}{{ end }}
          {{ end }}
        color: 'danger'

  # Alertas de warning
  - name: 'warning-alerts'
    slack_configs:
      - api_url: 'https://hooks.slack.com/services/YOUR/SLACK/WEBHOOK'
        channel: '#monitoring-alerts'
        title: '⚠️ Warning Alert'
        text: '{{ range .Alerts }}{{ .Annotations.summary }}{{ end }}'
        color: 'warning'
    
    email_configs:
      - to: 'monitoring@company.com'
        subject: '⚠️ WARNING: {{ .GroupLabels.alertname }}'
        body: |
          Warning Alert Details:
          {{ range .Alerts }}
          - {{ .Annotations.summary }}
          {{ end }}

  # Equipo Backend
  - name: 'backend-team'
    slack_configs:
      - api_url: 'https://hooks.slack.com/services/YOUR/SLACK/WEBHOOK'
        channel: '#backend-alerts'
        title: 'Backend Service Alert'
        text: |
          {{ range .Alerts }}
          *Service:* {{ .Labels.service }}
          *Alert:* {{ .Annotations.summary }}
          {{ end }}
    
    email_configs:
      - to: 'backend-team@company.com'
        subject: 'Backend Alert: {{ .GroupLabels.service }}'

  # Equipo Frontend
  - name: 'frontend-team'
    slack_configs:
      - api_url: 'https://hooks.slack.com/services/YOUR/SLACK/WEBHOOK'
        channel: '#frontend-alerts'
        title: 'Frontend Alert'
        text: |
          {{ range .Alerts }}
          *Service:* {{ .Labels.service }}
          *Alert:* {{ .Annotations.summary }}
          {{ end }}

  # Alertas de desarrollo
  - name: 'dev-alerts'
    slack_configs:
      - api_url: 'https://hooks.slack.com/services/YOUR/SLACK/WEBHOOK'
        channel: '#dev-alerts'
        title: 'Development Environment Alert'
        text: '{{ range .Alerts }}{{ .Annotations.summary }}{{ end }}'
        color: 'good'

# Configuración de intervalos de tiempo activos
time_intervals:
  - name: 'business-hours'
    time_intervals:
      - times:
          - start_time: '09:00'
            end_time: '17:00'
        weekdays: ['monday:friday']
        location: 'America/New_York'

# Configuración de inhibiciones
inhibit_rules:
  # Si hay una alerta crítica, inhibir warnings relacionadas
  - source_matchers:
      - severity="critical"
    target_matchers:
      - severity="warning"
    equal: ['alertname', 'instance']

  # Si el cluster está down, inhibir alertas de servicios individuales
  - source_matchers:
      - alertname="ClusterDown"
    target_matchers:
      - alertname=~"Service.*Down"
    equal: ['cluster']

  # Si hay alerta de disco lleno, inhibir alertas de performance
  - source_matchers:
      - alertname="DiskSpaceFull"
    target_matchers:
      - alertname=~".*SlowResponse|.*HighLatency"
    equal: ['instance']
```

---

## 📧 Configuración de Receptores

### 1. Configuración de Email

```yaml
# Configuración avanzada de email
receivers:
  - name: 'email-alerts'
    email_configs:
      - to: 'alerts@company.com'
        from: 'alertmanager@company.com'
        subject: '[{{ .Status | toUpper }}] {{ .GroupLabels.alertname }} ({{ len .Alerts }} alerts)'
        body: |
          {{ if gt (len .Alerts.Firing) 0 }}
          **FIRING ALERTS:**
          {{ range .Alerts.Firing }}
          - **{{ .Labels.alertname }}** on {{ .Labels.instance }}
            Summary: {{ .Annotations.summary }}
            Description: {{ .Annotations.description }}
            Started: {{ .StartsAt.Format "2006-01-02 15:04:05" }}
          {{ end }}
          {{ end }}
          
          {{ if gt (len .Alerts.Resolved) 0 }}
          **RESOLVED ALERTS:**
          {{ range .Alerts.Resolved }}
          - **{{ .Labels.alertname }}** on {{ .Labels.instance }}
            Resolved: {{ .EndsAt.Format "2006-01-02 15:04:05" }}
          {{ end }}
          {{ end }}
        
        # Configuración SMTP específica
        smarthost: 'smtp.gmail.com:587'
        auth_username: 'your-email@gmail.com'
        auth_password: 'your-app-password'
        auth_secret: 'your-secret'
        auth_identity: 'your-email@gmail.com'
        
        # Headers personalizados
        headers:
          X-Priority: 'high'
          X-Alertmanager: 'true'
        
        # Configuración TLS
        require_tls: true
```

### 2. Configuración de Slack

```yaml
receivers:
  - name: 'slack-alerts'
    slack_configs:
      - api_url: 'https://hooks.slack.com/services/YOUR_TEAM_ID/YOUR_CHANNEL_ID/YOUR_WEBHOOK_TOKEN'
        channel: '#alerts'
        username: 'Alertmanager'
        icon_emoji: ':exclamation:'
        
        # Título dinámico
        title: |
          {{ if eq .Status "firing" }}🔥{{ else }}✅{{ end }} 
          {{ .GroupLabels.alertname }} 
          ({{ len .Alerts.Firing }} firing, {{ len .Alerts.Resolved }} resolved)
        
        # Mensaje principal
        text: |
          {{ range .Alerts }}
          {{ if eq .Status "firing" }}🚨{{ else }}✅{{ end }} 
          *{{ .Labels.alertname }}* on `{{ .Labels.instance }}`
          {{ .Annotations.summary }}
          {{ if .Annotations.runbook_url }}
          <{{ .Annotations.runbook_url }}|Runbook>
          {{ end }}
          *Started:* {{ .StartsAt.Format "2006-01-02 15:04:05" }}
          {{ if ne .Status "firing" }}*Resolved:* {{ .EndsAt.Format "2006-01-02 15:04:05" }}{{ end }}
          {{ end }}
        
        # Color basado en severidad
        color: |
          {{ if eq .Status "firing" }}
            {{ if eq .CommonLabels.severity "critical" }}danger{{ else if eq .CommonLabels.severity "warning" }}warning{{ else }}good{{ end }}
          {{ else }}good{{ end }}
        
        # Campos adicionales
        fields:
          - title: 'Environment'
            value: '{{ .CommonLabels.environment }}'
            short: true
          - title: 'Service'
            value: '{{ .CommonLabels.service }}'
            short: true
        
        # Acciones
        actions:
          - type: 'button'
            text: 'Runbook'
            url: '{{ (index .Alerts 0).Annotations.runbook_url }}'
          - type: 'button'
            text: 'Silence'
            url: 'http://alertmanager:9093/#/silences/new'
          - type: 'button'
            text: 'Dashboard'
            url: '{{ (index .Alerts 0).Annotations.dashboard_url }}'

        send_resolved: true
        
        # Link names y unfurl
        link_names: true
        mrkdwn_in: ['text', 'pretext']
```

### 3. Configuración de PagerDuty

```yaml
receivers:
  - name: 'pagerduty-alerts'
    pagerduty_configs:
      - routing_key: 'YOUR_PAGERDUTY_INTEGRATION_KEY'
        
        # Descripción de la alerta
        description: |
          {{ range .Alerts.Firing }}
          {{ .Labels.alertname }}: {{ .Annotations.summary }}
          {{ end }}
        
        # Severidad dinámica
        severity: |
          {{ if eq .CommonLabels.severity "critical" }}critical
          {{ else if eq .CommonLabels.severity "warning" }}warning
          {{ else }}info{{ end }}
        
        # Clase de servicio
        class: '{{ .CommonLabels.service | default "unknown" }}'
        
        # Componente afectado
        component: '{{ .CommonLabels.instance }}'
        
        # Grupo para deduplicación
        group: '{{ .GroupLabels.alertname }}'
        
        # Detalles adicionales
        details:
          alertname: '{{ .GroupLabels.alertname }}'
          environment: '{{ .CommonLabels.environment }}'
          instance: '{{ .CommonLabels.instance }}'
          job: '{{ .CommonLabels.job }}'
          severity: '{{ .CommonLabels.severity }}'
          firing_alerts: '{{ len .Alerts.Firing }}'
          resolved_alerts: '{{ len .Alerts.Resolved }}'
        
        # URLs de contexto
        client: 'Alertmanager'
        client_url: 'http://alertmanager:9093'
        
        # Configuración de TLS
        http_config:
          tls_config:
            insecure_skip_verify: false
```

### 4. Configuración de Microsoft Teams

```yaml
receivers:
  - name: 'msteams-alerts'
    msteams_configs:
      - webhook_url: 'https://outlook.office.com/webhook/...'
        
        # Título de la carta
        title: |
          {{ if eq .Status "firing" }}🔥 ALERT FIRING{{ else }}✅ ALERT RESOLVED{{ end }}
          {{ .GroupLabels.alertname }}
        
        # Resumen
        summary: |
          {{ len .Alerts.Firing }} firing, {{ len .Alerts.Resolved }} resolved
        
        # Texto principal
        text: |
          {{ range .Alerts }}
          **{{ .Labels.alertname }}** on {{ .Labels.instance }}
          
          {{ .Annotations.summary }}
          {{ if .Annotations.description }}
          {{ .Annotations.description }}
          {{ end }}
          
          **Status:** {{ .Status | title }}
          **Severity:** {{ .Labels.severity | title }}
          **Started:** {{ .StartsAt.Format "2006-01-02 15:04:05 MST" }}
          {{ if ne .Status "firing" }}**Resolved:** {{ .EndsAt.Format "2006-01-02 15:04:05 MST" }}{{ end }}
          
          ---
          {{ end }}

        # Color temático
        theme_color: |
          {{ if eq .Status "firing" }}
            {{ if eq .CommonLabels.severity "critical" }}FF0000{{ else if eq .CommonLabels.severity "warning" }}FFA500{{ else }}FFFF00{{ end }}
          {{ else }}00FF00{{ end }}
```

---

## 🔕 Silenciamientos e Inhibiciones

### 1. Configuración de Silenciamientos

```bash
#!/bin/bash
# silence_management.sh

ALERTMANAGER_URL="http://localhost:9093"

# Crear silenciamiento por mantenimiento
create_maintenance_silence() {
    local service="$1"
    local duration="$2"
    local comment="$3"
    
    curl -X POST "$ALERTMANAGER_URL/api/v1/silences" \
         -H "Content-Type: application/json" \
         -d "{
           \"matchers\": [
             {
               \"name\": \"service\",
               \"value\": \"$service\",
               \"isRegex\": false
             }
           ],
           \"startsAt\": \"$(date -u +%Y-%m-%dT%H:%M:%S.000Z)\",
           \"endsAt\": \"$(date -u -d \"+$duration\" +%Y-%m-%dT%H:%M:%S.000Z)\",
           \"createdBy\": \"maintenance-script\",
           \"comment\": \"$comment\"
         }"
}

# Silenciar por instancia específica
silence_instance() {
    local instance="$1"
    local duration="$2"
    local reason="$3"
    
    curl -X POST "$ALERTMANAGER_URL/api/v1/silences" \
         -H "Content-Type: application/json" \
         -d "{
           \"matchers\": [
             {
               \"name\": \"instance\",
               \"value\": \"$instance\",
               \"isRegex\": false
             }
           ],
           \"startsAt\": \"$(date -u +%Y-%m-%dT%H:%M:%S.000Z)\",
           \"endsAt\": \"$(date -u -d \"+$duration\" +%Y-%m-%dT%H:%M:%S.000Z)\",
           \"createdBy\": \"ops-team\",
           \"comment\": \"$reason\"
         }"
}

# Silenciar alertas por regex
silence_by_regex() {
    local alert_pattern="$1"
    local duration="$2"
    local comment="$3"
    
    curl -X POST "$ALERTMANAGER_URL/api/v1/silences" \
         -H "Content-Type: application/json" \
         -d "{
           \"matchers\": [
             {
               \"name\": \"alertname\",
               \"value\": \"$alert_pattern\",
               \"isRegex\": true
             }
           ],
           \"startsAt\": \"$(date -u +%Y-%m-%dT%H:%M:%S.000Z)\",
           \"endsAt\": \"$(date -u -d \"+$duration\" +%Y-%m-%dT%H:%M:%S.000Z)\",
           \"createdBy\": \"automation\",
           \"comment\": \"$comment\"
         }"
}

# Listar silenciamientos activos
list_active_silences() {
    curl -s "$ALERTMANAGER_URL/api/v1/silences" | \
    jq -r '.data[] | select(.status.state == "active") | 
           "\(.id): \(.comment) (expires: \(.endsAt))"'
}

# Eliminar silenciamiento
delete_silence() {
    local silence_id="$1"
    
    curl -X DELETE "$ALERTMANAGER_URL/api/v1/silence/$silence_id"
}

# Uso de funciones
case "$1" in
    "maintenance")
        create_maintenance_silence "$2" "$3" "$4"
        ;;
    "instance")
        silence_instance "$2" "$3" "$4"
        ;;
    "regex")
        silence_by_regex "$2" "$3" "$4"
        ;;
    "list")
        list_active_silences
        ;;
    "delete")
        delete_silence "$2"
        ;;
    *)
        echo "Usage: $0 {maintenance|instance|regex|list|delete} [args...]"
        echo "Examples:"
        echo "  $0 maintenance web-service 2h 'Planned maintenance'"
        echo "  $0 instance server01:9100 30m 'Reboot for updates'"
        echo "  $0 regex 'Disk.*Warning' 1h 'Known disk issue'"
        echo "  $0 list"
        echo "  $0 delete abc123def"
        ;;
esac
```

### 2. Inhibiciones Avanzadas

```yaml
# alertmanager.yml - Sección de inhibiciones
inhibit_rules:
  # Inhibir alertas de servicios cuando el nodo está down
  - source_matchers:
      - alertname="NodeDown"
    target_matchers:
      - alertname=~"Service.*"
    equal: ['instance']

  # Inhibir alertas de disco cuando hay problemas de red
  - source_matchers:
      - alertname="NetworkUnreachable"
    target_matchers:
      - alertname=~"Disk.*|Filesystem.*"
    equal: ['instance']

  # Inhibir warnings cuando hay critical del mismo tipo
  - source_matchers:
      - severity="critical"
    target_matchers:
      - severity="warning"
    equal: ['alertname', 'instance']

  # Inhibir alertas de performance durante backup
  - source_matchers:
      - alertname="BackupRunning"
    target_matchers:
      - alertname=~".*HighLatency|.*SlowResponse|.*HighCPU"
    equal: ['instance']

  # Inhibir alertas de aplicación cuando hay problemas de base de datos
  - source_matchers:
      - alertname=~"Database.*Down|Database.*Unreachable"
    target_matchers:
      - alertname=~"Application.*Error|Service.*Timeout"
    equal: ['environment', 'cluster']

  # Inhibir alertas individuales durante mantenimiento del cluster
  - source_matchers:
      - alertname="MaintenanceMode"
    target_matchers:
      - alertname!="MaintenanceMode"
    equal: ['cluster']
```

---

## 📊 Plantillas Personalizadas

### 1. Plantilla HTML para Email

```html
<!-- alertmanager/templates/email.html.tmpl -->
{{ define "email.subject" }}
[{{ .Status | toUpper }}{{ if eq .Status "firing" }}: {{ .Alerts.Firing | len }}{{ end }}] 
{{ range .GroupLabels.SortedPairs }}{{ .Name }}={{ .Value }} {{ end }}
{{ end }}

{{ define "email.html" }}
<!DOCTYPE html>
<html>
<head>
    <meta charset="UTF-8">
    <title>Alert Notification</title>
    <style>
        body { font-family: Arial, sans-serif; margin: 0; padding: 20px; background-color: #f5f5f5; }
        .container { max-width: 800px; margin: 0 auto; background: white; padding: 20px; border-radius: 8px; }
        .header { background: {{ if eq .Status "firing" }}#d32f2f{{ else }}#388e3c{{ end }}; 
                  color: white; padding: 15px; border-radius: 4px; margin-bottom: 20px; }
        .alert { border-left: 4px solid {{ if eq .Status "firing" }}#f44336{{ else }}#4caf50{{ end }}; 
                 padding: 15px; margin: 10px 0; background: #f9f9f9; }
        .alert-critical { border-left-color: #d32f2f; background: #ffebee; }
        .alert-warning { border-left-color: #ff9800; background: #fff3e0; }
        .alert-info { border-left-color: #2196f3; background: #e3f2fd; }
        .label { background: #e0e0e0; padding: 2px 6px; border-radius: 3px; font-size: 12px; margin: 2px; }
        .resolved { opacity: 0.7; }
        .footer { margin-top: 20px; padding-top: 20px; border-top: 1px solid #ddd; 
                  font-size: 12px; color: #666; }
        a { color: #1976d2; text-decoration: none; }
        a:hover { text-decoration: underline; }
    </style>
</head>
<body>
    <div class="container">
        <div class="header">
            <h2>{{ if eq .Status "firing" }}🚨 Alert Firing{{ else }}✅ Alert Resolved{{ end }}</h2>
            <p>{{ len .Alerts.Firing }} firing, {{ len .Alerts.Resolved }} resolved</p>
        </div>

        {{ if gt (len .Alerts.Firing) 0 }}
        <h3>🔥 Firing Alerts</h3>
        {{ range .Alerts.Firing }}
        <div class="alert alert-{{ .Labels.severity | default "info" }}">
            <h4>{{ .Labels.alertname }}</h4>
            <p><strong>Summary:</strong> {{ .Annotations.summary }}</p>
            {{ if .Annotations.description }}
            <p><strong>Description:</strong> {{ .Annotations.description }}</p>
            {{ end }}
            
            <div style="margin: 10px 0;">
                <strong>Labels:</strong><br>
                {{ range .Labels.SortedPairs }}
                <span class="label">{{ .Name }}={{ .Value }}</span>
                {{ end }}
            </div>
            
            <p><strong>Started:</strong> {{ .StartsAt.Format "2006-01-02 15:04:05 MST" }}</p>
            
            {{ if .Annotations.runbook_url }}
            <p><a href="{{ .Annotations.runbook_url }}" style="color: #1976d2;">📖 View Runbook</a></p>
            {{ end }}
            
            {{ if .Annotations.dashboard_url }}
            <p><a href="{{ .Annotations.dashboard_url }}" style="color: #1976d2;">📊 View Dashboard</a></p>
            {{ end }}
        </div>
        {{ end }}
        {{ end }}

        {{ if gt (len .Alerts.Resolved) 0 }}
        <h3>✅ Resolved Alerts</h3>
        {{ range .Alerts.Resolved }}
        <div class="alert resolved">
            <h4>{{ .Labels.alertname }}</h4>
            <p><strong>Summary:</strong> {{ .Annotations.summary }}</p>
            <p><strong>Resolved:</strong> {{ .EndsAt.Format "2006-01-02 15:04:05 MST" }}</p>
            <p><strong>Duration:</strong> {{ .EndsAt.Sub .StartsAt }}</p>
        </div>
        {{ end }}
        {{ end }}

        <div class="footer">
            <p>This alert was sent by Alertmanager at {{ now.Format "2006-01-02 15:04:05 MST" }}</p>
            <p><a href="http://alertmanager:9093">Alertmanager Dashboard</a> | 
               <a href="http://prometheus:9090">Prometheus</a> | 
               <a href="http://grafana:3000">Grafana</a></p>
        </div>
    </div>
</body>
</html>
{{ end }}
```

### 2. Plantilla Avanzada para Slack

```go
// alertmanager/templates/slack.tmpl
{{ define "slack.title" }}
{{ if eq .Status "firing" }}🚨{{ else }}✅{{ end }} 
{{ range .GroupLabels.SortedPairs }}{{ .Name }}={{ .Value }} {{ end }}
{{ if eq .Status "firing" }}({{ .Alerts.Firing | len }} firing){{ else }}(resolved){{ end }}
{{ end }}

{{ define "slack.text" }}
{{ if eq .Status "firing" }}
*🔥 FIRING ALERTS*
{{ range .Alerts.Firing }}
*{{ .Labels.alertname }}* {{ if .Labels.severity }}[{{ .Labels.severity | title }}]{{ end }}
{{ if .Labels.instance }}📍 {{ .Labels.instance }}{{ end }}
{{ if .Annotations.summary }}📝 {{ .Annotations.summary }}{{ end }}
{{ if .Annotations.description }}📋 {{ .Annotations.description }}{{ end }}
⏰ Started: {{ .StartsAt.Format "Jan 02, 15:04 MST" }}
{{ if .Annotations.runbook_url }}📖 <{{ .Annotations.runbook_url }}|Runbook>{{ end }}
{{ if .Annotations.dashboard_url }}📊 <{{ .Annotations.dashboard_url }}|Dashboard>{{ end }}

{{ end }}
{{ else }}
*✅ RESOLVED ALERTS*
{{ range .Alerts.Resolved }}
*{{ .Labels.alertname }}* {{ if .Labels.severity }}[{{ .Labels.severity | title }}]{{ end }}
{{ if .Labels.instance }}📍 {{ .Labels.instance }}{{ end }}
⏰ Resolved: {{ .EndsAt.Format "Jan 02, 15:04 MST" }}
⌛ Duration: {{ .EndsAt.Sub .StartsAt }}

{{ end }}
{{ end }}

---
🔗 <http://alertmanager:9093|Alertmanager> | <http://prometheus:9090|Prometheus> | <http://grafana:3000|Grafana>
{{ end }}

{{ define "slack.color" }}
{{ if eq .Status "firing" }}
  {{ if eq .CommonLabels.severity "critical" }}danger
  {{ else if eq .CommonLabels.severity "warning" }}warning
  {{ else }}#ff9800{{ end }}
{{ else }}good{{ end }}
{{ end }}
```

### 3. Plantilla para Microsoft Teams

```json
{{/* alertmanager/templates/teams.tmpl */}}
{{ define "teams.title" }}
{{ if eq .Status "firing" }}🚨 ALERT FIRING{{ else }}✅ ALERTS RESOLVED{{ end }}
{{ end }}

{{ define "teams.text" }}
{{ if eq .Status "firing" }}
**🔥 FIRING ALERTS ({{ .Alerts.Firing | len }})**

{{ range .Alerts.Firing }}
**{{ .Labels.alertname }}** {{ if .Labels.severity }}[{{ .Labels.severity | upper }}]{{ end }}

{{ if .Annotations.summary }}📝 **Summary:** {{ .Annotations.summary }}{{ end }}
{{ if .Annotations.description }}📋 **Description:** {{ .Annotations.description }}{{ end }}
{{ if .Labels.instance }}📍 **Instance:** {{ .Labels.instance }}{{ end }}
{{ if .Labels.service }}🔧 **Service:** {{ .Labels.service }}{{ end }}
⏰ **Started:** {{ .StartsAt.Format "2006-01-02 15:04:05 MST" }}

{{ if .Annotations.runbook_url }}[📖 View Runbook]({{ .Annotations.runbook_url }}){{ end }}
{{ if .Annotations.dashboard_url }}[📊 View Dashboard]({{ .Annotations.dashboard_url }}){{ end }}

---
{{ end }}
{{ else }}
**✅ RESOLVED ALERTS ({{ .Alerts.Resolved | len }})**

{{ range .Alerts.Resolved }}
**{{ .Labels.alertname }}** was resolved
⏰ **Resolved:** {{ .EndsAt.Format "2006-01-02 15:04:05 MST" }}
⌛ **Duration:** {{ .EndsAt.Sub .StartsAt }}

{{ end }}
{{ end }}

[🔗 Alertmanager](http://alertmanager:9093) | [🔗 Prometheus](http://prometheus:9090) | [🔗 Grafana](http://grafana:3000)
{{ end }}
```

---

## 🔄 Escalación de Alertas

### 1. Configuración de Escalación

```yaml
# alertmanager.yml - Configuración de escalación
route:
  receiver: 'default-receiver'
  group_wait: 30s
  group_interval: 5m
  repeat_interval: 4h
  routes:
    # Escalación para alertas críticas
    - match:
        severity: critical
      receiver: 'level1-oncall'
      group_wait: 10s
      repeat_interval: 15m
      routes:
        # Escalación nivel 2 después de 30 minutos
        - match:
            escalation: level2
          receiver: 'level2-oncall'
          repeat_interval: 30m
          routes:
            # Escalación nivel 3 después de 1 hora
            - match:
                escalation: level3
              receiver: 'management-escalation'
              repeat_interval: 1h

receivers:
  # Nivel 1 - On-call engineer
  - name: 'level1-oncall'
    pagerduty_configs:
      - routing_key: 'level1-pagerduty-key'
        description: 'L1: {{ range .Alerts }}{{ .Annotations.summary }}{{ end }}'
    
    slack_configs:
      - api_url: 'https://hooks.slack.com/services/YOUR/SLACK/WEBHOOK'
        channel: '#oncall-level1'
        title: '🚨 Level 1 Critical Alert'

  # Nivel 2 - Senior engineers + Manager
  - name: 'level2-oncall'
    pagerduty_configs:
      - routing_key: 'level2-pagerduty-key'
        description: 'L2 ESCALATION: {{ range .Alerts }}{{ .Annotations.summary }}{{ end }}'
    
    email_configs:
      - to: 'senior-engineers@company.com, engineering-manager@company.com'
        subject: '🚨 LEVEL 2 ESCALATION: Critical Alert'
    
    slack_configs:
      - api_url: 'https://hooks.slack.com/services/YOUR/SLACK/WEBHOOK'
        channel: '#escalation-level2'
        title: '🚨🚨 Level 2 Escalation - Critical Alert'

  # Nivel 3 - Management escalation
  - name: 'management-escalation'
    email_configs:
      - to: 'cto@company.com, vp-engineering@company.com'
        subject: '🚨🚨🚨 MANAGEMENT ESCALATION: Unresolved Critical Alert'
    
    slack_configs:
      - api_url: 'https://hooks.slack.com/services/YOUR/SLACK/WEBHOOK'
        channel: '#management-alerts'
        title: '🚨🚨🚨 Management Escalation Required'
```

### 2. Script de Escalación Automática

```python
#!/usr/bin/env python3
# escalation_manager.py

import requests
import json
import time
from datetime import datetime, timedelta
import logging

logging.basicConfig(level=logging.INFO)
logger = logging.getLogger(__name__)

class EscalationManager:
    def __init__(self, alertmanager_url="http://localhost:9093"):
        self.alertmanager_url = alertmanager_url
        self.escalation_levels = {
            'level1': {'delay': 900, 'label': 'escalation=level1'},      # 15 min
            'level2': {'delay': 1800, 'label': 'escalation=level2'},     # 30 min
            'level3': {'delay': 3600, 'label': 'escalation=level3'},     # 60 min
        }
    
    def get_firing_alerts(self):
        """Obtiene alertas activas"""
        try:
            response = requests.get(f"{self.alertmanager_url}/api/v1/alerts")
            response.raise_for_status()
            return response.json()['data']
        except Exception as e:
            logger.error(f"Error getting alerts: {e}")
            return []
    
    def should_escalate(self, alert, level):
        """Determina si una alerta debe escalarse"""
        if alert['status']['state'] != 'active':
            return False
        
        # Verificar si ya está en este nivel de escalación
        if any(label.get('name') == 'escalation' and 
               label.get('value') == level for label in alert.get('labels', [])):
            return False
        
        # Calcular tiempo desde que comenzó la alerta
        starts_at = datetime.fromisoformat(alert['startsAt'].replace('Z', '+00:00'))
        now = datetime.now().astimezone()
        alert_duration = (now - starts_at).total_seconds()
        
        # Verificar si debe escalarse
        escalation_delay = self.escalation_levels[level]['delay']
        return alert_duration >= escalation_delay
    
    def create_escalation_alert(self, original_alert, level):
        """Crea una nueva alerta de escalación"""
        escalation_alert = {
            'labels': {**original_alert['labels'], 'escalation': level},
            'annotations': {
                **original_alert['annotations'],
                'escalation_level': level,
                'original_alert': original_alert['labels']['alertname'],
                'escalation_time': datetime.now().isoformat()
            },
            'generatorURL': original_alert.get('generatorURL', ''),
            'startsAt': datetime.now().isoformat() + 'Z'
        }
        
        # Enviar alerta escalada
        try:
            response = requests.post(
                f"{self.alertmanager_url}/api/v1/alerts",
                json=[escalation_alert],
                headers={'Content-Type': 'application/json'}
            )
            response.raise_for_status()
            logger.info(f"Escalated alert {original_alert['labels']['alertname']} to {level}")
            return True
        except Exception as e:
            logger.error(f"Error creating escalation alert: {e}")
            return False
    
    def process_escalations(self):
        """Procesa escalaciones para todas las alertas activas"""
        alerts = self.get_firing_alerts()
        
        for alert in alerts:
            # Solo procesar alertas críticas
            if alert['labels'].get('severity') != 'critical':
                continue
            
            # Verificar cada nivel de escalación
            for level in ['level1', 'level2', 'level3']:
                if self.should_escalate(alert, level):
                    self.create_escalation_alert(alert, level)
                    break  # Solo escalar un nivel a la vez
    
    def run_daemon(self, interval=300):  # 5 minutos
        """Ejecuta el manager como daemon"""
        logger.info("Starting escalation manager daemon")
        
        while True:
            try:
                self.process_escalations()
                time.sleep(interval)
            except KeyboardInterrupt:
                logger.info("Shutting down escalation manager")
                break
            except Exception as e:
                logger.error(f"Error in escalation loop: {e}")
                time.sleep(60)  # Esperar 1 minuto antes de reintentar

if __name__ == "__main__":
    manager = EscalationManager()
    manager.run_daemon()
```

---

## 🏗️ Clustering y Alta Disponibilidad

### 1. Configuración de Cluster

```yaml
# docker-compose.cluster.yml
version: '3.8'

services:
  alertmanager-1:
    image: prom/alertmanager:v0.26.0
    container_name: alertmanager-1
    hostname: alertmanager-1
    restart: unless-stopped
    ports:
      - "9093:9093"
      - "9094:9094"
    volumes:
      - ./alertmanager/config:/etc/alertmanager
      - alertmanager1_data:/alertmanager
    command:
      - '--config.file=/etc/alertmanager/alertmanager.yml'
      - '--storage.path=/alertmanager'
      - '--web.external-url=http://localhost:9093'
      - '--cluster.listen-address=0.0.0.0:9094'
      - '--cluster.advertise-address=alertmanager-1:9094'
      - '--log.level=info'
      - '--cluster.settle-timeout=30s'
      - '--cluster.gossip-interval=200ms'
    networks:
      - monitoring

  alertmanager-2:
    image: prom/alertmanager:v0.26.0
    container_name: alertmanager-2
    hostname: alertmanager-2
    restart: unless-stopped
    ports:
      - "9095:9093"
      - "9096:9094"
    volumes:
      - ./alertmanager/config:/etc/alertmanager
      - alertmanager2_data:/alertmanager
    command:
      - '--config.file=/etc/alertmanager/alertmanager.yml'
      - '--storage.path=/alertmanager'
      - '--web.external-url=http://localhost:9095'
      - '--cluster.listen-address=0.0.0.0:9094'
      - '--cluster.advertise-address=alertmanager-2:9094'
      - '--cluster.peer=alertmanager-1:9094'
      - '--log.level=info'
    networks:
      - monitoring
    depends_on:
      - alertmanager-1

  alertmanager-3:
    image: prom/alertmanager:v0.26.0
    container_name: alertmanager-3
    hostname: alertmanager-3
    restart: unless-stopped
    ports:
      - "9097:9093"
      - "9098:9094"
    volumes:
      - ./alertmanager/config:/etc/alertmanager
      - alertmanager3_data:/alertmanager
    command:
      - '--config.file=/etc/alertmanager/alertmanager.yml'
      - '--storage.path=/alertmanager'
      - '--web.external-url=http://localhost:9097'
      - '--cluster.listen-address=0.0.0.0:9094'
      - '--cluster.advertise-address=alertmanager-3:9094'
      - '--cluster.peer=alertmanager-1:9094'
      - '--cluster.peer=alertmanager-2:9094'
      - '--log.level=info'
    networks:
      - monitoring
    depends_on:
      - alertmanager-1
      - alertmanager-2

  # HAProxy para load balancing
  haproxy:
    image: haproxy:2.8-alpine
    container_name: alertmanager-lb
    ports:
      - "9090:9090"
    volumes:
      - ./haproxy/haproxy.cfg:/usr/local/etc/haproxy/haproxy.cfg:ro
    networks:
      - monitoring
    depends_on:
      - alertmanager-1
      - alertmanager-2
      - alertmanager-3

volumes:
  alertmanager1_data:
  alertmanager2_data:
  alertmanager3_data:

networks:
  monitoring:
    external: true
```

### 2. Configuración de HAProxy

```cfg
# haproxy/haproxy.cfg
global
    daemon
    log stdout local0 info

defaults
    mode http
    timeout connect 5000ms
    timeout client 50000ms
    timeout server 50000ms
    option httplog
    log global

frontend alertmanager_frontend
    bind *:9090
    default_backend alertmanager_backend

backend alertmanager_backend
    balance roundrobin
    option httpchk GET /api/v1/status
    http-check expect status 200
    
    server alertmanager-1 alertmanager-1:9093 check
    server alertmanager-2 alertmanager-2:9093 check
    server alertmanager-3 alertmanager-3:9093 check

# Estadísticas
listen stats
    bind *:8080
    stats enable
    stats uri /stats
    stats refresh 30s
    stats admin if TRUE
```

---

## 📋 Monitoreo de Alertmanager

### 1. Métricas de Alertmanager

```yaml
# alertmanager_monitoring.yml - Reglas de Prometheus
groups:
  - name: alertmanager.rules
    rules:
      # Alertmanager no está disponible
      - alert: AlertmanagerDown
        expr: up{job="alertmanager"} == 0
        for: 5m
        labels:
          severity: critical
          service: alertmanager
        annotations:
          summary: "Alertmanager instance is down"
          description: "Alertmanager instance {{ $labels.instance }} has been down for more than 5 minutes."
          runbook_url: "https://runbooks.company.com/alertmanager-down"

      # Alertmanager no puede cargar configuración
      - alert: AlertmanagerConfigInconsistent
        expr: alertmanager_config_hash != on(job,instance) group_left() alertmanager_config_hash{job="alertmanager"}
        for: 20m
        labels:
          severity: critical
          service: alertmanager
        annotations:
          summary: "Alertmanager configuration is inconsistent"
          description: "Alertmanager instances have inconsistent configurations."

      # Fallo en entrega de notificaciones
      - alert: AlertmanagerNotificationsFailed
        expr: rate(alertmanager_notifications_failed_total[5m]) > 0.01
        for: 5m
        labels:
          severity: warning
          service: alertmanager
        annotations:
          summary: "Alertmanager notifications are failing"
          description: "Alertmanager is failing to send notifications."
          
      # Cluster no sincronizado
      - alert: AlertmanagerClusterDown
        expr: |
          alertmanager_cluster_members != on (job) group_left () count by (job) (up{job="alertmanager"} == 1)
        for: 15m
        labels:
          severity: critical
          service: alertmanager
        annotations:
          summary: "Alertmanager cluster is not fully synchronized"
          description: "Alertmanager cluster is not in sync. Some instances may be down."

      # Alto número de alertas
      - alert: AlertmanagerHighAlertVolume
        expr: sum(rate(alertmanager_alerts_received_total[5m])) > 50
        for: 10m
        labels:
          severity: warning
          service: alertmanager
        annotations:
          summary: "High volume of alerts received"
          description: "Alertmanager is receiving more than 50 alerts per second."
```

### 2. Dashboard de Monitoreo

```python
#!/usr/bin/env python3
# alertmanager_metrics.py

import requests
import json
from datetime import datetime
import time

class AlertmanagerMetrics:
    def __init__(self, alertmanager_url="http://localhost:9093"):
        self.alertmanager_url = alertmanager_url

    def get_status(self):
        """Obtiene el estado de Alertmanager"""
        try:
            response = requests.get(f"{self.alertmanager_url}/api/v1/status")
            return response.json()
        except Exception as e:
            return {"error": str(e)}

    def get_alerts_summary(self):
        """Obtiene resumen de alertas"""
        try:
            response = requests.get(f"{self.alertmanager_url}/api/v1/alerts")
            alerts = response.json()['data']
            
            summary = {
                'total': len(alerts),
                'firing': len([a for a in alerts if a['status']['state'] == 'active']),
                'resolved': len([a for a in alerts if a['status']['state'] == 'suppressed']),
                'by_severity': {},
                'by_service': {}
            }
            
            for alert in alerts:
                # Por severidad
                severity = alert['labels'].get('severity', 'unknown')
                summary['by_severity'][severity] = summary['by_severity'].get(severity, 0) + 1
                
                # Por servicio
                service = alert['labels'].get('service', 'unknown')
                summary['by_service'][service] = summary['by_service'].get(service, 0) + 1
            
            return summary
        except Exception as e:
            return {"error": str(e)}

    def get_silences_summary(self):
        """Obtiene resumen de silenciamientos"""
        try:
            response = requests.get(f"{self.alertmanager_url}/api/v1/silences")
            silences = response.json()['data']
            
            summary = {
                'total': len(silences),
                'active': len([s for s in silences if s['status']['state'] == 'active']),
                'pending': len([s for s in silences if s['status']['state'] == 'pending']),
                'expired': len([s for s in silences if s['status']['state'] == 'expired'])
            }
            
            return summary
        except Exception as e:
            return {"error": str(e)}

    def get_receivers_status(self):
        """Obtiene estado de receptores"""
        try:
            # Esta información se obtendría de las métricas de Prometheus
            # Por ahora, retornamos un placeholder
            return {
                'email': {'sent': 0, 'failed': 0},
                'slack': {'sent': 0, 'failed': 0},
                'pagerduty': {'sent': 0, 'failed': 0}
            }
        except Exception as e:
            return {"error": str(e)}

    def generate_health_report(self):
        """Genera reporte de salud"""
        status = self.get_status()
        alerts = self.get_alerts_summary()
        silences = self.get_silences_summary()
        receivers = self.get_receivers_status()
        
        report = f"""
Alertmanager Health Report
Generated: {datetime.now().strftime('%Y-%m-%d %H:%M:%S')}

=== STATUS ===
Uptime: {status.get('uptime', 'Unknown')}
Version: {status.get('versionInfo', {}).get('version', 'Unknown')}

=== ALERTS ===
Total: {alerts['total']}
Firing: {alerts['firing']}
Resolved: {alerts['resolved']}

By Severity:
{chr(10).join([f"  {k}: {v}" for k, v in alerts.get('by_severity', {}).items()])}

By Service:
{chr(10).join([f"  {k}: {v}" for k, v in alerts.get('by_service', {}).items()])}

=== SILENCES ===
Total: {silences['total']}
Active: {silences['active']}
Pending: {silences['pending']}
Expired: {silences['expired']}

=== RECEIVERS STATUS ===
{chr(10).join([f"{k}: Sent={v['sent']}, Failed={v['failed']}" for k, v in receivers.items()])}
"""
        return report

if __name__ == "__main__":
    metrics = AlertmanagerMetrics()
    print(metrics.generate_health_report())
```

---

## 📋 Ejercicios Prácticos

### Ejercicio 1: Configuración Básica de Alertmanager

**Objetivo**: Configurar Alertmanager con múltiples receptores.

**Tareas**:

1. Configurar Alertmanager con Docker Compose
2. Crear configuración para email y Slack
3. Configurar routing básico por severidad
4. Probar envío de alertas de prueba

### Ejercicio 2: Implementar Escalación de Alertas

**Objetivo**: Crear sistema de escalación automática.

**Componentes**:

- Configuración de escalación por niveles
- Script de escalación automática
- Integración con PagerDuty
- Monitoreo de escalaciones

### Ejercicio 3: Alta Disponibilidad

**Objetivo**: Configurar cluster de Alertmanager.

**Tareas**:

- Configurar clustering con 3 nodos
- Implementar load balancer
- Probar failover
- Monitorear sincronización

---

## 🧪 Laboratorio

### Lab 1: Configuración Completa de Alertmanager

1. **Desplegar el stack completo**:

   ```bash
   docker-compose -f docker-compose.alertmanager.yml up -d
   ```

2. **Configurar integración con Prometheus**:
   - Actualizar prometheus.yml para usar Alertmanager
   - Crear reglas de alerta básicas
   - Probar envío de alertas

3. **Configurar receptores**:
   - Email con plantillas HTML
   - Slack con formatting avanzado
   - Webhook personalizado

### Lab 2: Gestión de Silenciamientos

1. **Crear silenciamientos programáticos**
2. **Implementar inhibiciones avanzadas**
3. **Script de gestión de silenciamientos**
4. **Automatización de silenciamientos por mantenimiento**

---

## 📖 Best Practices

### 1. Configuración de Routing

```yaml
routing_best_practices:
  structure:
    - "Usar jerarquía de rutas clara"
    - "Agrupar por criticidad primero"
    - "Luego por equipo o servicio"
    - "Configurar catch-all al final"
  
  timing:
    - "group_wait: 30s para alertas normales"
    - "group_wait: 10s para críticas"
    - "repeat_interval: 4h para evitar spam"
    - "group_interval: 5m para agrupación"
  
  grouping:
    - "Agrupar por ['alertname', 'cluster', 'service']"
    - "Evitar agrupación excesiva"
    - "Considerar contexto de negocio"
```

### 2. Gestión de Plantillas

```yaml
template_best_practices:
  organization:
    - "Separar plantillas por receptor"
    - "Usar funciones helper reutilizables"
    - "Mantener consistencia en formato"
  
  content:
    - "Incluir información contextual"
    - "Proporcionar enlaces a runbooks"
    - "Mostrar duración de alertas"
    - "Incluir acciones sugeridas"
  
  formatting:
    - "Usar HTML para emails ricos"
    - "Markdown para Slack/Teams"
    - "Colores según severidad"
    - "Emojis para reconocimiento rápido"
```

---

## ✅ Resumen del Módulo

En este módulo has aprendido:

1. **Instalación y configuración** de Alertmanager
2. **Routing avanzado** de alertas con criterios complejos
3. **Integración de múltiples receptores** (Email, Slack, PagerDuty, Teams)
4. **Configuración de agrupación** y deduplicación inteligente
5. **Implementación de silenciamientos** e inhibiciones
6. **Sistema de escalación** automática de alertas
7. **Clustering y alta disponibilidad** para redundancia
8. **Plantillas personalizadas** para notificaciones ricas
9. **Monitoreo del propio Alertmanager**

### Próximas etapas

- **Módulo 06**: Exploraremos distributed tracing con Jaeger y Zipkin
- **Módulo 07**: Implementaremos Application Performance Monitoring (APM)
- **Módulo 08**: Configuraremos sistemas de logging centralizados avanzados

¡Continúa con el siguiente módulo para dominar el distributed tracing!
