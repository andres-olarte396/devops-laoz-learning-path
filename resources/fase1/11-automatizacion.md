# Automatización en DevOps

## Introducción

La **automatización** es uno de los pilares fundamentales de DevOps. Consiste en usar tecnología para realizar tareas con mínima intervención humana, aumentando la eficiencia, reduciendo errores y permitiendo que los equipos se enfoquen en trabajo de mayor valor.

## ¿Qué es la Automatización?

### Definición

La automatización en DevOps es el uso de herramientas y scripts para ejecutar tareas repetitivas de forma automática, desde la compilación y prueba de código hasta el despliegue y monitoreo de aplicaciones.

### Objetivos Principales

1. **Reducir errores humanos**
2. **Aumentar velocidad de entrega**
3. **Mejorar consistencia**
4. **Liberar tiempo para tareas estratégicas**
5. **Habilitar escalabilidad**

## Áreas de Automatización en DevOps

### 1. **Automatización de Build (Construcción)**

#### **Compilación Automática**

```yaml
# GitHub Actions - Build automático
name: Build Application

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

jobs:
  build:
    runs-on: ubuntu-latest
    
    steps:
    - uses: actions/checkout@v3
    
    - name: Setup .NET
      uses: actions/setup-dotnet@v3
      with:
        dotnet-version: '6.0.x'
    
    - name: Restore dependencies
      run: dotnet restore
    
    - name: Build application
      run: dotnet build --no-restore --configuration Release
    
    - name: Upload build artifacts
      uses: actions/upload-artifact@v3
      with:
        name: build-artifacts
        path: ./bin/Release/
```

#### **Build Script Personalizado**

```bash
#!/bin/bash
# build.sh - Script de build automático

set -e  # Salir si cualquier comando falla

echo "🔨 Iniciando proceso de build..."

# Limpiar build anterior
echo "🧹 Limpiando artifacts anteriores..."
rm -rf ./dist ./build

# Instalar dependencias
echo "📦 Instalando dependencias..."
npm ci

# Ejecutar linter
echo "🔍 Ejecutando linter..."
npm run lint

# Ejecutar tests
echo "🧪 Ejecutando tests..."
npm run test:coverage

# Compilar aplicación
echo "⚙️ Compilando aplicación..."
npm run build

# Verificar que los archivos se generaron
if [ ! -d "./dist" ]; then
  echo "❌ Error: directorio dist no encontrado"
  exit 1
fi

echo "✅ Build completado exitosamente!"
```

### 2. **Automatización de Testing**

#### **Tests Unitarios Automatizados**

```csharp
// Ejemplo en .NET con NUnit
[TestFixture]
public class CalculatorTests
{
    private Calculator _calculator;

    [SetUp]
    public void Setup()
    {
        _calculator = new Calculator();
    }

    [Test]
    [TestCase(2, 3, 5)]
    [TestCase(-1, 1, 0)]
    [TestCase(0, 0, 0)]
    public void Add_ShouldReturnCorrectSum(int a, int b, int expected)
    {
        // Act
        var result = _calculator.Add(a, b);

        // Assert
        Assert.AreEqual(expected, result);
    }
}
```

#### **Pipeline de Testing**

```yaml
# Azure DevOps Pipeline
trigger:
- main

pool:
  vmImage: 'ubuntu-latest'

stages:
- stage: Test
  displayName: 'Testing Stage'
  jobs:
  - job: UnitTests
    displayName: 'Unit Tests'
    steps:
    - task: DotNetCoreCLI@2
      displayName: 'Run Unit Tests'
      inputs:
        command: 'test'
        projects: '**/*Tests.csproj'
        arguments: '--configuration Release --collect:"XPlat Code Coverage"'
    
    - task: PublishCodeCoverageResults@1
      displayName: 'Publish Code Coverage'
      inputs:
        codeCoverageTool: 'Cobertura'
        summaryFileLocation: '$(Agent.TempDirectory)/**/coverage.cobertura.xml'

  - job: IntegrationTests
    displayName: 'Integration Tests'
    dependsOn: UnitTests
    steps:
    - script: |
        docker-compose up -d database
        sleep 30
        dotnet test IntegrationTests/ --configuration Release
        docker-compose down
      displayName: 'Run Integration Tests'
```

### 3. **Automatización de Deployment**

#### **Deployment Script**

```bash
#!/bin/bash
# deploy.sh - Script de deployment automático

ENVIRONMENT=$1
VERSION=$2

if [ -z "$ENVIRONMENT" ] || [ -z "$VERSION" ]; then
    echo "Uso: ./deploy.sh <environment> <version>"
    echo "Ejemplo: ./deploy.sh production v1.2.3"
    exit 1
fi

echo "🚀 Desplegando versión $VERSION en $ENVIRONMENT..."

# Validar ambiente
case $ENVIRONMENT in
    "development"|"staging"|"production")
        echo "✅ Ambiente válido: $ENVIRONMENT"
        ;;
    *)
        echo "❌ Ambiente inválido: $ENVIRONMENT"
        exit 1
        ;;
esac

# Backup de la versión actual
echo "💾 Creando backup..."
kubectl get deployment app -o yaml > backup-$(date +%Y%m%d-%H%M%S).yaml

# Actualizar imagen de contenedor
echo "🔄 Actualizando deployment..."
kubectl set image deployment/app app=myapp:$VERSION

# Esperar que el deployment complete
echo "⏳ Esperando deployment..."
kubectl rollout status deployment/app --timeout=600s

# Verificar health check
echo "🏥 Verificando health check..."
for i in {1..10}; do
    if curl -f http://app.$ENVIRONMENT.local/health; then
        echo "✅ Deployment exitoso!"
        exit 0
    fi
    echo "⏳ Esperando health check... ($i/10)"
    sleep 30
done

echo "❌ Health check falló, realizando rollback..."
kubectl rollout undo deployment/app
exit 1
```

### 4. **Automatización de Infraestructura**

#### **Terraform Automation**

```hcl
# terraform/main.tf
resource "aws_instance" "web_server" {
  ami           = var.ami_id
  instance_type = var.instance_type
  key_name      = var.key_name

  user_data = templatefile("${path.module}/user_data.sh", {
    app_version = var.app_version
    environment = var.environment
  })

  tags = {
    Name        = "web-server-${var.environment}"
    Environment = var.environment
    ManagedBy   = "Terraform"
  }
}

resource "aws_security_group" "web_sg" {
  name_description = "Security group for web server"

  ingress {
    from_port   = 80
    to_port     = 80
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  ingress {
    from_port   = 443
    to_port     = 443
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }
}
```

#### **User Data Script**

```bash
#!/bin/bash
# user_data.sh - Script de inicialización automática

# Variables desde Terraform
APP_VERSION="${app_version}"
ENVIRONMENT="${environment}"

# Actualizar sistema
yum update -y

# Instalar Docker
yum install -y docker
systemctl start docker
systemctl enable docker

# Agregar usuario ec2-user al grupo docker
usermod -a -G docker ec2-user

# Descargar y ejecutar aplicación
docker pull myapp:$APP_VERSION
docker run -d \
  --name myapp \
  -p 80:8080 \
  -e ENVIRONMENT=$ENVIRONMENT \
  myapp:$APP_VERSION

# Configurar log rotation
cat > /etc/logrotate.d/myapp << EOF
/var/log/myapp.log {
    daily
    rotate 7
    compress
    missingok
    notifempty
}
EOF

echo "✅ Servidor configurado correctamente"
```

## Herramientas de Automatización

### 1. **Jenkins**

#### **Jenkinsfile Declarativo**

```groovy
pipeline {
    agent any
    
    environment {
        REGISTRY = 'docker-registry.company.com'
        IMAGE_NAME = 'myapp'
        KUBECONFIG = credentials('kubeconfig')
    }
    
    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        
        stage('Build') {
            steps {
                script {
                    def buildNumber = env.BUILD_NUMBER
                    sh """
                        docker build -t ${REGISTRY}/${IMAGE_NAME}:${buildNumber} .
                        docker build -t ${REGISTRY}/${IMAGE_NAME}:latest .
                    """
                }
            }
        }
        
        stage('Test') {
            parallel {
                stage('Unit Tests') {
                    steps {
                        sh 'docker run --rm ${REGISTRY}/${IMAGE_NAME}:${BUILD_NUMBER} npm test'
                    }
                }
                stage('Security Scan') {
                    steps {
                        sh 'docker run --rm -v $(pwd):/app clair-scanner ${REGISTRY}/${IMAGE_NAME}:${BUILD_NUMBER}'
                    }
                }
            }
        }
        
        stage('Push') {
            when {
                branch 'main'
            }
            steps {
                script {
                    docker.withRegistry("https://${REGISTRY}", 'docker-registry-creds') {
                        sh """
                            docker push ${REGISTRY}/${IMAGE_NAME}:${BUILD_NUMBER}
                            docker push ${REGISTRY}/${IMAGE_NAME}:latest
                        """
                    }
                }
            }
        }
        
        stage('Deploy') {
            when {
                branch 'main'
            }
            steps {
                sh """
                    helm upgrade --install myapp ./helm/myapp \
                        --set image.tag=${BUILD_NUMBER} \
                        --set environment=production \
                        --namespace production
                """
            }
        }
    }
    
    post {
        always {
            cleanWs()
        }
        success {
            slackSend channel: '#deployments',
                      color: 'good',
                      message: "✅ Deployment exitoso: ${env.JOB_NAME} - ${env.BUILD_NUMBER}"
        }
        failure {
            slackSend channel: '#deployments',
                      color: 'danger',
                      message: "❌ Deployment falló: ${env.JOB_NAME} - ${env.BUILD_NUMBER}"
        }
    }
}
```

### 2. **GitHub Actions**

#### **Workflow Completo**

```yaml
name: CI/CD Pipeline

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

env:
  REGISTRY: ghcr.io
  IMAGE_NAME: ${{ github.repository }}

jobs:
  test:
    runs-on: ubuntu-latest
    
    services:
      postgres:
        image: postgres:13
        env:
          POSTGRES_PASSWORD: postgres
        options: >-
          --health-cmd pg_isready
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5
    
    steps:
    - uses: actions/checkout@v3
    
    - name: Setup Node.js
      uses: actions/setup-node@v3
      with:
        node-version: '18'
        cache: 'npm'
    
    - name: Install dependencies
      run: npm ci
    
    - name: Run linter
      run: npm run lint
    
    - name: Run tests
      run: npm test -- --coverage
      env:
        DATABASE_URL: postgresql://postgres:postgres@localhost:5432/test
    
    - name: Upload coverage to Codecov
      uses: codecov/codecov-action@v3

  build-and-push:
    needs: test
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main'
    
    permissions:
      contents: read
      packages: write
    
    steps:
    - uses: actions/checkout@v3
    
    - name: Log in to Container Registry
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
          type=sha,prefix={{branch}}-
          type=raw,value=latest,enable={{is_default_branch}}
    
    - name: Build and push Docker image
      uses: docker/build-push-action@v4
      with:
        context: .
        push: true
        tags: ${{ steps.meta.outputs.tags }}
        labels: ${{ steps.meta.outputs.labels }}

  deploy:
    needs: build-and-push
    runs-on: ubuntu-latest
    environment: production
    
    steps:
    - uses: actions/checkout@v3
    
    - name: Deploy to Kubernetes
      run: |
        echo "${{ secrets.KUBECONFIG }}" | base64 -d > kubeconfig
        export KUBECONFIG=kubeconfig
        
        kubectl set image deployment/myapp \
          myapp=${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}:main-${{ github.sha }}
        
        kubectl rollout status deployment/myapp --timeout=300s
```

### 3. **Ansible**

#### **Playbook de Deployment**

```yaml
---
# deploy.yml
- name: Deploy Application
  hosts: webservers
  become: yes
  vars:
    app_version: "{{ version | default('latest') }}"
    app_port: 8080
    
  tasks:
    - name: Update system packages
      yum:
        name: "*"
        state: latest
      
    - name: Install Docker
      yum:
        name: docker
        state: present
        
    - name: Start Docker service
      service:
        name: docker
        state: started
        enabled: yes
        
    - name: Pull application image
      docker_image:
        name: "myapp:{{ app_version }}"
        source: pull
        
    - name: Stop existing container
      docker_container:
        name: myapp
        state: absent
      ignore_errors: yes
      
    - name: Start new container
      docker_container:
        name: myapp
        image: "myapp:{{ app_version }}"
        state: started
        restart_policy: always
        ports:
          - "80:{{ app_port }}"
        env:
          ENVIRONMENT: production
          
    - name: Wait for application to be ready
      uri:
        url: "http://{{ inventory_hostname }}/health"
        method: GET
        status_code: 200
      register: result
      until: result.status == 200
      retries: 30
      delay: 10
      
    - name: Verify deployment
      debug:
        msg: "Application deployed successfully on {{ inventory_hostname }}"
```

#### **Inventory File**

```ini
# inventory/production
[webservers]
web1 ansible_host=10.0.1.10
web2 ansible_host=10.0.1.11
web3 ansible_host=10.0.1.12

[webservers:vars]
ansible_user=ec2-user
ansible_ssh_private_key_file=~/.ssh/production-key.pem
```

## Automatización de Monitoreo

### 1. **Health Checks Automatizados**

```python
#!/usr/bin/env python3
# health_monitor.py

import requests
import smtplib
import time
from email.mime.text import MIMEText
from datetime import datetime

class HealthMonitor:
    def __init__(self, services):
        self.services = services
        
    def check_service(self, service):
        try:
            response = requests.get(
                service['url'], 
                timeout=service.get('timeout', 30)
            )
            
            if response.status_code == 200:
                return {
                    'name': service['name'],
                    'status': 'healthy',
                    'response_time': response.elapsed.total_seconds()
                }
            else:
                return {
                    'name': service['name'],
                    'status': 'unhealthy',
                    'error': f'HTTP {response.status_code}'
                }
                
        except requests.RequestException as e:
            return {
                'name': service['name'],
                'status': 'unhealthy',
                'error': str(e)
            }
    
    def send_alert(self, unhealthy_services):
        if not unhealthy_services:
            return
            
        subject = f"🚨 ALERT: {len(unhealthy_services)} service(s) down"
        
        body = "The following services are unhealthy:\n\n"
        for service in unhealthy_services:
            body += f"- {service['name']}: {service['error']}\n"
        
        # Enviar email (configurar SMTP)
        print(f"ALERT: {subject}")
        print(body)
    
    def monitor(self):
        print(f"[{datetime.now()}] Starting health check...")
        
        results = []
        unhealthy = []
        
        for service in self.services:
            result = self.check_service(service)
            results.append(result)
            
            if result['status'] == 'unhealthy':
                unhealthy.append(result)
            
            print(f"  {result['name']}: {result['status']}")
        
        if unhealthy:
            self.send_alert(unhealthy)
        
        return results

# Configuración
services = [
    {
        'name': 'Web App',
        'url': 'https://myapp.com/health',
        'timeout': 10
    },
    {
        'name': 'API Gateway',
        'url': 'https://api.myapp.com/health',
        'timeout': 15
    },
    {
        'name': 'Database',
        'url': 'https://db.myapp.com/health',
        'timeout': 20
    }
]

if __name__ == '__main__':
    monitor = HealthMonitor(services)
    
    while True:
        monitor.monitor()
        time.sleep(300)  # Check every 5 minutes
```

### 2. **Automatización de Logs**

```bash
#!/bin/bash
# log_rotation.sh - Rotación automática de logs

LOG_DIR="/var/log/myapp"
RETENTION_DAYS=30
BACKUP_DIR="/var/log/backup"

# Crear directorio de backup si no existe
mkdir -p $BACKUP_DIR

# Comprimir logs antiguos
find $LOG_DIR -name "*.log" -mtime +1 -exec gzip {} \;

# Mover logs comprimidos al backup
find $LOG_DIR -name "*.gz" -exec mv {} $BACKUP_DIR/ \;

# Eliminar backups antiguos
find $BACKUP_DIR -name "*.gz" -mtime +$RETENTION_DAYS -delete

# Reiniciar el servicio para crear nuevos logs
systemctl reload myapp

echo "$(date): Log rotation completed"
```

## Automatización de Seguridad

### 1. **Security Scanning**

```yaml
# security-scan.yml - GitHub Actions
name: Security Scan

on:
  push:
  schedule:
    - cron: '0 2 * * *'  # Daily at 2 AM

jobs:
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
    
    - name: Run CodeQL Analysis
      uses: github/codeql-action/analyze@v2
      with:
        languages: javascript
    
    - name: Run dependency check
      run: |
        npm audit --audit-level=high
        npm audit fix
```

### 2. **Automated Backup**

```bash
#!/bin/bash
# backup.sh - Script de backup automatizado

DATABASE_NAME="myapp_prod"
BACKUP_DIR="/backups"
S3_BUCKET="myapp-backups"
RETENTION_DAYS=7

# Crear timestamp
TIMESTAMP=$(date +%Y%m%d_%H%M%S)
BACKUP_FILE="$BACKUP_DIR/${DATABASE_NAME}_${TIMESTAMP}.sql"

# Crear directorio de backup
mkdir -p $BACKUP_DIR

# Realizar backup de la base de datos
echo "Creating database backup..."
pg_dump $DATABASE_NAME > $BACKUP_FILE

# Comprimir backup
echo "Compressing backup..."
gzip $BACKUP_FILE

# Subir a S3
echo "Uploading to S3..."
aws s3 cp "${BACKUP_FILE}.gz" "s3://$S3_BUCKET/"

# Limpiar backups locales antiguos
echo "Cleaning old local backups..."
find $BACKUP_DIR -name "*.gz" -mtime +$RETENTION_DAYS -delete

# Limpiar backups de S3 antiguos
echo "Cleaning old S3 backups..."
aws s3 ls "s3://$S3_BUCKET/" | while read -r line; do
    backup_date=$(echo $line | awk '{print $1}')
    backup_file=$(echo $line | awk '{print $4}')
    
    if [[ $(date -d "$backup_date" +%s) -lt $(date -d "$RETENTION_DAYS days ago" +%s) ]]; then
        aws s3 rm "s3://$S3_BUCKET/$backup_file"
    fi
done

echo "Backup completed successfully: ${BACKUP_FILE}.gz"
```

## Mejores Prácticas de Automatización

### 1. **Principios de Automatización**

#### **Idempotencia**
```bash
# ✅ Idempotente
if ! command -v docker &> /dev/null; then
    curl -fsSL https://get.docker.com -o get-docker.sh
    sh get-docker.sh
fi

# ❌ No idempotente
curl -fsSL https://get.docker.com -o get-docker.sh
sh get-docker.sh
```

#### **Fail Fast**
```bash
#!/bin/bash
set -euo pipefail  # Salir ante cualquier error

# Validaciones al inicio
if [ -z "${ENVIRONMENT:-}" ]; then
    echo "Error: ENVIRONMENT variable is required"
    exit 1
fi

if [ ! -f "config/${ENVIRONMENT}.conf" ]; then
    echo "Error: Configuration file not found"
    exit 1
fi
```

### 2. **Logging y Monitoreo**

```python
import logging
import time
from functools import wraps

# Configurar logging
logging.basicConfig(
    level=logging.INFO,
    format='%(asctime)s - %(name)s - %(levelname)s - %(message)s'
)
logger = logging.getLogger(__name__)

def monitor_execution(func):
    """Decorator para monitorear ejecución de funciones"""
    @wraps(func)
    def wrapper(*args, **kwargs):
        start_time = time.time()
        logger.info(f"Starting {func.__name__}")
        
        try:
            result = func(*args, **kwargs)
            execution_time = time.time() - start_time
            logger.info(f"Completed {func.__name__} in {execution_time:.2f}s")
            return result
            
        except Exception as e:
            execution_time = time.time() - start_time
            logger.error(f"Failed {func.__name__} after {execution_time:.2f}s: {e}")
            raise
            
    return wrapper

@monitor_execution
def deploy_application(version):
    """Deploy application with monitoring"""
    logger.info(f"Deploying version {version}")
    # Deployment logic here
    pass
```

### 3. **Testing de Automatización**

```python
# test_automation.py
import unittest
from unittest.mock import patch, Mock
import automation_scripts

class TestAutomation(unittest.TestCase):
    
    @patch('automation_scripts.subprocess.run')
    def test_docker_build_success(self, mock_run):
        # Arrange
        mock_run.return_value = Mock(returncode=0)
        
        # Act
        result = automation_scripts.build_docker_image('myapp', 'v1.0.0')
        
        # Assert
        self.assertTrue(result)
        mock_run.assert_called_once_with(
            ['docker', 'build', '-t', 'myapp:v1.0.0', '.'],
            capture_output=True,
            text=True
        )
    
    def test_health_check_validation(self):
        # Test que el health check valida URLs correctamente
        valid_urls = [
            'https://api.example.com/health',
            'http://localhost:8080/health'
        ]
        
        for url in valid_urls:
            self.assertTrue(automation_scripts.is_valid_health_url(url))
```

## Herramientas de Automatización

### 1. **Orchestration Tools**

- **Jenkins:** Pipeline automation
- **GitHub Actions:** Git-based automation
- **GitLab CI/CD:** Integrated DevOps platform
- **Azure DevOps:** Microsoft ecosystem
- **CircleCI:** Cloud-native CI/CD

### 2. **Configuration Management**

- **Ansible:** Agentless automation
- **Puppet:** Declarative configuration
- **Chef:** Infrastructure as code
- **SaltStack:** Event-driven automation

### 3. **Monitoring and Alerting**

- **Prometheus + Grafana:** Metrics and visualization
- **ELK Stack:** Logs and analytics
- **Datadog:** APM and monitoring
- **PagerDuty:** Incident management

## Ejercicios Prácticos

### Ejercicio 1: Pipeline Básico
Crear un pipeline que:
1. Compile una aplicación
2. Ejecute tests unitarios
3. Genere reporte de cobertura
4. Publique artefactos

### Ejercicio 2: Deployment Automation
Automatizar deployment con:
1. Blue-green deployment
2. Health checks
3. Rollback automático en caso de falla
4. Notificaciones

### Ejercicio 3: Infrastructure Automation
Crear scripts para:
1. Provisionar infraestructura con Terraform
2. Configurar servidores con Ansible
3. Monitorear recursos
4. Cleanup automático

## Recursos Adicionales

### Documentación
- [Jenkins Documentation](https://www.jenkins.io/doc/)
- [GitHub Actions Documentation](https://docs.github.com/en/actions)
- [Ansible Documentation](https://docs.ansible.com/)

### Tutoriales
- [DevOps Automation Best Practices](https://docs.microsoft.com/en-us/azure/devops/learn/)
- [CI/CD Pipeline Tutorials](https://www.redhat.com/en/topics/devops/what-cicd-pipeline)

---

## Próximos Pasos

Una vez que domines la automatización básica, estarás listo para:

1. **Automatización avanzada:** Orquestación compleja y workflows
2. **Infrastructure as Code:** Terraform, CloudFormation
3. **Containerización:** Docker y Kubernetes automation
4. **Monitoring automation:** Observabilidad y alerting

¡La automatización es el corazón de DevOps y la clave para entregar software de calidad de forma rápida y confiable!