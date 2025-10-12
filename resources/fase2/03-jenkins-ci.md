# 03. Configuración de Jenkins

**Domina Jenkins para CI/CD profesional!**Este módulo te enseñará a configurar y gestionar Jenkins desde cero hasta implementaciones empresariales avanzadas.

## Objetivos de Aprendizaje

Al completar este módulo serás capaz de:

- **Instalar y configurar**Jenkins en diferentes entornos
- **Crear pipelines**declarativos e imperativos
- **Gestionar plugins**y configuraciones avanzadas
- **Implementar seguridad**y control de acceso
- **Configurar agentes**distribuidos
- **Integrar con Git**y sistemas de control de versiones
- **Automatizar deployments**con Jenkins

## Contenido del Módulo

### 1. Instalación y Configuración Inicial

#### **Instalación con Docker (Recomendado)**

```yaml
# docker-compose.yml
version: '3.8'
services:
  jenkins:
    image: jenkins/jenkins:lts
    container_name: jenkins
    restart: unless-stopped
    ports:
      - "8080:8080"
      - "50000:50000"
    volumes:
      - jenkins_home:/var/jenkins_home
      - /var/run/docker.sock:/var/run/docker.sock
      - ./jenkins_plugins.txt:/usr/share/jenkins/ref/plugins.txt
    environment:
      - JAVA_OPTS=-Djenkins.install.runSetupWizard=false
      - JENKINS_ADMIN_ID=admin
      - JENKINS_ADMIN_PASSWORD=admin123
    networks:
      - jenkins_network

  jenkins-agent:
    image: jenkins/ssh-agent
    container_name: jenkins-agent
    environment:
      - JENKINS_AGENT_SSH_PUBKEY=ssh-rsa AAAAB3NzaC1yc2E...
    networks:
      - jenkins_network

volumes:
  jenkins_home:

networks:
  jenkins_network:
    driver: bridge
```

```bash
# plugins.txt - Plugins esenciales
ant:latest
build-timeout:latest
credentials-binding:latest
email-ext:latest
git:latest
github:latest
gradle:latest
ldap:latest
mailer:latest
matrix-auth:latest
pam-auth:latest
pipeline-github-lib:latest
pipeline-stage-view:latest
ssh-slaves:latest
timestamper:latest
workflow-aggregator:latest
ws-cleanup:latest
docker-workflow:latest
docker-pipeline:latest
kubernetes:latest
```

```bash
# Iniciar Jenkins
docker-compose up -d

# Ver logs
docker-compose logs -f jenkins

# Obtener password inicial (si no se configuró)
docker exec jenkins cat /var/jenkins_home/secrets/initialAdminPassword
```

#### **Instalación en Ubuntu/Debian**

```bash
#!/bin/bash
# install-jenkins.sh

echo " Instalando Jenkins en Ubuntu/Debian"

# Actualizar sistema
sudo apt update && sudo apt upgrade -y

# Instalar Java 11
sudo apt install -y openjdk-11-jdk

# Verificar Java
java -version

# Agregar repositorio de Jenkins
wget -q -O - https://pkg.jenkins.io/debian-stable/jenkins.io.key | sudo apt-key add -
sudo sh -c 'echo deb https://pkg.jenkins.io/debian-stable binary/ > /etc/apt/sources.list.d/jenkins.list'

# Instalar Jenkins
sudo apt update
sudo apt install -y jenkins

# Habilitar y iniciar Jenkins
sudo systemctl enable jenkins
sudo systemctl start jenkins

# Verificar estado
sudo systemctl status jenkins

# Configurar firewall
sudo ufw allow 8080
sudo ufw enable

echo " Jenkins instalado exitosamente"
echo " Acceder a http://localhost:8080"
echo " Password inicial:"
sudo cat /var/lib/jenkins/secrets/initialAdminPassword
```

#### **Configuración Inicial Automatizada**

```groovy
// init.groovy.d/basic-security.groovy
import jenkins.model.*
import hudson.security.*
import hudson.security.csrf.DefaultCrumbIssuer
import jenkins.security.s2m.AdminWhitelistRule

def instance = Jenkins.getInstance()

// Configurar seguridad básica
def hudsonRealm = new HudsonPrivateSecurityRealm(false)
hudsonRealm.createAccount("admin", "admin123")
instance.setSecurityRealm(hudsonRealm)

// Configurar autorización
def strategy = new FullControlOnceLoggedInAuthorizationStrategy()
strategy.setAllowAnonymousRead(false)
instance.setAuthorizationStrategy(strategy)

// Habilitar CSRF protection
instance.setCrumbIssuer(new DefaultCrumbIssuer(true))

// Configurar agente-master security
instance.getInjector().getInstance(AdminWhitelistRule.class).setMasterKillSwitch(false)

instance.save()
```

### 2. Pipelines Declarativos

#### **Pipeline Básico**

```groovy
// Jenkinsfile
pipeline {
    agent any

    environment {
        APP_NAME = 'mi-aplicacion'
        VERSION = '1.0.0'
        DOCKER_REGISTRY = 'registry.company.com'
    }

    options {
        buildDiscarder(logRotator(numToKeepStr: '10'))
        timeout(time: 30, unit: 'MINUTES')
        timestamps()
    }

    triggers {
        cron('H 2 * * *')  // Build nocturno
        pollSCM('H/5 * * * *')  // Poll SCM cada 5 minutos
    }

    stages {
        stage('Checkout') {
            steps {
                echo ' Descargando código fuente...'
                checkout scm
                script {
                    env.GIT_COMMIT_HASH = sh(
                        script: 'git rev-parse --short HEAD',
                        returnStdout: true
                    ).trim()
                }
            }
        }

        stage('Build') {
            steps {
                echo ' Construyendo aplicación...'
                sh 'npm ci'
                sh 'npm run build'
            }
            post {
                always {
                    archiveArtifacts artifacts: 'dist/**/*', allowEmptyArchive: true
                }
            }
        }

        stage('Test') {
            parallel {
                stage('Unit Tests') {
                    steps {
                        echo ' Ejecutando tests unitarios...'
                        sh 'npm run test:unit'
                    }
                    post {
                        always {
                            publishTestResults testResultsPattern: 'reports/junit.xml'
                            publishCoverageReports([
                                [
                                    reportType: 'COBERTURA',
                                    reportFile: 'coverage/cobertura-coverage.xml'
                                ]
                            ])
                        }
                    }
                }

                stage('Integration Tests') {
                    steps {
                        echo ' Ejecutando tests de integración...'
                        sh 'npm run test:integration'
                    }
                }

                stage('Security Scan') {
                    steps {
                        echo ' Ejecutando análisis de seguridad...'
                        sh 'npm audit --audit-level moderate'
                        sh 'docker run --rm -v $(pwd):/app securecodewarrior/semgrep --config=auto /app'
                    }
                }
            }
        }

        stage('Quality Gate') {
            steps {
                echo ' Verificando quality gate...'
                script {
                    def qualityGate = waitForQualityGate()
                    if (qualityGate.status != 'OK') {
                        error "Quality Gate falló: ${qualityGate.status}"
                    }
                }
            }
        }

        stage('Docker Build') {
            when {
                anyOf {
                    branch 'main'
                    branch 'develop'
                    changeRequest()
                }
            }
            steps {
                echo ' Construyendo imagen Docker...'
                script {
                    def image = docker.build("${DOCKER_REGISTRY}/${APP_NAME}:${env.GIT_COMMIT_HASH}")
                    docker.withRegistry("https://${DOCKER_REGISTRY}", 'docker-registry-credentials') {
                        image.push()
                        image.push('latest')
                    }
                }
            }
        }

        stage('Deploy to Staging') {
            when {
                branch 'develop'
            }
            steps {
                echo ' Desplegando a staging...'
                script {
                    build job: 'deploy-to-staging', parameters: [
                        string(name: 'IMAGE_TAG', value: env.GIT_COMMIT_HASH),
                        string(name: 'APP_NAME', value: APP_NAME)
                    ]
                }
            }
        }

        stage('Deploy to Production') {
            when {
                branch 'main'
            }
            steps {
                echo ' Desplegando a producción...'
                input message: '¿Desplegar a producción?', ok: 'Deploy!',
                      submitterParameter: 'DEPLOYER'

                script {
                    echo "Desplegado por: ${env.DEPLOYER}"
                    build job: 'deploy-to-production', parameters: [
                        string(name: 'IMAGE_TAG', value: env.GIT_COMMIT_HASH),
                        string(name: 'APP_NAME', value: APP_NAME),
                        string(name: 'DEPLOYER', value: env.DEPLOYER)
                    ]
                }
            }
        }
    }

    post {
        always {
            echo ' Limpieza...'
            cleanWs()
        }
        success {
            echo ' Pipeline completado exitosamente!'
            slackSend channel: '#deployments',
                     color: 'good',
                     message: " ${APP_NAME} ${VERSION} desplegado exitosamente\nCommit: ${env.GIT_COMMIT_HASH}\nPor: ${env.CHANGE_AUTHOR ?: env.BUILD_USER}"
        }
        failure {
            echo ' Pipeline falló!'
            slackSend channel: '#deployments',
                     color: 'danger',
                     message: " ${APP_NAME} ${VERSION} falló en el pipeline\nCommit: ${env.GIT_COMMIT_HASH}\nStage: ${env.STAGE_NAME}"
        }
        unstable {
            echo ' Pipeline inestable!'
            slackSend channel: '#deployments',
                     color: 'warning',
                     message: " ${APP_NAME} ${VERSION} completado con warnings\nCommit: ${env.GIT_COMMIT_HASH}"
        }
    }
}
```

#### **Pipeline Multibranch Avanzado**

```groovy
// Jenkinsfile.multibranch
pipeline {
    agent none

    environment {
        APP_NAME = 'mi-aplicacion'
        DOCKER_REGISTRY = 'registry.company.com'
        SLACK_CHANNEL = '#ci-cd'
    }

    stages {
        stage('Branch Strategy') {
            steps {
                script {
                    // Configurar variables según la rama
                    switch(env.BRANCH_NAME) {
                        case 'main':
                            env.ENVIRONMENT = 'production'
                            env.DEPLOY_APPROVAL = 'true'
                            env.RUN_PERFORMANCE_TESTS = 'true'
                            break
                        case 'develop':
                            env.ENVIRONMENT = 'staging'
                            env.DEPLOY_APPROVAL = 'false'
                            env.RUN_PERFORMANCE_TESTS = 'true'
                            break
                        case ~/^feature\/.*/:
                            env.ENVIRONMENT = 'feature'
                            env.DEPLOY_APPROVAL = 'false'
                            env.RUN_PERFORMANCE_TESTS = 'false'
                            break
                        case ~/^hotfix\/.*/:
                            env.ENVIRONMENT = 'hotfix'
                            env.DEPLOY_APPROVAL = 'true'
                            env.RUN_PERFORMANCE_TESTS = 'false'
                            break
                        default:
                            env.ENVIRONMENT = 'development'
                            env.DEPLOY_APPROVAL = 'false'
                            env.RUN_PERFORMANCE_TESTS = 'false'
                    }

                    echo "Rama: ${env.BRANCH_NAME}"
                    echo "Entorno: ${env.ENVIRONMENT}"
                    echo "Requiere aprobación: ${env.DEPLOY_APPROVAL}"
                }
            }
        }

        stage('Build & Test') {
            agent {
                docker {
                    image 'node:18-alpine'
                    args '-v /var/run/docker.sock:/var/run/docker.sock'
                }
            }
            stages {
                stage('Install Dependencies') {
                    steps {
                        sh 'npm ci'
                    }
                }

                stage('Code Quality') {
                    parallel {
                        stage('Lint') {
                            steps {
                                sh 'npm run lint'
                            }
                        }
                        stage('Type Check') {
                            steps {
                                sh 'npm run type-check'
                            }
                        }
                        stage('Security Audit') {
                            steps {
                                sh 'npm audit --audit-level moderate'
                            }
                        }
                    }
                }

                stage('Tests') {
                    parallel {
                        stage('Unit Tests') {
                            steps {
                                sh 'npm run test:unit -- --coverage'
                            }
                            post {
                                always {
                                    publishTestResults testResultsPattern: 'reports/jest-junit.xml'
                                    publishCoverageReports([
                                        [
                                            reportType: 'COBERTURA',
                                            reportFile: 'coverage/cobertura-coverage.xml'
                                        ]
                                    ])
                                }
                            }
                        }

                        stage('Integration Tests') {
                            when {
                                not { changeRequest() }
                            }
                            steps {
                                sh 'npm run test:integration'
                            }
                        }

                        stage('E2E Tests') {
                            when {
                                anyOf {
                                    branch 'main'
                                    branch 'develop'
                                }
                            }
                            steps {
                                sh 'npm run test:e2e'
                            }
                            post {
                                always {
                                    publishHTML([
                                        allowMissing: false,
                                        alwaysLinkToLastBuild: true,
                                        keepAll: true,
                                        reportDir: 'e2e-reports',
                                        reportFiles: 'index.html',
                                        reportName: 'E2E Test Report'
                                    ])
                                }
                            }
                        }
                    }
                }

                stage('Build Application') {
                    steps {
                        sh 'npm run build'
                    }
                    post {
                        success {
                            archiveArtifacts artifacts: 'dist/**/*', allowEmptyArchive: true
                        }
                    }
                }
            }
        }

        stage('Performance Tests') {
            when {
                expression { env.RUN_PERFORMANCE_TESTS == 'true' }
            }
            agent {
                docker {
                    image 'loadimpact/k6:latest'
                }
            }
            steps {
                echo ' Ejecutando tests de performance...'
                sh 'k6 run --out json=performance-results.json tests/performance/load-test.js'
            }
            post {
                always {
                    archiveArtifacts artifacts: 'performance-results.json', allowEmptyArchive: true
                    publishHTML([
                        allowMissing: false,
                        alwaysLinkToLastBuild: true,
                        keepAll: true,
                        reportDir: 'performance-reports',
                        reportFiles: 'index.html',
                        reportName: 'Performance Test Report'
                    ])
                }
            }
        }

        stage('Docker Build & Push') {
            agent any
            when {
                not { changeRequest() }
            }
            steps {
                script {
                    def imageTag = "${env.BRANCH_NAME}-${env.BUILD_NUMBER}-${env.GIT_COMMIT[0..6]}"

                    echo " Construyendo imagen: ${DOCKER_REGISTRY}/${APP_NAME}:${imageTag}"

                    def image = docker.build("${DOCKER_REGISTRY}/${APP_NAME}:${imageTag}")

                    docker.withRegistry("https://${DOCKER_REGISTRY}", 'docker-registry-credentials') {
                        image.push()

                        // Taggear según la rama
                        if (env.BRANCH_NAME == 'main') {
                            image.push('latest')
                            image.push('stable')
                        } else if (env.BRANCH_NAME == 'develop') {
                            image.push('develop')
                        }
                    }

                    env.DOCKER_IMAGE_TAG = imageTag
                }
            }
        }

        stage('Security Scanning') {
            agent any
            when {
                not { changeRequest() }
            }
            parallel {
                stage('Container Scan') {
                    steps {
                        echo ' Escaneando imagen Docker...'
                        sh """
                            docker run --rm -v /var/run/docker.sock:/var/run/docker.sock \\
                                aquasec/trivy image --format json --output container-scan.json \\
                                ${DOCKER_REGISTRY}/${APP_NAME}:${env.DOCKER_IMAGE_TAG}
                        """
                    }
                    post {
                        always {
                            archiveArtifacts artifacts: 'container-scan.json', allowEmptyArchive: true
                        }
                    }
                }

                stage('SAST Scan') {
                    steps {
                        echo ' Análisis estático de seguridad...'
                        sh 'docker run --rm -v $(pwd):/src sonarqube/sonar-scanner-cli'
                    }
                }
            }
        }

        stage('Deploy') {
            when {
                not { changeRequest() }
            }
            steps {
                script {
                    if (env.DEPLOY_APPROVAL == 'true') {
                        timeout(time: 10, unit: 'MINUTES') {
                            input message: "¿Desplegar ${APP_NAME} a ${env.ENVIRONMENT}?",
                                  ok: 'Deploy!',
                                  submitterParameter: 'DEPLOYER'
                        }
                    }

                    echo " Desplegando a ${env.ENVIRONMENT}..."

                    // Deployment usando Helm
                    sh """
                        helm upgrade --install ${APP_NAME}-${env.ENVIRONMENT} ./helm/${APP_NAME} \\
                            --namespace ${env.ENVIRONMENT} \\
                            --set image.tag=${env.DOCKER_IMAGE_TAG} \\
                            --set environment=${env.ENVIRONMENT} \\
                            --wait --timeout=5m
                    """

                    // Health check
                    sh """
                        kubectl wait --for=condition=ready pod \\
                            -l app=${APP_NAME} \\
                            -n ${env.ENVIRONMENT} \\
                            --timeout=300s
                    """
                }
            }
            post {
                success {
                    echo ' Deployment exitoso!'
                    script {
                        def deployUrl = getDeploymentUrl(env.ENVIRONMENT)
                        slackSend channel: env.SLACK_CHANNEL,
                                 color: 'good',
                                 message: " ${APP_NAME} desplegado a ${env.ENVIRONMENT}\nURL: ${deployUrl}\nTag: ${env.DOCKER_IMAGE_TAG}\nPor: ${env.DEPLOYER ?: 'Automático'}"
                    }
                }
                failure {
                    echo ' Deployment falló!'
                    slackSend channel: env.SLACK_CHANNEL,
                             color: 'danger',
                             message: " Falló deployment de ${APP_NAME} a ${env.ENVIRONMENT}\nTag: ${env.DOCKER_IMAGE_TAG}"
                }
            }
        }

        stage('Post-Deploy Tests') {
            when {
                anyOf {
                    branch 'main'
                    branch 'develop'
                }
            }
            parallel {
                stage('Smoke Tests') {
                    steps {
                        echo ' Ejecutando smoke tests...'
                        script {
                            def deployUrl = getDeploymentUrl(env.ENVIRONMENT)
                            sh "newman run tests/postman/smoke-tests.json --env-var base_url=${deployUrl}"
                        }
                    }
                }

                stage('Health Check') {
                    steps {
                        echo ' Verificando health check...'
                        script {
                            def deployUrl = getDeploymentUrl(env.ENVIRONMENT)
                            timeout(time: 5, unit: 'MINUTES') {
                                waitUntil {
                                    script {
                                        def response = httpRequest url: "${deployUrl}/health", validResponseCodes: '200'
                                        return response.status == 200
                                    }
                                }
                            }
                        }
                    }
                }
            }
        }
    }

    post {
        always {
            cleanWs()
        }
        success {
            script {
                if (env.BRANCH_NAME == 'main') {
                    // Crear tag de release
                    sh """
                        git tag -a v${env.BUILD_NUMBER} -m "Release v${env.BUILD_NUMBER}"
                        git push origin v${env.BUILD_NUMBER}
                    """
                }
            }
        }
        failure {
            script {
                // Notificar a desarrolladores responsables
                def culprits = emailextrecipients([
                    [$class: 'CulpritsRecipientProvider'],
                    [$class: 'DevelopersRecipientProvider']
                ])

                if (culprits) {
                    emailext (
                        subject: " Pipeline falló: ${APP_NAME} - ${env.BRANCH_NAME}",
                        body: """
                            <h2>Pipeline Falló</h2>
                            <p><strong>Proyecto:</strong> ${APP_NAME}</p>
                            <p><strong>Rama:</strong> ${env.BRANCH_NAME}</p>
                            <p><strong>Build:</strong> ${env.BUILD_NUMBER}</p>
                            <p><strong>Commit:</strong> ${env.GIT_COMMIT}</p>
                            <p><strong>URL:</strong> ${env.BUILD_URL}</p>
                        """,
                        to: culprits,
                        mimeType: 'text/html'
                    )
                }
            }
        }
    }
}

// Función helper para obtener URL de deployment
def getDeploymentUrl(environment) {
    def urls = [
        'production': 'https://app.company.com',
        'staging': 'https://staging.app.company.com',
        'development': 'https://dev.app.company.com'
    ]
    return urls[environment] ?: 'http://localhost:8080'
}
```

### 3. Configuración de Agentes

#### **Agente Docker**

```groovy
// Configuración de agente Docker en Jenkinsfile
pipeline {
    agent none

    stages {
        stage('Build with Node') {
            agent {
                docker {
                    image 'node:18-alpine'
                    args '-v /var/run/docker.sock:/var/run/docker.sock'
                }
            }
            steps {
                sh 'npm install'
                sh 'npm run build'
            }
        }

        stage('Test with Different Versions') {
            parallel {
                stage('Node 16') {
                    agent {
                        docker { image 'node:16-alpine' }
                    }
                    steps {
                        sh 'npm test'
                    }
                }
                stage('Node 18') {
                    agent {
                        docker { image 'node:18-alpine' }
                    }
                    steps {
                        sh 'npm test'
                    }
                }
                stage('Node 20') {
                    agent {
                        docker { image 'node:20-alpine' }
                    }
                    steps {
                        sh 'npm test'
                    }
                }
            }
        }
    }
}
```

#### **Agente Kubernetes**

```yaml
# jenkins-agent-pod.yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    jenkins: agent
spec:
  containers:
  - name: jnlp
    image: jenkins/inbound-agent
    resources:
      requests:
        memory: "256Mi"
        cpu: "100m"
      limits:
        memory: "512Mi"
        cpu: "500m"
  - name: docker
    image: docker:dind
    securityContext:
      privileged: true
    volumeMounts:
    - name: docker-sock
      mountPath: /var/run/docker.sock
  - name: kubectl
    image: bitnami/kubectl
    command:
    - cat
    tty: true
  - name: helm
    image: alpine/helm
    command:
    - cat
    tty: true
  volumes:
  - name: docker-sock
    hostPath:
      path: /var/run/docker.sock
```

```groovy
// Uso del agente Kubernetes
pipeline {
    agent {
        kubernetes {
            yaml readTrusted('jenkins-agent-pod.yaml')
        }
    }

    stages {
        stage('Build') {
            steps {
                container('docker') {
                    sh 'docker build -t myapp .'
                }
            }
        }

        stage('Deploy') {
            steps {
                container('kubectl') {
                    sh 'kubectl apply -f k8s/'
                }
                container('helm') {
                    sh 'helm upgrade --install myapp ./helm'
                }
            }
        }
    }
}
```

### 4. Plugins Esenciales y Configuración

#### **Configuración de Plugins Críticos**

```groovy
// plugins.groovy - Configuración automatizada de plugins
import jenkins.model.*
import hudson.security.*

def instance = Jenkins.getInstance()

// Plugin: Blue Ocean
def blueOceanPlugin = instance.getPlugin("blueocean")
if (blueOceanPlugin) {
    println "Blue Ocean configurado"
}

// Plugin: Pipeline
def pipelinePlugin = instance.getPlugin("workflow-aggregator")
if (pipelinePlugin) {
    println "Pipeline plugin configurado"
}

// Plugin: Docker
def dockerPlugin = instance.getPlugin("docker-workflow")
if (dockerPlugin) {
    println "Docker plugin configurado"
}

// Plugin: Kubernetes
def k8sPlugin = instance.getPlugin("kubernetes")
if (k8sPlugin) {
    println "Kubernetes plugin configurado"
}

instance.save()
```

#### **Configuración de Notificaciones**

```groovy
// email-config.groovy
import jenkins.model.*
import hudson.tasks.Mailer

def instance = Jenkins.getInstance()
def descriptor = instance.getDescriptor("hudson.tasks.Mailer")

descriptor.setSmtpHost("smtp.company.com")
descriptor.setSmtpPort("587")
descriptor.setUseSsl(true)
descriptor.setCharset("UTF-8")
descriptor.setSmtpAuth("username", "password")

descriptor.save()
```

```groovy
// slack-config.groovy
import jenkins.plugins.slack.*

def instance = Jenkins.getInstance()
def slack = instance.getDescriptor(SlackNotifier.class)

slack.baseUrl = "https://company.slack.com/services/hooks/jenkins-ci/"
slack.token = "your-slack-token"
slack.room = "#ci-cd"
slack.sendAs = "Jenkins CI"

slack.save()
```

## Laboratorios Prácticos

### Lab 1: Jenkins con Docker

**Objetivo**: Configurar Jenkins usando Docker y crear primer pipeline.

```bash
# 1. Crear docker-compose.yml
cat > docker-compose.yml << 'EOF'
version: '3.8'
services:
  jenkins:
    image: jenkins/jenkins:lts
    ports:
      - "8080:8080"
      - "50000:50000"
    volumes:
      - jenkins_home:/var/jenkins_home
    environment:
      - JAVA_OPTS=-Djenkins.install.runSetupWizard=false

volumes:
  jenkins_home:
EOF

# 2. Iniciar Jenkins
docker-compose up -d

# 3. Obtener password inicial
docker-compose exec jenkins cat /var/jenkins_home/secrets/initialAdminPassword

# 4. Acceder a http://localhost:8080
```

### Lab 2: Pipeline Multibranch

**Objetivo**: Crear un pipeline que se ejecute diferente según la rama.

1. **Crear repositorio con ramas**:

```bash
git init myapp
cd myapp
echo "# Mi App" > README.md
git add . && git commit -m "Initial commit"

git checkout -b develop
echo "console.log('develop');" > app.js
git add . && git commit -m "Add develop app"

git checkout -b feature/new-feature
echo "console.log('feature');" > feature.js
git add . && git commit -m "Add feature"
```

2. **Crear Jenkinsfile**:

```groovy
pipeline {
    agent any
    stages {
        stage('Branch Detection') {
            steps {
                script {
                    echo "Branch: ${env.BRANCH_NAME}"
                    switch(env.BRANCH_NAME) {
                        case 'main':
                            echo " Production deployment"
                            break
                        case 'develop':
                            echo " Staging deployment"
                            break
                        case ~/^feature\/.*/:
                            echo " Feature branch testing"
                            break
                        default:
                            echo " Development branch"
                    }
                }
            }
        }
    }
}
```

### Lab 3: Integración con Git

**Objetivo**: Configurar webhooks y triggers automáticos.

1. **Configurar webhook en GitHub**:
   - Ir a Settings → Webhooks
   - URL: `http://jenkins-url/github-webhook/`
   - Content type: `application/json`
   - Events: Push, Pull requests

2. **Configurar job multibranch**:
   - New Item → Multibranch Pipeline
   - Branch Sources → GitHub
   - Configurar credenciales y repositorio

## Ejercicios Prácticos

### Ejercicio 1: Pipeline de Microservicios

Crear un pipeline que:

- Detecte cambios en múltiples servicios
- Construya solo los servicios modificados
- Ejecute tests de integración entre servicios
- Despliegue usando blue-green strategy

### Ejercicio 2: Pipeline de Seguridad

Desarrollar un pipeline que incluya:

- Análisis de dependencias vulnerables
- Scan de código con SonarQube
- Scan de imágenes Docker con Trivy
- Tests de penetración automatizados

### Ejercicio 3: Disaster Recovery

Implementar:

- Backup automático de configuración Jenkins
- Restore automático desde backup
- Pipeline de recuperación de desastres
- Documentación de procedimientos

## Troubleshooting

### Problemas Comunes

1. **Out of Memory**:

```bash
# Aumentar memoria JVM
JAVA_OPTS="-Xmx2048m -XX:MaxPermSize=512m"
```

2. **Workspace Cleanup**:

```groovy
// En pipeline
post {
    always {
        cleanWs()
    }
}
```

3. **Plugin Conflicts**:

```bash
# Verificar plugins
curl -s http://localhost:8080/pluginManager/api/json?depth=1
```

### Monitoreo y Logs

```bash
# Logs de Jenkins
docker-compose logs -f jenkins

# Logs específicos
tail -f /var/jenkins_home/logs/jenkins.log

# Métricas de performance
curl http://localhost:8080/metrics
```

## Recursos Adicionales

- [Jenkins Documentation](https://www.jenkins.io/doc/)
- [Pipeline Syntax Reference](https://www.jenkins.io/doc/book/pipeline/syntax/)
- [Plugin Index](https://plugins.jenkins.io/)
- [Best Practices](https://www.jenkins.io/doc/book/pipeline/pipeline-best-practices/)

## Checklist de Completado

- [ ] Instalado Jenkins con Docker
- [ ] Configurado seguridad básica
- [ ] Creado pipeline declarativo básico
- [ ] Implementado pipeline multibranch
- [ ] Configurado agentes (Docker/Kubernetes)
- [ ] Integrado con Git/GitHub
- [ ] Configurado notificaciones (email/Slack)
- [ ] Implementado quality gates
- [ ] Configurado plugins esenciales
- [ ] Creado jobs de deployment automatizado

---

**Excelente!**Ahora dominas Jenkins para CI/CD profesional. Continúa con el siguiente módulo para explorar plataformas CI/CD modernas como GitHub Actions y GitLab CI/CD.
