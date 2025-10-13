# Helm: Gestión de Charts para Kubernetes

## 🎯 Objetivos de Aprendizaje

Al finalizar este tema serás capaz de:

- Instalar y configurar Helm en tu entorno
- Crear y gestionar Charts de Helm personalizados
- Implementar templates dinámicos con valores parametrizados
- Gestionar releases y versionado de aplicaciones
- Usar repositorios de Charts públicos y privados

## 📚 ¿Qué es Helm?

### Definición

**Helm** es el gestor de paquetes para Kubernetes, conocido como "el apt/yum de Kubernetes". Permite:

- **Empaquetar** aplicaciones como Charts
- **Distribuir** aplicaciones de forma consistente
- **Versionar** deployments y rollbacks
- **Parametrizar** configuraciones
- **Gestionar** dependencias entre aplicaciones

### Conceptos Clave

1. **Chart**: Paquete de archivos YAML que describe una aplicación
2. **Release**: Instancia de un Chart desplegado en el cluster
3. **Repository**: Colección de Charts disponibles
4. **Values**: Parámetros de configuración para personalizar Charts

## 🛠️ Instalación de Helm

### Instalación en Windows

```powershell
# Usando Chocolatey
choco install kubernetes-helm

# Usando Scoop
scoop install helm

# Descarga directa
# Ir a https://github.com/helm/helm/releases
# Descargar el binario y añadir al PATH
```

### Instalación en Linux/macOS

```bash
# Script oficial
curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash

# Usando Homebrew (macOS)
brew install helm

# Usando Snap (Linux)
sudo snap install helm --classic
```

### Verificar Instalación

```bash
# Versión de Helm
helm version

# Agregar repositorio oficial
helm repo add stable https://charts.helm.sh/stable
helm repo add bitnami https://charts.bitnami.com/bitnami

# Actualizar repositorios
helm repo update

# Listar repositorios
helm repo list
```

## 📦 Estructura de un Chart

### Crear Chart Básico

```bash
# Crear nuevo chart
helm create mi-webapp

# Estructura generada
mi-webapp/
├── Chart.yaml          # Metadatos del chart
├── values.yaml         # Valores por defecto
├── charts/             # Dependencias del chart
├── templates/          # Templates de Kubernetes
│   ├── deployment.yaml
│   ├── service.yaml
│   ├── ingress.yaml
│   ├── _helpers.tpl    # Template helpers
│   ├── NOTES.txt       # Notas post-instalación
│   └── tests/          # Tests del chart
└── .helmignore         # Archivos a ignorar
```

### Chart.yaml

```yaml
# Chart.yaml
apiVersion: v2
name: mi-webapp
description: Una aplicación web de ejemplo
type: application
version: 0.1.0
appVersion: "1.16.0"
home: https://github.com/mi-org/mi-webapp
sources:
  - https://github.com/mi-org/mi-webapp
maintainers:
  - name: Juan Pérez
    email: juan@example.com
    url: https://github.com/juanperez
keywords:
  - web
  - frontend
  - nginx
annotations:
  category: WebServer
dependencies:
  - name: postgresql
    version: "11.6.12"
    repository: https://charts.bitnami.com/bitnami
    condition: postgresql.enabled
  - name: redis
    version: "16.13.2"
    repository: https://charts.bitnami.com/bitnami
    condition: redis.enabled
```

### values.yaml

```yaml
# values.yaml
# Configuración por defecto para el chart

# Configuración de imagen
image:
  repository: nginx
  pullPolicy: IfNotPresent
  tag: "1.21.0"

# Configuración de réplicas
replicaCount: 1

# Configuración del servicio
service:
  type: ClusterIP
  port: 80
  targetPort: 8080

# Configuración de Ingress
ingress:
  enabled: false
  className: nginx
  annotations: {}
    # kubernetes.io/ingress.class: nginx
    # cert-manager.io/cluster-issuer: letsencrypt-prod
  hosts:
    - host: chart-example.local
      paths:
        - path: /
          pathType: ImplementationSpecific
  tls: []
    # - secretName: chart-example-tls
    #   hosts:
    #     - chart-example.local

# Recursos
resources:
  limits:
    cpu: 500m
    memory: 512Mi
  requests:
    cpu: 250m
    memory: 256Mi

# Autoscaling
autoscaling:
  enabled: false
  minReplicas: 1
  maxReplicas: 100
  targetCPUUtilizationPercentage: 80
  targetMemoryUtilizationPercentage: 80

# Security Context
securityContext:
  runAsNonRoot: true
  runAsUser: 1000
  fsGroup: 2000

# Pod Security Context
podSecurityContext:
  capabilities:
    drop:
    - ALL
  readOnlyRootFilesystem: true
  runAsNonRoot: true
  runAsUser: 1000

# Configuración de base de datos
postgresql:
  enabled: true
  auth:
    postgresPassword: "secretpassword"
    database: "myapp"
  primary:
    persistence:
      enabled: true
      size: 8Gi

# Configuración de cache
redis:
  enabled: false
  auth:
    enabled: false

# Variables de entorno
env:
  - name: APP_ENV
    value: "production"
  - name: LOG_LEVEL
    value: "info"

# ConfigMaps
configMaps:
  app-config:
    data:
      database_url: "postgresql://user:pass@postgres:5432/myapp"
      redis_url: "redis://redis:6379"

# Secrets
secrets:
  app-secrets:
    data:
      api_key: "bXlfc2VjcmV0X2FwaV9rZXk="  # base64 encoded

# Volúmenes persistentes
persistence:
  enabled: true
  storageClass: ""
  accessMode: ReadWriteOnce
  size: 10Gi

# Configuración de nodos
nodeSelector: {}
tolerations: []
affinity: {}

# Service Account
serviceAccount:
  create: true
  annotations: {}
  name: ""

# RBAC
rbac:
  create: true

# Network Policies
networkPolicy:
  enabled: false

# Pod Disruption Budget
podDisruptionBudget:
  enabled: false
  minAvailable: 1
```

## 🎨 Templates de Helm

### Deployment Template

```yaml
# templates/deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ include "mi-webapp.fullname" . }}
  labels:
    {{- include "mi-webapp.labels" . | nindent 4 }}
spec:
  {{- if not .Values.autoscaling.enabled }}
  replicas: {{ .Values.replicaCount }}
  {{- end }}
  selector:
    matchLabels:
      {{- include "mi-webapp.selectorLabels" . | nindent 6 }}
  template:
    metadata:
      annotations:
        checksum/config: {{ include (print $.Template.BasePath "/configmap.yaml") . | sha256sum }}
        {{- with .Values.podAnnotations }}
        {{- toYaml . | nindent 8 }}
        {{- end }}
      labels:
        {{- include "mi-webapp.selectorLabels" . | nindent 8 }}
    spec:
      {{- with .Values.imagePullSecrets }}
      imagePullSecrets:
        {{- toYaml . | nindent 8 }}
      {{- end }}
      serviceAccountName: {{ include "mi-webapp.serviceAccountName" . }}
      securityContext:
        {{- toYaml .Values.podSecurityContext | nindent 8 }}
      containers:
        - name: {{ .Chart.Name }}
          securityContext:
            {{- toYaml .Values.securityContext | nindent 12 }}
          image: "{{ .Values.image.repository }}:{{ .Values.image.tag | default .Chart.AppVersion }}"
          imagePullPolicy: {{ .Values.image.pullPolicy }}
          ports:
            - name: http
              containerPort: {{ .Values.service.targetPort }}
              protocol: TCP
          env:
            {{- range .Values.env }}
            - name: {{ .name }}
              value: {{ .value | quote }}
            {{- end }}
            {{- if .Values.postgresql.enabled }}
            - name: DB_HOST
              value: {{ include "mi-webapp.postgresql.fullname" . }}
            - name: DB_PASSWORD
              valueFrom:
                secretKeyRef:
                  name: {{ include "mi-webapp.postgresql.secretName" . }}
                  key: postgres-password
            {{- end }}
          livenessProbe:
            httpGet:
              path: /health
              port: http
            initialDelaySeconds: 30
            periodSeconds: 10
          readinessProbe:
            httpGet:
              path: /ready
              port: http
            initialDelaySeconds: 5
            periodSeconds: 5
          resources:
            {{- toYaml .Values.resources | nindent 12 }}
          {{- if .Values.persistence.enabled }}
          volumeMounts:
            - name: data
              mountPath: /app/data
          {{- end }}
      {{- if .Values.persistence.enabled }}
      volumes:
        - name: data
          persistentVolumeClaim:
            claimName: {{ include "mi-webapp.fullname" . }}
      {{- end }}
      {{- with .Values.nodeSelector }}
      nodeSelector:
        {{- toYaml . | nindent 8 }}
      {{- end }}
      {{- with .Values.affinity }}
      affinity:
        {{- toYaml . | nindent 8 }}
      {{- end }}
      {{- with .Values.tolerations }}
      tolerations:
        {{- toYaml . | nindent 8 }}
      {{- end }}
```

### Service Template

```yaml
# templates/service.yaml
apiVersion: v1
kind: Service
metadata:
  name: {{ include "mi-webapp.fullname" . }}
  labels:
    {{- include "mi-webapp.labels" . | nindent 4 }}
spec:
  type: {{ .Values.service.type }}
  ports:
    - port: {{ .Values.service.port }}
      targetPort: http
      protocol: TCP
      name: http
      {{- if and (eq .Values.service.type "NodePort") .Values.service.nodePort }}
      nodePort: {{ .Values.service.nodePort }}
      {{- end }}
  selector:
    {{- include "mi-webapp.selectorLabels" . | nindent 4 }}
  {{- if eq .Values.service.type "LoadBalancer" }}
  {{- with .Values.service.loadBalancerIP }}
  loadBalancerIP: {{ . }}
  {{- end }}
  {{- with .Values.service.loadBalancerSourceRanges }}
  loadBalancerSourceRanges:
    {{- range . }}
    - {{ . }}
    {{- end }}
  {{- end }}
  {{- end }}
```

### Ingress Template

```yaml
# templates/ingress.yaml
{{- if .Values.ingress.enabled -}}
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: {{ include "mi-webapp.fullname" . }}
  labels:
    {{- include "mi-webapp.labels" . | nindent 4 }}
  {{- with .Values.ingress.annotations }}
  annotations:
    {{- toYaml . | nindent 4 }}
  {{- end }}
spec:
  {{- if .Values.ingress.className }}
  ingressClassName: {{ .Values.ingress.className }}
  {{- end }}
  {{- if .Values.ingress.tls }}
  tls:
    {{- range .Values.ingress.tls }}
    - hosts:
        {{- range .hosts }}
        - {{ . | quote }}
        {{- end }}
      secretName: {{ .secretName }}
    {{- end }}
  {{- end }}
  rules:
    {{- range .Values.ingress.hosts }}
    - host: {{ .host | quote }}
      http:
        paths:
          {{- range .paths }}
          - path: {{ .path }}
            pathType: {{ .pathType }}
            backend:
              service:
                name: {{ include "mi-webapp.fullname" $ }}
                port:
                  number: {{ $.Values.service.port }}
          {{- end }}
    {{- end }}
{{- end }}
```

### ConfigMap Template

```yaml
# templates/configmap.yaml
{{- range $name, $config := .Values.configMaps }}
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ include "mi-webapp.fullname" $ }}-{{ $name }}
  labels:
    {{- include "mi-webapp.labels" $ | nindent 4 }}
data:
  {{- range $key, $value := $config.data }}
  {{ $key }}: {{ $value | quote }}
  {{- end }}
---
{{- end }}
```

### Secret Template

```yaml
# templates/secret.yaml
{{- range $name, $secret := .Values.secrets }}
apiVersion: v1
kind: Secret
metadata:
  name: {{ include "mi-webapp.fullname" $ }}-{{ $name }}
  labels:
    {{- include "mi-webapp.labels" $ | nindent 4 }}
type: Opaque
data:
  {{- range $key, $value := $secret.data }}
  {{ $key }}: {{ $value }}
  {{- end }}
---
{{- end }}
```

### HPA Template

```yaml
# templates/hpa.yaml
{{- if .Values.autoscaling.enabled }}
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: {{ include "mi-webapp.fullname" . }}
  labels:
    {{- include "mi-webapp.labels" . | nindent 4 }}
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: {{ include "mi-webapp.fullname" . }}
  minReplicas: {{ .Values.autoscaling.minReplicas }}
  maxReplicas: {{ .Values.autoscaling.maxReplicas }}
  metrics:
    {{- if .Values.autoscaling.targetCPUUtilizationPercentage }}
    - type: Resource
      resource:
        name: cpu
        target:
          type: Utilization
          averageUtilization: {{ .Values.autoscaling.targetCPUUtilizationPercentage }}
    {{- end }}
    {{- if .Values.autoscaling.targetMemoryUtilizationPercentage }}
    - type: Resource
      resource:
        name: memory
        target:
          type: Utilization
          averageUtilization: {{ .Values.autoscaling.targetMemoryUtilizationPercentage }}
    {{- end }}
{{- end }}
```

### Template Helpers

```yaml
# templates/_helpers.tpl
{{/*
Nombre completo expandido
*/}}
{{- define "mi-webapp.fullname" -}}
{{- if .Values.fullnameOverride }}
{{- .Values.fullnameOverride | trunc 63 | trimSuffix "-" }}
{{- else }}
{{- $name := default .Chart.Name .Values.nameOverride }}
{{- if contains $name .Release.Name }}
{{- .Release.Name | trunc 63 | trimSuffix "-" }}
{{- else }}
{{- printf "%s-%s" .Release.Name $name | trunc 63 | trimSuffix "-" }}
{{- end }}
{{- end }}
{{- end }}

{{/*
Nombre del chart
*/}}
{{- define "mi-webapp.chart" -}}
{{- printf "%s-%s" .Chart.Name .Chart.Version | replace "+" "_" | trunc 63 | trimSuffix "-" }}
{{- end }}

{{/*
Labels comunes
*/}}
{{- define "mi-webapp.labels" -}}
helm.sh/chart: {{ include "mi-webapp.chart" . }}
{{ include "mi-webapp.selectorLabels" . }}
{{- if .Chart.AppVersion }}
app.kubernetes.io/version: {{ .Chart.AppVersion | quote }}
{{- end }}
app.kubernetes.io/managed-by: {{ .Release.Service }}
{{- end }}

{{/*
Labels de selección
*/}}
{{- define "mi-webapp.selectorLabels" -}}
app.kubernetes.io/name: {{ include "mi-webapp.name" . }}
app.kubernetes.io/instance: {{ .Release.Name }}
{{- end }}

{{/*
Nombre del ServiceAccount
*/}}
{{- define "mi-webapp.serviceAccountName" -}}
{{- if .Values.serviceAccount.create }}
{{- default (include "mi-webapp.fullname" .) .Values.serviceAccount.name }}
{{- else }}
{{- default "default" .Values.serviceAccount.name }}
{{- end }}
{{- end }}

{{/*
Nombre de PostgreSQL
*/}}
{{- define "mi-webapp.postgresql.fullname" -}}
{{- printf "%s-%s" .Release.Name "postgresql" | trunc 63 | trimSuffix "-" }}
{{- end }}

{{/*
Nombre del Secret de PostgreSQL
*/}}
{{- define "mi-webapp.postgresql.secretName" -}}
{{- if .Values.postgresql.auth.existingSecret }}
{{- .Values.postgresql.auth.existingSecret }}
{{- else }}
{{- include "mi-webapp.postgresql.fullname" . }}
{{- end }}
{{- end }}
```

## 🚀 Comandos de Helm

### Gestión de Charts

```bash
# Crear chart
helm create mi-app

# Validar sintaxis
helm lint mi-app/

# Renderizar templates (debug)
helm template mi-app mi-app/ --values mi-app/values.yaml

# Renderizar con valores personalizados
helm template mi-app mi-app/ --set image.tag=2.0.0

# Empaquetar chart
helm package mi-app/

# Verificar chart empaquetado
helm verify mi-app-0.1.0.tgz
```

### Instalación y Gestión de Releases

```bash
# Instalar chart
helm install mi-release mi-app/

# Instalar con valores personalizados
helm install mi-release mi-app/ \
  --set replicaCount=3 \
  --set image.tag=v2.0.0 \
  --set ingress.enabled=true

# Instalar con archivo de valores
helm install mi-release mi-app/ -f valores-produccion.yaml

# Instalar en namespace específico
helm install mi-release mi-app/ -n production --create-namespace

# Upgrade de release
helm upgrade mi-release mi-app/ --set image.tag=v2.1.0

# Rollback a versión anterior
helm rollback mi-release 1

# Ver releases
helm list
helm list --all-namespaces

# Ver historial
helm history mi-release

# Obtener valores de release
helm get values mi-release

# Ver manifiestos instalados
helm get manifest mi-release

# Desinstalar release
helm uninstall mi-release
```

### Repositorios

```bash
# Añadir repositorio
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts

# Actualizar repositorios
helm repo update

# Buscar charts
helm search repo nginx
helm search repo prometheus

# Ver información de chart
helm show chart bitnami/nginx
helm show values bitnami/nginx

# Instalar desde repositorio
helm install mi-nginx bitnami/nginx
```

## 📁 Repositorio de Charts

### Crear Repositorio Local

```bash
# Crear estructura de repositorio
mkdir mi-repo
cd mi-repo

# Copiar charts empaquetados
cp ../mi-app-0.1.0.tgz .
cp ../otro-chart-1.0.0.tgz .

# Generar índice
helm repo index . --url http://mi-servidor.com/charts

# Contenido de index.yaml generado
cat index.yaml
```

### Repositorio con GitHub Pages

```yaml
# .github/workflows/helm-release.yml
name: Release Charts

on:
  push:
    branches:
      - main

jobs:
  release:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v3
        with:
          fetch-depth: 0

      - name: Configure Git
        run: |
          git config user.name "$GITHUB_ACTOR"
          git config user.email "$GITHUB_ACTOR@users.noreply.github.com"

      - name: Install Helm
        uses: azure/setup-helm@v3
        with:
          version: v3.10.0

      - name: Run chart-releaser
        uses: helm/chart-releaser-action@v1.4.0
        env:
          CR_TOKEN: "${{ secrets.GITHUB_TOKEN }}"
```

### Harbor Registry

```bash
# Configurar Harbor como registry OCI
helm registry login https://harbor.example.com

# Empaquetar y subir chart
helm package mi-app/
helm push mi-app-0.1.0.tgz oci://harbor.example.com/library

# Instalar desde Harbor
helm install mi-release oci://harbor.example.com/library/mi-app --version 0.1.0
```

## 🔧 Valores y Configuración

### Archivos de Valores por Entorno

```yaml
# values-development.yaml
image:
  tag: "latest"
  pullPolicy: Always

replicaCount: 1

resources:
  requests:
    cpu: 100m
    memory: 128Mi

postgresql:
  enabled: false

externalDatabase:
  host: "localhost"
  port: 5432

ingress:
  enabled: false

autoscaling:
  enabled: false
```

```yaml
# values-production.yaml
image:
  tag: "1.2.3"
  pullPolicy: IfNotPresent

replicaCount: 3

resources:
  limits:
    cpu: 1000m
    memory: 1Gi
  requests:
    cpu: 500m
    memory: 512Mi

postgresql:
  enabled: true
  auth:
    postgresPassword: "super-secure-password"
  primary:
    persistence:
      enabled: true
      size: 20Gi

ingress:
  enabled: true
  className: nginx
  annotations:
    cert-manager.io/cluster-issuer: letsencrypt-prod
    nginx.ingress.kubernetes.io/rate-limit: "100"
  hosts:
    - host: myapp.example.com
      paths:
        - path: /
          pathType: ImplementationSpecific
  tls:
    - secretName: myapp-tls
      hosts:
        - myapp.example.com

autoscaling:
  enabled: true
  minReplicas: 3
  maxReplicas: 20
  targetCPUUtilizationPercentage: 70

podDisruptionBudget:
  enabled: true
  minAvailable: 2
```

### Uso de Secrets en Charts

```yaml
# values-secrets.yaml
secrets:
  database:
    host: "cG9zdGdyZXNxbC5kYXRhYmFzZS5henVyZS5jb20="  # base64
    password: "c3VwZXItc2VjdXJlLXBhc3N3b3Jk"               # base64
  api:
    key: "YWJjZGVmZ2hpams="                              # base64
    secret: "bXlfc2VjcmV0X2FwaV9rZXk="                    # base64
```

### Template con Condicionales Avanzadas

```yaml
# templates/database-config.yaml
{{- if .Values.postgresql.enabled }}
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ include "mi-webapp.fullname" . }}-db-config
data:
  database_url: "postgresql://{{ .Values.postgresql.auth.username }}@{{ include "mi-webapp.postgresql.fullname" . }}:5432/{{ .Values.postgresql.auth.database }}"
{{- else if .Values.externalDatabase.enabled }}
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ include "mi-webapp.fullname" . }}-db-config
data:
  database_url: "postgresql://{{ .Values.externalDatabase.username }}@{{ .Values.externalDatabase.host }}:{{ .Values.externalDatabase.port }}/{{ .Values.externalDatabase.database }}"
{{- end }}
```

## 🧪 Testing de Charts

### Tests de Chart

```yaml
# templates/tests/test-connection.yaml
apiVersion: v1
kind: Pod
metadata:
  name: "{{ include "mi-webapp.fullname" . }}-test"
  labels:
    {{- include "mi-webapp.labels" . | nindent 4 }}
  annotations:
    "helm.sh/hook": test
    "helm.sh/hook-weight": "1"
    "helm.sh/hook-delete-policy": before-hook-creation,hook-succeeded
spec:
  restartPolicy: Never
  containers:
    - name: wget
      image: busybox
      command: ['wget']
      args: ['{{ include "mi-webapp.fullname" . }}:{{ .Values.service.port }}']
    - name: curl-test
      image: curlimages/curl
      command: ['curl']
      args: ['-f', 'http://{{ include "mi-webapp.fullname" . }}:{{ .Values.service.port }}/health']
```

### Ejecutar Tests

```bash
# Instalar chart con tests
helm install mi-release mi-app/

# Ejecutar tests
helm test mi-release

# Ver logs de tests
helm test mi-release --logs

# Test con timeout
helm test mi-release --timeout 300s
```

### Chart Testing con Docker

```dockerfile
# Dockerfile.test
FROM alpine/helm:latest

WORKDIR /charts

# Copiar chart
COPY . .

# Instalar herramientas de testing
RUN apk add --no-cache \
    bash \
    curl \
    jq \
    yamllint

# Script de testing
COPY test-script.sh /usr/local/bin/
RUN chmod +x /usr/local/bin/test-script.sh

ENTRYPOINT ["/usr/local/bin/test-script.sh"]
```

```bash
#!/bin/bash
# test-script.sh

# Validar sintaxis YAML
yamllint charts/

# Lint de Helm
helm lint charts/mi-app/

# Template rendering test
helm template test-release charts/mi-app/ \
  --values charts/mi-app/values.yaml \
  --output-dir /tmp/rendered

# Validar que los manifiestos se generan correctamente
if [ ! -f "/tmp/rendered/mi-app/templates/deployment.yaml" ]; then
  echo "Error: Deployment template not generated"
  exit 1
fi

echo "All tests passed!"
```

## 🏗️ Ejercicio Práctico: Stack LAMP con Helm

### 1. Crear Chart LAMP

```bash
helm create lamp-stack
cd lamp-stack
```

### 2. Chart.yaml

```yaml
# Chart.yaml
apiVersion: v2
name: lamp-stack
description: A complete LAMP stack deployment
type: application
version: 0.1.0
appVersion: "1.0"
dependencies:
  - name: mysql
    version: "9.4.6"
    repository: https://charts.bitnami.com/bitnami
  - name: phpmyadmin
    version: "10.3.4"
    repository: https://charts.bitnami.com/bitnami
    condition: phpmyadmin.enabled
```

### 3. Values LAMP

```yaml
# values.yaml
# Apache + PHP Configuration
apache:
  image:
    repository: php
    tag: "8.1-apache"
  replicaCount: 2
  service:
    type: ClusterIP
    port: 80
  
  # PHP Configuration
  phpConfig: |
    upload_max_filesize = 64M
    post_max_size = 64M
    max_execution_time = 300
    memory_limit = 256M
  
  # Virtual Host Configuration
  vhostConfig: |
    <VirtualHost *:80>
        DocumentRoot /var/www/html
        ErrorLog ${APACHE_LOG_DIR}/error.log
        CustomLog ${APACHE_LOG_DIR}/access.log combined
        
        <Directory /var/www/html>
            AllowOverride All
            Require all granted
        </Directory>
    </VirtualHost>

# MySQL Configuration (Bitnami dependency)
mysql:
  auth:
    rootPassword: "lamp-root-password"
    database: "lampdb"
    username: "lampuser"
    password: "lamp-password"
  primary:
    persistence:
      enabled: true
      size: 8Gi

# PHPMyAdmin Configuration (Bitnami dependency)
phpmyadmin:
  enabled: true
  db:
    host: lamp-stack-mysql
    port: 3306
    allowArbitraryServer: false

# Ingress Configuration
ingress:
  enabled: true
  className: nginx
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
  hosts:
    - host: lamp.local
      paths:
        - path: /
          pathType: Prefix
          service: apache
        - path: /phpmyadmin
          pathType: Prefix
          service: phpmyadmin

# Application Code ConfigMap
applicationCode:
  enabled: true
  files:
    index.php: |
      <?php
      $servername = "lamp-stack-mysql";
      $username = "lampuser";
      $password = "lamp-password";
      $dbname = "lampdb";
      
      try {
          $pdo = new PDO("mysql:host=$servername;dbname=$dbname", $username, $password);
          $pdo->setAttribute(PDO::ATTR_ERRMODE, PDO::ERRMODE_EXCEPTION);
          echo "<h1>LAMP Stack funcionando!</h1>";
          echo "<p>Conexión a base de datos: ✅ Exitosa</p>";
          echo "<p>Servidor: " . gethostname() . "</p>";
          echo "<p>Fecha: " . date('Y-m-d H:i:s') . "</p>";
          
          // Crear tabla de ejemplo
          $sql = "CREATE TABLE IF NOT EXISTS users (
              id INT(6) UNSIGNED AUTO_INCREMENT PRIMARY KEY,
              firstname VARCHAR(30) NOT NULL,
              lastname VARCHAR(30) NOT NULL,
              email VARCHAR(50),
              reg_date TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
          )";
          $pdo->exec($sql);
          
          echo "<p>Tabla 'users' creada/verificada correctamente</p>";
          echo "<a href='/phpmyadmin'>Acceder a PHPMyAdmin</a>";
          
      } catch(PDOException $e) {
          echo "<h1>Error de conexión a base de datos</h1>";
          echo "<p>Error: " . $e->getMessage() . "</p>";
      }
      ?>
    
    info.php: |
      <?php
      phpinfo();
      ?>
```

### 4. Templates LAMP

```yaml
# templates/apache-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ include "lamp-stack.fullname" . }}-apache
  labels:
    {{- include "lamp-stack.labels" . | nindent 4 }}
    component: apache
spec:
  replicas: {{ .Values.apache.replicaCount }}
  selector:
    matchLabels:
      {{- include "lamp-stack.selectorLabels" . | nindent 6 }}
      component: apache
  template:
    metadata:
      labels:
        {{- include "lamp-stack.selectorLabels" . | nindent 8 }}
        component: apache
    spec:
      containers:
        - name: apache-php
          image: "{{ .Values.apache.image.repository }}:{{ .Values.apache.image.tag }}"
          ports:
            - name: http
              containerPort: 80
              protocol: TCP
          env:
            - name: MYSQL_HOST
              value: "{{ include "lamp-stack.fullname" . }}-mysql"
            - name: MYSQL_DATABASE
              value: {{ .Values.mysql.auth.database }}
            - name: MYSQL_USER
              value: {{ .Values.mysql.auth.username }}
            - name: MYSQL_PASSWORD
              valueFrom:
                secretKeyRef:
                  name: {{ include "lamp-stack.fullname" . }}-mysql
                  key: mysql-password
          volumeMounts:
            - name: php-config
              mountPath: /usr/local/etc/php/conf.d/custom.ini
              subPath: custom.ini
            - name: apache-config
              mountPath: /etc/apache2/sites-available/000-default.conf
              subPath: 000-default.conf
            {{- if .Values.applicationCode.enabled }}
            - name: app-code
              mountPath: /var/www/html
            {{- end }}
          livenessProbe:
            httpGet:
              path: /
              port: http
            initialDelaySeconds: 30
            periodSeconds: 10
          readinessProbe:
            httpGet:
              path: /
              port: http
            initialDelaySeconds: 5
            periodSeconds: 5
      volumes:
        - name: php-config
          configMap:
            name: {{ include "lamp-stack.fullname" . }}-php-config
        - name: apache-config
          configMap:
            name: {{ include "lamp-stack.fullname" . }}-apache-config
        {{- if .Values.applicationCode.enabled }}
        - name: app-code
          configMap:
            name: {{ include "lamp-stack.fullname" . }}-app-code
        {{- end }}
```

```yaml
# templates/configmaps.yaml
{{- if .Values.applicationCode.enabled }}
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ include "lamp-stack.fullname" . }}-app-code
  labels:
    {{- include "lamp-stack.labels" . | nindent 4 }}
data:
  {{- range $filename, $content := .Values.applicationCode.files }}
  {{ $filename }}: |
{{ $content | indent 4 }}
  {{- end }}
---
{{- end }}
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ include "lamp-stack.fullname" . }}-php-config
  labels:
    {{- include "lamp-stack.labels" . | nindent 4 }}
data:
  custom.ini: |
{{ .Values.apache.phpConfig | indent 4 }}
---
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ include "lamp-stack.fullname" . }}-apache-config
  labels:
    {{- include "lamp-stack.labels" . | nindent 4 }}
data:
  000-default.conf: |
{{ .Values.apache.vhostConfig | indent 4 }}
```

### 5. Desplegar Stack LAMP

```bash
# Actualizar dependencias
helm dependency update

# Instalar stack
helm install mi-lamp . \
  --set mysql.auth.rootPassword=supersecret \
  --set ingress.hosts[0].host=lamp.local

# Verificar instalación
kubectl get pods
kubectl get services
kubectl get ingress

# Agregar entrada a /etc/hosts (Windows: C:\Windows\System32\drivers\etc\hosts)
echo "127.0.0.1 lamp.local" >> /etc/hosts

# Acceder a la aplicación
curl http://lamp.local
```

## 📊 Hooks de Helm

### Pre-install Hook

```yaml
# templates/pre-install-job.yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: {{ include "mi-webapp.fullname" . }}-pre-install
  labels:
    {{- include "mi-webapp.labels" . | nindent 4 }}
  annotations:
    "helm.sh/hook": pre-install
    "helm.sh/hook-weight": "-5"
    "helm.sh/hook-delete-policy": hook-succeeded
spec:
  template:
    spec:
      restartPolicy: Never
      containers:
      - name: pre-install
        image: busybox
        command:
        - /bin/sh
        - -c
        - |
          echo "Preparando instalación..."
          echo "Verificando prerrequisitos..."
          sleep 10
          echo "Prerrequisitos verificados ✓"
```

### Post-install Hook

```yaml
# templates/post-install-job.yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: {{ include "mi-webapp.fullname" . }}-post-install
  labels:
    {{- include "mi-webapp.labels" . | nindent 4 }}
  annotations:
    "helm.sh/hook": post-install
    "helm.sh/hook-weight": "1"
    "helm.sh/hook-delete-policy": hook-succeeded
spec:
  template:
    spec:
      restartPolicy: Never
      containers:
      - name: post-install
        image: curlimages/curl
        command:
        - /bin/sh
        - -c
        - |
          echo "Verificando que la aplicación esté funcionando..."
          until curl -f http://{{ include "mi-webapp.fullname" . }}:{{ .Values.service.port }}/health; do
            echo "Esperando que la aplicación esté lista..."
            sleep 5
          done
          echo "Aplicación desplegada correctamente ✓"
```

### Pre-upgrade Hook

```yaml
# templates/pre-upgrade-backup.yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: {{ include "mi-webapp.fullname" . }}-backup-{{ .Release.Revision }}
  labels:
    {{- include "mi-webapp.labels" . | nindent 4 }}
  annotations:
    "helm.sh/hook": pre-upgrade
    "helm.sh/hook-weight": "-10"
spec:
  template:
    spec:
      restartPolicy: Never
      containers:
      - name: backup
        image: postgres:13
        env:
        - name: PGPASSWORD
          valueFrom:
            secretKeyRef:
              name: {{ include "mi-webapp.postgresql.secretName" . }}
              key: postgres-password
        command:
        - /bin/bash
        - -c
        - |
          echo "Creando backup antes de actualización..."
          pg_dump -h {{ include "mi-webapp.postgresql.fullname" . }} \
                  -U {{ .Values.postgresql.auth.username }} \
                  -d {{ .Values.postgresql.auth.database }} \
                  > /backup/backup-$(date +%Y%m%d-%H%M%S).sql
          echo "Backup completado ✓"
        volumeMounts:
        - name: backup-storage
          mountPath: /backup
      volumes:
      - name: backup-storage
        persistentVolumeClaim:
          claimName: backup-pvc
```

## 🎯 Mejores Prácticas

### Estructura de Chart

1. **Nombres consistentes** en templates
2. **Valores por defecto** seguros
3. **Documentación completa** en NOTES.txt
4. **Tests automatizados** para validar funcionalidad
5. **Versionado semántico** del chart

### Seguridad

1. **No incluir secrets** en values.yaml
2. **Usar security contexts** apropiados
3. **Validar inputs** con esquemas JSON
4. **Implementar RBAC** cuando sea necesario
5. **Usar imágenes firmadas** y escaneadas

### Performance

1. **Resource limits** apropiados
2. **Health checks** configurados
3. **Strategies de actualización** definidas
4. **PodDisruptionBudgets** para alta disponibilidad
5. **Affinity rules** para distribución

## 🎯 Resumen

Has aprendido:

- [x] Instalación y configuración de Helm
- [x] Estructura y componentes de Charts
- [x] Creación de templates dinámicos
- [x] Gestión de valores y configuraciones
- [x] Implementación de hooks y tests
- [x] Mejores prácticas para Charts de producción

## 📚 Recursos Adicionales

- [Documentación oficial de Helm](https://helm.sh/docs/)
- [Best Practices Guide](https://helm.sh/docs/chart_best_practices/)
- [Chart Repository Guide](https://helm.sh/docs/topics/chart_repository/)
- [Helm Hub](https://artifacthub.io/)
- [Security Best Practices](https://helm.sh/docs/topics/advanced/)

---

**Próximo tema:** [Práctica: Aplicación Multi-contenedor](./10-practica-multicontenedor.md)