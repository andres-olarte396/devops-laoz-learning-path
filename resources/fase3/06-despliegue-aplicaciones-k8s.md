# Despliegue de Aplicaciones en Kubernetes

## 🎯 Objetivos de Aprendizaje

Al finalizar este tema serás capaz de:
- Comprender y usar Deployments para gestión de aplicaciones
- Implementar ReplicaSets para alta disponibilidad
- Configurar DaemonSets para servicios del sistema
- Gestionar Jobs y CronJobs para tareas programadas
- Implementar estrategias de despliegue seguras

## 📚 Controladores de Kubernetes

Los **controladores** gestionan el ciclo de vida de los recursos y mantienen el estado deseado del sistema.

### Jerarquía de Controladores

```
Deployment
    ↓
ReplicaSet
    ↓
Pod(s)
```

## 🚀 Deployments

### ¿Qué es un Deployment?

Un **Deployment** proporciona actualizaciones declarativas para Pods y ReplicaSets.

```yaml
# deployment-basic.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: webapp-deployment
  labels:
    app: webapp
spec:
  replicas: 3
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
        image: nginx:1.20
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

### Características de Deployments

1. **Gestión de ReplicaSets**: Crea y gestiona ReplicaSets automáticamente
2. **Rolling Updates**: Actualizaciones sin tiempo de inactividad
3. **Rollback**: Reversión a versiones anteriores
4. **Escalado**: Modificación dinámica del número de réplicas
5. **Historial**: Mantiene historial de revisiones

### Comandos de Deployment

```bash
# Crear deployment
kubectl apply -f deployment.yaml

# Ver deployments
kubectl get deployments

# Información detallada
kubectl describe deployment webapp-deployment

# Escalar deployment
kubectl scale deployment webapp-deployment --replicas=5

# Actualizar imagen
kubectl set image deployment/webapp-deployment webapp=nginx:1.21

# Ver historial de rollouts
kubectl rollout history deployment/webapp-deployment

# Rollback a versión anterior
kubectl rollout undo deployment/webapp-deployment

# Rollback a versión específica
kubectl rollout undo deployment/webapp-deployment --to-revision=2

# Ver estado del rollout
kubectl rollout status deployment/webapp-deployment
```

### Deployment Avanzado con Estrategias

```yaml
# deployment-avanzado.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: webapp-production
  labels:
    app: webapp
    environment: production
spec:
  replicas: 6
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxUnavailable: 1
      maxSurge: 2
  selector:
    matchLabels:
      app: webapp
  template:
    metadata:
      labels:
        app: webapp
        environment: production
    spec:
      containers:
      - name: webapp
        image: myapp:v2.1.0
        ports:
        - containerPort: 8080
        env:
        - name: ENVIRONMENT
          value: "production"
        - name: LOG_LEVEL
          value: "info"
        resources:
          requests:
            cpu: 200m
            memory: 256Mi
          limits:
            cpu: 500m
            memory: 512Mi
        livenessProbe:
          httpGet:
            path: /health
            port: 8080
          initialDelaySeconds: 30
          periodSeconds: 10
          timeoutSeconds: 5
          failureThreshold: 3
        readinessProbe:
          httpGet:
            path: /ready
            port: 8080
          initialDelaySeconds: 5
          periodSeconds: 5
          timeoutSeconds: 3
          failureThreshold: 3
        volumeMounts:
        - name: config-volume
          mountPath: /app/config
        - name: logs-volume
          mountPath: /app/logs
      volumes:
      - name: config-volume
        configMap:
          name: webapp-config
      - name: logs-volume
        emptyDir: {}
      imagePullSecrets:
      - name: registry-secret
```

## 📋 ReplicaSets

### ¿Qué es un ReplicaSet?

Un **ReplicaSet** asegura que un número específico de réplicas de Pod estén ejecutándose.

```yaml
# replicaset.yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: webapp-replicaset
  labels:
    app: webapp
spec:
  replicas: 3
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
```

### Diferencias: Deployment vs ReplicaSet

| Deployment | ReplicaSet |
|------------|------------|
| Gestiona ReplicaSets | Gestiona Pods directamente |
| Rolling updates automáticos | Updates manuales |
| Historial de versiones | Sin historial |
| Rollback automático | Sin rollback |
| **Recomendado para producción** | Uso directo limitado |

## 🔄 DaemonSets

### ¿Qué es un DaemonSet?

Un **DaemonSet** asegura que todos (o algunos) nodos ejecuten una copia de un Pod.

```yaml
# daemonset-logging.yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: log-collector
  labels:
    app: log-collector
spec:
  selector:
    matchLabels:
      app: log-collector
  template:
    metadata:
      labels:
        app: log-collector
    spec:
      tolerations:
      # Permite ejecutar en nodos master
      - key: node-role.kubernetes.io/master
        effect: NoSchedule
      containers:
      - name: log-collector
        image: fluentd:v1.14
        resources:
          limits:
            memory: 200Mi
            cpu: 100m
          requests:
            cpu: 100m
            memory: 200Mi
        volumeMounts:
        - name: varlog
          mountPath: /var/log
        - name: varlibdockercontainers
          mountPath: /var/lib/docker/containers
          readOnly: true
      terminationGracePeriodSeconds: 30
      volumes:
      - name: varlog
        hostPath:
          path: /var/log
      - name: varlibdockercontainers
        hostPath:
          path: /var/lib/docker/containers
```

### Casos de Uso de DaemonSets

1. **Monitoring agents**: Prometheus node exporter
2. **Log collectors**: Fluentd, Filebeat
3. **Network plugins**: CNI daemons
4. **Storage daemons**: Ceph, GlusterFS
5. **Security agents**: Antivirus, compliance scanners

### DaemonSet con Node Selector

```yaml
# daemonset-selective.yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: ssd-monitor
spec:
  selector:
    matchLabels:
      app: ssd-monitor
  template:
    metadata:
      labels:
        app: ssd-monitor
    spec:
      nodeSelector:
        disk-type: ssd
      containers:
      - name: ssd-monitor
        image: ssd-monitoring-tool:latest
```

## ⚡ Jobs y CronJobs

### Jobs - Tareas de Una Vez

```yaml
# job-backup.yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: database-backup
spec:
  completions: 1
  parallelism: 1
  backoffLimit: 3
  template:
    spec:
      containers:
      - name: backup
        image: mysql:8.0
        command:
        - /bin/bash
        - -c
        - |
          mysqldump -h $DB_HOST -u $DB_USER -p$DB_PASSWORD $DB_NAME > /backup/backup-$(date +%Y%m%d).sql
        env:
        - name: DB_HOST
          value: "mysql-service"
        - name: DB_USER
          valueFrom:
            secretKeyRef:
              name: db-secret
              key: username
        - name: DB_PASSWORD
          valueFrom:
            secretKeyRef:
              name: db-secret
              key: password
        - name: DB_NAME
          value: "production"
        volumeMounts:
        - name: backup-storage
          mountPath: /backup
      volumes:
      - name: backup-storage
        persistentVolumeClaim:
          claimName: backup-pvc
      restartPolicy: OnFailure
```

### Jobs Paralelos

```yaml
# job-parallel.yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: data-processing
spec:
  completions: 10
  parallelism: 3
  template:
    spec:
      containers:
      - name: worker
        image: data-processor:latest
        command: ["python", "process_batch.py"]
      restartPolicy: Never
```

### CronJobs - Tareas Programadas

```yaml
# cronjob-cleanup.yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: log-cleanup
spec:
  schedule: "0 2 * * *"  # Todos los días a las 2:00 AM
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: cleanup
            image: alpine:latest
            command:
            - /bin/sh
            - -c
            - find /logs -name "*.log" -mtime +7 -delete
            volumeMounts:
            - name: logs
              mountPath: /logs
          volumes:
          - name: logs
            hostPath:
              path: /var/log/myapp
          restartPolicy: OnFailure
  successfulJobsHistoryLimit: 3
  failedJobsHistoryLimit: 1
```

### Sintaxis de Cron

```bash
# Formato: minute hour day month day-of-week
# ┌───────────── minuto (0 - 59)
# │ ┌───────────── hora (0 - 23)
# │ │ ┌───────────── día del mes (1 - 31)
# │ │ │ ┌───────────── mes (1 - 12)
# │ │ │ │ ┌───────────── día de la semana (0 - 6) (Domingo=0)
# │ │ │ │ │
# * * * * *

# Ejemplos:
# "0 0 * * *"     # Todos los días a medianoche
# "0 6 * * 1"     # Todos los lunes a las 6:00 AM
# "*/15 * * * *"  # Cada 15 minutos
# "0 2 1 * *"     # El primer día de cada mes a las 2:00 AM
# "0 0 * * 0"     # Todos los domingos a medianoche
```

## 🔄 Estrategias de Despliegue

### 1. Rolling Update (Por Defecto)

```yaml
spec:
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxUnavailable: 25%
      maxSurge: 25%
```

**Ventajas**:
- Sin tiempo de inactividad
- Rollback automático en caso de fallos
- Gradual y controlado

**Desventajas**:
- Versiones mixtas temporalmente
- Proceso más lento

### 2. Recreate

```yaml
spec:
  strategy:
    type: Recreate
```

**Ventajas**:
- Evita versiones mixtas
- Más simple

**Desventajas**:
- Tiempo de inactividad
- No apto para producción

### 3. Blue-Green Deployment

```yaml
# Deployment Blue (versión actual)
apiVersion: apps/v1
kind: Deployment
metadata:
  name: webapp-blue
  labels:
    version: blue
spec:
  replicas: 3
  selector:
    matchLabels:
      app: webapp
      version: blue
  template:
    metadata:
      labels:
        app: webapp
        version: blue
    spec:
      containers:
      - name: webapp
        image: myapp:v1.0
---
# Deployment Green (nueva versión)
apiVersion: apps/v1
kind: Deployment
metadata:
  name: webapp-green
  labels:
    version: green
spec:
  replicas: 3
  selector:
    matchLabels:
      app: webapp
      version: green
  template:
    metadata:
      labels:
        app: webapp
        version: green
    spec:
      containers:
      - name: webapp
        image: myapp:v2.0
---
# Service que cambia entre blue y green
apiVersion: v1
kind: Service
metadata:
  name: webapp-service
spec:
  selector:
    app: webapp
    version: blue  # Cambiar a 'green' para switch
  ports:
  - port: 80
    targetPort: 8080
```

### 4. Canary Deployment

```yaml
# Deployment principal (90% tráfico)
apiVersion: apps/v1
kind: Deployment
metadata:
  name: webapp-main
spec:
  replicas: 9
  selector:
    matchLabels:
      app: webapp
      track: stable
  template:
    metadata:
      labels:
        app: webapp
        track: stable
    spec:
      containers:
      - name: webapp
        image: myapp:v1.0
---
# Deployment canary (10% tráfico)
apiVersion: apps/v1
kind: Deployment
metadata:
  name: webapp-canary
spec:
  replicas: 1
  selector:
    matchLabels:
      app: webapp
      track: canary
  template:
    metadata:
      labels:
        app: webapp
        track: canary
    spec:
      containers:
      - name: webapp
        image: myapp:v2.0
---
# Service que distribuye tráfico
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
```

## 🛠️ Gestión de Configuración

### Variables de Entorno

```yaml
# deployment-con-env.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: webapp-with-env
spec:
  replicas: 3
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
        image: myapp:latest
        env:
        # Variable simple
        - name: ENVIRONMENT
          value: "production"
        
        # Desde ConfigMap
        - name: DATABASE_URL
          valueFrom:
            configMapKeyRef:
              name: app-config
              key: database.url
        
        # Desde Secret
        - name: API_KEY
          valueFrom:
            secretKeyRef:
              name: api-secrets
              key: api-key
        
        # Información del Pod
        - name: POD_NAME
          valueFrom:
            fieldRef:
              fieldPath: metadata.name
        
        # Recursos del contenedor
        - name: MEMORY_LIMIT
          valueFrom:
            resourceFieldRef:
              resource: limits.memory
```

### Montar ConfigMaps y Secrets

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: webapp-with-volumes
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
        image: myapp:latest
        volumeMounts:
        # ConfigMap como archivo
        - name: app-config
          mountPath: /app/config
          readOnly: true
        
        # Secret como archivo
        - name: db-credentials
          mountPath: /app/secrets
          readOnly: true
        
        # Archivo específico de ConfigMap
        - name: nginx-config
          mountPath: /etc/nginx/nginx.conf
          subPath: nginx.conf
          readOnly: true
      
      volumes:
      # ConfigMap completo
      - name: app-config
        configMap:
          name: webapp-config
      
      # Secret completo
      - name: db-credentials
        secret:
          secretName: db-secret
          defaultMode: 0400
      
      # Archivo específico
      - name: nginx-config
        configMap:
          name: nginx-config
          items:
          - key: nginx.conf
            path: nginx.conf
```

## 🔍 Monitoreo y Debugging

### Health Checks

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: webapp-with-probes
spec:
  replicas: 3
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
        image: myapp:latest
        ports:
        - containerPort: 8080
        
        # Verifica si el contenedor está vivo
        livenessProbe:
          httpGet:
            path: /health
            port: 8080
          initialDelaySeconds: 30
          periodSeconds: 10
          timeoutSeconds: 5
          failureThreshold: 3
          successThreshold: 1
        
        # Verifica si el contenedor está listo para tráfico
        readinessProbe:
          httpGet:
            path: /ready
            port: 8080
          initialDelaySeconds: 5
          periodSeconds: 5
          timeoutSeconds: 3
          failureThreshold: 3
          successThreshold: 1
        
        # Verifica si el contenedor inició correctamente
        startupProbe:
          httpGet:
            path: /startup
            port: 8080
          initialDelaySeconds: 10
          periodSeconds: 10
          timeoutSeconds: 3
          failureThreshold: 30
          successThreshold: 1
```

### Tipos de Probes

```yaml
# HTTP GET
livenessProbe:
  httpGet:
    path: /health
    port: 8080
    httpHeaders:
    - name: Custom-Header
      value: Awesome

# TCP Socket
livenessProbe:
  tcpSocket:
    port: 8080

# Comando exec
livenessProbe:
  exec:
    command:
    - cat
    - /tmp/healthy
```

### Comandos de Debugging

```bash
# Ver estado de deployments
kubectl get deployments
kubectl get replicasets
kubectl get pods

# Información detallada
kubectl describe deployment webapp-deployment
kubectl describe pod <pod-name>

# Logs de aplicación
kubectl logs deployment/webapp-deployment
kubectl logs -f deployment/webapp-deployment
kubectl logs deployment/webapp-deployment --previous

# Eventos del cluster
kubectl get events --sort-by=.metadata.creationTimestamp

# Entrar al contenedor
kubectl exec -it <pod-name> -- /bin/bash

# Port forwarding para testing
kubectl port-forward deployment/webapp-deployment 8080:80

# Ver recursos utilizados
kubectl top pods
kubectl top nodes
```

## 🎯 Ejercicio Práctico Completo

### Desplegar Aplicación WordPress

```yaml
# 1. MySQL Deployment
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mysql
  labels:
    app: mysql
spec:
  replicas: 1
  selector:
    matchLabels:
      app: mysql
  template:
    metadata:
      labels:
        app: mysql
    spec:
      containers:
      - name: mysql
        image: mysql:8.0
        env:
        - name: MYSQL_ROOT_PASSWORD
          valueFrom:
            secretKeyRef:
              name: mysql-secret
              key: password
        - name: MYSQL_DATABASE
          value: wordpress
        ports:
        - containerPort: 3306
        volumeMounts:
        - name: mysql-storage
          mountPath: /var/lib/mysql
      volumes:
      - name: mysql-storage
        emptyDir: {}
---
# 2. MySQL Service
apiVersion: v1
kind: Service
metadata:
  name: mysql-service
spec:
  selector:
    app: mysql
  ports:
  - port: 3306
    targetPort: 3306
---
# 3. WordPress Deployment
apiVersion: apps/v1
kind: Deployment
metadata:
  name: wordpress
  labels:
    app: wordpress
spec:
  replicas: 2
  selector:
    matchLabels:
      app: wordpress
  template:
    metadata:
      labels:
        app: wordpress
    spec:
      containers:
      - name: wordpress
        image: wordpress:latest
        env:
        - name: WORDPRESS_DB_HOST
          value: mysql-service:3306
        - name: WORDPRESS_DB_PASSWORD
          valueFrom:
            secretKeyRef:
              name: mysql-secret
              key: password
        - name: WORDPRESS_DB_NAME
          value: wordpress
        ports:
        - containerPort: 80
        volumeMounts:
        - name: wordpress-storage
          mountPath: /var/www/html
      volumes:
      - name: wordpress-storage
        emptyDir: {}
---
# 4. WordPress Service
apiVersion: v1
kind: Service
metadata:
  name: wordpress-service
spec:
  type: LoadBalancer
  selector:
    app: wordpress
  ports:
  - port: 80
    targetPort: 80
---
# 5. Secret para MySQL
apiVersion: v1
kind: Secret
metadata:
  name: mysql-secret
type: Opaque
data:
  password: cGFzc3dvcmQxMjM=  # password123 en base64
```

### Comandos de Despliegue

```bash
# Aplicar configuración
kubectl apply -f wordpress-stack.yaml

# Verificar estado
kubectl get all

# Ver logs
kubectl logs deployment/wordpress
kubectl logs deployment/mysql

# Obtener IP externa (si usa LoadBalancer)
kubectl get service wordpress-service

# Port forward para testing local
kubectl port-forward service/wordpress-service 8080:80
```

## 📝 Mejores Prácticas

### Diseño de Aplicaciones

1. **Usa Deployments** para aplicaciones stateless
2. **Implementa health checks** apropiados
3. **Define resource limits** para evitar consumo excesivo
4. **Separa configuración** del código con ConfigMaps
5. **Gestiona secretos** apropiadamente

### Estrategias de Despliegue

1. **Usa rolling updates** para disponibilidad
2. **Implementa canary deployments** para releases críticas
3. **Mantén historial** de versiones para rollback
4. **Prueba en staging** antes de producción
5. **Monitorea métricas** durante despliegues

### Gestión de Recursos

1. **Especifica requests y limits** de CPU/memoria
2. **Usa node selectors** para placement específico
3. **Implementa pod disruption budgets**
4. **Configura auto-scaling** según necesidad
5. **Planifica capacidad** del cluster

## 🎯 Resumen

Has aprendido:
- ✅ Gestión de aplicaciones con Deployments
- ✅ Uso de ReplicaSets y DaemonSets
- ✅ Implementación de Jobs y CronJobs
- ✅ Estrategias de despliegue avanzadas
- ✅ Configuración y monitoreo de aplicaciones

## 📚 Recursos Adicionales

- [Kubernetes Deployments](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/)
- [Jobs and CronJobs](https://kubernetes.io/docs/concepts/workloads/controllers/job/)
- [DaemonSet Documentation](https://kubernetes.io/docs/concepts/workloads/controllers/daemonset/)
- [Deployment Strategies](https://kubernetes.io/docs/concepts/cluster-administration/manage-deployment/)

---

**Próximo tema:** [Gestión de configuraciones y secretos](./07-configuraciones-secretos.md)