# 06. Estrategias de Deployment Avanzadas

**Domina estrategias de deployment de nivel empresarial!**Este módulo te enseñará técnicas avanzadas para deployments seguros, escalables y sin downtime.

## Objetivos de Aprendizaje

Al completar este módulo serás capaz de:

- **Implementar Blue-Green deployments**para zero downtime
- **Configurar Canary deployments**para releases seguras
- **Usar Rolling deployments**efectivamente
- **Implementar Feature Flags**y A/B testing
- **Configurar GitOps**workflows completos
- **Gestionar rollbacks**automáticos
- **Monitorear deployments**en tiempo real

## Contenido del Módulo

### 1. Blue-Green Deployment

#### **Implementación con Kubernetes**

```yaml
# blue-green-deployment.yaml
apiVersion: v1
kind: Service
metadata:
  name: myapp-service
  labels:
    app: myapp
spec:
  selector:
    app: myapp
    version: blue  # Cambia entre 'blue' y 'green'
  ports:
    - port: 80
      targetPort: 3000
  type: LoadBalancer

---
# Blue Deployment
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp-blue
  labels:
    app: myapp
    version: blue
spec:
  replicas: 3
  selector:
    matchLabels:
      app: myapp
      version: blue
  template:
    metadata:
      labels:
        app: myapp
        version: blue
    spec:
      containers:
      - name: myapp
        image: myapp:v1.0.0
        ports:
        - containerPort: 3000
        env:
        - name: APP_VERSION
          value: "v1.0.0-blue"
        readinessProbe:
          httpGet:
            path: /health
            port: 3000
          initialDelaySeconds: 10
          periodSeconds: 5
        livenessProbe:
          httpGet:
            path: /health
            port: 3000
          initialDelaySeconds: 30
          periodSeconds: 10

---
# Green Deployment
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp-green
  labels:
    app: myapp
    version: green
spec:
  replicas: 3
  selector:
    matchLabels:
      app: myapp
      version: green
  template:
    metadata:
      labels:
        app: myapp
        version: green
    spec:
      containers:
      - name: myapp
        image: myapp:v2.0.0
        ports:
        - containerPort: 3000
        env:
        - name: APP_VERSION
          value: "v2.0.0-green"
        readinessProbe:
          httpGet:
            path: /health
            port: 3000
          initialDelaySeconds: 10
          periodSeconds: 5
        livenessProbe:
          httpGet:
            path: /health
            port: 3000
          initialDelaySeconds: 30
          periodSeconds: 10
```

```bash
#!/bin/bash
# blue-green-deploy.sh

APP_NAME="myapp"
NEW_VERSION=$1
NAMESPACE=${NAMESPACE:-default}

if [ -z "$NEW_VERSION" ]; then
    echo "Usage: $0 <new-version>"
    exit 1
fi

echo " Starting Blue-Green deployment for $APP_NAME:$NEW_VERSION"

# Determinar el color activo actual
CURRENT_COLOR=$(kubectl get service $APP_NAME-service -n $NAMESPACE -o jsonpath='{.spec.selector.version}')
if [ "$CURRENT_COLOR" = "blue" ]; then
    TARGET_COLOR="green"
else
    TARGET_COLOR="blue"
fi

echo " Current active color: $CURRENT_COLOR"
echo " Target color: $TARGET_COLOR"

# Actualizar el deployment inactivo
echo " Updating $TARGET_COLOR deployment with version $NEW_VERSION"
kubectl set image deployment/$APP_NAME-$TARGET_COLOR \
    $APP_NAME=myapp:$NEW_VERSION \
    -n $NAMESPACE

# Esperar a que el deployment esté listo
echo "⏳ Waiting for $TARGET_COLOR deployment to be ready..."
kubectl rollout status deployment/$APP_NAME-$TARGET_COLOR -n $NAMESPACE --timeout=300s

if [ $? -ne 0 ]; then
    echo " Deployment failed! Rolling back..."
    exit 1
fi

# Verificar health checks
echo " Running health checks on $TARGET_COLOR environment..."
TARGET_POD=$(kubectl get pods -n $NAMESPACE -l app=$APP_NAME,version=$TARGET_COLOR -o jsonpath='{.items[0].metadata.name}')

for i in {1..10}; do
    if kubectl exec -n $NAMESPACE $TARGET_POD -- curl -f http://localhost:3000/health; then
        echo " Health check passed"
        break
    fi

    if [ $i -eq 10 ]; then
        echo " Health checks failed after 10 attempts"
        exit 1
    fi

    echo "⏳ Health check attempt $i failed, retrying..."
    sleep 10
done

# Ejecutar smoke tests
echo " Running smoke tests..."
kubectl exec -n $NAMESPACE $TARGET_POD -- npm run test:smoke

if [ $? -ne 0 ]; then
    echo " Smoke tests failed!"
    exit 1
fi

# Cambiar el tráfico al nuevo color
echo " Switching traffic to $TARGET_COLOR"
kubectl patch service $APP_NAME-service -n $NAMESPACE -p '{"spec":{"selector":{"version":"'$TARGET_COLOR'"}}}'

# Verificar que el tráfico se está sirviendo correctamente
echo " Verifying traffic switch..."
sleep 30

# Obtener la IP del servicio
SERVICE_IP=$(kubectl get service $APP_NAME-service -n $NAMESPACE -o jsonpath='{.status.loadBalancer.ingress[0].ip}')

for i in {1..5}; do
    RESPONSE=$(curl -s http://$SERVICE_IP/version)
    if echo "$RESPONSE" | grep -q "$NEW_VERSION-$TARGET_COLOR"; then
        echo " Traffic successfully switched to $TARGET_COLOR"
        break
    fi

    if [ $i -eq 5 ]; then
        echo " Traffic switch verification failed"
        # Rollback
        kubectl patch service $APP_NAME-service -n $NAMESPACE -p '{"spec":{"selector":{"version":"'$CURRENT_COLOR'"}}}'
        exit 1
    fi

    sleep 10
done

# Escalar down el deployment anterior
echo " Scaling down $CURRENT_COLOR deployment"
kubectl scale deployment $APP_NAME-$CURRENT_COLOR --replicas=0 -n $NAMESPACE

echo " Blue-Green deployment completed successfully!"
echo " Monitor the application and run the following to complete the cleanup:"
echo "kubectl delete deployment $APP_NAME-$CURRENT_COLOR -n $NAMESPACE"
```

#### **Blue-Green con AWS ECS**

```json
{
  "family": "myapp-task-definition",
  "networkMode": "awsvpc",
  "requiresCompatibilities": ["FARGATE"],
  "cpu": "256",
  "memory": "512",
  "executionRoleArn": "arn:aws:iam::123456789012:role/ecsTaskExecutionRole",
  "containerDefinitions": [
    {
      "name": "myapp",
      "image": "myregistry/myapp:${IMAGE_TAG}",
      "portMappings": [
        {
          "containerPort": 3000,
          "protocol": "tcp"
        }
      ],
      "healthCheck": {
        "command": [
          "CMD-SHELL",
          "curl -f http://localhost:3000/health || exit 1"
        ],
        "interval": 30,
        "timeout": 5,
        "retries": 3,
        "startPeriod": 60
      },
      "logConfiguration": {
        "logDriver": "awslogs",
        "options": {
          "awslogs-group": "/ecs/myapp",
          "awslogs-region": "us-east-1",
          "awslogs-stream-prefix": "ecs"
        }
      }
    }
  ]
}
```

```bash
#!/bin/bash
# ecs-blue-green-deploy.sh

CLUSTER_NAME="myapp-cluster"
SERVICE_NAME="myapp-service"
NEW_IMAGE_TAG=$1
REGION="us-east-1"

if [ -z "$NEW_IMAGE_TAG" ]; then
    echo "Usage: $0 <new-image-tag>"
    exit 1
fi

echo " Starting ECS Blue-Green deployment"

# Obtener la task definition actual
CURRENT_TASK_DEF=$(aws ecs describe-services \
    --cluster $CLUSTER_NAME \
    --services $SERVICE_NAME \
    --region $REGION \
    --query 'services[0].taskDefinition' \
    --output text)

echo " Current task definition: $CURRENT_TASK_DEF"

# Crear nueva task definition
NEW_TASK_DEF_JSON=$(aws ecs describe-task-definition \
    --task-definition $CURRENT_TASK_DEF \
    --region $REGION \
    --query 'taskDefinition' \
    --output json | \
    jq --arg tag "$NEW_IMAGE_TAG" '.containerDefinitions[0].image = "myregistry/myapp:" + $tag' | \
    jq 'del(.taskDefinitionArn, .revision, .status, .requiresAttributes, .placementConstraints, .compatibilities, .registeredAt, .registeredBy)')

# Registrar nueva task definition
NEW_TASK_DEF_ARN=$(echo "$NEW_TASK_DEF_JSON" | \
    aws ecs register-task-definition \
    --region $REGION \
    --cli-input-json file:///dev/stdin \
    --query 'taskDefinition.taskDefinitionArn' \
    --output text)

echo " New task definition: $NEW_TASK_DEF_ARN"

# Crear un nuevo servicio temporal (Green)
GREEN_SERVICE_NAME="${SERVICE_NAME}-green"

aws ecs create-service \
    --cluster $CLUSTER_NAME \
    --service-name $GREEN_SERVICE_NAME \
    --task-definition $NEW_TASK_DEF_ARN \
    --desired-count 2 \
    --launch-type FARGATE \
    --network-configuration "awsvpcConfiguration={subnets=[subnet-12345,subnet-67890],securityGroups=[sg-abcdef],assignPublicIp=ENABLED}" \
    --region $REGION

# Esperar a que el servicio Green esté estable
echo "⏳ Waiting for Green service to be stable..."
aws ecs wait services-stable \
    --cluster $CLUSTER_NAME \
    --services $GREEN_SERVICE_NAME \
    --region $REGION

# Ejecutar health checks
echo " Running health checks..."
GREEN_TASK_ARN=$(aws ecs list-tasks \
    --cluster $CLUSTER_NAME \
    --service-name $GREEN_SERVICE_NAME \
    --region $REGION \
    --query 'taskArns[0]' \
    --output text)

# Aquí ejecutarías tus health checks específicos
# Por ejemplo, usando AWS CLI para ejecutar comandos en el contenedor

# Actualizar el Target Group del ALB para apuntar al servicio Green
# (Esto requiere configuración previa del ALB y Target Groups)

echo " Switching ALB traffic to Green service"
# aws elbv2 modify-listener --listener-arn $LISTENER_ARN --default-actions ...

# Verificar el tráfico
echo " Verifying traffic switch..."
sleep 60

# Si todo está bien, eliminar el servicio Blue
echo " Cleaning up Blue service"
aws ecs update-service \
    --cluster $CLUSTER_NAME \
    --service $SERVICE_NAME \
    --desired-count 0 \
    --region $REGION

aws ecs delete-service \
    --cluster $CLUSTER_NAME \
    --service $SERVICE_NAME \
    --region $REGION

# Renombrar el servicio Green como el principal
echo " Blue-Green deployment completed successfully!"
```

### 2. Canary Deployment

#### **Canary con Istio**

```yaml
# canary-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp-v2
  labels:
    app: myapp
    version: v2
spec:
  replicas: 1  # Comenzar con pocas réplicas
  selector:
    matchLabels:
      app: myapp
      version: v2
  template:
    metadata:
      labels:
        app: myapp
        version: v2
    spec:
      containers:
      - name: myapp
        image: myapp:v2.0.0
        ports:
        - containerPort: 3000

---
apiVersion: networking.istio.io/v1beta1
kind: VirtualService
metadata:
  name: myapp-virtual-service
spec:
  http:
  - match:
    - headers:
        canary:
          exact: "true"
    route:
    - destination:
        host: myapp-service
        subset: v2
  - route:
    - destination:
        host: myapp-service
        subset: v1
      weight: 90
    - destination:
        host: myapp-service
        subset: v2
      weight: 10  # 10% del tráfico a la nueva versión

---
apiVersion: networking.istio.io/v1beta1
kind: DestinationRule
metadata:
  name: myapp-destination-rule
spec:
  host: myapp-service
  subsets:
  - name: v1
    labels:
      version: v1
  - name: v2
    labels:
      version: v2
```

```bash
#!/bin/bash
# canary-deploy.sh

APP_NAME="myapp"
NEW_VERSION=$1
NAMESPACE=${NAMESPACE:-default}

if [ -z "$NEW_VERSION" ]; then
    echo "Usage: $0 <new-version>"
    exit 1
fi

echo " Starting Canary deployment for $APP_NAME:$NEW_VERSION"

# Desplegar la nueva versión con 1 réplica
echo " Deploying canary version..."
kubectl apply -f - <<EOF
apiVersion: apps/v1
kind: Deployment
metadata:
  name: $APP_NAME-v2
  namespace: $NAMESPACE
  labels:
    app: $APP_NAME
    version: v2
spec:
  replicas: 1
  selector:
    matchLabels:
      app: $APP_NAME
      version: v2
  template:
    metadata:
      labels:
        app: $APP_NAME
        version: v2
    spec:
      containers:
      - name: $APP_NAME
        image: $APP_NAME:$NEW_VERSION
        ports:
        - containerPort: 3000
        readinessProbe:
          httpGet:
            path: /health
            port: 3000
          initialDelaySeconds: 10
          periodSeconds: 5
EOF

# Esperar a que el canary esté listo
echo "⏳ Waiting for canary deployment..."
kubectl rollout status deployment/$APP_NAME-v2 -n $NAMESPACE

# Configurar tráfico canary (5%)
echo " Configuring 5% canary traffic..."
kubectl apply -f - <<EOF
apiVersion: networking.istio.io/v1beta1
kind: VirtualService
metadata:
  name: $APP_NAME-vs
  namespace: $NAMESPACE
spec:
  http:
  - route:
    - destination:
        host: $APP_NAME-service
        subset: v1
      weight: 95
    - destination:
        host: $APP_NAME-service
        subset: v2
      weight: 5
EOF

# Monitorear métricas por 5 minutos
echo " Monitoring canary metrics for 5 minutes..."
monitor_canary() {
    for i in {1..10}; do
        echo " Monitoring iteration $i/10"

        # Obtener métricas de error rate
        ERROR_RATE=$(kubectl exec -n istio-system deployment/prometheus -- \
            promtool query instant \
            'rate(istio_request_total{destination_service_name="'$APP_NAME'",destination_version="v2",response_code!~"2.*"}[1m]) / rate(istio_request_total{destination_service_name="'$APP_NAME'",destination_version="v2"}[1m])' | \
            tail -1 | awk '{print $2}')

        # Verificar si error rate es aceptable (< 1%)
        if (( $(echo "$ERROR_RATE > 0.01" | bc -l) )); then
            echo " Error rate too high: $ERROR_RATE"
            rollback_canary
            exit 1
        fi

        # Obtener métricas de latencia
        P99_LATENCY=$(kubectl exec -n istio-system deployment/prometheus -- \
            promtool query instant \
            'histogram_quantile(0.99, rate(istio_request_duration_milliseconds_bucket{destination_service_name="'$APP_NAME'",destination_version="v2"}[1m]))' | \
            tail -1 | awk '{print $2}')

        # Verificar si latencia es aceptable (< 500ms)
        if (( $(echo "$P99_LATENCY > 500" | bc -l) )); then
            echo " P99 latency too high: ${P99_LATENCY}ms"
            rollback_canary
            exit 1
        fi

        echo " Metrics OK - Error rate: $ERROR_RATE, P99: ${P99_LATENCY}ms"
        sleep 30
    done
}

rollback_canary() {
    echo " Rolling back canary deployment..."
    kubectl delete deployment $APP_NAME-v2 -n $NAMESPACE
    kubectl patch virtualservice $APP_NAME-vs -n $NAMESPACE --type='json' -p='[{"op": "replace", "path": "/spec/http/0/route", "value": [{"destination": {"host": "'$APP_NAME'-service", "subset": "v1"}, "weight": 100}]}]'
}

promote_canary() {
    echo " Promoting canary to 50% traffic..."
    kubectl patch virtualservice $APP_NAME-vs -n $NAMESPACE --type='json' -p='[{"op": "replace", "path": "/spec/http/0/route", "value": [{"destination": {"host": "'$APP_NAME'-service", "subset": "v1"}, "weight": 50}, {"destination": {"host": "'$APP_NAME'-service", "subset": "v2"}, "weight": 50}]}]'

    # Escalar el canary
    kubectl scale deployment $APP_NAME-v2 --replicas=3 -n $NAMESPACE

    # Monitorear por otros 5 minutos
    sleep 300

    echo " Promoting canary to 100% traffic..."
    kubectl patch virtualservice $APP_NAME-vs -n $NAMESPACE --type='json' -p='[{"op": "replace", "path": "/spec/http/0/route", "value": [{"destination": {"host": "'$APP_NAME'-service", "subset": "v2"}, "weight": 100}]}]'

    # Limpiar la versión anterior
    kubectl delete deployment $APP_NAME-v1 -n $NAMESPACE
    kubectl patch deployment $APP_NAME-v2 -n $NAMESPACE --type='json' -p='[{"op": "replace", "path": "/metadata/name", "value": "'$APP_NAME'"}]'
}

# Ejecutar monitoreo
monitor_canary

# Si llegamos aquí, el canary está funcionando bien
echo " Canary metrics look good, promoting..."
promote_canary

echo " Canary deployment completed successfully!"
```

#### **Canary Automatizado con Flagger**

```yaml
# flagger-canary.yaml
apiVersion: flagger.app/v1beta1
kind: Canary
metadata:
  name: myapp
  namespace: production
spec:
  # Deployment reference
  targetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: myapp

  # Progress deadline
  progressDeadlineSeconds: 60

  # HPA reference (optional)
  autoscalerRef:
    apiVersion: autoscaling/v2beta2
    kind: HorizontalPodAutoscaler
    name: myapp

  service:
    # Service port number
    port: 80
    # Container port number or name
    targetPort: 3000
    # Istio gateways (optional)
    gateways:
    - myapp-gateway
    # Istio virtual service host names (optional)
    hosts:
    - myapp.example.com

  analysis:
    # Schedule interval
    interval: 1m
    # Max number of failed metric checks before rollback
    threshold: 5
    # Max traffic percentage routed to canary
    maxWeight: 50
    # Canary increment step
    stepWeight: 10
    # Istio Prometheus checks
    metrics:
    - name: request-success-rate
      # Minimum req success rate (non 5xx responses)
      thresholdRange:
        min: 99
      interval: 1m
    - name: request-duration
      # Maximum req duration P99
      thresholdRange:
        max: 500
      interval: 30s
    # Testing (optional)
    webhooks:
      - name: acceptance-test
        type: pre-rollout
        url: http://flagger-loadtester.test/
        timeout: 30s
        metadata:
          type: bash
          cmd: "curl -sd 'test' http://myapp-canary/token | grep token"
      - name: load-test
        url: http://flagger-loadtester.test/
        timeout: 5s
        metadata:
          cmd: "hey -z 1m -q 10 -c 2 http://myapp-canary/"
```

### 3. Rolling Deployment

#### **Rolling Update con Kubernetes**

```yaml
# rolling-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp
  labels:
    app: myapp
spec:
  replicas: 6
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxUnavailable: 1      # Máximo 1 pod no disponible
      maxSurge: 2           # Máximo 2 pods adicionales
  selector:
    matchLabels:
      app: myapp
  template:
    metadata:
      labels:
        app: myapp
    spec:
      containers:
      - name: myapp
        image: myapp:v2.0.0
        ports:
        - containerPort: 3000
        readinessProbe:
          httpGet:
            path: /health
            port: 3000
          initialDelaySeconds: 10
          periodSeconds: 5
          successThreshold: 1
          failureThreshold: 3
        livenessProbe:
          httpGet:
            path: /health
            port: 3000
          initialDelaySeconds: 30
          periodSeconds: 10
          successThreshold: 1
          failureThreshold: 3
        lifecycle:
          preStop:
            exec:
              command: ["/bin/sh", "-c", "sleep 15"]  # Graceful shutdown
        resources:
          requests:
            memory: "128Mi"
            cpu: "100m"
          limits:
            memory: "256Mi"
            cpu: "200m"
      terminationGracePeriodSeconds: 30
```

```bash
#!/bin/bash
# rolling-deploy.sh

APP_NAME="myapp"
NEW_IMAGE=$1
NAMESPACE=${NAMESPACE:-default}
TIMEOUT=${TIMEOUT:-300}

if [ -z "$NEW_IMAGE" ]; then
    echo "Usage: $0 <new-image>"
    exit 1
fi

echo " Starting Rolling deployment for $APP_NAME"
echo " New image: $NEW_IMAGE"

# Verificar que el deployment existe
if ! kubectl get deployment $APP_NAME -n $NAMESPACE &> /dev/null; then
    echo " Deployment $APP_NAME not found in namespace $NAMESPACE"
    exit 1
fi

# Guardar la imagen actual para rollback
CURRENT_IMAGE=$(kubectl get deployment $APP_NAME -n $NAMESPACE -o jsonpath='{.spec.template.spec.containers[0].image}')
echo " Current image: $CURRENT_IMAGE"

# Actualizar la imagen
echo " Updating deployment image..."
kubectl set image deployment/$APP_NAME $APP_NAME=$NEW_IMAGE -n $NAMESPACE

if [ $? -ne 0 ]; then
    echo " Failed to update deployment image"
    exit 1
fi

# Monitorear el rollout
echo "⏳ Monitoring rollout progress..."
kubectl rollout status deployment/$APP_NAME -n $NAMESPACE --timeout=${TIMEOUT}s

ROLLOUT_STATUS=$?

if [ $ROLLOUT_STATUS -ne 0 ]; then
    echo " Rollout failed or timed out"

    # Automatic rollback
    echo " Initiating automatic rollback..."
    kubectl rollout undo deployment/$APP_NAME -n $NAMESPACE

    echo "⏳ Waiting for rollback to complete..."
    kubectl rollout status deployment/$APP_NAME -n $NAMESPACE --timeout=${TIMEOUT}s

    if [ $? -eq 0 ]; then
        echo " Rollback completed successfully"
    else
        echo " Rollback failed"
    fi

    exit 1
fi

# Verificar que todos los pods están healthy
echo " Verifying pod health..."
READY_REPLICAS=$(kubectl get deployment $APP_NAME -n $NAMESPACE -o jsonpath='{.status.readyReplicas}')
DESIRED_REPLICAS=$(kubectl get deployment $APP_NAME -n $NAMESPACE -o jsonpath='{.spec.replicas}')

if [ "$READY_REPLICAS" != "$DESIRED_REPLICAS" ]; then
    echo " Not all replicas are ready: $READY_REPLICAS/$DESIRED_REPLICAS"
    exit 1
fi

# Ejecutar smoke tests
echo " Running smoke tests..."
run_smoke_tests() {
    SERVICE_URL=$(kubectl get service $APP_NAME -n $NAMESPACE -o jsonpath='{.status.loadBalancer.ingress[0].ip}')

    if [ -z "$SERVICE_URL" ]; then
        # Usar port-forward si no hay LoadBalancer
        kubectl port-forward service/$APP_NAME 8080:80 -n $NAMESPACE &
        PORT_FORWARD_PID=$!
        SERVICE_URL="localhost:8080"
        sleep 5
    fi

    # Test básico de conectividad
    for i in {1..5}; do
        if curl -f http://$SERVICE_URL/health &> /dev/null; then
            echo " Smoke test $i passed"
        else
            echo " Smoke test $i failed"

            # Cleanup port-forward if used
            if [ ! -z "$PORT_FORWARD_PID" ]; then
                kill $PORT_FORWARD_PID
            fi

            return 1
        fi
        sleep 2
    done

    # Cleanup port-forward if used
    if [ ! -z "$PORT_FORWARD_PID" ]; then
        kill $PORT_FORWARD_PID
    fi

    return 0
}

if ! run_smoke_tests; then
    echo " Smoke tests failed, rolling back..."
    kubectl rollout undo deployment/$APP_NAME -n $NAMESPACE
    kubectl rollout status deployment/$APP_NAME -n $NAMESPACE --timeout=${TIMEOUT}s
    exit 1
fi

echo " Rolling deployment completed successfully!"
echo " Deployment details:"
kubectl get deployment $APP_NAME -n $NAMESPACE
kubectl get pods -l app=$APP_NAME -n $NAMESPACE
```

### 4. Feature Flags y A/B Testing

#### **Implementación con LaunchDarkly**

```javascript
// feature-flags-service.js
import { LDClient } from 'launchdarkly-node-server-sdk'

class FeatureFlagService {
  constructor(sdkKey) {
    this.client = LDClient.init(sdkKey)
    this.isReady = false

    this.client.waitForInitialization().then(() => {
      this.isReady = true
      console.log(' LaunchDarkly client initialized')
    }).catch(error => {
      console.error(' LaunchDarkly initialization failed:', error)
    })
  }

  async isFeatureEnabled(flagKey, user, defaultValue = false) {
    if (!this.isReady) {
      console.warn(' LaunchDarkly client not ready, returning default value')
      return defaultValue
    }

    try {
      return await this.client.variation(flagKey, user, defaultValue)
    } catch (error) {
      console.error(` Error evaluating flag ${flagKey}:`, error)
      return defaultValue
    }
  }

  async getFeatureVariation(flagKey, user, defaultValue = 'control') {
    if (!this.isReady) {
      return defaultValue
    }

    try {
      return await this.client.variation(flagKey, user, defaultValue)
    } catch (error) {
      console.error(` Error getting variation for ${flagKey}:`, error)
      return defaultValue
    }
  }

  async getAllFlags(user) {
    if (!this.isReady) {
      return {}
    }

    try {
      return await this.client.allFlagsState(user)
    } catch (error) {
      console.error(' Error getting all flags:', error)
      return {}
    }
  }

  trackEvent(eventKey, user, data = null, metricValue = null) {
    if (!this.isReady) {
      console.warn(' LaunchDarkly client not ready, event not tracked')
      return
    }

    try {
      this.client.track(eventKey, user, data, metricValue)
    } catch (error) {
      console.error(` Error tracking event ${eventKey}:`, error)
    }
  }

  close() {
    if (this.client) {
      this.client.close()
    }
  }
}

// Uso en la aplicación
const featureFlags = new FeatureFlagService(process.env.LAUNCHDARKLY_SDK_KEY)

// Middleware para inyectar feature flags
export const featureFlagMiddleware = async (req, res, next) => {
  const user = {
    key: req.user?.id || req.sessionID,
    email: req.user?.email,
    country: req.headers['cf-ipcountry'] || 'US',
    custom: {
      userAgent: req.headers['user-agent'],
      beta: req.user?.isBetaUser || false
    }
  }

  // Obtener todas las feature flags para este usuario
  req.featureFlags = await featureFlags.getAllFlags(user)
  req.user = user

  // Helper functions
  req.isFeatureEnabled = (flagKey, defaultValue = false) => {
    return featureFlags.isFeatureEnabled(flagKey, user, defaultValue)
  }

  req.getFeatureVariation = (flagKey, defaultValue = 'control') => {
    return featureFlags.getFeatureVariation(flagKey, user, defaultValue)
  }

  req.trackEvent = (eventKey, data = null, metricValue = null) => {
    return featureFlags.trackEvent(eventKey, user, data, metricValue)
  }

  next()
}

export { featureFlags }
```

```javascript
// a-b-testing-controller.js
import { featureFlags } from './feature-flags-service.js'

export class ABTestingController {
  async handleCheckoutFlow(req, res) {
    const user = req.user

    // A/B Test: Nueva UI de checkout
    const checkoutVariation = await req.getFeatureVariation('checkout-ui-test', 'control')

    // Track que el usuario entró al checkout
    req.trackEvent('checkout-entered', {
      variation: checkoutVariation,
      source: req.query.source || 'direct'
    })

    switch (checkoutVariation) {
      case 'variant-a':
        return res.render('checkout-new-ui', {
          featureFlags: req.featureFlags,
          experiment: 'checkout-ui-test-a'
        })

      case 'variant-b':
        return res.render('checkout-simplified', {
          featureFlags: req.featureFlags,
          experiment: 'checkout-ui-test-b'
        })

      default: // control
        return res.render('checkout-original', {
          featureFlags: req.featureFlags,
          experiment: 'checkout-ui-test-control'
        })
    }
  }

  async handleCheckoutComplete(req, res) {
    const { orderId, amount, variation } = req.body

    // Track conversion event
    req.trackEvent('checkout-completed', {
      orderId,
      variation: variation || 'control'
    }, amount)

    // Feature flag para mostrar upsells
    const showUpsells = await req.isFeatureEnabled('post-checkout-upsells', false)

    res.json({
      success: true,
      orderId,
      showUpsells,
      redirectUrl: showUpsells ? '/upsells' : '/thank-you'
    })
  }

  async handleRecommendationEngine(req, res) {
    const user = req.user

    // A/B Test: Motor de recomendaciones
    const recommendationAlgorithm = await req.getFeatureVariation('recommendation-algorithm', 'collaborative')

    let recommendations = []

    switch (recommendationAlgorithm) {
      case 'ml-enhanced':
        recommendations = await this.getMLRecommendations(user)
        break

      case 'hybrid':
        recommendations = await this.getHybridRecommendations(user)
        break

      default: // collaborative
        recommendations = await this.getCollaborativeRecommendations(user)
    }

    // Track recommendation view
    req.trackEvent('recommendations-viewed', {
      algorithm: recommendationAlgorithm,
      count: recommendations.length
    })

    res.json({
      recommendations,
      algorithm: recommendationAlgorithm
    })
  }

  // Métodos de recomendación (implementación simplificada)
  async getMLRecommendations(user) {
    // Implementación del algoritmo ML
    return []
  }

  async getHybridRecommendations(user) {
    // Implementación híbrida
    return []
  }

  async getCollaborativeRecommendations(user) {
    // Implementación colaborativa
    return []
  }
}
```

#### **Feature Flags con Deployment**

```yaml
# feature-flag-deployment.yml
apiVersion: v1
kind: ConfigMap
metadata:
  name: feature-flags-config
data:
  features.json: |
    {
      "new-checkout-ui": {
        "enabled": false,
        "rollout": 0,
        "conditions": {
          "beta_users": true,
          "regions": ["US", "CA"]
        }
      },
      "advanced-search": {
        "enabled": true,
        "rollout": 25,
        "conditions": {
          "premium_users": true
        }
      },
      "recommendation-engine-v2": {
        "enabled": true,
        "rollout": 10,
        "conditions": {}
      }
    }

---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp
spec:
  replicas: 3
  template:
    spec:
      containers:
      - name: myapp
        image: myapp:latest
        env:
        - name: FEATURE_FLAGS_CONFIG
          valueFrom:
            configMapKeyRef:
              name: feature-flags-config
              key: features.json
        - name: LAUNCHDARKLY_SDK_KEY
          valueFrom:
            secretKeyRef:
              name: launchdarkly-secret
              key: sdk-key
        volumeMounts:
        - name: feature-flags-volume
          mountPath: /app/config
      volumes:
      - name: feature-flags-volume
        configMap:
          name: feature-flags-config
```

### 5. GitOps Workflow

#### **ArgoCD Configuration**

```yaml
# argocd-application.yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: myapp-production
  namespace: argocd
  finalizers:
    - resources-finalizer.argocd.argoproj.io
spec:
  project: default

  source:
    repoURL: https://github.com/mycompany/myapp-manifests
    targetRevision: HEAD
    path: production
    helm:
      valueFiles:
      - values-production.yaml
      parameters:
      - name: image.tag
        value: v2.0.0
      - name: replicas
        value: "5"

  destination:
    server: https://kubernetes.default.svc
    namespace: production

  syncPolicy:
    automated:
      prune: true
      selfHeal: true
      allowEmpty: false
    syncOptions:
    - CreateNamespace=true
    - PrunePropagationPolicy=foreground
    - PruneLast=true

    retry:
      limit: 5
      backoff:
        duration: 5s
        factor: 2
        maxDuration: 3m

  revisionHistoryLimit: 10

  # Health checks personalizados
  ignoreDifferences:
  - group: apps
    kind: Deployment
    jsonPointers:
    - /spec/replicas  # Ignorar cambios en replicas si son manejados por HPA

  # Hooks de sincronización
  hooks:
  - name: pre-sync-migration
    phase: PreSync
    template:
      spec:
        containers:
        - name: migrate
          image: migrate/migrate
          command: ["migrate", "-path", "/migrations", "-database", "postgres://...", "up"]
```

```bash
#!/bin/bash
# gitops-deploy.sh

REPO_URL="https://github.com/mycompany/myapp-manifests"
NEW_IMAGE_TAG=$1
ENVIRONMENT=${2:-staging}
BRANCH=${3:-main}

if [ -z "$NEW_IMAGE_TAG" ]; then
    echo "Usage: $0 <image-tag> [environment] [branch]"
    exit 1
fi

echo " Starting GitOps deployment"
echo " Image tag: $NEW_IMAGE_TAG"
echo " Environment: $ENVIRONMENT"
echo " Branch: $BRANCH"

# Clonar el repositorio de manifiestos
TEMP_DIR=$(mktemp -d)
cd $TEMP_DIR

git clone $REPO_URL .
git checkout $BRANCH

# Actualizar el tag de la imagen en los manifiestos
echo " Updating image tag in manifests..."

if [ -f "${ENVIRONMENT}/values.yaml" ]; then
    # Helm chart
    yq eval ".image.tag = \"$NEW_IMAGE_TAG\"" -i ${ENVIRONMENT}/values.yaml
elif [ -f "${ENVIRONMENT}/kustomization.yaml" ]; then
    # Kustomize
    cd $ENVIRONMENT
    kustomize edit set image myapp=myapp:$NEW_IMAGE_TAG
    cd ..
else
    # Plain Kubernetes manifests
    find ${ENVIRONMENT}/ -name "*.yaml" -exec sed -i "s|image: myapp:.*|image: myapp:$NEW_IMAGE_TAG|g" {} \;
fi

# Verificar cambios
echo " Changes to be committed:"
git diff

# Commit y push
echo " Committing changes..."
git add .
git commit -m "Deploy $NEW_IMAGE_TAG to $ENVIRONMENT

Automated deployment via GitOps
- Image: myapp:$NEW_IMAGE_TAG
- Environment: $ENVIRONMENT
- Timestamp: $(date -Iseconds)"

git push origin $BRANCH

echo " GitOps deployment triggered successfully!"
echo " Monitor deployment progress in ArgoCD UI"

# Opcional: Esperar a que ArgoCD sincronice
if command -v argocd &> /dev/null; then
    echo "⏳ Waiting for ArgoCD sync..."

    # Login a ArgoCD (requiere configuración previa)
    argocd app sync myapp-$ENVIRONMENT --prune

    # Esperar a que el sync complete
    argocd app wait myapp-$ENVIRONMENT --timeout 600

    if [ $? -eq 0 ]; then
        echo " ArgoCD sync completed successfully!"
    else
        echo " ArgoCD sync failed or timed out"
        exit 1
    fi
fi

# Cleanup
cd /
rm -rf $TEMP_DIR
```

## Laboratorios Prácticos

### Lab 1: Blue-Green con Health Checks

**Objetivo**: Implementar Blue-Green deployment con verificaciones automáticas.

1. **Configurar entornos Blue y Green**
2. **Implementar health checks robustos**
3. **Automatizar el switch de tráfico**
4. **Configurar rollback automático**

### Lab 2: Canary con Métricas

**Objetivo**: Crear Canary deployment basado en métricas de negocio.

1. **Configurar Prometheus y Grafana**
2. **Definir métricas de éxito**
3. **Automatizar promoción/rollback**
4. **Implementar alertas**

### Lab 3: GitOps Completo

**Objetivo**: Implementar workflow GitOps end-to-end.

1. **Configurar ArgoCD**
2. **Crear repositorio de manifiestos**
3. **Automatizar actualizaciones**
4. **Implementar multi-environment**

## Ejercicios Prácticos

### Ejercicio 1: Multi-Environment Pipeline

Crear un pipeline que:

- Despliegue automáticamente a Dev
- Requiera aprobación para Staging
- Use Canary para Production
- Implemente rollback automático

### Ejercicio 2: Disaster Recovery

Implementar estrategia de DR que:

- Detecte fallos automáticamente
- Cambie a región de backup
- Sincronice datos
- Notifique al equipo

### Ejercicio 3: Cost Optimization

Desarrollar estrategias para:

- Optimizar recursos por ambiente
- Implementar auto-scaling inteligente
- Reducir costos de deployment
- Monitorear gastos por release

## Troubleshooting

### Problemas Comunes

1. **Deployments Lentos**:

```bash
# Optimizar readiness probes
readinessProbe:
  httpGet:
    path: /health
    port: 3000
  initialDelaySeconds: 5  # Reducir delay inicial
  periodSeconds: 2        # Verificar más frecuentemente
```

2. **Rollbacks Fallidos**:

```bash
# Verificar historial de revisiones
kubectl rollout history deployment/myapp

# Rollback a versión específica
kubectl rollout undo deployment/myapp --to-revision=2
```

3. **Traffic Split Issues**:

```bash
# Verificar configuración de Istio
kubectl get virtualservice -o yaml
kubectl get destinationrule -o yaml
```

## Recursos Adicionales

- [Kubernetes Deployment Strategies](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/)
- [Istio Traffic Management](https://istio.io/latest/docs/concepts/traffic-management/)
- [ArgoCD Documentation](https://argo-cd.readthedocs.io/)
- [Flagger Documentation](https://docs.flagger.app/)
- [LaunchDarkly Best Practices](https://docs.launchdarkly.com/guides)

## Checklist de Completado

- [ ] Implementado Blue-Green deployment
- [ ] Configurado Canary deployment con métricas
- [ ] Implementado Rolling deployment optimizado
- [ ] Configurado Feature Flags
- [ ] Implementado A/B testing
- [ ] Configurado GitOps workflow
- [ ] Implementado rollback automático
- [ ] Configurado monitoring de deployments
- [ ] Implementado multi-environment strategy
- [ ] Documentado procesos de deployment

---

**Felicitaciones!**Has completado la Fase 2 del curso DevOps y dominas Automation y CI/CD a nivel profesional. Ahora estás preparado para implementar estrategias de deployment de clase mundial.
