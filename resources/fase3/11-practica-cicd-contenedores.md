# Práctica: CI/CD para Contenedores

## 🎯 Objetivos de la Práctica

Al completar esta práctica serás capaz de:

- Implementar pipelines de CI/CD completos para aplicaciones containerizadas
- Configurar automated testing para contenedores
- Gestionar registries de contenedores públicos y privados
- Implementar estrategias de despliegue avanzadas (Blue/Green, Canary)
- Configurar monitoreo y rollback automático
- Integrar herramientas de seguridad en el pipeline

## 📋 Descripción del Proyecto

Crearemos un **pipeline de CI/CD completo** para una aplicación web moderna que incluye:

**Aplicación**: API REST con frontend web
**Tecnologías**: Docker, Kubernetes, GitHub Actions, ArgoCD
**Estrategias**: Multi-stage builds, security scanning, automated testing
**Despliegue**: Multiple environments (dev, staging, production)

## 🏗️ Arquitectura del Pipeline

```
┌─────────────────────────────────────────────────────────────┐
│                    Developer Workflow                       │
└────────────────────┬────────────────────────────────────────┘
                     │
┌────────────────────▼────────────────────────────────────────┐
│                 Source Control                             │
│                    (GitHub)                                │
└────────────────────┬────────────────────────────────────────┘
                     │
┌────────────────────▼────────────────────────────────────────┐
│                Continuous Integration                       │
│   • Build & Test • Security Scan • Image Build & Push     │
└────────────────────┬────────────────────────────────────────┘
                     │
┌────────────────────▼────────────────────────────────────────┐
│              Container Registry                             │
│          (Docker Hub / Harbor / ECR)                       │
└────────────────────┬────────────────────────────────────────┘
                     │
┌────────────────────▼────────────────────────────────────────┐
│             Continuous Deployment                          │
│  Dev → Staging → Production (Blue/Green + Canary)         │
└────────────────────┬────────────────────────────────────────┘
                     │
┌────────────────────▼────────────────────────────────────────┐
│           Monitoring & Observability                       │
│    Metrics • Logs • Alerts • Automatic Rollback          │
└─────────────────────────────────────────────────────────────┘
```

## 📂 Estructura del Proyecto

```
cicd-containers-demo/
├── .github/
│   └── workflows/
│       ├── ci.yml
│       ├── cd-dev.yml
│       ├── cd-staging.yml
│       ├── cd-production.yml
│       ├── security-scan.yml
│       └── cleanup.yml
├── app/
│   ├── Dockerfile
│   ├── Dockerfile.dev
│   ├── package.json
│   ├── src/
│   ├── tests/
│   └── .dockerignore
├── k8s/
│   ├── base/
│   │   ├── deployment.yaml
│   │   ├── service.yaml
│   │   ├── configmap.yaml
│   │   └── kustomization.yaml
│   ├── overlays/
│   │   ├── dev/
│   │   ├── staging/
│   │   └── production/
│   └── argocd/
├── scripts/
│   ├── build.sh
│   ├── test.sh
│   ├── deploy.sh
│   └── rollback.sh
├── monitoring/
│   ├── prometheus/
│   ├── grafana/
│   └── alertmanager/
├── security/
│   ├── policies/
│   └── scanners/
├── docker-compose.yml
├── docker-compose.test.yml
├── Makefile
└── README.md
```

## 🐳 Aplicación de Ejemplo

### app/Dockerfile (Multi-stage optimizado)

```dockerfile
# Multi-stage Dockerfile para producción
FROM node:18-alpine AS dependencies

WORKDIR /app

# Copy package files for better caching
COPY package*.json ./
RUN npm ci --only=production && npm cache clean --force

# Development dependencies for building
FROM node:18-alpine AS build-deps
WORKDIR /app
COPY package*.json ./
RUN npm ci

# Build stage
FROM build-deps AS build
WORKDIR /app
COPY . .
RUN npm run build
RUN npm run test:unit

# Security scanning stage
FROM build AS security-scan
RUN npm audit --audit-level=high
RUN npm run security:check

# Production stage
FROM node:18-alpine AS production

# Create non-root user
RUN addgroup -g 1001 -S nodejs && \
    adduser -S nodejs -u 1001

# Install dumb-init for proper signal handling
RUN apk add --no-cache dumb-init

# Set working directory
WORKDIR /app

# Copy production dependencies
COPY --from=dependencies --chown=nodejs:nodejs /app/node_modules ./node_modules

# Copy built application
COPY --from=build --chown=nodejs:nodejs /app/dist ./dist
COPY --from=build --chown=nodejs:nodejs /app/package*.json ./

# Security: Remove unnecessary packages and files
RUN apk del --purge \
    && rm -rf /var/cache/apk/* \
    && rm -rf /tmp/* \
    && rm -rf /var/tmp/*

# Switch to non-root user
USER nodejs

# Health check
HEALTHCHECK --interval=30s --timeout=10s --start-period=5s --retries=3 \
  CMD node health-check.js

# Expose port
EXPOSE 3000

# Use dumb-init to handle signals properly
ENTRYPOINT ["dumb-init", "--"]
CMD ["node", "dist/index.js"]

# Labels for metadata
LABEL maintainer="devops-team@company.com" \
      version="1.0.0" \
      description="Production-ready Node.js application" \
      org.opencontainers.image.source="https://github.com/company/app" \
      org.opencontainers.image.documentation="https://docs.company.com/app" \
      org.opencontainers.image.licenses="MIT"
```

### app/Dockerfile.dev (Desarrollo)

```dockerfile
FROM node:18-alpine AS development

WORKDIR /app

# Install development tools
RUN apk add --no-cache git curl

# Copy package files
COPY package*.json ./
RUN npm ci

# Copy source code
COPY . .

# Install development dependencies
RUN npm install --include=dev

# Expose port and debugger port
EXPOSE 3000 9229

# Development command with hot reload and debugging
CMD ["npm", "run", "dev:debug"]
```

### app/package.json

```json
{
  "name": "cicd-demo-app",
  "version": "1.0.0",
  "description": "Demo application for CI/CD pipeline",
  "main": "dist/index.js",
  "scripts": {
    "dev": "nodemon src/index.js",
    "dev:debug": "nodemon --inspect=0.0.0.0:9229 src/index.js",
    "build": "webpack --mode=production",
    "start": "node dist/index.js",
    "test": "npm run test:unit && npm run test:integration && npm run test:e2e",
    "test:unit": "jest --testMatch='**/*.unit.test.js'",
    "test:integration": "jest --testMatch='**/*.integration.test.js'",
    "test:e2e": "playwright test",
    "test:coverage": "jest --coverage",
    "lint": "eslint src/ --ext .js",
    "lint:fix": "eslint src/ --ext .js --fix",
    "security:check": "npm audit && snyk test",
    "docker:build": "docker build -t cicd-demo-app .",
    "docker:run": "docker run -p 3000:3000 cicd-demo-app",
    "docker:test": "docker-compose -f docker-compose.test.yml up --abort-on-container-exit"
  },
  "dependencies": {
    "express": "^4.18.2",
    "helmet": "^7.0.0",
    "cors": "^2.8.5",
    "dotenv": "^16.3.1",
    "compression": "^1.7.4",
    "morgan": "^1.10.0",
    "joi": "^17.9.2",
    "prometheus-client": "^14.2.0"
  },
  "devDependencies": {
    "nodemon": "^3.0.1",
    "jest": "^29.6.1",
    "supertest": "^6.3.3",
    "playwright": "^1.36.2",
    "eslint": "^8.45.0",
    "webpack": "^5.88.2",
    "snyk": "^1.1192.0"
  }
}
```

## ⚙️ GitHub Actions Workflows

### .github/workflows/ci.yml (Continuous Integration)

```yaml
name: Continuous Integration

on:
  push:
    branches: [ main, develop ]
  pull_request:
    branches: [ main, develop ]

env:
  REGISTRY: ghcr.io
  IMAGE_NAME: ${{ github.repository }}

jobs:
  # ================================
  # Code Quality & Testing
  # ================================
  test:
    name: Test & Quality Checks
    runs-on: ubuntu-latest
    
    strategy:
      matrix:
        node-version: [16, 18, 20]
    
    steps:
    - name: Checkout code
      uses: actions/checkout@v4

    - name: Setup Node.js ${{ matrix.node-version }}
      uses: actions/setup-node@v4
      with:
        node-version: ${{ matrix.node-version }}
        cache: 'npm'

    - name: Install dependencies
      run: npm ci

    - name: Run linting
      run: npm run lint

    - name: Run unit tests
      run: npm run test:unit

    - name: Run integration tests
      run: npm run test:integration

    - name: Generate test coverage
      run: npm run test:coverage

    - name: Upload coverage to Codecov
      uses: codecov/codecov-action@v3
      with:
        token: ${{ secrets.CODECOV_TOKEN }}
        file: ./coverage/lcov.info

  # ================================
  # Security Scanning
  # ================================
  security:
    name: Security Scan
    runs-on: ubuntu-latest
    needs: test
    
    steps:
    - name: Checkout code
      uses: actions/checkout@v4

    - name: Setup Node.js
      uses: actions/setup-node@v4
      with:
        node-version: '18'
        cache: 'npm'

    - name: Install dependencies
      run: npm ci

    - name: Run npm audit
      run: npm audit --audit-level=high

    - name: Run Snyk security scan
      uses: snyk/actions/node@master
      env:
        SNYK_TOKEN: ${{ secrets.SNYK_TOKEN }}
      with:
        args: --severity-threshold=high

    - name: Upload Snyk results to GitHub Code Scanning
      uses: github/codeql-action/upload-sarif@v2
      if: always()
      with:
        sarif_file: snyk.sarif

  # ================================
  # Docker Build & Scan
  # ================================
  docker-build:
    name: Docker Build & Security Scan
    runs-on: ubuntu-latest
    needs: [test, security]
    
    permissions:
      contents: read
      packages: write
      security-events: write

    steps:
    - name: Checkout code
      uses: actions/checkout@v4

    - name: Set up Docker Buildx
      uses: docker/setup-buildx-action@v3

    - name: Login to Container Registry
      uses: docker/login-action@v3
      with:
        registry: ${{ env.REGISTRY }}
        username: ${{ github.actor }}
        password: ${{ secrets.GITHUB_TOKEN }}

    - name: Extract metadata
      id: meta
      uses: docker/metadata-action@v5
      with:
        images: ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}
        tags: |
          type=ref,event=branch
          type=ref,event=pr
          type=sha,prefix={{branch}}-
          type=raw,value=latest,enable={{is_default_branch}}

    - name: Build Docker image
      uses: docker/build-push-action@v5
      with:
        context: ./app
        push: false
        tags: ${{ steps.meta.outputs.tags }}
        labels: ${{ steps.meta.outputs.labels }}
        cache-from: type=gha
        cache-to: type=gha,mode=max
        target: production

    - name: Run Trivy vulnerability scanner
      uses: aquasecurity/trivy-action@master
      with:
        image-ref: ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}:${{ github.sha }}
        format: 'sarif'
        output: 'trivy-results.sarif'

    - name: Upload Trivy scan results
      uses: github/codeql-action/upload-sarif@v2
      if: always()
      with:
        sarif_file: 'trivy-results.sarif'

    - name: Run container structure test
      run: |
        curl -LO https://storage.googleapis.com/container-structure-test/latest/container-structure-test-linux-amd64
        chmod +x container-structure-test-linux-amd64
        ./container-structure-test-linux-amd64 test --image ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}:${{ github.sha }} --config ./app/container-structure-test.yaml

    - name: Push Docker image
      if: github.event_name != 'pull_request'
      uses: docker/build-push-action@v5
      with:
        context: ./app
        push: true
        tags: ${{ steps.meta.outputs.tags }}
        labels: ${{ steps.meta.outputs.labels }}
        cache-from: type=gha
        cache-to: type=gha,mode=max

  # ================================
  # E2E Testing with Docker
  # ================================
  e2e-test:
    name: End-to-End Tests
    runs-on: ubuntu-latest
    needs: docker-build
    if: github.event_name != 'pull_request'

    steps:
    - name: Checkout code
      uses: actions/checkout@v4

    - name: Start application stack
      run: |
        docker-compose -f docker-compose.test.yml up -d
        sleep 30

    - name: Wait for application to be ready
      run: |
        timeout 60 bash -c 'until curl -f http://localhost:3000/health; do sleep 2; done'

    - name: Setup Node.js for E2E tests
      uses: actions/setup-node@v4
      with:
        node-version: '18'
        cache: 'npm'

    - name: Install E2E test dependencies
      run: npm ci

    - name: Install Playwright browsers
      run: npx playwright install

    - name: Run E2E tests
      run: npm run test:e2e

    - name: Upload E2E test results
      uses: actions/upload-artifact@v3
      if: always()
      with:
        name: e2e-test-results
        path: test-results/

    - name: Stop application stack
      if: always()
      run: docker-compose -f docker-compose.test.yml down -v

  # ================================
  # Performance Testing
  # ================================
  performance-test:
    name: Performance Tests
    runs-on: ubuntu-latest
    needs: e2e-test
    if: github.ref == 'refs/heads/main'

    steps:
    - name: Checkout code
      uses: actions/checkout@v4

    - name: Start application
      run: |
        docker run -d -p 3000:3000 --name app ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}:${{ github.sha }}
        sleep 30

    - name: Install k6
      run: |
        sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv-keys C5AD17C747E3415A3642D57D77C6C491D6AC1D69
        echo "deb https://dl.k6.io/deb stable main" | sudo tee /etc/apt/sources.list.d/k6.list
        sudo apt-get update
        sudo apt-get install k6

    - name: Run performance tests
      run: k6 run --out json=performance-results.json ./tests/performance/load-test.js

    - name: Upload performance results
      uses: actions/upload-artifact@v3
      with:
        name: performance-results
        path: performance-results.json

    - name: Stop application
      if: always()
      run: docker stop app && docker rm app
```

### .github/workflows/cd-production.yml (Production Deployment)

```yaml
name: Production Deployment

on:
  workflow_run:
    workflows: ["Continuous Integration"]
    types:
      - completed
    branches: [main]

env:
  REGISTRY: ghcr.io
  IMAGE_NAME: ${{ github.repository }}
  KUBE_NAMESPACE: production

jobs:
  # ================================
  # Pre-deployment Checks
  # ================================
  pre-deployment:
    name: Pre-deployment Validation
    runs-on: ubuntu-latest
    if: ${{ github.event.workflow_run.conclusion == 'success' }}

    outputs:
      image-tag: ${{ steps.meta.outputs.tags }}
      deployment-id: ${{ steps.deployment.outputs.deployment_id }}

    steps:
    - name: Checkout code
      uses: actions/checkout@v4

    - name: Create GitHub deployment
      id: deployment
      uses: actions/github-script@v6
      with:
        script: |
          const deployment = await github.rest.repos.createDeployment({
            owner: context.repo.owner,
            repo: context.repo.repo,
            ref: context.sha,
            environment: 'production',
            required_contexts: [],
            auto_merge: false
          });
          return deployment.data.id;

    - name: Extract metadata
      id: meta
      uses: docker/metadata-action@v5
      with:
        images: ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}
        tags: |
          type=sha,prefix=main-

    - name: Verify image exists
      run: |
        docker manifest inspect ${{ steps.meta.outputs.tags }}

  # ================================
  # Blue-Green Deployment
  # ================================
  blue-green-deploy:
    name: Blue-Green Deployment
    runs-on: ubuntu-latest
    needs: pre-deployment
    environment: 
      name: production
      url: https://app.production.company.com

    steps:
    - name: Checkout code
      uses: actions/checkout@v4

    - name: Setup kubectl
      uses: azure/setup-kubectl@v3
      with:
        version: 'v1.28.0'

    - name: Configure kubeconfig
      run: |
        mkdir -p $HOME/.kube
        echo "${{ secrets.KUBE_CONFIG }}" | base64 -d > $HOME/.kube/config
        chmod 600 $HOME/.kube/config

    - name: Get current deployment color
      id: current-color
      run: |
        CURRENT_COLOR=$(kubectl get service app-service -n ${{ env.KUBE_NAMESPACE }} -o jsonpath='{.spec.selector.color}' || echo "blue")
        if [ "$CURRENT_COLOR" = "blue" ]; then
          NEW_COLOR="green"
        else
          NEW_COLOR="blue"
        fi
        echo "current-color=$CURRENT_COLOR" >> $GITHUB_OUTPUT
        echo "new-color=$NEW_COLOR" >> $GITHUB_OUTPUT

    - name: Deploy to ${{ steps.current-color.outputs.new-color }} environment
      run: |
        # Update kustomization with new image
        cd k8s/overlays/production
        kustomize edit set image app-image=${{ needs.pre-deployment.outputs.image-tag }}
        
        # Apply blue-green deployment
        kubectl apply -k . -n ${{ env.KUBE_NAMESPACE }}
        
        # Update deployment with new color label
        kubectl patch deployment app-deployment-${{ steps.current-color.outputs.new-color }} \
          -n ${{ env.KUBE_NAMESPACE }} \
          -p '{"spec":{"template":{"metadata":{"labels":{"color":"${{ steps.current-color.outputs.new-color }}"}}}}}'

    - name: Wait for deployment to be ready
      run: |
        kubectl rollout status deployment/app-deployment-${{ steps.current-color.outputs.new-color }} \
          -n ${{ env.KUBE_NAMESPACE }} \
          --timeout=600s

    - name: Run smoke tests
      run: |
        # Get the ClusterIP of the new deployment
        NEW_SERVICE_IP=$(kubectl get service app-service-${{ steps.current-color.outputs.new-color }} -n ${{ env.KUBE_NAMESPACE }} -o jsonpath='{.spec.clusterIP}')
        
        # Run health checks
        kubectl run smoke-test-${{ github.run_id }} \
          --image=curlimages/curl:latest \
          --rm -i --restart=Never \
          -n ${{ env.KUBE_NAMESPACE }} \
          -- curl -f http://$NEW_SERVICE_IP:3000/health

    - name: Switch traffic to ${{ steps.current-color.outputs.new-color }}
      run: |
        # Update main service to point to new deployment
        kubectl patch service app-service \
          -n ${{ env.KUBE_NAMESPACE }} \
          -p '{"spec":{"selector":{"color":"${{ steps.current-color.outputs.new-color }}"}}}'

    - name: Post-deployment verification
      run: |
        sleep 30
        # Verify external endpoint is working
        curl -f https://app.production.company.com/health
        
        # Check metrics for errors
        EXTERNAL_IP=$(kubectl get service app-service -n ${{ env.KUBE_NAMESPACE }} -o jsonpath='{.status.loadBalancer.ingress[0].ip}')
        curl -f http://$EXTERNAL_IP:3000/metrics | grep -E "(http_requests_total|error_rate)"

    - name: Scale down old deployment
      run: |
        kubectl scale deployment app-deployment-${{ steps.current-color.outputs.current-color }} \
          --replicas=0 \
          -n ${{ env.KUBE_NAMESPACE }}

    - name: Update deployment status - Success
      if: success()
      uses: actions/github-script@v6
      with:
        script: |
          github.rest.repos.createDeploymentStatus({
            owner: context.repo.owner,
            repo: context.repo.repo,
            deployment_id: ${{ needs.pre-deployment.outputs.deployment-id }},
            state: 'success',
            environment_url: 'https://app.production.company.com',
            description: 'Blue-green deployment completed successfully'
          });

    - name: Update deployment status - Failure
      if: failure()
      uses: actions/github-script@v6
      with:
        script: |
          github.rest.repos.createDeploymentStatus({
            owner: context.repo.owner,
            repo: context.repo.repo,
            deployment_id: ${{ needs.pre-deployment.outputs.deployment-id }},
            state: 'failure',
            description: 'Blue-green deployment failed'
          });

  # ================================
  # Canary Deployment (Alternative)
  # ================================
  canary-deploy:
    name: Canary Deployment
    runs-on: ubuntu-latest
    needs: pre-deployment
    if: false # Set to true to use canary instead of blue-green
    environment: 
      name: production-canary

    steps:
    - name: Deploy canary (10% traffic)
      run: |
        # Deploy canary version
        kubectl apply -f k8s/overlays/production/canary/ -n ${{ env.KUBE_NAMESPACE }}
        
        # Configure traffic split (90% stable, 10% canary)
        kubectl apply -f - <<EOF
        apiVersion: networking.istio.io/v1alpha3
        kind: VirtualService
        metadata:
          name: app-vs
          namespace: ${{ env.KUBE_NAMESPACE }}
        spec:
          hosts:
          - app.production.company.com
          http:
          - match:
            - headers:
                canary:
                  exact: "true"
            route:
            - destination:
                host: app-service-canary
          - route:
            - destination:
                host: app-service-stable
              weight: 90
            - destination:
                host: app-service-canary
              weight: 10
        EOF

    - name: Monitor canary metrics
      run: |
        # Monitor for 10 minutes
        for i in {1..20}; do
          # Check error rate
          ERROR_RATE=$(curl -s "http://prometheus:9090/api/v1/query?query=rate(http_requests_total{code!~\"2..\",service=\"app-canary\"}[5m])" | jq -r '.data.result[0].value[1]')
          
          if (( $(echo "$ERROR_RATE > 0.01" | bc -l) )); then
            echo "High error rate detected: $ERROR_RATE"
            exit 1
          fi
          
          sleep 30
        done

    - name: Promote canary to 100%
      run: |
        # Gradually increase traffic: 10% -> 25% -> 50% -> 100%
        for weight in 25 50 100; do
          stable_weight=$((100 - weight))
          kubectl patch virtualservice app-vs -n ${{ env.KUBE_NAMESPACE }} \
            --type='merge' -p="{\"spec\":{\"http\":[{\"route\":[{\"destination\":{\"host\":\"app-service-stable\"},\"weight\":$stable_weight},{\"destination\":{\"host\":\"app-service-canary\"},\"weight\":$weight}]}]}}"
          
          sleep 300 # Wait 5 minutes between traffic increases
        done
```

## 🔒 Security Scanning Configuration

### security/policies/opa-policies.rego

```rego
# OPA Policy for container security
package kubernetes.admission

deny[msg] {
  input.request.kind.kind == "Pod"
  input.request.object.spec.containers[_].securityContext.runAsUser == 0
  msg := "Containers must not run as root user"
}

deny[msg] {
  input.request.kind.kind == "Pod"
  input.request.object.spec.containers[_].securityContext.privileged == true
  msg := "Privileged containers are not allowed"
}

deny[msg] {
  input.request.kind.kind == "Pod"
  container := input.request.object.spec.containers[_]
  not container.securityContext.readOnlyRootFilesystem
  msg := "Root filesystem must be read-only"
}

deny[msg] {
  input.request.kind.kind == "Pod"
  container := input.request.object.spec.containers[_]
  not container.resources.limits.memory
  msg := "Memory limits must be specified"
}
```

### app/container-structure-test.yaml

```yaml
schemaVersion: 2.0.0

commandTests:
  - name: "application starts successfully"
    command: ["node", "--version"]
    expectedOutput: ["v18.*"]

  - name: "non-root user"
    command: ["whoami"]
    expectedOutput: ["nodejs"]

  - name: "required files exist"
    command: ["ls", "/app"]
    expectedOutput: ["dist", "node_modules", "package.json"]

fileExistenceTests:
  - name: "application files"
    path: "/app/dist/index.js"
    shouldExist: true

  - name: "package.json exists"
    path: "/app/package.json"
    shouldExist: true

  - name: "no .git directory"
    path: "/app/.git"
    shouldExist: false

fileContentTests:
  - name: "package.json contains expected name"
    path: "/app/package.json"
    expectedContents: ["cicd-demo-app"]

metadataTest:
  exposedPorts: ["3000"]
  env:
    - key: "NODE_ENV"
      value: "production"
  user: "nodejs"
```

## ☸️ Kubernetes Manifests

### k8s/base/deployment.yaml

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: app-deployment
  labels:
    app: demo-app
    version: v1
spec:
  replicas: 3
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxUnavailable: 1
      maxSurge: 1
  selector:
    matchLabels:
      app: demo-app
  template:
    metadata:
      labels:
        app: demo-app
        version: v1
      annotations:
        prometheus.io/scrape: "true"
        prometheus.io/port: "3000"
        prometheus.io/path: "/metrics"
    spec:
      securityContext:
        runAsNonRoot: true
        runAsUser: 1001
        fsGroup: 2000
      containers:
      - name: app
        image: app-image # Kustomize will replace this
        ports:
        - containerPort: 3000
          name: http
        env:
        - name: NODE_ENV
          value: "production"
        - name: PORT
          value: "3000"
        envFrom:
        - configMapRef:
            name: app-config
        - secretRef:
            name: app-secrets
        resources:
          requests:
            memory: "128Mi"
            cpu: "100m"
          limits:
            memory: "512Mi"
            cpu: "500m"
        securityContext:
          allowPrivilegeEscalation: false
          readOnlyRootFilesystem: true
          runAsNonRoot: true
          runAsUser: 1001
          capabilities:
            drop:
            - ALL
        livenessProbe:
          httpGet:
            path: /health
            port: http
          initialDelaySeconds: 30
          periodSeconds: 10
          timeoutSeconds: 5
          failureThreshold: 3
        readinessProbe:
          httpGet:
            path: /ready
            port: http
          initialDelaySeconds: 5
          periodSeconds: 5
          timeoutSeconds: 3
          failureThreshold: 3
        volumeMounts:
        - name: tmp
          mountPath: /tmp
        - name: cache
          mountPath: /app/cache
      volumes:
      - name: tmp
        emptyDir: {}
      - name: cache
        emptyDir: {}
      affinity:
        podAntiAffinity:
          preferredDuringSchedulingIgnoredDuringExecution:
          - weight: 100
            podAffinityTerm:
              labelSelector:
                matchExpressions:
                - key: app
                  operator: In
                  values:
                  - demo-app
              topologyKey: kubernetes.io/hostname
```

### k8s/base/service.yaml

```yaml
apiVersion: v1
kind: Service
metadata:
  name: app-service
  labels:
    app: demo-app
  annotations:
    service.beta.kubernetes.io/aws-load-balancer-type: nlb
    service.beta.kubernetes.io/aws-load-balancer-cross-zone-load-balancing-enabled: "true"
spec:
  selector:
    app: demo-app
  ports:
  - name: http
    port: 80
    targetPort: http
    protocol: TCP
  type: LoadBalancer
  sessionAffinity: None
---
apiVersion: v1
kind: Service
metadata:
  name: app-service-internal
  labels:
    app: demo-app
spec:
  selector:
    app: demo-app
  ports:
  - name: http
    port: 3000
    targetPort: http
    protocol: TCP
  type: ClusterIP
```

### k8s/overlays/production/kustomization.yaml

```yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization

namespace: production

resources:
- ../../base
- hpa.yaml
- pdb.yaml
- network-policy.yaml
- service-monitor.yaml

images:
- name: app-image
  newName: ghcr.io/company/cicd-demo-app
  newTag: main-abc123 # This will be updated by CI/CD

replicas:
- name: app-deployment
  count: 5

patchesStrategicMerge:
- deployment-patch.yaml
- service-patch.yaml

configMapGenerator:
- name: app-config
  literals:
  - LOG_LEVEL=info
  - MAX_CONNECTIONS=1000
  - REDIS_URL=redis://redis-cluster:6379

secretGenerator:
- name: app-secrets
  literals:
  - DATABASE_URL=postgresql://user:pass@db:5432/prod
  - JWT_SECRET=super-secret-jwt-key
  type: Opaque

patches:
- target:
    kind: Deployment
    name: app-deployment
  patch: |-
    - op: add
      path: /spec/template/spec/tolerations
      value:
      - key: "production"
        operator: "Equal"
        value: "true"
        effect: "NoSchedule"
```

### k8s/overlays/production/hpa.yaml

```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: app-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: app-deployment
  minReplicas: 3
  maxReplicas: 20
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
  - type: Pods
    pods:
      metric:
        name: http_requests_per_second
      target:
        type: AverageValue
        averageValue: "100"
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

## 🎯 ArgoCD Configuration

### k8s/argocd/application.yaml

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: demo-app-production
  namespace: argocd
  labels:
    app: demo-app
  finalizers:
    - resources-finalizer.argocd.argoproj.io
spec:
  project: production
  source:
    repoURL: https://github.com/company/cicd-demo-app
    targetRevision: main
    path: k8s/overlays/production
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
---
apiVersion: argoproj.io/v1alpha1
kind: AppProject
metadata:
  name: production
  namespace: argocd
spec:
  description: Production applications
  sourceRepos:
  - 'https://github.com/company/*'
  destinations:
  - namespace: production
    server: https://kubernetes.default.svc
  - namespace: monitoring
    server: https://kubernetes.default.svc
  clusterResourceWhitelist:
  - group: ''
    kind: Namespace
  - group: 'rbac.authorization.k8s.io'
    kind: ClusterRole
  - group: 'rbac.authorization.k8s.io'
    kind: ClusterRoleBinding
  namespaceResourceBlacklist:
  - group: ''
    kind: ResourceQuota
  - group: ''
    kind: LimitRange
  - group: ''
    kind: NetworkPolicy
  roles:
  - name: production-admin
    description: Production administrator
    policies:
    - p, proj:production:production-admin, applications, *, production/*, allow
    - p, proj:production:production-admin, repositories, *, *, allow
    groups:
    - company:production-team
```

## 📊 Monitoring & Observability

### monitoring/prometheus/alerts.yml

```yaml
groups:
- name: application
  rules:
  - alert: HighErrorRate
    expr: rate(http_requests_total{code!~"2.."}[5m]) > 0.1
    for: 5m
    labels:
      severity: critical
    annotations:
      summary: "High error rate detected"
      description: "Error rate is {{ $value }} for {{ $labels.instance }}"

  - alert: HighResponseTime
    expr: histogram_quantile(0.95, rate(http_request_duration_seconds_bucket[5m])) > 0.5
    for: 2m
    labels:
      severity: warning
    annotations:
      summary: "High response time"
      description: "95th percentile latency is {{ $value }}s"

  - alert: PodCrashLooping
    expr: rate(kube_pod_container_status_restarts_total[15m]) > 0
    for: 5m
    labels:
      severity: critical
    annotations:
      summary: "Pod is crash looping"
      description: "Pod {{ $labels.pod }} is restarting frequently"

  - alert: DeploymentReplicasMismatch
    expr: kube_deployment_spec_replicas != kube_deployment_status_ready_replicas
    for: 10m
    labels:
      severity: warning
    annotations:
      summary: "Deployment has mismatched replicas"
      description: "Deployment {{ $labels.deployment }} has {{ $value }} ready replicas"

- name: infrastructure
  rules:
  - alert: HighCPUUsage
    expr: 100 - (avg by(instance) (rate(node_cpu_seconds_total{mode="idle"}[5m])) * 100) > 80
    for: 5m
    labels:
      severity: warning
    annotations:
      summary: "High CPU usage"
      description: "CPU usage is {{ $value }}% on {{ $labels.instance }}"

  - alert: HighMemoryUsage
    expr: (1 - (node_memory_MemAvailable_bytes / node_memory_MemTotal_bytes)) * 100 > 85
    for: 5m
    labels:
      severity: warning
    annotations:
      summary: "High memory usage"
      description: "Memory usage is {{ $value }}% on {{ $labels.instance }}"

  - alert: DiskSpaceLow
    expr: (1 - (node_filesystem_avail_bytes / node_filesystem_size_bytes)) * 100 > 90
    for: 5m
    labels:
      severity: critical
    annotations:
      summary: "Low disk space"
      description: "Disk usage is {{ $value }}% on {{ $labels.instance }}"
```

## 🔄 Rollback Automation

### scripts/rollback.sh

```bash
#!/bin/bash

set -e

NAMESPACE=${1:-production}
PREVIOUS_REVISION=${2:-}

echo "🔄 Iniciando rollback en namespace: $NAMESPACE"

# Función para obtener la última revisión exitosa
get_last_successful_revision() {
    kubectl rollout history deployment/app-deployment -n $NAMESPACE | \
    grep -E "^\s*[0-9]+\s+" | \
    tail -2 | head -1 | \
    awk '{print $1}'
}

# Función para verificar salud del deployment
check_deployment_health() {
    local max_attempts=30
    local attempt=1
    
    while [ $attempt -le $max_attempts ]; do
        echo "Verificando salud del deployment (intento $attempt/$max_attempts)..."
        
        # Verificar que el rollout esté completo
        if kubectl rollout status deployment/app-deployment -n $NAMESPACE --timeout=10s; then
            echo "✅ Rollout completado exitosamente"
            
            # Verificar health endpoint
            if kubectl run health-check-$RANDOM --rm -i --restart=Never \
               --image=curlimages/curl:latest -n $NAMESPACE \
               -- curl -f http://app-service:80/health; then
                echo "✅ Health check exitoso"
                return 0
            fi
        fi
        
        attempt=$((attempt + 1))
        sleep 10
    done
    
    echo "❌ Deployment no saludable después de $max_attempts intentos"
    return 1
}

# Función para notificar Slack
notify_slack() {
    local message=$1
    local color=${2:-"danger"}
    
    if [ -n "$SLACK_WEBHOOK_URL" ]; then
        curl -X POST -H 'Content-type: application/json' \
            --data "{
                \"attachments\": [{
                    \"color\": \"$color\",
                    \"title\": \"🔄 Rollback Notification\",
                    \"text\": \"$message\",
                    \"fields\": [
                        {\"title\": \"Environment\", \"value\": \"$NAMESPACE\", \"short\": true},
                        {\"title\": \"Timestamp\", \"value\": \"$(date)\", \"short\": true}
                    ]
                }]
            }" \
            $SLACK_WEBHOOK_URL
    fi
}

# Función principal de rollback
perform_rollback() {
    local target_revision=$1
    
    echo "📋 Información del deployment actual:"
    kubectl describe deployment app-deployment -n $NAMESPACE | head -20
    
    echo "📊 Estado actual de los pods:"
    kubectl get pods -l app=demo-app -n $NAMESPACE
    
    if [ -z "$target_revision" ]; then
        target_revision=$(get_last_successful_revision)
        echo "🎯 Revision objetivo detectada automáticamente: $target_revision"
    else
        echo "🎯 Revision objetivo especificada: $target_revision"
    fi
    
    if [ -z "$target_revision" ]; then
        echo "❌ No se pudo determinar la revision objetivo"
        exit 1
    fi
    
    # Confirmar rollback
    echo "⚠️  Se realizará rollback a la revision $target_revision"
    echo "¿Desea continuar? (y/N)"
    read -r confirmation
    
    if [[ ! "$confirmation" =~ ^[Yy]$ ]]; then
        echo "Rollback cancelado por el usuario"
        exit 0
    fi
    
    # Realizar rollback
    echo "🚀 Ejecutando rollback..."
    kubectl rollout undo deployment/app-deployment --to-revision=$target_revision -n $NAMESPACE
    
    # Verificar salud post-rollback
    if check_deployment_health; then
        echo "✅ Rollback completado exitosamente"
        notify_slack "Rollback to revision $target_revision completed successfully in $NAMESPACE" "good"
        
        # Información post-rollback
        echo "📋 Estado post-rollback:"
        kubectl get deployment app-deployment -n $NAMESPACE
        kubectl get pods -l app=demo-app -n $NAMESPACE
        
    else
        echo "❌ Rollback falló - deployment no saludable"
        notify_slack "Rollback to revision $target_revision failed in $NAMESPACE" "danger"
        
        # Obtener logs para diagnóstico
        echo "📋 Logs de diagnóstico:"
        kubectl logs -l app=demo-app -n $NAMESPACE --tail=50
        
        exit 1
    fi
}

# Función de rollback automático basado en métricas
auto_rollback_on_metrics() {
    echo "🤖 Verificando métricas para rollback automático..."
    
    # Verificar error rate
    local error_rate=$(curl -s "http://prometheus:9090/api/v1/query?query=rate(http_requests_total{code!~\"2..\",namespace=\"$NAMESPACE\"}[5m])" | \
                      jq -r '.data.result[0].value[1] // 0')
    
    # Verificar latencia P95
    local p95_latency=$(curl -s "http://prometheus:9090/api/v1/query?query=histogram_quantile(0.95,rate(http_request_duration_seconds_bucket{namespace=\"$NAMESPACE\"}[5m]))" | \
                       jq -r '.data.result[0].value[1] // 0')
    
    echo "📊 Métricas actuales:"
    echo "   Error rate: ${error_rate}"
    echo "   P95 latency: ${p95_latency}s"
    
    # Thresholds para rollback automático
    local error_threshold=0.05  # 5%
    local latency_threshold=2.0 # 2 segundos
    
    if (( $(echo "$error_rate > $error_threshold" | bc -l) )) || \
       (( $(echo "$p95_latency > $latency_threshold" | bc -l) )); then
        echo "⚠️  Métricas exceden thresholds - iniciando rollback automático"
        notify_slack "Auto-rollback triggered due to high error rate ($error_rate) or latency ($p95_latency)" "warning"
        
        local last_revision=$(get_last_successful_revision)
        perform_rollback $last_revision
    else
        echo "✅ Métricas dentro de límites normales"
    fi
}

# Verificar argumentos
case "${1:-}" in
    "auto")
        auto_rollback_on_metrics
        ;;
    "manual")
        perform_rollback $PREVIOUS_REVISION
        ;;
    *)
        echo "Uso: $0 {auto|manual} [namespace] [revision]"
        echo "  auto   - Rollback automático basado en métricas"
        echo "  manual - Rollback manual a revision específica"
        exit 1
        ;;
esac
```

## 🚀 Comandos de Despliegue

### Makefile

```makefile
.PHONY: help build test deploy rollback clean

# Variables
APP_NAME := cicd-demo-app
REGISTRY := ghcr.io/company
TAG := $(shell git rev-parse --short HEAD)
NAMESPACE := production
ENVIRONMENT := production

help: ## Mostrar ayuda
	@grep -E '^[a-zA-Z_-]+:.*?## .*$$' $(MAKEFILE_LIST) | sort | awk 'BEGIN {FS = ":.*?## "}; {printf "\033[36m%-20s\033[0m %s\n", $$1, $$2}'

build: ## Construir imagen Docker
	docker build -t $(APP_NAME):$(TAG) -t $(APP_NAME):latest ./app
	docker tag $(APP_NAME):$(TAG) $(REGISTRY)/$(APP_NAME):$(TAG)
	docker tag $(APP_NAME):latest $(REGISTRY)/$(APP_NAME):latest

test: ## Ejecutar tests
	docker-compose -f docker-compose.test.yml up --abort-on-container-exit
	docker-compose -f docker-compose.test.yml down -v

security-scan: ## Escanear vulnerabilidades
	trivy image --exit-code 1 --severity HIGH,CRITICAL $(APP_NAME):$(TAG)
	docker run --rm -v /var/run/docker.sock:/var/run/docker.sock \
		-v $(PWD):/src aquasec/trivy fs --exit-code 1 --severity HIGH,CRITICAL /src

push: ## Subir imagen al registry
	docker push $(REGISTRY)/$(APP_NAME):$(TAG)
	docker push $(REGISTRY)/$(APP_NAME):latest

deploy-dev: ## Desplegar a desarrollo
	kubectl apply -k k8s/overlays/dev
	kubectl set image deployment/app-deployment app=$(REGISTRY)/$(APP_NAME):$(TAG) -n dev
	kubectl rollout status deployment/app-deployment -n dev

deploy-staging: ## Desplegar a staging
	kubectl apply -k k8s/overlays/staging
	kubectl set image deployment/app-deployment app=$(REGISTRY)/$(APP_NAME):$(TAG) -n staging
	kubectl rollout status deployment/app-deployment -n staging

deploy-production: ## Desplegar a producción
	@echo "🚀 Desplegando a producción..."
	kubectl apply -k k8s/overlays/production
	kubectl set image deployment/app-deployment app=$(REGISTRY)/$(APP_NAME):$(TAG) -n $(NAMESPACE)
	kubectl rollout status deployment/app-deployment -n $(NAMESPACE) --timeout=600s
	@echo "✅ Despliegue completado"

rollback: ## Rollback a versión anterior
	./scripts/rollback.sh manual $(NAMESPACE)

health-check: ## Verificar salud de la aplicación
	kubectl get pods -l app=demo-app -n $(NAMESPACE)
	kubectl top pods -l app=demo-app -n $(NAMESPACE)
	@echo "Testing health endpoint..."
	curl -f http://$(shell kubectl get service app-service -n $(NAMESPACE) -o jsonpath='{.status.loadBalancer.ingress[0].ip}')/health

logs: ## Ver logs de la aplicación
	kubectl logs -l app=demo-app -n $(NAMESPACE) --tail=100 -f

clean: ## Limpiar recursos locales
	docker system prune -f
	docker volume prune -f

monitor: ## Abrir dashboards de monitoreo
	@echo "Abriendo dashboards..."
	@echo "Grafana: http://localhost:3000"
	@echo "Prometheus: http://localhost:9090"
	@echo "Kibana: http://localhost:5601"

# Pipeline completo
ci: build test security-scan ## Ejecutar pipeline de CI completo

cd-dev: ci push deploy-dev ## Pipeline completo para desarrollo

cd-staging: ci push deploy-staging ## Pipeline completo para staging

cd-production: ci push deploy-production health-check ## Pipeline completo para producción

# Comando de emergencia
emergency-rollback: ## Rollback de emergencia
	./scripts/rollback.sh auto $(NAMESPACE)
```

## 🎯 Validación y Testing

### tests/e2e/app.spec.js (Playwright)

```javascript
const { test, expect } = require('@playwright/test');

test.describe('E2E Application Tests', () => {
  const baseURL = process.env.BASE_URL || 'http://localhost:3000';

  test('homepage loads successfully', async ({ page }) => {
    await page.goto(baseURL);
    await expect(page).toHaveTitle(/Demo App/);
    
    // Verificar que elementos críticos estén presentes
    await expect(page.locator('header')).toBeVisible();
    await expect(page.locator('main')).toBeVisible();
  });

  test('health endpoint returns ok', async ({ request }) => {
    const response = await request.get(`${baseURL}/health`);
    expect(response.ok()).toBeTruthy();
    
    const data = await response.json();
    expect(data.status).toBe('healthy');
  });

  test('API endpoints work correctly', async ({ request }) => {
    // Test API endpoints
    const apiResponse = await request.get(`${baseURL}/api/users`);
    expect(apiResponse.ok()).toBeTruthy();
    
    const users = await apiResponse.json();
    expect(Array.isArray(users)).toBeTruthy();
  });

  test('application handles errors gracefully', async ({ page }) => {
    // Test 404 page
    await page.goto(`${baseURL}/non-existent-page`);
    await expect(page).toHaveTitle(/404/);
  });

  test('performance metrics are acceptable', async ({ page }) => {
    await page.goto(baseURL);
    
    // Measure page load time
    const startTime = Date.now();
    await page.waitForLoadState('networkidle');
    const loadTime = Date.now() - startTime;
    
    expect(loadTime).toBeLessThan(3000); // Less than 3 seconds
  });
});
```

### Ejecución del Pipeline Completo

```bash
# 1. Desarrollo local
make ci                    # Build, test, security scan
make deploy-dev           # Deploy to development

# 2. Staging
make cd-staging           # Full pipeline to staging

# 3. Production
make cd-production        # Full pipeline to production

# 4. Monitoreo
make monitor              # Open monitoring dashboards

# 5. Rollback si es necesario
make emergency-rollback   # Automatic rollback based on metrics
```

## 🎯 Mejores Prácticas Implementadas

### CI/CD
1. **Multi-stage pipelines** con gates de calidad
2. **Security scanning** integrado en cada etapa
3. **Automated testing** completo (unit, integration, e2e)
4. **Progressive deployment** (dev → staging → production)
5. **Rollback automation** basado en métricas

### Seguridad
1. **Container scanning** con Trivy y Snyk
2. **Policy enforcement** con OPA
3. **Secret management** con Kubernetes secrets
4. **Non-root containers** obligatorio
5. **Network policies** implementadas

### Observabilidad
1. **Distributed tracing** preparado
2. **Metrics collection** con Prometheus
3. **Log aggregation** configurado
4. **Alerting rules** definidas
5. **Automated rollback** en caso de problemas

## 🎯 Resultados Esperados

Al completar esta práctica habrás implementado:

- ✅ Pipeline CI/CD completo y automatizado
- ✅ Múltiples estrategias de despliegue (Blue/Green, Canary)
- ✅ Security scanning integrado
- ✅ Monitoreo y observabilidad completos
- ✅ Rollback automático basado en métricas
- ✅ Prácticas de seguridad y compliance

## 📚 Recursos Adicionales

- [GitHub Actions Documentation](https://docs.github.com/en/actions)
- [ArgoCD Best Practices](https://argo-cd.readthedocs.io/en/stable/user-guide/best_practices/)
- [Kubernetes Security Best Practices](https://kubernetes.io/docs/concepts/security/)
- [Container Security Guide](https://cloud.google.com/security/containers)
- [CI/CD Pipeline Patterns](https://martinfowler.com/articles/cd-patterns.html)

---

**🎉 ¡Felicitaciones! Has completado toda la Fase 3: Contenedores, Redes y Orquestación**

**Próxima fase:** [Fase 4: Infraestructura como Código](../../fase4/README.md)