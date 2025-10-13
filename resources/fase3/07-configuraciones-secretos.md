# Gestión de Configuraciones y Secretos

## 🎯 Objetivos de Aprendizaje

Al finalizar este tema serás capaz de:
- Gestionar configuración de aplicaciones con ConfigMaps
- Almacenar información sensible usando Secrets
- Implementar buenas prácticas de seguridad
- Configurar montaje de volúmenes para configuración
- Usar herramientas para gestión de secretos

## 📚 ConfigMaps - Gestión de Configuración

### ¿Qué es un ConfigMap?

Un **ConfigMap** almacena datos de configuración no confidenciales en pares clave-valor.

```yaml
# configmap-simple.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  # Configuración simple
  database.host: "mysql.example.com"
  database.port: "3306"
  log.level: "info"
  
  # Configuración multilinea
  app.properties: |
    server.port=8080
    server.servlet.context-path=/api
    spring.profiles.active=production
    
  # Archivo de configuración completo
  nginx.conf: |
    server {
        listen 80;
        server_name example.com;
        
        location / {
            proxy_pass http://backend:8080;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
        }
    }
```

### Crear ConfigMaps

#### Desde archivos

```bash
# Crear desde archivo
kubectl create configmap app-config --from-file=app.properties

# Crear desde múltiples archivos
kubectl create configmap nginx-config \
  --from-file=nginx.conf \
  --from-file=mime.types

# Crear desde directorio
kubectl create configmap web-config --from-file=config/

# Con clave personalizada
kubectl create configmap db-config \
  --from-file=database-config=db.properties
```

#### Desde literales

```bash
# Desde valores literales
kubectl create configmap app-config \
  --from-literal=database.host=mysql.example.com \
  --from-literal=database.port=3306 \
  --from-literal=log.level=debug
```

#### Desde variables de entorno

```bash
# Desde archivo .env
kubectl create configmap env-config --from-env-file=.env
```

### Usar ConfigMaps en Pods

#### Como Variables de Entorno

```yaml
# pod-con-configmap.yaml
apiVersion: v1
kind: Pod
metadata:
  name: webapp-pod
spec:
  containers:
  - name: webapp
    image: myapp:latest
    env:
    # Variable individual de ConfigMap
    - name: DATABASE_HOST
      valueFrom:
        configMapKeyRef:
          name: app-config
          key: database.host
    
    # Todas las variables del ConfigMap
    envFrom:
    - configMapRef:
        name: app-config
    
    # Con prefijo
    envFrom:
    - configMapRef:
        name: db-config
      prefix: DB_
```

#### Como Volúmenes

```yaml
# deployment-con-volumen-configmap.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: webapp-deployment
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
        volumeMounts:
        # Montar ConfigMap completo
        - name: config-volume
          mountPath: /etc/config
          readOnly: true
        
        # Montar archivo específico
        - name: nginx-config
          mountPath: /etc/nginx/nginx.conf
          subPath: nginx.conf
          readOnly: true
        
        # Montar con permisos específicos
        - name: app-config
          mountPath: /app/config
          readOnly: true
      
      volumes:
      # ConfigMap completo
      - name: config-volume
        configMap:
          name: app-config
      
      # Archivo específico
      - name: nginx-config
        configMap:
          name: nginx-config
          items:
          - key: nginx.conf
            path: nginx.conf
            mode: 0644
      
      # Con permisos por defecto
      - name: app-config
        configMap:
          name: app-config
          defaultMode: 0755
```

### ConfigMap Avanzado

```yaml
# configmap-avanzado.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: webapp-config
  labels:
    app: webapp
    environment: production
data:
  # Configuración de la aplicación
  application.yaml: |
    spring:
      datasource:
        url: jdbc:mysql://mysql:3306/webapp
        username: webapp_user
        driver-class-name: com.mysql.cj.jdbc.Driver
      jpa:
        hibernate:
          ddl-auto: validate
        show-sql: false
      redis:
        host: redis
        port: 6379
        timeout: 2000ms
    
    logging:
      level:
        com.example: INFO
        org.springframework: WARN
      pattern:
        console: "%d{yyyy-MM-dd HH:mm:ss} - %msg%n"
    
    management:
      endpoints:
        web:
          exposure:
            include: health,metrics,info
  
  # Configuración de Nginx
  nginx.conf: |
    events {
        worker_connections 1024;
    }
    
    http {
        upstream backend {
            server webapp:8080;
        }
        
        server {
            listen 80;
            
            location / {
                proxy_pass http://backend;
                proxy_set_header Host $host;
                proxy_set_header X-Real-IP $remote_addr;
                proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            }
            
            location /health {
                access_log off;
                return 200 "healthy\n";
            }
        }
    }
  
  # Script de inicialización
  init.sh: |
    #!/bin/bash
    echo "Iniciando aplicación..."
    
    # Verificar conexión a base de datos
    until mysqladmin ping -h mysql --silent; do
        echo "Esperando MySQL..."
        sleep 2
    done
    
    echo "MySQL disponible, iniciando aplicación"
    exec "$@"
```

## 🔐 Secrets - Información Sensible

### ¿Qué es un Secret?

Un **Secret** almacena información sensible como contraseñas, tokens OAuth, claves SSH, etc.

```yaml
# secret-basic.yaml
apiVersion: v1
kind: Secret
metadata:
  name: db-secret
type: Opaque
data:
  # Valores en base64
  username: YWRtaW4=        # admin
  password: MWYyZDFlMmU2N2Rm # 1f2d1e2e67df
```

### Tipos de Secrets

#### 1. Opaque (Generic)

```bash
# Crear secret opaque
kubectl create secret generic db-secret \
  --from-literal=username=admin \
  --from-literal=password=secret123

# Desde archivos
kubectl create secret generic ssl-certs \
  --from-file=tls.crt=server.crt \
  --from-file=tls.key=server.key
```

#### 2. TLS Certificates

```bash
# Crear secret TLS
kubectl create secret tls webapp-tls \
  --cert=path/to/tls.crt \
  --key=path/to/tls.key
```

```yaml
# secret-tls.yaml
apiVersion: v1
kind: Secret
metadata:
  name: webapp-tls
type: kubernetes.io/tls
data:
  tls.crt: LS0tLS1CRUdJTi... # certificado base64
  tls.key: LS0tLS1CRUdJTi... # clave privada base64
```

#### 3. Docker Registry

```bash
# Crear secret para registry privado
kubectl create secret docker-registry regcred \
  --docker-server=myregistry.com \
  --docker-username=myuser \
  --docker-password=mypassword \
  --docker-email=my@email.com
```

```yaml
# secret-docker-registry.yaml
apiVersion: v1
kind: Secret
metadata:
  name: regcred
type: kubernetes.io/dockerconfigjson
data:
  .dockerconfigjson: eyJhdXRoc... # config JSON base64
```

#### 4. Service Account Token

```yaml
# secret-service-account.yaml
apiVersion: v1
kind: Secret
metadata:
  name: mysecretname
  annotations:
    kubernetes.io/service-account.name: myserviceaccount
type: kubernetes.io/service-account-token
```

### Usar Secrets en Pods

#### Como Variables de Entorno

```yaml
# pod-con-secrets.yaml
apiVersion: v1
kind: Pod
metadata:
  name: webapp-pod
spec:
  containers:
  - name: webapp
    image: myapp:latest
    env:
    # Variable individual del Secret
    - name: DB_USERNAME
      valueFrom:
        secretKeyRef:
          name: db-secret
          key: username
    
    - name: DB_PASSWORD
      valueFrom:
        secretKeyRef:
          name: db-secret
          key: password
    
    # Todas las variables del Secret
    envFrom:
    - secretRef:
        name: db-secret
```

#### Como Volúmenes

```yaml
# deployment-con-secret-volumes.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: webapp-with-secrets
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
        # Secret como archivos
        - name: db-credentials
          mountPath: /app/secrets
          readOnly: true
        
        # Certificados TLS
        - name: tls-certs
          mountPath: /etc/ssl/certs
          readOnly: true
        
        # Archivo específico del secret
        - name: api-key
          mountPath: /app/config/api.key
          subPath: api-key
          readOnly: true
      
      volumes:
      # Secret completo
      - name: db-credentials
        secret:
          secretName: db-secret
          defaultMode: 0400
      
      # Certificados TLS
      - name: tls-certs
        secret:
          secretName: webapp-tls
      
      # Archivo específico
      - name: api-key
        secret:
          secretName: api-secrets
          items:
          - key: api-key
            path: api.key
            mode: 0600
      
      # Para usar Docker registry privado
      imagePullSecrets:
      - name: regcred
```

### Secret con Múltiples Tipos de Datos

```yaml
# secret-completo.yaml
apiVersion: v1
kind: Secret
metadata:
  name: webapp-secrets
  labels:
    app: webapp
type: Opaque
data:
  # Credenciales de base de datos
  db-username: d2ViYXBwX3VzZXI=    # webapp_user
  db-password: c3VwZXJzZWNyZXQ=    # supersecret
  
  # API Keys
  github-token: Z2hwX3h4eHh4eHh4  # ghp_xxxxxxxx
  stripe-key: c2tfdGVzdF94eHh4    # sk_test_xxxx
  
  # Certificados
  ca.crt: LS0tLS1CRUdJTi...
  client.crt: LS0tLS1CRUdJTi...
  client.key: LS0tLS1CRUdJTi...
  
  # Configuración SSH
  ssh-privatekey: LS0tLS1CRUdJTi...
  ssh-publickey: c3NoLXJzYSBBQUFB...
  
  # Configuración JSON
  service-account.json: eyJhbGciOiJSUzI1...
```

## 🛡️ Buenas Prácticas de Seguridad

### 1. Principio de Menor Privilegio

```yaml
# Usar ServiceAccount específico
apiVersion: v1
kind: ServiceAccount
metadata:
  name: webapp-sa
  namespace: production
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: webapp
spec:
  template:
    spec:
      serviceAccountName: webapp-sa
      securityContext:
        runAsNonRoot: true
        runAsUser: 1000
        fsGroup: 2000
      containers:
      - name: webapp
        image: myapp:latest
        securityContext:
          allowPrivilegeEscalation: false
          readOnlyRootFilesystem: true
          capabilities:
            drop:
            - ALL
```

### 2. Encriptación en Reposo

```yaml
# Habilitar encriptación de etcd (a nivel de cluster)
apiVersion: apiserver.k8s.io/v1
kind: EncryptionConfiguration
resources:
- resources:
  - secrets
  providers:
  - aescbc:
      keys:
      - name: key1
        secret: <32-byte base64-encoded key>
  - identity: {}
```

### 3. RBAC para Secrets

```yaml
# role-secrets.yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: production
  name: secret-reader
rules:
- apiGroups: [""]
  resources: ["secrets"]
  verbs: ["get", "list"]
  resourceNames: ["db-secret", "api-secret"]
---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: secret-reader-binding
  namespace: production
subjects:
- kind: ServiceAccount
  name: webapp-sa
  namespace: production
roleRef:
  kind: Role
  name: secret-reader
  apiGroup: rbac.authorization.k8s.io
```

### 4. Secrets Inmutables

```yaml
# secret-inmutable.yaml
apiVersion: v1
kind: Secret
metadata:
  name: immutable-secret
type: Opaque
immutable: true  # No se puede modificar
data:
  api-key: YWJjZGVmZ2hpamtsbW5vcA==
```

## 🔧 Herramientas Avanzadas de Gestión

### 1. External Secrets Operator

```yaml
# Instalar External Secrets Operator
# helm install external-secrets external-secrets/external-secrets -n external-secrets-system --create-namespace

# external-secret.yaml
apiVersion: external-secrets.io/v1beta1
kind: ExternalSecret
metadata:
  name: vault-secret
spec:
  refreshInterval: 15s
  secretStoreRef:
    name: vault-backend
    kind: SecretStore
  target:
    name: vault-secret
    creationPolicy: Owner
  data:
  - secretKey: password
    remoteRef:
      key: secret/database
      property: password
---
apiVersion: external-secrets.io/v1beta1
kind: SecretStore
metadata:
  name: vault-backend
spec:
  provider:
    vault:
      server: "https://vault.example.com"
      path: "secret"
      version: "v2"
      auth:
        kubernetes:
          mountPath: "kubernetes"
          role: "webapp"
```

### 2. Sealed Secrets

```bash
# Instalar Sealed Secrets Controller
kubectl apply -f https://github.com/bitnami-labs/sealed-secrets/releases/download/v0.18.0/controller.yaml

# Instalar kubeseal CLI
wget https://github.com/bitnami-labs/sealed-secrets/releases/download/v0.18.0/kubeseal-linux-amd64 -O kubeseal
chmod +x kubeseal
sudo mv kubeseal /usr/local/bin/
```

```yaml
# Crear secret normal
apiVersion: v1
kind: Secret
metadata:
  name: mysecret
  namespace: mynamespace
data:
  username: YWRtaW4=
  password: MWYyZDFlMmU2N2Rm
```

```bash
# Convertir a SealedSecret
kubeseal -o yaml < secret.yaml > sealed-secret.yaml
```

```yaml
# sealed-secret.yaml (resultado)
apiVersion: bitnami.com/v1alpha1
kind: SealedSecret
metadata:
  name: mysecret
  namespace: mynamespace
spec:
  encryptedData:
    username: AgBy3i4OJSWK+PiTySYZZA9rO43cGDEQAx...
    password: AgBy3i4OJSWK+PiTySYZZA9rO43cGDEQAx...
  template:
    metadata:
      name: mysecret
      namespace: mynamespace
```

### 3. Bank-Vaults

```yaml
# vault-secret.yaml
apiVersion: v1
kind: Secret
metadata:
  name: vault-secret
  annotations:
    vault.security.banzaicloud.io/vault-addr: "https://vault.example.com"
    vault.security.banzaicloud.io/vault-role: "webapp"
    vault.security.banzaicloud.io/vault-path: "kubernetes"
type: Opaque
data:
  # El valor será inyectado por Bank-Vaults
  password: vault:secret/data/database#password
  username: vault:secret/data/database#username
```

## 🔄 Actualización de ConfigMaps y Secrets

### Hot Reload con Reloader

```bash
# Instalar Reloader
kubectl apply -f https://raw.githubusercontent.com/stakater/Reloader/master/deployments/kubernetes/reloader.yaml
```

```yaml
# deployment-con-reloader.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: webapp
  annotations:
    reloader.stakater.com/auto: "true"
    # O específico:
    # reloader.stakater.com/search: "true"
    # configmap.reloader.stakater.com/reload: "app-config"
    # secret.reloader.stakater.com/reload: "db-secret"
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
        envFrom:
        - configMapRef:
            name: app-config
        - secretRef:
            name: db-secret
```

### Rolling Update Manual

```bash
# Actualizar ConfigMap
kubectl patch configmap app-config -p '{"data":{"log.level":"debug"}}'

# Forzar rolling update del deployment
kubectl rollout restart deployment/webapp

# Verificar estado
kubectl rollout status deployment/webapp
```

## 🛠️ Ejercicio Práctico: Stack Completo

### 1. ConfigMaps

```yaml
# configmaps.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: webapp-config
data:
  database.host: "mysql"
  database.port: "3306"
  database.name: "webapp"
  redis.host: "redis"
  redis.port: "6379"
  log.level: "info"
  
  application.yaml: |
    server:
      port: 8080
    spring:
      datasource:
        url: jdbc:mysql://${DATABASE_HOST}:${DATABASE_PORT}/${DATABASE_NAME}
        username: ${DATABASE_USERNAME}
        password: ${DATABASE_PASSWORD}
      redis:
        host: ${REDIS_HOST}
        port: ${REDIS_PORT}
    logging:
      level:
        com.example: ${LOG_LEVEL}
---
apiVersion: v1
kind: ConfigMap
metadata:
  name: nginx-config
data:
  nginx.conf: |
    events {
        worker_connections 1024;
    }
    http {
        upstream webapp {
            server webapp:8080;
        }
        server {
            listen 80;
            location / {
                proxy_pass http://webapp;
                proxy_set_header Host $host;
            }
            location /health {
                return 200 "OK";
            }
        }
    }
```

### 2. Secrets

```bash
# Crear secrets
kubectl create secret generic db-secret \
  --from-literal=username=webapp_user \
  --from-literal=password=supersecret123

kubectl create secret generic api-secrets \
  --from-literal=jwt-secret=jwt-super-secret-key \
  --from-literal=api-key=api-key-12345

kubectl create secret docker-registry regcred \
  --docker-server=myregistry.com \
  --docker-username=myuser \
  --docker-password=mypass \
  --docker-email=my@email.com
```

### 3. Deployment con Configuración

```yaml
# webapp-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: webapp
  annotations:
    reloader.stakater.com/auto: "true"
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
        image: myregistry.com/webapp:v1.0
        ports:
        - containerPort: 8080
        env:
        # Desde ConfigMap
        - name: DATABASE_HOST
          valueFrom:
            configMapKeyRef:
              name: webapp-config
              key: database.host
        - name: DATABASE_PORT
          valueFrom:
            configMapKeyRef:
              name: webapp-config
              key: database.port
        - name: DATABASE_NAME
          valueFrom:
            configMapKeyRef:
              name: webapp-config
              key: database.name
        
        # Desde Secrets
        - name: DATABASE_USERNAME
          valueFrom:
            secretKeyRef:
              name: db-secret
              key: username
        - name: DATABASE_PASSWORD
          valueFrom:
            secretKeyRef:
              name: db-secret
              key: password
        - name: JWT_SECRET
          valueFrom:
            secretKeyRef:
              name: api-secrets
              key: jwt-secret
        
        volumeMounts:
        # ConfigMap como archivo
        - name: app-config
          mountPath: /app/config
          readOnly: true
        
        # Secret como archivo
        - name: api-credentials
          mountPath: /app/secrets
          readOnly: true
        
        livenessProbe:
          httpGet:
            path: /health
            port: 8080
          initialDelaySeconds: 30
          periodSeconds: 10
        
        readinessProbe:
          httpGet:
            path: /ready
            port: 8080
          initialDelaySeconds: 5
          periodSeconds: 5
      
      volumes:
      - name: app-config
        configMap:
          name: webapp-config
      - name: api-credentials
        secret:
          secretName: api-secrets
          defaultMode: 0400
      
      imagePullSecrets:
      - name: regcred
```

### 4. Verificación

```bash
# Aplicar configuración
kubectl apply -f configmaps.yaml
kubectl apply -f webapp-deployment.yaml

# Verificar configuración
kubectl get configmaps
kubectl get secrets

# Ver variables de entorno en el pod
kubectl exec -it <pod-name> -- env | grep DATABASE

# Ver archivos montados
kubectl exec -it <pod-name> -- ls -la /app/config
kubectl exec -it <pod-name> -- ls -la /app/secrets

# Ver contenido de configuración
kubectl exec -it <pod-name> -- cat /app/config/application.yaml
```

## 📝 Mejores Prácticas

### ConfigMaps

1. **Separa configuración por entorno** (dev, staging, prod)
2. **Usa nombres descriptivos** para las claves
3. **Documenta la configuración** con comentarios
4. **Valida configuración** antes del despliegue
5. **Versionado** de ConfigMaps con labels

### Secrets

1. **Nunca hardcodees secrets** en imágenes
2. **Usa RBAC** para controlar acceso
3. **Rota secrets regularmente**
4. **Implementa encriptación** en reposo
5. **Audita acceso** a secrets

### Generales

1. **Principio de menor privilegio**
2. **Monitoreo de cambios** en configuración
3. **Backup** de configuración crítica
4. **Testing** de configuración en staging
5. **Documentación** de configuración

## 🎯 Resumen

Has aprendido:
- [x] Gestión de configuración con ConfigMaps
- [x] Almacenamiento seguro con Secrets
- [x] Técnicas de montaje de volúmenes
- [x] Herramientas avanzadas de gestión
- [x] Mejores prácticas de seguridad

## 📚 Recursos Adicionales

- [ConfigMaps Documentation](https://kubernetes.io/docs/concepts/configuration/configmap/)
- [Secrets Documentation](https://kubernetes.io/docs/concepts/configuration/secret/)
- [External Secrets Operator](https://external-secrets.io/)
- [Sealed Secrets](https://github.com/bitnami-labs/sealed-secrets)
- [Bank-Vaults](https://github.com/banzaicloud/bank-vaults)

---

**Próximo tema:** [Escalado y balanceo de carga](./08-escalado-balanceo.md)