# Integración Continua (CI)

La Integración Continua es el **corazón del desarrollo moderno de software**. En este módulo aprenderás a implementar pipelines automatizados que validan, construyen y prueban código cada vez que se realizan cambios, eliminando la integración manual y reduciendo drasticamente el tiempo de detección de errores.

---

## **¿Qué es la Integración Continua?**

La **Integración Continua (CI)**es una práctica de desarrollo donde los desarrolladores integran código en un repositorio compartido frecuentemente, idealmente varias veces al día. Cada integración se verifica automáticamente mediante:

- **Build automatizado**del código
- **Ejecución de tests**automatizados
- **Análisis de calidad**de código
- **Validaciones de seguridad**
- **Generación de artefactos**desplegables

### **El problema que resuelve CI**

**Antes de CI (Integration Hell): **
```
Developer A: "Funciona en mi máquina"
Developer B: "Mi código no compila con los cambios de A"
Developer C: "Los tests fallan cuando integro todo"
Team Lead: "El release se retrasa 2 semanas"
```

**Con CI: **
```
 Cada commit se valida automáticamente
 Los problemas se detectan en minutos, no semanas
 El código en el repositorio principal siempre está en estado desplegable
 Los desarrolladores reciben feedback inmediato
```

### **Beneficios de la Integración Continua**

- **Detección temprana de errores**: Issues detectados en minutos vs días/semanas
- **Reducción de riesgos**: Cambios pequeños y frecuentes son menos riesgosos
- **Confianza en el código**: Validación automática constante
- **Productividad mejorada**: Menos tiempo dedicado a debugging de integración
- **Calidad consistente**: Estándares aplicados automáticamente

---

## **1. Principios Fundamentales de CI**

### **Los pilares de CI**

#### **1. Un repositorio único**
- Todo el código fuente en un repositorio versionado (Git)
- Incluye tests, scripts de build, configuración
- Una sola fuente de verdad para el equipo

#### **2. Builds automatizados**
- Proceso de build completamente automatizado
- Sin intervención manual
- Reproducible en cualquier entorno

#### **3. Testing automatizado**
- Tests unitarios, integración, funcionales
- Ejecución automática en cada cambio
- Cobertura de código medible

#### **4. Commits frecuentes**
- Integración diaria (mínimo) por desarrollador
- Cambios pequeños e incrementales
- Evitar ramas de larga duración

#### **5. Build rápido**
- Feedback en menos de 10 minutos
- Parallelización cuando sea posible
- Tests optimizados para velocidad

#### **6. Ambiente similar a producción**
- Tests ejecutados en ambiente que simula producción
- Mismas dependencias y configuraciones
- Containerización para consistencia

#### **7. Fácil acceso a deliverables**
- Artefactos disponibles para todo el equipo
- Versionado claro de releases
- Trazabilidad de cambios

#### **8. Visibilidad del estado**
- Dashboard con estado actual del build
- Notificaciones inmediatas de fallos
- Métricas y tendencias visibles

### **El flujo típico de CI**

```mermaid
graph LR
    A[Developer Push] --> B[Trigger CI Pipeline]
    B --> C[Checkout Code]
    C --> D[Install Dependencies]
    D --> E[Run Linting]
    E --> F[Run Unit Tests]
    F --> G[Build Application]
    G --> H[Run Integration Tests]
    H --> I[Security Scan]
    I --> J[Generate Artifacts]
    J --> K[Notify Results]

    K --> L{Success?}
    L -->|Yes| M[ Ready for CD]
    L -->|No| N[ Notify Developer]
```

---

## **2. Jenkins: El Servidor CI Clásico**

Jenkins es uno de los servidores de CI/CD más populares y flexibles, especialmente para organizaciones con necesidades complejas de automatización.

### **Instalación y configuración inicial**

#### **Instalación con Docker**

```bash
# Crear volumen para persistencia
docker volume create jenkins_home

# Ejecutar Jenkins
docker run -d \
  --name jenkins \
  -p 8080:8080 \
  -p 50000:50000 \
  -v jenkins_home:/var/jenkins_home \
  -v /var/run/docker.sock:/var/run/docker.sock \
  jenkins/jenkins:lts

# Obtener password inicial
docker exec jenkins cat /var/jenkins_home/secrets/initialAdminPassword
```

#### **Instalación en Ubuntu**

```bash
# Agregar clave GPG de Jenkins
curl -fsSL https://pkg.jenkins.io/debian/jenkins.io-2023.key | sudo tee \
  /usr/share/keyrings/jenkins-keyring.asc > /dev/null

# Agregar repositorio
echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] \
  https://pkg.jenkins.io/debian binary/ | sudo tee \
  /etc/apt/sources.list.d/jenkins.list > /dev/null

# Instalar Java y Jenkins
sudo apt update
sudo apt install openjdk-11-jdk jenkins

# Iniciar Jenkins
sudo systemctl start jenkins
sudo systemctl enable jenkins

# Verificar estado
sudo systemctl status jenkins
```

### **Pipeline como Código (Jenkinsfile)**

#### **Jenkinsfile básico**

```groovy
// Jenkinsfile
pipeline {
    agent any

    environment {
        NODE_VERSION = '18'
        APP_NAME = 'my-web-app'
    }

    stages {
        stage('Checkout') {
            steps {
                echo 'Checking out source code...'
                checkout scm
            }
        }

        stage('Install Dependencies') {
            steps {
                echo 'Installing Node.js dependencies...'
                sh 'npm ci'
            }
        }

        stage('Code Quality') {
            parallel {
                stage('Linting') {
                    steps {
                        echo 'Running ESLint...'
                        sh 'npm run lint'
                    }
                }

                stage('Security Scan') {
                    steps {
                        echo 'Running security audit...'
                        sh 'npm audit --audit-level=high'
                    }
                }
            }
        }

        stage('Unit Tests') {
            steps {
                echo 'Running unit tests...'
                sh 'npm run test:unit'
            }
            post {
                always {
                    publishTestResults testResultsPattern: 'test-results.xml'
                    publishHTML([
                        allowMissing: false,
                        alwaysLinkToLastBuild: false,
                        keepAll: true,
                        reportDir: 'coverage',
                        reportFiles: 'index.html',
                        reportName: 'Coverage Report'
                    ])
                }
            }
        }

        stage('Build') {
            steps {
                echo 'Building application...'
                sh 'npm run build'
            }
        }

        stage('Integration Tests') {
            steps {
                echo 'Running integration tests...'
                sh 'npm run test:integration'
            }
        }

        stage('Docker Build') {
            steps {
                script {
                    def image = docker.build("${APP_NAME}:${BUILD_NUMBER}")
                    docker.withRegistry('https://registry.hub.docker.com', 'docker-hub-credentials') {
                        image.push()
                        image.push("latest")
                    }
                }
            }
        }
    }

    post {
        always {
            echo 'Pipeline completed'
            cleanWs()
        }

        success {
            echo ' Pipeline succeeded!'
            slackSend(
                channel: '#deployments',
                color: 'good',
                message: " Build #${BUILD_NUMBER} succeeded for ${APP_NAME}"
            )
        }

        failure {
            echo ' Pipeline failed!'
            slackSend(
                channel: '#deployments',
                color: 'danger',
                message: " Build #${BUILD_NUMBER} failed for ${APP_NAME}"
            )
        }
    }
}
```

#### **Jenkinsfile avanzado con múltiples entornos**

```groovy
// Jenkinsfile-advanced
pipeline {
    agent any

    parameters {
        choice(
            name: 'ENVIRONMENT',
            choices: ['development', 'staging', 'production'],
            description: 'Target deployment environment'
        )

        booleanParam(
            name: 'SKIP_TESTS',
            defaultValue: false,
            description: 'Skip test execution (not recommended for production)'
        )

        string(
            name: 'DOCKER_TAG',
            defaultValue: 'latest',
            description: 'Docker image tag'
        )
    }

    environment {
        APP_NAME = 'my-web-app'
        DOCKER_REGISTRY = 'your-registry.com'
        SONAR_PROJECT_KEY = 'my-web-app'
    }

    tools {
        nodejs 'NodeJS-18'
    }

    stages {
        stage('Pre-flight Checks') {
            steps {
                script {
                    echo " Starting pipeline for ${APP_NAME}"
                    echo " Parameters:"
                    echo "   Environment: ${params.ENVIRONMENT}"
                    echo "   Skip Tests: ${params.SKIP_TESTS}"
                    echo "   Docker Tag: ${params.DOCKER_TAG}"

                    // Verificar que la rama sea correcta para producción
                    if (params.ENVIRONMENT == 'production' && env.BRANCH_NAME != 'main') {
                        error("Production deployments only allowed from main branch")
                    }
                }
            }
        }

        stage('Code Checkout') {
            steps {
                checkout scm
                script {
                    env.GIT_COMMIT_SHORT = sh(
                        script: "git rev-parse --short HEAD",
                        returnStdout: true
                    ).trim()
                    env.BUILD_VERSION = "${env.BUILD_NUMBER}-${env.GIT_COMMIT_SHORT}"
                }
            }
        }

        stage('Install Dependencies') {
            steps {
                sh '''
                    echo " Installing dependencies..."
                    npm ci --only=production
                    npm ci --only=development
                '''
            }
        }

        stage('Code Analysis') {
            parallel {
                stage('Linting') {
                    steps {
                        sh '''
                            echo " Running linter..."
                            npm run lint -- --format=checkstyle --output-file=lint-results.xml
                        '''
                    }
                    post {
                        always {
                            recordIssues(
                                enabledForFailure: true,
                                aggregatingResults: true,
                                tools: [checkStyle(pattern: 'lint-results.xml')]
                            )
                        }
                    }
                }

                stage('Security Audit') {
                    steps {
                        sh '''
                            echo " Running security audit..."
                            npm audit --audit-level=moderate --json > audit-results.json || true
                        '''
                    }
                }

                stage('Dependency Check') {
                    steps {
                        sh '''
                            echo " Checking dependencies..."
                            npm outdated > outdated-dependencies.txt || true
                            npm ls --depth=0 > dependency-tree.txt
                        '''
                    }
                }
            }
        }

        stage('SonarQube Analysis') {
            when {
                anyOf {
                    branch 'main'
                    branch 'develop'
                }
            }
            steps {
                script {
                    def scannerHome = tool 'SonarQubeScanner'
                    withSonarQubeEnv('SonarQube') {
                        sh """
                            ${scannerHome}/bin/sonar-scanner \
                                -Dsonar.projectKey=${SONAR_PROJECT_KEY} \
                                -Dsonar.sources=src \
                                -Dsonar.tests=test \
                                -Dsonar.javascript.lcov.reportPaths=coverage/lcov.info \
                                -Dsonar.testExecutionReportPaths=test-results.xml
                        """
                    }
                }
            }
        }

        stage('Unit Tests') {
            when {
                not { params.SKIP_TESTS }
            }
            steps {
                sh '''
                    echo " Running unit tests..."
                    npm run test:unit -- --reporter=xunit --outputFile=test-results.xml
                '''
            }
            post {
                always {
                    publishTestResults testResultsPattern: 'test-results.xml'
                    publishHTML([
                        allowMissing: false,
                        alwaysLinkToLastBuild: false,
                        keepAll: true,
                        reportDir: 'coverage',
                        reportFiles: 'index.html',
                        reportName: 'Unit Test Coverage'
                    ])
                }
            }
        }

        stage('Build Application') {
            steps {
                sh '''
                    echo " Building application..."
                    npm run build:${ENVIRONMENT}

                    echo " Creating build artifact..."
                    tar -czf ${APP_NAME}-${BUILD_VERSION}.tar.gz dist/
                '''
            }
            post {
                success {
                    archiveArtifacts artifacts: "${APP_NAME}-${BUILD_VERSION}.tar.gz"
                }
            }
        }

        stage('Integration Tests') {
            when {
                not { params.SKIP_TESTS }
            }
            steps {
                sh '''
                    echo " Starting test environment..."
                    docker-compose -f docker-compose.test.yml up -d

                    echo "⏳ Waiting for services to be ready..."
                    sleep 30

                    echo " Running integration tests..."
                    npm run test:integration
                '''
            }
            post {
                always {
                    sh 'docker-compose -f docker-compose.test.yml down || true'
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    def image = docker.build(
                        "${DOCKER_REGISTRY}/${APP_NAME}:${params.DOCKER_TAG}",
                        "--build-arg BUILD_VERSION=${BUILD_VERSION} ."
                    )

                    docker.withRegistry("https://${DOCKER_REGISTRY}", 'docker-registry-credentials') {
                        image.push()
                        image.push("${BUILD_VERSION}")
                    }
                }
            }
        }

        stage('Security Scan - Docker Image') {
            steps {
                sh """
                    echo " Scanning Docker image for vulnerabilities..."
                    docker run --rm -v /var/run/docker.sock:/var/run/docker.sock \
                        aquasec/trivy image --format json --output image-security-report.json \
                        ${DOCKER_REGISTRY}/${APP_NAME}:${params.DOCKER_TAG}
                """
            }
            post {
                always {
                    archiveArtifacts artifacts: 'image-security-report.json'
                }
            }
        }
    }

    post {
        always {
            echo ' Cleaning up workspace...'
            cleanWs()
        }

        success {
            echo ' Pipeline completed successfully!'
            script {
                def message = """
                     *Build Successful*

                    *Project:* ${APP_NAME}
                    *Build:* #${BUILD_NUMBER}
                    *Version:* ${BUILD_VERSION}
                    *Environment:* ${params.ENVIRONMENT}
                    *Branch:* ${env.BRANCH_NAME}
                    *Commit:* ${env.GIT_COMMIT_SHORT}

                    *Artifacts:* Available in Jenkins
                    *Docker Image:* ${DOCKER_REGISTRY}/${APP_NAME}:${params.DOCKER_TAG}
                """

                slackSend(
                    channel: '#ci-cd',
                    color: 'good',
                    message: message
                )
            }
        }

        failure {
            echo ' Pipeline failed!'
            script {
                def message = """
                     *Build Failed*

                    *Project:* ${APP_NAME}
                    *Build:* #${BUILD_NUMBER}
                    *Branch:* ${env.BRANCH_NAME}
                    *Stage:* ${env.STAGE_NAME}

                    *View Logs:* ${BUILD_URL}console
                """

                slackSend(
                    channel: '#ci-cd',
                    color: 'danger',
                    message: message
                )
            }
        }

        unstable {
            echo ' Pipeline unstable (tests failed but build succeeded)'
        }
    }
}
```

---

## **3. GitHub Actions: CI en la Nube**

GitHub Actions es la plataforma de CI/CD nativa de GitHub, integrada directamente con repositorios Git.

### **Workflow básico de CI**

```yaml
# .github/workflows/ci.yml
name: Continuous Integration

on:
  push:
    branches: [ main, develop ]
  pull_request:
    branches: [ main ]

env:
  NODE_VERSION: '18'
  APP_NAME: 'my-web-app'

jobs:
  ci:
    runs-on: ubuntu-latest

    services:
      postgres:
        image: postgres:13
        env:
          POSTGRES_PASSWORD: postgres
          POSTGRES_DB: testdb
        options: >-
          --health-cmd pg_isready
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5
        ports:
          - 5432:5432

    steps:
    - name: Checkout code
      uses: actions/checkout@v3
      with:
        fetch-depth: 0  # Necesario para SonarCloud

    - name: Setup Node.js
      uses: actions/setup-node@v3
      with:
        node-version: ${{ env.NODE_VERSION }}
        cache: 'npm'

    - name: Install dependencies
      run: |
        npm ci
        echo " Dependencies installed successfully"

    - name: Run linter
      run: |
        npm run lint
        echo " Linting passed"

    - name: Run unit tests
      run: |
        npm run test:unit -- --coverage
        echo " Unit tests passed"

    - name: Run integration tests
      env:
        DATABASE_URL: postgresql://postgres:postgres@localhost:5432/testdb
      run: |
        npm run test:integration
        echo " Integration tests passed"

    - name: Build application
      run: |
        npm run build
        echo " Build completed successfully"

    - name: Upload test coverage to Codecov
      uses: codecov/codecov-action@v3
      with:
        file: ./coverage/lcov.info
        flags: unittests
        name: codecov-umbrella

    - name: SonarCloud Scan
      uses: SonarSource/sonarcloud-github-action@master
      env:
        GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
        SONAR_TOKEN: ${{ secrets.SONAR_TOKEN }}

    - name: Build Docker image
      run: |
        docker build -t ${{ env.APP_NAME }}:${{ github.sha }} .
        docker tag ${{ env.APP_NAME }}:${{ github.sha }} ${{ env.APP_NAME }}:latest
        echo " Docker image built successfully"

    - name: Run security scan
      run: |
        docker run --rm -v /var/run/docker.sock:/var/run/docker.sock \
          aquasec/trivy image --exit-code 1 --severity HIGH,CRITICAL \
          ${{ env.APP_NAME }}:${{ github.sha }}

    - name: Upload build artifacts
      uses: actions/upload-artifact@v3
      with:
        name: build-artifacts
        path: |
          dist/
          package.json
          package-lock.json
        retention-days: 30
```

### **Workflow con matrix builds**

```yaml
# .github/workflows/matrix-ci.yml
name: Matrix CI

on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]

jobs:
  test:
    runs-on: ${{ matrix.os }}

    strategy:
      matrix:
        os: [ubuntu-latest, windows-latest, macos-latest]
        node-version: [16, 18, 20]
        include:
          - os: ubuntu-latest
            node-version: 18
            coverage: true
        exclude:
          - os: windows-latest
            node-version: 16

    steps:
    - uses: actions/checkout@v3

    - name: Setup Node.js ${{ matrix.node-version }}
      uses: actions/setup-node@v3
      with:
        node-version: ${{ matrix.node-version }}
        cache: 'npm'

    - name: Install dependencies
      run: npm ci

    - name: Run tests
      run: npm test

    - name: Upload coverage (Ubuntu + Node 18 only)
      if: matrix.coverage
      uses: codecov/codecov-action@v3
```

### **Workflow avanzado con deployment condicional**

```yaml
# .github/workflows/advanced-ci.yml
name: Advanced CI/CD Pipeline

on:
  push:
    branches: [ main, develop ]
    tags: [ 'v*' ]
  pull_request:
    branches: [ main ]

env:
  REGISTRY: ghcr.io
  IMAGE_NAME: ${{ github.repository }}

jobs:
  changes:
    runs-on: ubuntu-latest
    outputs:
      backend: ${{ steps.changes.outputs.backend }}
      frontend: ${{ steps.changes.outputs.frontend }}
      docker: ${{ steps.changes.outputs.docker }}
    steps:
    - uses: actions/checkout@v3
    - uses: dorny/paths-filter@v2
      id: changes
      with:
        filters: |
          backend:
            - 'server/**'
            - 'package.json'
          frontend:
            - 'client/**'
            - 'public/**'
          docker:
            - 'Dockerfile'
            - 'docker-compose*.yml'

  backend-tests:
    needs: changes
    if: needs.changes.outputs.backend == 'true'
    runs-on: ubuntu-latest

    services:
      postgres:
        image: postgres:13
        env:
          POSTGRES_PASSWORD: postgres
        options: --health-cmd pg_isready --health-interval 10s --health-timeout 5s --health-retries 5
        ports:
          - 5432:5432

      redis:
        image: redis:6
        options: --health-cmd "redis-cli ping" --health-interval 10s --health-timeout 5s --health-retries 5
        ports:
          - 6379:6379

    steps:
    - uses: actions/checkout@v3

    - name: Setup Node.js
      uses: actions/setup-node@v3
      with:
        node-version: '18'
        cache: 'npm'
        cache-dependency-path: server/package-lock.json

    - name: Install backend dependencies
      run: |
        cd server
        npm ci

    - name: Run backend linting
      run: |
        cd server
        npm run lint

    - name: Run backend unit tests
      run: |
        cd server
        npm run test:unit
      env:
        NODE_ENV: test
        DATABASE_URL: postgresql://postgres:postgres@localhost:5432/postgres
        REDIS_URL: redis://localhost:6379

    - name: Run backend integration tests
      run: |
        cd server
        npm run test:integration
      env:
        NODE_ENV: test
        DATABASE_URL: postgresql://postgres:postgres@localhost:5432/postgres
        REDIS_URL: redis://localhost:6379

  frontend-tests:
    needs: changes
    if: needs.changes.outputs.frontend == 'true'
    runs-on: ubuntu-latest

    steps:
    - uses: actions/checkout@v3

    - name: Setup Node.js
      uses: actions/setup-node@v3
      with:
        node-version: '18'
        cache: 'npm'
        cache-dependency-path: client/package-lock.json

    - name: Install frontend dependencies
      run: |
        cd client
        npm ci

    - name: Run frontend linting
      run: |
        cd client
        npm run lint

    - name: Run frontend unit tests
      run: |
        cd client
        npm run test:unit -- --coverage

    - name: Build frontend
      run: |
        cd client
        npm run build

    - name: Run E2E tests
      run: |
        cd client
        npm run test:e2e
      env:
        CI: true

  security-scan:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v3

    - name: Run Trivy vulnerability scanner
      uses: aquasecurity/trivy-action@master
      with:
        scan-type: 'fs'
        scan-ref: '.'
        format: 'sarif'
        output: 'trivy-results.sarif'

    - name: Upload Trivy scan results
      uses: github/codeql-action/upload-sarif@v2
      with:
        sarif_file: 'trivy-results.sarif'

  build-and-push:
    needs: [backend-tests, frontend-tests, security-scan]
    if: always() && (needs.backend-tests.result == 'success' || needs.backend-tests.result == 'skipped') && (needs.frontend-tests.result == 'success' || needs.frontend-tests.result == 'skipped')
    runs-on: ubuntu-latest

    permissions:
      contents: read
      packages: write

    outputs:
      image-tag: ${{ steps.meta.outputs.tags }}
      image-digest: ${{ steps.build.outputs.digest }}

    steps:
    - name: Checkout
      uses: actions/checkout@v3

    - name: Login to Container Registry
      uses: docker/login-action@v2
      with:
        registry: ${{ env.REGISTRY }}
        username: ${{ github.actor }}
        password: ${{ secrets.GITHUB_TOKEN }}

    - name: Extract metadata
      id: meta
      uses: docker/metadata-action@v4
      with:
        images: ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}
        tags: |
          type=ref,event=branch
          type=ref,event=pr
          type=semver,pattern={{version}}
          type=semver,pattern={{major}}.{{minor}}
          type=sha,prefix={{branch}}-

    - name: Build and push
      id: build
      uses: docker/build-push-action@v4
      with:
        context: .
        push: true
        tags: ${{ steps.meta.outputs.tags }}
        labels: ${{ steps.meta.outputs.labels }}
        build-args: |
          BUILD_VERSION=${{ github.sha }}
          BUILD_DATE=${{ github.event.head_commit.timestamp }}

  deploy-staging:
    needs: build-and-push
    if: github.ref == 'refs/heads/develop'
    runs-on: ubuntu-latest
    environment: staging

    steps:
    - name: Deploy to staging
      run: |
        echo " Deploying to staging environment"
        echo "Image: ${{ needs.build-and-push.outputs.image-tag }}"
        # Aquí iría la lógica real de deployment

  deploy-production:
    needs: build-and-push
    if: startsWith(github.ref, 'refs/tags/v')
    runs-on: ubuntu-latest
    environment: production

    steps:
    - name: Deploy to production
      run: |
        echo " Deploying to production environment"
        echo "Image: ${{ needs.build-and-push.outputs.image-tag }}"
        # Aquí iría la lógica real de deployment
```

---

## **4. GitLab CI/CD**

GitLab CI/CD está integrado nativamente en GitLab y ofrece potentes capacidades de pipeline.

### **Pipeline básico (.gitlab-ci.yml)**

```yaml
# .gitlab-ci.yml
stages:
  - validate
  - build
  - test
  - security
  - package
  - deploy

variables:
  NODE_VERSION: "18"
  DOCKER_REGISTRY: $CI_REGISTRY
  APP_NAME: "my-web-app"

# Cache para optimizar builds
.node_cache: &node_cache
  cache:
    key:
      files:
        - package-lock.json
    paths:
      - node_modules/
    policy: pull

before_script:
  - echo " Starting CI pipeline for $APP_NAME"
  - echo " Job: $CI_JOB_NAME"
  - echo " Branch: $CI_COMMIT_REF_NAME"
  - echo " Commit: $CI_COMMIT_SHORT_SHA"

# Validación de código
validate:lint:
  stage: validate
  image: node:$NODE_VERSION-alpine
  <<: *node_cache
  before_script:
    - npm ci --cache .npm --prefer-offline
  script:
    - echo " Running linter..."
    - npm run lint -- --format=junit --output-file=lint-report.xml
  artifacts:
    when: always
    reports:
      junit: lint-report.xml
    expire_in: 1 week
  rules:
    - if: $CI_PIPELINE_SOURCE == "merge_request_event"
    - if: $CI_COMMIT_BRANCH == "main"
    - if: $CI_COMMIT_BRANCH == "develop"

validate:format:
  stage: validate
  image: node:$NODE_VERSION-alpine
  <<: *node_cache
  script:
    - echo " Checking code formatting..."
    - npm run format:check
  allow_failure: true

# Build de la aplicación
build:app:
  stage: build
  image: node:$NODE_VERSION-alpine
  cache:
    key:
      files:
        - package-lock.json
    paths:
      - node_modules/
    policy: pull-push
  script:
    - echo " Installing dependencies..."
    - npm ci --cache .npm --prefer-offline
    - echo " Building application..."
    - npm run build
    - echo " Build completed"
  artifacts:
    paths:
      - dist/
      - node_modules/
    expire_in: 1 hour
  rules:
    - if: $CI_PIPELINE_SOURCE == "merge_request_event"
    - if: $CI_COMMIT_BRANCH

# Tests
test:unit:
  stage: test
  image: node:$NODE_VERSION-alpine
  <<: *node_cache
  dependencies:
    - build:app
  script:
    - echo " Running unit tests..."
    - npm run test:unit -- --coverage --reporters=default,junit
  coverage: '/Lines\s*:\s*(\d+\.\d+)%/'
  artifacts:
    when: always
    reports:
      junit: junit.xml
      coverage_report:
        coverage_format: cobertura
        path: coverage/cobertura-coverage.xml
    paths:
      - coverage/
    expire_in: 1 week

test:integration:
  stage: test
  image: node:$NODE_VERSION-alpine
  services:
    - name: postgres:13-alpine
      alias: postgres
    - name: redis:6-alpine
      alias: redis
  variables:
    POSTGRES_DB: testdb
    POSTGRES_USER: test
    POSTGRES_PASSWORD: test
    DATABASE_URL: "postgresql://test:test@postgres:5432/testdb"
    REDIS_URL: "redis://redis:6379"
  dependencies:
    - build:app
  script:
    - echo " Running integration tests..."
    - npm run test:integration
  artifacts:
    when: always
    reports:
      junit: integration-test-results.xml
    expire_in: 1 week

# Análisis de seguridad
security:sast:
  stage: security
  image: returntocorp/semgrep:latest
  script:
    - echo " Running SAST scan..."
    - semgrep --config=auto --json --output=sast-report.json .
  artifacts:
    reports:
      sast: sast-report.json
    expire_in: 1 week
  allow_failure: true

security:dependency:
  stage: security
  image: node:$NODE_VERSION-alpine
  dependencies:
    - build:app
  script:
    - echo " Scanning dependencies for vulnerabilities..."
    - npm audit --audit-level=moderate --json > dependency-scan.json || true
  artifacts:
    paths:
      - dependency-scan.json
    expire_in: 1 week
  allow_failure: true

# Packaging
package:docker:
  stage: package
  image: docker:20.10.16
  services:
    - docker:20.10.16-dind
  dependencies:
    - build:app
  variables:
    DOCKER_HOST: tcp://docker:2376
    DOCKER_TLS_CERTDIR: "/certs"
  before_script:
    - echo $CI_REGISTRY_PASSWORD | docker login -u $CI_REGISTRY_USER --password-stdin $CI_REGISTRY
  script:
    - echo " Building Docker image..."
    - |
      docker build \
        --build-arg BUILD_VERSION=$CI_COMMIT_SHORT_SHA \
        --build-arg BUILD_DATE=$(date -u +"%Y-%m-%dT%H:%M:%SZ") \
        --tag $CI_REGISTRY_IMAGE:$CI_COMMIT_SHORT_SHA \
        --tag $CI_REGISTRY_IMAGE:latest \
        .
    - echo " Pushing Docker image..."
    - docker push $CI_REGISTRY_IMAGE:$CI_COMMIT_SHORT_SHA
    - docker push $CI_REGISTRY_IMAGE:latest
  rules:
    - if: $CI_COMMIT_BRANCH == "main"
    - if: $CI_COMMIT_BRANCH == "develop"
    - if: $CI_COMMIT_TAG

# Deployment
deploy:staging:
  stage: deploy
  image: alpine:latest
  environment:
    name: staging
    url: https://staging.example.com
  dependencies:
    - package:docker
  before_script:
    - apk add --no-cache curl
  script:
    - echo " Deploying to staging..."
    - |
      curl -X POST \
        -H "Authorization: Bearer $DEPLOY_TOKEN" \
        -H "Content-Type: application/json" \
        -d "{
          \"environment\": \"staging\",
          \"image\": \"$CI_REGISTRY_IMAGE:$CI_COMMIT_SHORT_SHA\",
          \"version\": \"$CI_COMMIT_SHORT_SHA\"
        }" \
        $DEPLOYMENT_WEBHOOK_URL
    - echo " Deployment to staging completed"
  rules:
    - if: $CI_COMMIT_BRANCH == "develop"
  when: manual

deploy:production:
  stage: deploy
  image: alpine:latest
  environment:
    name: production
    url: https://app.example.com
  dependencies:
    - package:docker
  before_script:
    - apk add --no-cache curl
  script:
    - echo " Deploying to production..."
    - |
      curl -X POST \
        -H "Authorization: Bearer $DEPLOY_TOKEN" \
        -H "Content-Type: application/json" \
        -d "{
          \"environment\": \"production\",
          \"image\": \"$CI_REGISTRY_IMAGE:$CI_COMMIT_SHORT_SHA\",
          \"version\": \"$CI_COMMIT_SHORT_SHA\"
        }" \
        $DEPLOYMENT_WEBHOOK_URL
    - echo " Deployment to production completed"
  rules:
    - if: $CI_COMMIT_TAG
  when: manual
```

---

## **5. CircleCI: CI en la Nube Optimizado**

CircleCI es conocido por su velocidad y facilidad de configuración.

### **Configuración básica (.circleci/config.yml)**

```yaml
# .circleci/config.yml
version: 2.1

orbs:
  node: circleci/node@5.0.3
  docker: circleci/docker@2.2.0
  heroku: circleci/heroku@2.0.0

executors:
  node-executor:
    docker:
      - image: cimg/node:18.17
    resource_class: medium

  node-postgres-executor:
    docker:
      - image: cimg/node:18.17
      - image: cimg/postgres:13.11
        environment:
          POSTGRES_USER: test
          POSTGRES_PASSWORD: test
          POSTGRES_DB: testdb

commands:
  install-dependencies:
    description: "Install npm dependencies with caching"
    steps:
      - restore_cache:
          keys:
            - v1-dependencies-{{ checksum "package-lock.json" }}
            - v1-dependencies-
      - run:
          name: Install dependencies
          command: npm ci
      - save_cache:
          paths:
            - node_modules
          key: v1-dependencies-{{ checksum "package-lock.json" }}

jobs:
  checkout-and-install:
    executor: node-executor
    steps:
      - checkout
      - install-dependencies
      - persist_to_workspace:
          root: .
          paths:
            - .

  lint-and-format:
    executor: node-executor
    steps:
      - attach_workspace:
          at: .
      - run:
          name: Run linter
          command: |
            npm run lint -- --format=junit --output-file=reports/lint-results.xml
      - run:
          name: Check formatting
          command: npm run format:check
      - store_test_results:
          path: reports
      - store_artifacts:
          path: reports

  unit-tests:
    executor: node-executor
    steps:
      - attach_workspace:
          at: .
      - run:
          name: Run unit tests
          command: |
            npm run test:unit -- --coverage --reporters=default,jest-junit
          environment:
            JEST_JUNIT_OUTPUT_DIR: reports
            JEST_JUNIT_OUTPUT_NAME: unit-test-results.xml
      - run:
          name: Upload coverage to Codecov
          command: npx codecov
      - store_test_results:
          path: reports
      - store_artifacts:
          path: coverage

  integration-tests:
    executor: node-postgres-executor
    steps:
      - attach_workspace:
          at: .
      - run:
          name: Wait for database
          command: dockerize -wait tcp://localhost:5432 -timeout 1m
      - run:
          name: Run database migrations
          command: npm run db:migrate
          environment:
            DATABASE_URL: postgresql://test:test@localhost:5432/testdb
      - run:
          name: Run integration tests
          command: npm run test:integration
          environment:
            DATABASE_URL: postgresql://test:test@localhost:5432/testdb

  build-app:
    executor: node-executor
    steps:
      - attach_workspace:
          at: .
      - run:
          name: Build application
          command: npm run build
      - persist_to_workspace:
          root: .
          paths:
            - dist

  security-scan:
    executor: node-executor
    steps:
      - attach_workspace:
          at: .
      - run:
          name: Security audit
          command: |
            npm audit --audit-level=moderate --json > security-audit.json || true
      - run:
          name: Install Snyk
          command: npm install -g snyk
      - run:
          name: Run Snyk security scan
          command: |
            snyk auth $SNYK_TOKEN
            snyk test --json > snyk-results.json || true
      - store_artifacts:
          path: security-audit.json
      - store_artifacts:
          path: snyk-results.json

  build-docker:
    executor: docker/docker
    steps:
      - setup_remote_docker:
          docker_layer_caching: true
      - attach_workspace:
          at: .
      - docker/build:
          image: $CIRCLE_PROJECT_USERNAME/$CIRCLE_PROJECT_REPONAME
          tag: $CIRCLE_SHA1,latest
      - docker/push:
          image: $CIRCLE_PROJECT_USERNAME/$CIRCLE_PROJECT_REPONAME
          tag: $CIRCLE_SHA1,latest

  deploy-staging:
    executor: node-executor
    steps:
      - attach_workspace:
          at: .
      - heroku/deploy-via-git:
          app-name: $HEROKU_STAGING_APP_NAME

workflows:
  ci-cd:
    jobs:
      - checkout-and-install

      - lint-and-format:
          requires:
            - checkout-and-install

      - unit-tests:
          requires:
            - checkout-and-install

      - integration-tests:
          requires:
            - checkout-and-install

      - security-scan:
          requires:
            - checkout-and-install

      - build-app:
          requires:
            - lint-and-format
            - unit-tests
            - integration-tests

      - build-docker:
          requires:
            - build-app
            - security-scan
          filters:
            branches:
              only:
                - main
                - develop

      - deploy-staging:
          requires:
            - build-docker
          filters:
            branches:
              only: develop
```

---

## **6. Mejores Prácticas de CI**

### **Principios de pipeline efectivo**

#### **1. Fail Fast**
```yaml
# Ejemplo: Ejecutar linting antes que tests pesados
stages:
  - validate    # Linting, formato (rápido)
  - test        # Unit tests (medio)
  - integration # Integration tests (lento)
  - build       # Build final (lento)
```

#### **2. Paralelización inteligente**
```yaml
# GitHub Actions - Jobs paralelos
jobs:
  lint:
    runs-on: ubuntu-latest
    steps: [...]

  unit-tests:
    runs-on: ubuntu-latest
    steps: [...]

  integration-tests:
    runs-on: ubuntu-latest
    needs: [lint, unit-tests]  # Solo después de que pasen lint y unit tests
    steps: [...]
```

#### **3. Cache estratégico**
```yaml
# Cache de dependencias
- uses: actions/cache@v3
  with:
    path: ~/.npm
    key: ${{ runner.os }}-node-${{ hashFiles('**/package-lock.json') }}
    restore-keys: |
      ${{ runner.os }}-node-

# Cache de Docker layers
- uses: docker/build-push-action@v4
  with:
    cache-from: type=gha
    cache-to: type=gha,mode=max
```

#### **4. Environments y secretos**
```yaml
# Manejo seguro de secretos
jobs:
  deploy:
    environment: production
    steps:
      - name: Deploy
        env:
          API_KEY: ${{ secrets.PRODUCTION_API_KEY }}
          DB_PASSWORD: ${{ secrets.DB_PASSWORD }}
        run: ./deploy.sh
```

### **Métricas de CI importantes**

#### **Build Time Optimization**
```bash
# Script para medir tiempos de build
#!/bin/bash
echo " Build Metrics Report"
echo "=="

start_time=$(date +%s)

# Medir tiempo de instalación
install_start=$(date +%s)
npm ci
install_end=$(date +%s)
install_time=$((install_end - install_start))

# Medir tiempo de linting
lint_start=$(date +%s)
npm run lint
lint_end=$(date +%s)
lint_time=$((lint_end - lint_start))

# Medir tiempo de tests
test_start=$(date +%s)
npm run test
test_end=$(date +%s)
test_time=$((test_end - test_start))

# Medir tiempo de build
build_start=$(date +%s)
npm run build
build_end=$(date +%s)
build_time=$((build_end - build_start))

end_time=$(date +%s)
total_time=$((end_time - start_time))

echo "  Timing Results:"
echo "   Install: ${install_time}s"
echo "   Linting: ${lint_time}s"
echo "   Testing: ${test_time}s"
echo "   Building: ${build_time}s"
echo "   Total: ${total_time}s"

# Alertar si el build es muy lento
if [ $total_time -gt 600 ]; then
    echo "  Build time exceeds 10 minutes - optimization needed"
fi
```

#### **Dashboard de métricas**
```json
{
  "ci_metrics": {
    "build_success_rate": "98.5%",
    "average_build_time": "4m 32s",
    "test_coverage": "87%",
    "security_issues": 0,
    "deployment_frequency": "5.2 per day",
    "mttr": "12 minutes",
    "change_failure_rate": "2.3%"
  }
}
```

---

## **7. Troubleshooting Common CI Issues**

### **Problemas frecuentes y soluciones**

#### **1. Tests flaky (intermitentes)**
```javascript
//  Test flaky - depende de timing
test('async operation', async () => {
  startAsyncOperation();
  await new Promise(resolve => setTimeout(resolve, 100)); // Tiempo fijo
  expect(getResult()).toBe('expected');
});

//  Test estable - espera condición
test('async operation', async () => {
  startAsyncOperation();
  await waitFor(() => expect(getResult()).toBe('expected'), {
    timeout: 5000
  });
});
```

#### **2. Dependencias de red**
```yaml
#  Tests que dependen de servicios externos
- name: Test external API
  run: curl https://api.external-service.com/health

#  Mock de servicios externos
- name: Start mock server
  run: docker run -d --name mock-api wiremock/wiremock
- name: Test with mocked API
  run: npm run test:integration
  env:
    API_URL: http://localhost:8080
```

#### **3. Problemas de recursos**
```yaml
#  Configurar límites de memoria
jobs:
  test:
    runs-on: ubuntu-latest
    env:
      NODE_OPTIONS: '--max-old-space-size=4096'
    steps:
      - name: Run memory-intensive tests
        run: npm run test:heavy
```

#### **4. Debug de pipeline failures**
```yaml
# Agregar debugging steps
- name: Debug environment
  if: failure()
  run: |
    echo "Node version: $(node --version)"
    echo "NPM version: $(npm --version)"
    echo "Environment variables:"
    printenv | grep -E "(NODE|NPM|CI)" | sort
    echo "Disk usage:"
    df -h
    echo "Memory usage:"
    free -h

- name: Upload debug logs
  if: failure()
  uses: actions/upload-artifact@v3
  with:
    name: debug-logs
    path: |
      *.log
      /tmp/*.log
```

---

## **8. Casos de Estudio Prácticos**

### **Caso 1: Migración de Jenkins a GitHub Actions**

**Situación inicial: **
- Pipeline Jenkins complejo con 45 minutos de build time
- Dependencias en infrastructure on-premise
- Dificultades para escalar runners

**Solución implementada: **
```yaml
# Estrategia de migración gradual
# .github/workflows/migration-test.yml
name: GitHub Actions Test (Parallel to Jenkins)

on:
  push:
    branches: [ feature/github-actions-migration ]

jobs:
  parallel-test:
    runs-on: ubuntu-latest
    steps:
      # Ejecutar mismo pipeline que Jenkins
      # Comparar resultados y tiempos
      # Reportar diferencias
```

**Resultados: **
- Reducción de build time: 45min → 8min
- Eliminación de maintenance de infrastructure
- Mejor paralelización y caching

### **Caso 2: CI para Monorepo**

**Desafío: **
- Múltiples aplicaciones en un repositorio
- Evitar builds innecesarios
- Coordinar deployments

**Solución: **
```yaml
# .github/workflows/monorepo-ci.yml
name: Monorepo CI

on:
  push:
    branches: [main]

jobs:
  changes:
    outputs:
      frontend: ${{ steps.filter.outputs.frontend }}
      backend: ${{ steps.filter.outputs.backend }}
      shared: ${{ steps.filter.outputs.shared }}
    steps:
      - uses: dorny/paths-filter@v2
        id: filter
        with:
          filters: |
            frontend:
              - 'apps/frontend/**'
            backend:
              - 'apps/backend/**'
            shared:
              - 'packages/shared/**'

  frontend-ci:
    needs: changes
    if: needs.changes.outputs.frontend == 'true' || needs.changes.outputs.shared == 'true'
    runs-on: ubuntu-latest
    steps:
      - name: Build and test frontend
        run: |
          cd apps/frontend
          npm run ci-pipeline

  backend-ci:
    needs: changes
    if: needs.changes.outputs.backend == 'true' || needs.changes.outputs.shared == 'true'
    runs-on: ubuntu-latest
    steps:
      - name: Build and test backend
        run: |
          cd apps/backend
          npm run ci-pipeline
```

---

## **Próximos Pasos**

Una vez que domines la Integración Continua, estarás listo para avanzar a:

- **Pruebas Automatizadas**: Estrategias de testing en CI/CD
- **Entrega/Despliegue Continuo (CD)**: Automatizar deployments
- **Monitoreo de Pipelines**: Observabilidad y métricas

La CI es el **fundamento**de DevOps moderno. Un pipeline de CI bien diseñado es la diferencia entre un equipo que entrega software confiable rápidamente y uno que lucha con bugs e integraciones problemáticas.

---

## **Recursos y Referencias**

### **Documentación oficial**
- [Jenkins Documentation](https://www.jenkins.io/doc/)
- [GitHub Actions Documentation](https://docs.github.com/en/actions)
- [GitLab CI/CD Documentation](https://docs.gitlab.com/ee/ci/)
- [CircleCI Documentation](https://circleci.com/docs/)

### **Herramientas de testing y calidad**
- [SonarQube](https://www.sonarqube.org/) - Análisis de calidad de código
- [Codecov](https://codecov.io/) - Cobertura de código
- [Snyk](https://snyk.io/) - Seguridad de dependencias

### **Plantillas y ejemplos**
- [GitHub Actions Starter Workflows](https://github.com/actions/starter-workflows)
- [GitLab CI/CD Templates](https://gitlab.com/gitlab-org/gitlab-foss/-/tree/master/lib/gitlab/ci/templates)
- [CircleCI Orbs Registry](https://circleci.com/developer/orbs)

Felicidades! Has completado el módulo de Integración Continua. Ahora tienes las habilidades para implementar pipelines automatizados que validarán tu código consistentemente y te permitirán detectar problemas temprano en el ciclo de desarrollo.
