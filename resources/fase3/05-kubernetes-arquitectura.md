# Arquitectura de Kubernetes: Pods, Nodes, Clusters, Services

## 🎯 Objetivos de Aprendizaje

Al finalizar este tema serás capaz de:
- Comprender la arquitectura completa de Kubernetes
- Diferenciar entre componentes del Control Plane y Worker Nodes
- Entender el concepto y funcionamiento de Pods
- Configurar y gestionar Services para comunicación
- Trabajar con Namespaces para organización de recursos

## 📚 ¿Qué es Kubernetes?

**Kubernetes** (K8s) es una plataforma de orquestación de contenedores que automatiza el despliegue, escalado y gestión de aplicaciones contenedorizadas.

### 🔍 Características Principales

1. **Orquestación**: Gestión automática de contenedores
2. **Auto-escalado**: Escala aplicaciones según demanda
3. **Auto-reparación**: Reemplaza contenedores fallidos
4. **Balanceo de carga**: Distribuye tráfico automáticamente
5. **Gestión de configuración**: Secrets y ConfigMaps
6. **Rolling updates**: Actualizaciones sin tiempo de inactividad

## 🏗️ Arquitectura de Kubernetes

```
┌─────────────────────────────────────────────────────────────┐
│                    KUBERNETES CLUSTER                       │
├─────────────────────────────────────────────────────────────┤
│                    CONTROL PLANE                           │
│  ┌─────────────┐ ┌─────────────┐ ┌─────────────┐ ┌──────── │
│  │   API       │ │    etcd     │ │ Controller  │ │ Scheduler│
│  │  Server     │ │             │ │  Manager    │ │          │
│  └─────────────┘ └─────────────┘ └─────────────┘ └──────── │
├─────────────────────────────────────────────────────────────┤
│                    WORKER NODES                            │
│  ┌─────────────────────────┐ ┌─────────────────────────┐   │
│  │       NODE 1            │ │       NODE 2            │   │
│  │  ┌─────────┐ ┌────────┐ │ │  ┌─────────┐ ┌────────┐ │   │
│  │  │ kubelet │ │ Pod 1  │ │ │  │ kubelet │ │ Pod 3  │ │   │
│  │  │         │ │ Pod 2  │ │ │  │         │ │ Pod 4  │ │   │
│  │  └─────────┘ └────────┘ │ │  └─────────┘ └────────┘ │   │
│  │  ┌─────────┐ ┌────────┐ │ │  ┌─────────┐ ┌────────┐ │   │
│  │  │kube-proxy│ │Container│ │ │  │kube-proxy│ │Container│ │   │
│  │  │         │ │Runtime │ │ │  │         │ │Runtime │ │   │
│  │  └─────────┘ └────────┘ │ │  └─────────┘ └────────┘ │   │
│  └─────────────────────────┘ └─────────────────────────┘   │
└─────────────────────────────────────────────────────────────┘
```

## 🧠 Control Plane Components

### API Server

**Función**: Punto de entrada central para todas las operaciones del cluster.

```bash
# Todas las interacciones pasan por el API Server
kubectl get pods        # → API Server → etcd
kubectl apply -f app.yaml  # → API Server → etcd + Controllers
```

**Características**:
- RESTful API para gestión del cluster
- Autenticación y autorización
- Validación de recursos
- Punto único de acceso a etcd

### etcd

**Función**: Base de datos distribuida que almacena el estado del cluster.

```bash
# etcd almacena toda la configuración del cluster
# - Definiciones de Pods, Services, ConfigMaps
# - Estado actual vs deseado
# - Metadatos de recursos
```

**Características**:
- Consistent and distributed key-value store
- Backup del estado completo del cluster
- Consenso distribuido via algoritmo Raft

### Controller Manager

**Función**: Ejecuta controladores que mantienen el estado deseado.

```yaml
# Ejemplos de controladores:
# - Deployment Controller: Gestiona ReplicaSets
# - Service Controller: Configura LoadBalancers
# - Node Controller: Monitorea estado de nodos
# - Job Controller: Ejecuta trabajos batch
```

### Scheduler

**Función**: Decide en qué nodo ejecutar cada Pod.

```bash
# Factores que considera el Scheduler:
# - Recursos disponibles (CPU, memoria)
# - Afinidad/anti-afinidad de nodos
# - Taints y tolerations
# - Restricciones de política
```

## 🖥️ Worker Node Components

### kubelet

**Función**: Agente que ejecuta en cada nodo y gestiona Pods.

```bash
# Responsabilidades del kubelet:
# - Registrar el nodo en el API Server
# - Recibir especificaciones de Pods
# - Gestionar ciclo de vida de contenedores
# - Reportar estado de salud
# - Ejecutar health checks
```

### kube-proxy

**Función**: Gestiona reglas de red y balanceado de carga.

```bash
# kube-proxy implementa:
# - Service discovery
# - Load balancing
# - Network routing
# - Port forwarding
```

### Container Runtime

**Función**: Software que ejecuta los contenedores.

```bash
# Runtimes soportados:
# - containerd (recomendado)
# - CRI-O
# - Docker Engine (deprecated)
```

## 📦 Pods - La Unidad Básica

### ¿Qué es un Pod?

Un **Pod** es la unidad de despliegue más pequeña en Kubernetes que puede contener uno o más contenedores.

```yaml
# pod-simple.yaml
apiVersion: v1
kind: Pod
metadata:
  name: mi-app
  labels:
    app: webapp
spec:
  containers:
  - name: web-container
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
```

### Características de los Pods

1. **Red compartida**: Todos los contenedores comparten IP y puertos
2. **Almacenamiento compartido**: Volúmenes accesibles por todos los contenedores
3. **Efímeros**: Los Pods pueden ser creados y destruidos dinámicamente
4. **Unidad atómica**: Se programa y escala como una unidad

### Pod Multi-Contenedor

```yaml
# pod-multicontenedor.yaml
apiVersion: v1
kind: Pod
metadata:
  name: webapp-con-proxy
spec:
  containers:
  # Contenedor principal de aplicación
  - name: webapp
    image: myapp:1.0
    ports:
    - containerPort: 8080
    
  # Contenedor sidecar para proxy
  - name: proxy
    image: nginx:alpine
    ports:
    - containerPort: 80
    volumeMounts:
    - name: nginx-config
      mountPath: /etc/nginx/nginx.conf
      subPath: nginx.conf
      
  # Contenedor para logging
  - name: log-collector
    image: fluentd:latest
    volumeMounts:
    - name: shared-logs
      mountPath: /var/log
      
  volumes:
  - name: nginx-config
    configMap:
      name: nginx-proxy-config
  - name: shared-logs
    emptyDir: {}
```

### Patrones de Pod Multi-Contenedor

#### 1. Sidecar Pattern

```yaml
# Contenedor principal + contenedor auxiliar
spec:
  containers:
  - name: app
    image: myapp:latest
  - name: log-collector  # Sidecar
    image: fluentd:latest
```

#### 2. Ambassador Pattern

```yaml
# Proxy para conexiones externas
spec:
  containers:
  - name: app
    image: myapp:latest
  - name: ambassador  # Proxy
    image: ambassador:latest
```

#### 3. Adapter Pattern

```yaml
# Adapta la salida del contenedor principal
spec:
  containers:
  - name: app
    image: legacy-app:latest
  - name: adapter  # Convierte formato de logs
    image: log-adapter:latest
```

## 🌐 Services - Comunicación y Descubrimiento

### ¿Qué es un Service?

Un **Service** define una forma estable de acceder a un conjunto de Pods.

```yaml
# service-clusterip.yaml
apiVersion: v1
kind: Service
metadata:
  name: webapp-service
spec:
  selector:
    app: webapp
  ports:
  - port: 80
    targetPort: 8080
    protocol: TCP
  type: ClusterIP
```

### Tipos de Services

#### 1. ClusterIP (Default)

```yaml
# Acceso interno al cluster
apiVersion: v1
kind: Service
metadata:
  name: internal-service
spec:
  type: ClusterIP
  selector:
    app: backend
  ports:
  - port: 80
    targetPort: 8080
```

#### 2. NodePort

```yaml
# Expone servicio en puerto específico de cada nodo
apiVersion: v1
kind: Service
metadata:
  name: nodeport-service
spec:
  type: NodePort
  selector:
    app: webapp
  ports:
  - port: 80
    targetPort: 8080
    nodePort: 30080  # Puerto 30080 en todos los nodos
```

#### 3. LoadBalancer

```yaml
# Crea un load balancer externo (cloud provider)
apiVersion: v1
kind: Service
metadata:
  name: loadbalancer-service
spec:
  type: LoadBalancer
  selector:
    app: webapp
  ports:
  - port: 80
    targetPort: 8080
```

#### 4. ExternalName

```yaml
# Mapea servicio a nombre DNS externo
apiVersion: v1
kind: Service
metadata:
  name: external-database
spec:
  type: ExternalName
  externalName: db.example.com
```

### Endpoints y Service Discovery

```bash
# Los Services crean automáticamente Endpoints
kubectl get endpoints webapp-service

# DNS interno automático:
# <service-name>.<namespace>.svc.cluster.local
# webapp-service.default.svc.cluster.local
```

## 🗂️ Namespaces - Organización de Recursos

### ¿Qué son los Namespaces?

Los **Namespaces** proporcionan aislamiento lógico de recursos dentro de un cluster.

```yaml
# namespace.yaml
apiVersion: v1
kind: Namespace
metadata:
  name: desarrollo
  labels:
    environment: dev
    team: backend
```

### Namespaces por Defecto

```bash
# Listar namespaces
kubectl get namespaces

# Namespaces del sistema:
# - default: Recursos sin namespace específico
# - kube-system: Componentes del sistema Kubernetes
# - kube-public: Recursos públicos del cluster
# - kube-node-lease: Información de lease de nodos
```

### Uso de Namespaces

```bash
# Crear namespace
kubectl create namespace produccion

# Aplicar recursos en namespace específico
kubectl apply -f app.yaml -n produccion

# Configurar namespace por defecto
kubectl config set-context --current --namespace=produccion

# Acceder a servicios entre namespaces
# service-name.namespace-name.svc.cluster.local
```

### Separación por Ambientes

```yaml
# Estructura de namespaces recomendada
---
apiVersion: v1
kind: Namespace
metadata:
  name: desarrollo
---
apiVersion: v1
kind: Namespace
metadata:
  name: testing
---
apiVersion: v1
kind: Namespace
metadata:
  name: produccion
```

## 🖥️ Nodes - Infraestructura del Cluster

### Información de Nodos

```bash
# Listar nodos
kubectl get nodes

# Información detallada de nodo
kubectl describe node <node-name>

# Ver recursos disponibles
kubectl top nodes
```

### Labels y Selectors

```yaml
# Agregar labels a nodos
apiVersion: v1
kind: Node
metadata:
  name: worker-1
  labels:
    hardware: high-memory
    zone: us-west-1a
    instance-type: m5.large
```

```yaml
# Usar node selector en Pods
apiVersion: v1
kind: Pod
metadata:
  name: memory-intensive-app
spec:
  nodeSelector:
    hardware: high-memory
  containers:
  - name: app
    image: memory-app:latest
```

### Taints y Tolerations

```bash
# Agregar taint a nodo
kubectl taint nodes worker-1 key=value:NoSchedule

# Remover taint
kubectl taint nodes worker-1 key=value:NoSchedule-
```

```yaml
# Pod que tolera el taint
apiVersion: v1
kind: Pod
metadata:
  name: tolerant-pod
spec:
  tolerations:
  - key: "key"
    operator: "Equal"
    value: "value"
    effect: "NoSchedule"
  containers:
  - name: app
    image: myapp:latest
```

## 🛠️ Comandos Básicos de kubectl

### Gestión de Recursos

```bash
# Crear recursos
kubectl create -f archivo.yaml
kubectl apply -f archivo.yaml

# Listar recursos
kubectl get pods
kubectl get services
kubectl get nodes

# Información detallada
kubectl describe pod <pod-name>
kubectl describe service <service-name>

# Logs de contenedores
kubectl logs <pod-name>
kubectl logs <pod-name> -c <container-name>

# Ejecutar comandos en contenedores
kubectl exec -it <pod-name> -- /bin/bash
kubectl exec <pod-name> -c <container-name> -- ls -la

# Port forwarding
kubectl port-forward pod/<pod-name> 8080:80
kubectl port-forward service/<service-name> 8080:80
```

### Debugging y Troubleshooting

```bash
# Estado de recursos
kubectl get pods --show-labels
kubectl get pods -o wide
kubectl get events

# Información del cluster
kubectl cluster-info
kubectl get componentstatuses

# Recursos por namespace
kubectl get all -n <namespace>

# Eliminar recursos
kubectl delete pod <pod-name>
kubectl delete service <service-name>
kubectl delete -f archivo.yaml
```

## 🔧 Ejercicio Práctico: Aplicación Completa

### 1. Crear Namespace

```yaml
# namespace.yaml
apiVersion: v1
kind: Namespace
metadata:
  name: webapp-demo
```

### 2. Crear Pod

```yaml
# pod.yaml
apiVersion: v1
kind: Pod
metadata:
  name: webapp-pod
  namespace: webapp-demo
  labels:
    app: webapp
    version: v1
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

### 3. Crear Service

```yaml
# service.yaml
apiVersion: v1
kind: Service
metadata:
  name: webapp-service
  namespace: webapp-demo
spec:
  selector:
    app: webapp
  ports:
  - port: 80
    targetPort: 80
    protocol: TCP
  type: ClusterIP
```

### 4. Aplicar y Verificar

```bash
# Aplicar recursos
kubectl apply -f namespace.yaml
kubectl apply -f pod.yaml
kubectl apply -f service.yaml

# Verificar estado
kubectl get all -n webapp-demo

# Probar conectividad
kubectl port-forward -n webapp-demo service/webapp-service 8080:80

# Acceder en navegador: http://localhost:8080
```

## 📝 Mejores Prácticas

### Organización de Recursos

1. **Usa namespaces** para separar ambientes y equipos
2. **Aplica labels consistentes** para organización
3. **Define resource requests y limits**
4. **Implementa health checks** (liveness/readiness probes)
5. **Usa Services** para comunicación entre Pods

### Seguridad

1. **Evita privilegios de root** en contenedores
2. **Usa Network Policies** para aislamiento de red
3. **Implementa RBAC** para control de acceso
4. **Escanea imágenes** por vulnerabilidades
5. **Usa secrets** para información sensible

### Monitoreo

1. **Configura logging centralizado**
2. **Implementa métricas** de aplicación
3. **Usa health checks apropiados**
4. **Monitorea recursos** del cluster
5. **Configura alertas** para fallos

## 🎯 Resumen

Has aprendido:
- ✅ Arquitectura completa de Kubernetes
- ✅ Componentes del Control Plane y Worker Nodes
- ✅ Concepto y uso de Pods
- ✅ Configuración de Services para comunicación
- ✅ Organización con Namespaces
- ✅ Comandos básicos de kubectl

## 📚 Recursos Adicionales

- [Kubernetes Documentation](https://kubernetes.io/docs/)
- [Kubernetes Concepts](https://kubernetes.io/docs/concepts/)
- [kubectl Cheat Sheet](https://kubernetes.io/docs/reference/kubectl/cheatsheet/)
- [Kubernetes Academy](https://kubernetes.academy/)
- [Play with Kubernetes](https://labs.play-with-k8s.com/)

---

**Próximo tema:** [Despliegue de aplicaciones en Kubernetes](./06-despliegue-aplicaciones-k8s.md)