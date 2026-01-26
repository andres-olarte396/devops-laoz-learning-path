# Infraestructura como Código (IaC)

## Introducción

La **Infraestructura como Código (Infrastructure as Code - IaC)** es una práctica fundamental en DevOps que permite gestionar y aprovisionar infraestructura de TI a través de código legible por máquinas, en lugar de procesos manuales.

## ¿Qué es Infrastructure as Code?

### Definición

IaC es el proceso de gestionar y aprovisionar centros de datos informáticos a través de archivos de definición legibles por máquinas, en lugar de configuración física de hardware o herramientas de configuración interactivas.

### Principios Fundamentales

1. **Declarativo vs Imperativo**
   - **Declarativo:** Defines el estado deseado
   - **Imperativo:** Defines los pasos para llegar al estado deseado

2. **Versionado:** Todo el código de infraestructura se versiona
3. **Reproducibilidad:** Misma configuración, mismo resultado
4. **Automatización:** Sin intervención manual

## Beneficios de IaC

### 1. **Consistencia y Estandarización**

```yaml
# Ejemplo: Mismo servidor en diferentes ambientes
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  environment: "{{ .Values.environment }}"
  database_url: "{{ .Values.database.url }}"
```

### 2. **Versionado y Auditabilidad**

```bash
# Historial completo de cambios en infraestructura
git log --oneline infrastructure/
# a1b2c3d Agregar balanceador de carga
# d4e5f6g Actualizar configuración de base de datos
# g7h8i9j Crear entorno de staging
```

### 3. **Reutilización**

```hcl
# Terraform module reutilizable
module "web_server" {
  source = "./modules/web-server"
  
  instance_type = "t3.micro"
  environment   = "production"
  project_name  = "my-app"
}
```

### 4. **Recuperación ante Desastres**

```bash
# Recrear infraestructura completa en minutos
terraform plan
terraform apply
```

## Herramientas Principales de IaC

### 1. **Terraform**

**Características:**

- Multi-cloud (AWS, Azure, GCP, etc.)
- Lenguaje declarativo (HCL)
- State management
- Plan before apply

**Ejemplo básico:**

```hcl
# main.tf
provider "aws" {
  region = "us-west-2"
}

resource "aws_instance" "web_server" {
  ami           = "ami-0c55b159cbfafe1d0"
  instance_type = "t3.micro"
  
  tags = {
    Name        = "WebServer"
    Environment = "Production"
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
  
  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }
}
```

### 2. **AWS CloudFormation**

**Características:**

- Nativo de AWS
- Formato JSON/YAML
- Rollback automático
- Stack management

**Ejemplo básico:**

```yaml
# template.yaml
AWSTemplateFormatVersion: '2010-09-09'
Description: 'Web server with security group'

Resources:
  WebServerInstance:
    Type: AWS::EC2::Instance
    Properties:
      ImageId: ami-0c55b159cbfafe1d0
      InstanceType: t3.micro
      SecurityGroupIds:
        - !Ref WebSecurityGroup
      Tags:
        - Key: Name
          Value: WebServer

  WebSecurityGroup:
    Type: AWS::EC2::SecurityGroup
    Properties:
      GroupDescription: Security group for web server
      SecurityGroupIngress:
        - IpProtocol: tcp
          FromPort: 80
          ToPort: 80
          CidrIp: 0.0.0.0/0

Outputs:
  InstanceId:
    Description: Instance ID of the web server
    Value: !Ref WebServerInstance
```

### 3. **Azure Resource Manager (ARM) Templates**

**Ejemplo básico:**

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "vmName": {
      "type": "string",
      "defaultValue": "webserver"
    }
  },
  "resources": [
    {
      "type": "Microsoft.Compute/virtualMachines",
      "apiVersion": "2021-03-01",
      "name": "[parameters('vmName')]",
      "location": "[resourceGroup().location]",
      "properties": {
        "hardwareProfile": {
          "vmSize": "Standard_B1s"
        },
        "osProfile": {
          "computerName": "[parameters('vmName')]",
          "adminUsername": "azureuser"
        }
      }
    }
  ]
}
```

### 4. **Pulumi**

**Características:**

- Uso de lenguajes de programación reales (TypeScript, Python, Go, etc.)
- Mismo código para infraestructura y aplicación
- State management integrado

**Ejemplo en TypeScript:**

```typescript
import * as aws from "@pulumi/aws";

const webServer = new aws.ec2.Instance("webserver", {
    instanceType: "t3.micro",
    ami: "ami-0c55b159cbfafe1d0",
    tags: {
        Name: "WebServer",
        Environment: "Production"
    }
});

const securityGroup = new aws.ec2.SecurityGroup("web-sg", {
    description: "Security group for web server",
    ingress: [{
        protocol: "tcp",
        fromPort: 80,
        toPort: 80,
        cidrBlocks: ["0.0.0.0/0"]
    }]
});

export const instanceId = webServer.id;
export const securityGroupId = securityGroup.id;
```

## Conceptos Clave

### 1. **State Management**

El **estado** es la representación actual de tu infraestructura.

#### **Terraform State**

```bash
# Ver estado actual
terraform show

# Listar recursos en estado
terraform state list

# Importar recurso existente
terraform import aws_instance.web_server i-1234567890abcdef0

# Refrescar estado
terraform refresh
```

#### **State Backends**

```hcl
# Backend S3 para Terraform
terraform {
  backend "s3" {
    bucket = "my-terraform-state"
    key    = "prod/terraform.tfstate"
    region = "us-west-2"
  }
}
```

### 2. **Idempotencia**

La misma configuración aplicada múltiples veces produce el mismo resultado.

```bash
# Primera ejecución: crea recursos
terraform apply

# Segunda ejecución: sin cambios
terraform apply
# No changes. Infrastructure is up-to-date.
```

### 3. **Drift Detection**

Detectar cuando la infraestructura real difiere del código.

```bash
# Detectar drift
terraform plan

# Restaurar al estado deseado
terraform apply
```

## Mejores Prácticas

### 1. **Estructura de Proyecto**

```text
infrastructure/
├── environments/
│   ├── dev/
│   │   ├── main.tf
│   │   ├── variables.tf
│   │   └── terraform.tfvars
│   ├── staging/
│   └── prod/
├── modules/
│   ├── network/
│   ├── compute/
│   └── database/
├── shared/
│   └── backend.tf
└── README.md
```

### 2. **Uso de Variables**

```hcl
# variables.tf
variable "environment" {
  description = "Environment name"
  type        = string
  validation {
    condition     = contains(["dev", "staging", "prod"], var.environment)
    error_message = "Environment must be dev, staging, or prod."
  }
}

variable "instance_count" {
  description = "Number of instances"
  type        = number
  default     = 1
}

# terraform.tfvars
environment    = "prod"
instance_count = 3
```

### 3. **Modularización**

```hcl
# modules/web-app/main.tf
resource "aws_instance" "app" {
  count         = var.instance_count
  ami           = var.ami_id
  instance_type = var.instance_type
  
  tags = merge(var.common_tags, {
    Name = "${var.app_name}-${count.index + 1}"
  })
}

# modules/web-app/variables.tf
variable "instance_count" {
  description = "Number of instances to create"
  type        = number
  default     = 1
}

variable "ami_id" {
  description = "AMI ID for instances"
  type        = string
}

# Usage in main.tf
module "web_app" {
  source = "./modules/web-app"
  
  instance_count = 2
  ami_id         = "ami-0c55b159cbfafe1d0"
  instance_type  = "t3.micro"
  app_name       = "my-app"
  
  common_tags = {
    Environment = "production"
    Project     = "my-project"
  }
}
```

### 4. **Remote State**

```hcl
# backend.tf
terraform {
  backend "s3" {
    bucket         = "company-terraform-state"
    key            = "environments/prod/terraform.tfstate"
    region         = "us-west-2"
    encrypt        = true
    dynamodb_table = "terraform-locks"
  }
}
```

### 5. **Resource Tagging**

```hcl
locals {
  common_tags = {
    Environment   = var.environment
    Project       = var.project_name
    Owner         = var.owner
    CreatedBy     = "Terraform"
    CreatedDate   = formatdate("YYYY-MM-DD", timestamp())
  }
}

resource "aws_instance" "web" {
  ami           = var.ami_id
  instance_type = var.instance_type
  
  tags = merge(local.common_tags, {
    Name = "${var.project_name}-web-${var.environment}"
    Type = "WebServer"
  })
}
```

## Workflows de IaC

### 1. **Desarrollo Local**

```bash
# 1. Escribir código IaC
vim main.tf

# 2. Formatear código
terraform fmt

# 3. Validar sintaxis
terraform validate

# 4. Planificar cambios
terraform plan

# 5. Aplicar cambios
terraform apply
```

### 2. **CI/CD Pipeline**

```yaml
# .github/workflows/terraform.yml
name: Terraform

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  terraform:
    runs-on: ubuntu-latest
    
    steps:
    - uses: actions/checkout@v3
    
    - name: Setup Terraform
      uses: hashicorp/setup-terraform@v2
      with:
        terraform_version: 1.5.0
    
    - name: Terraform Format
      run: terraform fmt -check
    
    - name: Terraform Init
      run: terraform init
    
    - name: Terraform Validate
      run: terraform validate
    
    - name: Terraform Plan
      run: terraform plan
      
    - name: Terraform Apply
      if: github.ref == 'refs/heads/main'
      run: terraform apply -auto-approve
```

### 3. **GitOps for Infrastructure**

```bash
# Estructura GitOps
gitops-infrastructure/
├── clusters/
│   ├── production/
│   └── staging/
├── apps/
│   ├── monitoring/
│   └── ingress/
└── infrastructure/
    ├── terraform/
    └── helm/
```

## Seguridad en IaC

### 1. **Secret Management**

```hcl
# ❌ MAL: Hardcoded secrets
resource "aws_db_instance" "main" {
  password = "super-secret-password"  # ¡Nunca hagas esto!
}

# ✅ BIEN: Using AWS Secrets Manager
data "aws_secretsmanager_secret_version" "db_password" {
  secret_id = "prod/db/password"
}

resource "aws_db_instance" "main" {
  password = data.aws_secretsmanager_secret_version.db_password.secret_string
}
```

### 2. **Policy as Code**

```hcl
# Sentinel policy (Terraform Cloud/Enterprise)
import "tfplan"

main = rule {
  all tfplan.resource_changes as _, rc {
    rc.type is "aws_s3_bucket" implies
    rc.change.after.versioning[0].enabled is true
  }
}

# Open Policy Agent (OPA)
package terraform.s3

deny[msg] {
  input.resource_changes[_].type == "aws_s3_bucket"
  input.resource_changes[_].change.after.versioning[0].enabled != true
  msg := "S3 buckets must have versioning enabled"
}
```

### 3. **Security Scanning**

```bash
# Checkov (static analysis)
checkov -f main.tf

# Terrascan
terrascan scan -f main.tf

# tfsec
tfsec .
```

## Testing de Infraestructura

### 1. **Unit Testing**

```python
# test_terraform.py usando pytest
import unittest
import hcl2

class TestTerraformConfig(unittest.TestCase):
    def setUp(self):
        with open('main.tf', 'r') as file:
            self.tf_config = hcl2.load(file)
    
    def test_instance_type_is_correct(self):
        instances = self.tf_config['resource']['aws_instance']
        for instance in instances.values():
            self.assertIn(instance['instance_type'], ['t3.micro', 't3.small'])
    
    def test_security_group_has_ingress_rules(self):
        sgs = self.tf_config['resource']['aws_security_group']
        for sg in sgs.values():
            self.assertIsNotNone(sg.get('ingress'))
```

### 2. **Integration Testing**

```python
# test_integration.py usando Terratest
import pytest
import boto3
from terratest import terraform

class TestInfrastructure:
    def test_ec2_instance_is_running(self):
        # Apply Terraform
        terraform_options = {
            'terraform_dir': './terraform'
        }
        
        terraform.init_and_apply(terraform_options)
        
        # Get instance ID from Terraform output
        instance_id = terraform.output(terraform_options, 'instance_id')
        
        # Verify instance is running
        ec2 = boto3.client('ec2', region_name='us-west-2')
        response = ec2.describe_instances(InstanceIds=[instance_id])
        
        instance = response['Reservations'][0]['Instances'][0]
        assert instance['State']['Name'] == 'running'
        
        # Cleanup
        terraform.destroy(terraform_options)
```

## Herramientas y Ecosistema

### 1. **Terraform Ecosystem**

- **Terraform Cloud:** SaaS para colaboración
- **Terraform Enterprise:** On-premise solution
- **Terragrunt:** Wrapper para DRY configurations
- **Atlantis:** GitOps for Terraform

### 2. **Formateo y Linting**

```bash
# Terraform
terraform fmt
terraform validate

# tflint
tflint

# Prettier para YAML/JSON
prettier --write "**/*.{yaml,yml,json}"
```

### 3. **Documentation**

```bash
# terraform-docs
terraform-docs markdown table . > README.md

# Genera documentación automática de módulos
```

## Monitoreo y Observabilidad

### 1. **Drift Detection**

```bash
# Script para detectar drift
#!/bin/bash
cd /path/to/terraform

terraform plan -detailed-exitcode
EXIT_CODE=$?

if [ $EXIT_CODE -eq 1 ]; then
    echo "Terraform plan failed"
    exit 1
elif [ $EXIT_CODE -eq 2 ]; then
    echo "Infrastructure drift detected!"
    # Enviar alerta
    curl -X POST webhook-url -d "Infrastructure drift detected"
fi
```

### 2. **Cost Monitoring**

```hcl
# Ejemplo con AWS Cost Budget
resource "aws_budgets_budget" "infrastructure" {
  name         = "infrastructure-budget"
  budget_type  = "COST"
  limit_amount = "100"
  limit_unit   = "USD"
  time_unit    = "MONTHLY"

  cost_filters = {
    TagKey = ["CreatedBy"]
    TagValue = ["Terraform"]
  }
}
```

## Ejercicios Prácticos

### Ejercicio 1: Mi Primera Infraestructura

```hcl
# Crear un servidor web simple con Terraform
# 1. Instancia EC2
# 2. Security Group
# 3. Elastic IP
# 4. Output de la IP pública
```

### Ejercicio 2: Infraestructura Multi-Ambiente

```bash
# Crear la misma infraestructura para:
# - Desarrollo (t3.micro)
# - Staging (t3.small)  
# - Producción (t3.medium)
```

### Ejercicio 3: Módulo Reutilizable

```hcl
# Crear módulo para:
# - VPC con subnets públicas y privadas
# - Auto Scaling Group
# - Load Balancer
# - RDS Database
```

## Recursos Adicionales

### Documentación Oficial

- [Terraform Documentation](https://www.terraform.io/docs)
- [AWS CloudFormation User Guide](https://docs.aws.amazon.com/cloudformation/)
- [Azure ARM Templates](https://docs.microsoft.com/en-us/azure/azure-resource-manager/)
- [Pulumi Documentation](https://www.pulumi.com/docs/)

### Tutoriales y Cursos

- [HashiCorp Learn](https://learn.hashicorp.com/terraform)
- [AWS CloudFormation Workshop](https://cfn101.workshop.aws/)
- [Terraform Best Practices](https://www.terraform-best-practices.com/)

### Herramientas Útiles

- [Terraform Registry](https://registry.terraform.io/)
- [CloudFormation Templates](https://aws.amazon.com/cloudformation/templates/)
- [Awesome Terraform](https://github.com/shuaibiyy/awesome-terraform)

---

## Próximos Pasos

Una vez que domines los conceptos básicos de IaC, estarás listo para:

1. **Automatización avanzada:** Pipelines de CI/CD para infraestructura
2. **Multi-cloud strategies:** Gestión de infraestructura híbrida
3. **Compliance as Code:** Políticas de seguridad automatizadas
4. **GitOps:** Flujos de trabajo declarativos

¡La Infraestructura como Código es esencial para DevOps moderno y escalable!
