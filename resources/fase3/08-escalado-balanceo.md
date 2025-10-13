# Escalado y Balanceo de Carga

## 🎯 Objetivos de Aprendizaje

Al finalizar este tema serás capaz de:
- Implementar escalado horizontal automático (HPA)
- Configurar escalado vertical automático (VPA)
- Gestionar balanceadores de carga para alta disponibilidad
- Configurar Ingress Controllers para tráfico HTTP/HTTPS
- Implementar políticas de disrupción de Pods

## 📚 Escalado en Kubernetes

### Tipos de Escalado

1. **Horizontal Pod Autoscaler (HPA)**: Aumenta/disminuye número de Pods
2. **Vertical Pod Autoscaler (VPA)**: Aumenta/disminuye recursos de Pods
3. **Cluster Autoscaler**: Aumenta/disminuye número de nodos
4. **Escalado manual**: Control directo sobre réplicas

## 📈 Horizontal Pod Autoscaler (HPA)

### ¿Qué es HPA?

El **HPA** escala automáticamente el número de Pods basándose en métricas como CPU, memoria o métricas personalizadas.

```yaml
# hpa-cpu.yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: webapp-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: webapp-deployment
  minReplicas: 2
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70
  - type: Resource
    resource:
      name: memory
      target:
        type: Utilization
        averageUtilization: 80
  behavior:
    scaleUp:
      stabilizationWindowSeconds: 60
      policies:
      - type: Percent
        value: 100
        periodSeconds: 15
    scaleDown:
      stabilizationWindowSeconds: 300
      policies:
      - type: Percent
        value: 10
        periodSeconds: 60
```

### Configurar Metrics Server

```bash
# Instalar Metrics Server (requerido para HPA)
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml

# Verificar funcionamiento
kubectl top nodes
kubectl top pods
```

### HPA con Métricas Personalizadas

```yaml
# hpa-custom-metrics.yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: webapp-custom-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: webapp-deployment
  minReplicas: 3
  maxReplicas: 50
  metrics:
  # CPU utilization
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 60
  
  # Requests per second (requiere adapter de métricas)
  - type: Pods
    pods:
      metric:
        name: http_requests_per_second
      target:
        type: AverageValue
        averageValue: "100"
  
  # Longitud de cola (métrica externa)
  - type: External
    external:
      metric:
        name: queue_length
        selector:
          matchLabels:
            queue: worker-queue
      target:
        type: Value
        value: "10"
```

### Comandos de HPA

```bash
# Crear HPA
kubectl apply -f hpa.yaml

# Ver HPAs
kubectl get hpa

# Información detallada
kubectl describe hpa webapp-hpa

# Ver métricas actuales
kubectl top pods

# Escalar manualmente (para testing)
kubectl scale deployment webapp-deployment --replicas=1

# Generar carga para probar HPA
kubectl run -i --tty load-generator --rm --image=busybox --restart=Never -- /bin/sh
# En el pod: while true; do wget -q -O- http://webapp-service; done
```

## 📏 Vertical Pod Autoscaler (VPA)

### ¿Qué es VPA?

El **VPA** ajusta automáticamente los recursos (CPU/memoria) de los contenedores.

```bash
# Instalar VPA
git clone https://github.com/kubernetes/autoscaler.git
cd autoscaler/vertical-pod-autoscaler/
./hack/vpa-install.sh
```

```yaml
# vpa.yaml
apiVersion: autoscaling.k8s.io/v1
kind: VerticalPodAutoscaler
metadata:
  name: webapp-vpa
spec:
  targetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: webapp-deployment
  updatePolicy:
    updateMode: "Auto"  # Off, Initial, Auto
  resourcePolicy:
    containerPolicies:
    - containerName: webapp
      maxAllowed:
        cpu: 1
        memory: 2Gi
      minAllowed:
        cpu: 100m
        memory: 128Mi
      controlledResources: ["cpu", "memory"]
```

### Modos de VPA

1. **Off**: Solo genera recomendaciones
2. **Initial**: Establece recursos solo en creación
3. **Auto**: Actualiza recursos automáticamente

```bash
# Ver recomendaciones de VPA
kubectl describe vpa webapp-vpa
```

## 🌐 Balanceo de Carga con Services

### LoadBalancer Service

```yaml
# service-loadbalancer.yaml
apiVersion: v1
kind: Service
metadata:
  name: webapp-loadbalancer
  annotations:
    # Anotaciones específicas del proveedor cloud
    service.beta.kubernetes.io/aws-load-balancer-type: nlb
    service.beta.kubernetes.io/azure-load-balancer-internal: "true"
spec:
  type: LoadBalancer
  selector:
    app: webapp
  ports:
  - port: 80
    targetPort: 8080
    protocol: TCP
  sessionAffinity: ClientIP
  sessionAffinityConfig:
    clientIP:
      timeoutSeconds: 300
```

### Service con ExternalTrafficPolicy

```yaml
# service-external-traffic.yaml
apiVersion: v1
kind: Service
metadata:
  name: webapp-nodeport
spec:
  type: NodePort
  externalTrafficPolicy: Local  # Preserva IP origen
  selector:
    app: webapp
  ports:
  - port: 80
    targetPort: 8080
    nodePort: 30080
```

## 🚪 Ingress Controllers

### ¿Qué es un Ingress?

Un **Ingress** expone rutas HTTP y HTTPS desde fuera del cluster hacia servicios internos.

### Instalar NGINX Ingress Controller

```bash
# Usando Helm
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
helm repo update
helm install ingress-nginx ingress-nginx/ingress-nginx \
  --create-namespace \
  --namespace ingress-nginx

# Verificar instalación
kubectl get pods -n ingress-nginx
kubectl get svc -n ingress-nginx
```

### Ingress Básico

```yaml
# ingress-basic.yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: webapp-ingress
  annotations:
    kubernetes.io/ingress.class: nginx
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  rules:
  - host: webapp.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: webapp-service
            port:
              number: 80
```

### Ingress con TLS

```yaml
# ingress-tls.yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: webapp-tls-ingress
  annotations:
    kubernetes.io/ingress.class: nginx
    cert-manager.io/cluster-issuer: letsencrypt-prod
    nginx.ingress.kubernetes.io/ssl-redirect: "true"
spec:
  tls:
  - hosts:
    - webapp.example.com
    - api.example.com
    secretName: webapp-tls
  rules:
  - host: webapp.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: frontend-service
            port:
              number: 80
  - host: api.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: backend-service
            port:
              number: 8080
```

### Ingress Avanzado con Múltiples Servicios

```yaml
# ingress-avanzado.yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: multi-service-ingress
  annotations:
    kubernetes.io/ingress.class: nginx
    nginx.ingress.kubernetes.io/auth-type: basic
    nginx.ingress.kubernetes.io/auth-secret: basic-auth
    nginx.ingress.kubernetes.io/cors-allow-methods: "GET, POST, PUT, DELETE"
    nginx.ingress.kubernetes.io/cors-allow-origin: "*"
    nginx.ingress.kubernetes.io/rate-limit: "100"
    nginx.ingress.kubernetes.io/limit-connections: "10"
spec:
  tls:
  - hosts:
    - myapp.example.com
    secretName: myapp-tls
  rules:
  - host: myapp.example.com
    http:
      paths:
      # Frontend
      - path: /
        pathType: Prefix
        backend:
          service:
            name: frontend-service
            port:
              number: 80
      
      # API
      - path: /api
        pathType: Prefix
        backend:
          service:
            name: backend-service
            port:
              number: 8080
      
      # Websockets
      - path: /ws
        pathType: Prefix
        backend:
          service:
            name: websocket-service
            port:
              number: 8081
      
      # Archivos estáticos
      - path: /static
        pathType: Prefix
        backend:
          service:
            name: static-service
            port:
              number: 8082
```

### Cert-Manager para Certificados Automáticos

```bash
# Instalar cert-manager
kubectl apply -f https://github.com/cert-manager/cert-manager/releases/download/v1.11.0/cert-manager.yaml

# Crear issuer para Let's Encrypt
kubectl apply -f - <<EOF
apiVersion: cert-manager.io/v1
kind: ClusterIssuer
metadata:
  name: letsencrypt-prod
spec:
  acme:
    server: https://acme-v02.api.letsencrypt.org/directory
    email: your-email@example.com
    privateKeySecretRef:
      name: letsencrypt-prod
    solvers:
    - http01:
        ingress:
          class: nginx
EOF
```

## ⚖️ Pod Disruption Budgets

### ¿Qué es un PDB?

Un **Pod Disruption Budget** define el número mínimo de Pods que deben estar disponibles durante interrupciones voluntarias.

```yaml
# pdb.yaml
apiVersion: policy/v1
kind: PodDisruptionBudget
metadata:
  name: webapp-pdb
spec:
  minAvailable: 2
  # O usar maxUnavailable: 1
  selector:
    matchLabels:
      app: webapp
```

### PDB con Porcentajes

```yaml
# pdb-porcentaje.yaml
apiVersion: policy/v1
kind: PodDisruptionBudget
metadata:
  name: webapp-pdb-percent
spec:
  maxUnavailable: 25%
  selector:
    matchLabels:
      app: webapp
```

### PDB para Múltiples Deployments

```yaml
# pdb-multi-app.yaml
apiVersion: policy/v1
kind: PodDisruptionBudget
metadata:
  name: critical-apps-pdb
spec:
  minAvailable: 50%
  selector:
    matchExpressions:
    - key: tier
      operator: In
      values: ["frontend", "backend"]
    - key: environment
      operator: In
      values: ["production"]
```

## 🔄 Cluster Autoscaler

### Configuración de Cluster Autoscaler

```yaml
# cluster-autoscaler.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: cluster-autoscaler
  namespace: kube-system
spec:
  replicas: 1
  selector:
    matchLabels:
      app: cluster-autoscaler
  template:
    metadata:
      labels:
        app: cluster-autoscaler
    spec:
      serviceAccountName: cluster-autoscaler
      containers:
      - image: k8s.gcr.io/autoscaling/cluster-autoscaler:v1.21.0
        name: cluster-autoscaler
        resources:
          limits:
            cpu: 100m
            memory: 300Mi
          requests:
            cpu: 100m
            memory: 300Mi
        command:
        - ./cluster-autoscaler
        - --v=4
        - --stderrthreshold=info
        - --cloud-provider=aws
        - --skip-nodes-with-local-storage=false
        - --expander=least-waste
        - --node-group-auto-discovery=asg:tag=k8s.io/cluster-autoscaler/enabled,k8s.io/cluster-autoscaler/myCluster
        env:
        - name: AWS_REGION
          value: us-west-2
```

### Node Groups con Auto Scaling

```yaml
# nodegroup-autoscaling.yaml (ejemplo para AWS)
apiVersion: v1
kind: ConfigMap
metadata:
  name: cluster-autoscaler-status
  namespace: kube-system
data:
  nodes.max: "100"
  nodes.min: "3"
  
  node-groups: |
    - name: worker-nodes
      min: 3
      max: 20
      instance-type: m5.large
    - name: spot-nodes
      min: 0
      max: 50
      instance-type: m5.large
      lifecycle: spot
```

## 🛠️ Ejercicio Práctico: Auto-Scaling Completo

### 1. Deployment con Resource Requests

```yaml
# webapp-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: webapp-scalable
  labels:
    app: webapp
spec:
  replicas: 2
  selector:
    matchLabels:
      app: webapp
  template:
    metadata:
      labels:
        app: webapp
    spec:
      containers:
      - name: webapp
        image: nginx:alpine
        ports:
        - containerPort: 80
        resources:
          requests:
            cpu: 100m
            memory: 128Mi
          limits:
            cpu: 200m
            memory: 256Mi
        livenessProbe:
          httpGet:
            path: /
            port: 80
          initialDelaySeconds: 30
          periodSeconds: 10
        readinessProbe:
          httpGet:
            path: /
            port: 80
          initialDelaySeconds: 5
          periodSeconds: 5
```

### 2. Service

```yaml
# webapp-service.yaml
apiVersion: v1
kind: Service
metadata:
  name: webapp-service
spec:
  selector:
    app: webapp
  ports:
  - port: 80
    targetPort: 80
  type: ClusterIP
```

### 3. HPA

```yaml
# webapp-hpa.yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: webapp-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: webapp-scalable
  minReplicas: 2
  maxReplicas: 20
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 50
  - type: Resource
    resource:
      name: memory
      target:
        type: Utilization
        averageUtilization: 70
  behavior:
    scaleUp:
      stabilizationWindowSeconds: 60
      policies:
      - type: Percent
        value: 50
        periodSeconds: 60
      - type: Pods
        value: 2
        periodSeconds: 60
      selectPolicy: Max
    scaleDown:
      stabilizationWindowSeconds: 300
      policies:
      - type: Percent
        value: 10
        periodSeconds: 60
      selectPolicy: Min
```

### 4. Pod Disruption Budget

```yaml
# webapp-pdb.yaml
apiVersion: policy/v1
kind: PodDisruptionBudget
metadata:
  name: webapp-pdb
spec:
  minAvailable: 1
  selector:
    matchLabels:
      app: webapp
```

### 5. Ingress

```yaml
# webapp-ingress.yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: webapp-ingress
  annotations:
    kubernetes.io/ingress.class: nginx
    nginx.ingress.kubernetes.io/rate-limit: "100"
    nginx.ingress.kubernetes.io/limit-connections: "10"
spec:
  rules:
  - host: webapp.local
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: webapp-service
            port:
              number: 80
```

### 6. Load Testing y Verificación

```bash
# Aplicar configuración
kubectl apply -f webapp-deployment.yaml
kubectl apply -f webapp-service.yaml
kubectl apply -f webapp-hpa.yaml
kubectl apply -f webapp-pdb.yaml
kubectl apply -f webapp-ingress.yaml

# Verificar estado inicial
kubectl get deployments
kubectl get hpa
kubectl get pdb

# Generar carga para probar scaling
kubectl run load-generator -i --tty --rm --image=busybox --restart=Never -- /bin/sh

# En el load generator:
while true; do 
  for i in {1..100}; do 
    wget -q -O- http://webapp-service &
  done
  wait
  sleep 1
done

# En otra terminal, monitorear scaling
watch -n 2 kubectl get hpa
watch -n 2 kubectl get pods

# Ver eventos de scaling
kubectl get events --sort-by=.metadata.creationTimestamp
```

## 📊 Balanceadores de Carga Avanzados

### Istio Service Mesh

```yaml
# istio-virtual-service.yaml
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: webapp-vs
spec:
  hosts:
  - webapp.example.com
  http:
  - match:
    - headers:
        canary:
          exact: "true"
    route:
    - destination:
        host: webapp-service
        subset: v2
      weight: 100
  - route:
    - destination:
        host: webapp-service
        subset: v1
      weight: 90
    - destination:
        host: webapp-service
        subset: v2
      weight: 10
---
apiVersion: networking.istio.io/v1alpha3
kind: DestinationRule
metadata:
  name: webapp-dr
spec:
  host: webapp-service
  subsets:
  - name: v1
    labels:
      version: v1
  - name: v2
    labels:
      version: v2
  trafficPolicy:
    loadBalancer:
      simple: LEAST_CONN
```

### Envoy Proxy Configuration

```yaml
# envoy-proxy.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: envoy-config
data:
  envoy.yaml: |
    static_resources:
      listeners:
      - name: main
        address:
          socket_address:
            address: 0.0.0.0
            port_value: 80
        filter_chains:
        - filters:
          - name: envoy.filters.network.http_connection_manager
            typed_config:
              "@type": type.googleapis.com/envoy.extensions.filters.network.http_connection_manager.v3.HttpConnectionManager
              stat_prefix: ingress_http
              route_config:
                name: local_route
                virtual_hosts:
                - name: webapp
                  domains: ["*"]
                  routes:
                  - match:
                      prefix: "/"
                    route:
                      cluster: webapp_cluster
              http_filters:
              - name: envoy.filters.http.router
      clusters:
      - name: webapp_cluster
        type: STRICT_DNS
        lb_policy: ROUND_ROBIN
        load_assignment:
          cluster_name: webapp_cluster
          endpoints:
          - lb_endpoints:
            - endpoint:
                address:
                  socket_address:
                    address: webapp-service
                    port_value: 8080
```

## 🔍 Monitoreo de Escalado

### Métricas Importantes

```bash
# CPU y memoria de Pods
kubectl top pods --sort-by=cpu
kubectl top pods --sort-by=memory

# Estado de HPA
kubectl get hpa -w

# Eventos de escalado
kubectl get events --field-selector reason=SuccessfulCreate
kubectl get events --field-selector reason=Killing

# Métricas detalladas
kubectl describe hpa webapp-hpa
```

### Dashboard de Monitoreo

```yaml
# monitoring-stack.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: prometheus-config
data:
  prometheus.yml: |
    global:
      scrape_interval: 15s
    scrape_configs:
    - job_name: 'kubernetes-pods'
      kubernetes_sd_configs:
      - role: pod
      relabel_configs:
      - source_labels: [__meta_kubernetes_pod_annotation_prometheus_io_scrape]
        action: keep
        regex: true
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: prometheus
spec:
  replicas: 1
  selector:
    matchLabels:
      app: prometheus
  template:
    metadata:
      labels:
        app: prometheus
    spec:
      containers:
      - name: prometheus
        image: prom/prometheus:latest
        ports:
        - containerPort: 9090
        volumeMounts:
        - name: prometheus-config
          mountPath: /etc/prometheus
      volumes:
      - name: prometheus-config
        configMap:
          name: prometheus-config
```

## 📝 Mejores Prácticas

### Escalado

1. **Define resource requests** apropiadas
2. **Configura límites realistas** para scaling
3. **Usa múltiples métricas** para HPA
4. **Implementa health checks** robustos
5. **Monitorea patrones** de tráfico

### Balanceo de Carga

1. **Usa session affinity** cuando sea necesario
2. **Implementa circuit breakers** para resilencia
3. **Configura timeouts** apropiados
4. **Distribuye carga** geográficamente
5. **Monitorea latencia** y throughput

### Seguridad

1. **Usa TLS** para tráfico externo
2. **Implementa rate limiting**
3. **Configura CORS** apropiadamente
4. **Usa Web Application Firewall** (WAF)
5. **Audita configuración** regularmente

## 🎯 Resumen

Has aprendido:
- ✅ Escalado automático horizontal y vertical
- ✅ Configuración de balanceadores de carga
- ✅ Implementación de Ingress Controllers
- ✅ Gestión de Pod Disruption Budgets
- ✅ Monitoreo y optimización de rendimiento

## 📚 Recursos Adicionales

- [Horizontal Pod Autoscaler](https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale/)
- [Vertical Pod Autoscaler](https://github.com/kubernetes/autoscaler/tree/master/vertical-pod-autoscaler)
- [NGINX Ingress Controller](https://kubernetes.github.io/ingress-nginx/)
- [Cert-Manager](https://cert-manager.io/)
- [Istio Service Mesh](https://istio.io/)

---

**Próximo tema:** [Helm: Gestión de charts para Kubernetes](./09-helm-charts.md)