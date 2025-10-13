# 04. Variables y Outputs en Terraform

![Variables Badge](https://img.shields.io/badge/Variables-623CE4?style=for-the-badge&logo=terraform&logoColor=white)

> **"Las variables y outputs son los mecanismos que permiten parametrizar y reutilizar configuraciones de Terraform"**

## 🎯 Objetivos de Aprendizaje

Al completar este módulo serás capaz de:

- [x] Definir y usar variables de entrada
- [x] Implementar diferentes tipos de variables
- [x] Gestionar archivos de variables (.tfvars)
- [x] Crear outputs informativos
- [x] Usar locals para simplificar configuraciones
- [x] Implementar validaciones personalizadas
- [x] Manejar variables sensibles
- [x] Organizar variables en proyectos complejos

## 📥 Variables de Entrada (Input Variables)

### **¿Qué son las Variables?**

Las **variables de entrada** permiten parametrizar configuraciones de Terraform, haciendo que sean:

- **Reutilizables** en diferentes entornos
- **Flexibles** para diferentes casos de uso
- **Mantenibles** al centralizar valores
- **Seguras** al separar configuración de código

### **Sintaxis Básica**

```hcl
variable "nombre_variable" {
  description = "Descripción de la variable"
  type        = string
  default     = "valor_por_defecto"
  sensitive   = false

  validation {
    condition     = can(regex("^[a-z0-9-]+$", var.nombre_variable))
    error_message = "El nombre debe contener solo letras minúsculas, números y guiones."
  }
}
```

### **Tipos de Variables**

#### **Tipos Primitivos**

```hcl
# String
variable "region" {
  description = "Región de AWS"
  type        = string
  default     = "us-west-2"
}

# Number
variable "instance_count" {
  description = "Número de instancias"
  type        = number
  default     = 2
}

# Bool
variable "enable_monitoring" {
  description = "Habilitar monitoreo"
  type        = bool
  default     = true
}
```

#### **Tipos Complejos**

```hcl
# List
variable "availability_zones" {
  description = "Lista de zonas de disponibilidad"
  type        = list(string)
  default     = ["us-west-2a", "us-west-2b", "us-west-2c"]
}

# Set (similar a list pero sin duplicados)
variable "allowed_ports" {
  description = "Puertos permitidos"
  type        = set(number)
  default     = [80, 443, 22]
}

# Map
variable "instance_types" {
  description = "Tipos de instancia por entorno"
  type        = map(string)
  default = {
    development = "t3.micro"
    staging     = "t3.small"
    production  = "t3.medium"
  }
}

# Object
variable "database_config" {
  description = "Configuración de la base de datos"
  type = object({
    engine         = string
    engine_version = string
    instance_class = string
    allocated_storage = number
    multi_az       = bool
    backup_retention = number
  })

  default = {
    engine            = "mysql"
    engine_version    = "8.0"
    instance_class    = "db.t3.micro"
    allocated_storage = 20
    multi_az          = false
    backup_retention  = 7
  }
}

# Tuple (lista con tipos específicos)
variable "subnet_config" {
  description = "Configuración de subnets"
  type = tuple([
    object({
      name = string
      cidr = string
      az   = string
    }),
    object({
      name = string
      cidr = string
      az   = string
    })
  ])

  default = [
    {
      name = "subnet-1"
      cidr = "10.0.1.0/24"
      az   = "us-west-2a"
    },
    {
      name = "subnet-2"
      cidr = "10.0.2.0/24"
      az   = "us-west-2b"
    }
  ]
}
```

### **Validaciones de Variables**

```hcl
# Validación de región
variable "aws_region" {
  description = "Región de AWS"
  type        = string

  validation {
    condition = can(regex("^(us|eu|ap|sa|ca|af|me)-[a-z]+-[0-9]$", var.aws_region))
    error_message = "La región debe seguir el formato: us-west-2, eu-west-1, etc."
  }
}

# Validación de tamaño de instancia
variable "instance_type" {
  description = "Tipo de instancia EC2"
  type        = string

  validation {
    condition = contains([
      "t3.micro", "t3.small", "t3.medium", "t3.large",
      "m5.large", "m5.xlarge", "c5.large", "c5.xlarge"
    ], var.instance_type)
    error_message = "El tipo de instancia debe ser uno de los permitidos."
  }
}

# Validación de rango numérico
variable "min_capacity" {
  description = "Capacidad mínima del Auto Scaling Group"
  type        = number

  validation {
    condition     = var.min_capacity >= 1 && var.min_capacity <= 10
    error_message = "La capacidad mínima debe estar entre 1 y 10."
  }
}

# Validación múltiple
variable "environment" {
  description = "Entorno de despliegue"
  type        = string

  validation {
    condition = contains(["dev", "staging", "prod"], var.environment)
    error_message = "El entorno debe ser: dev, staging o prod."
  }

  validation {
    condition = length(var.environment) >= 3
    error_message = "El nombre del entorno debe tener al menos 3 caracteres."
  }
}

# Validación con funciones complejas
variable "tags" {
  description = "Tags para los recursos"
  type        = map(string)

  validation {
    condition = alltrue([
      for tag_key, tag_value in var.tags :
      can(regex("^[A-Za-z][A-Za-z0-9_-]*$", tag_key)) &&
      length(tag_value) > 0
    ])
    error_message = "Las claves de tags deben comenzar con letra y los valores no pueden estar vacíos."
  }
}
```

### **Variables Sensibles**

```hcl
# Variable sensible
variable "database_password" {
  description = "Contraseña de la base de datos"
  type        = string
  sensitive   = true

  validation {
    condition     = length(var.database_password) >= 12
    error_message = "La contraseña debe tener al menos 12 caracteres."
  }
}

# Variable con secreto de AWS Secrets Manager
variable "api_key" {
  description = "Clave API almacenada en Secrets Manager"
  type        = string
  sensitive   = true
}

# Usar variables sensibles
resource "aws_db_instance" "main" {
  identifier = "main-database"
  engine     = "mysql"

  username = "admin"
  password = var.database_password  # Valor sensible

  # La contraseña no aparecerá en logs ni plan
}
```

## 📤 Variables de Salida (Outputs)

### **¿Qué son los Outputs?**

Los **outputs** exponen información de recursos creados por Terraform:

- **Compartir** datos entre configuraciones
- **Mostrar** información importante después del apply
- **Integrar** con otros sistemas
- **Documentar** la infraestructura creada

### **Sintaxis Básica**

```hcl
output "nombre_output" {
  description = "Descripción del output"
  value       = expresion_terraform
  sensitive   = false
  depends_on  = [recurso.dependencia]
  precondition {
    condition     = expresion_booleana
    error_message = "Mensaje de error"
  }
}
```

### **Outputs Comunes**

```hcl
# Output simple
output "vpc_id" {
  description = "ID de la VPC creada"
  value       = aws_vpc.main.id
}

# Output con interpolación
output "instance_info" {
  description = "Información de la instancia"
  value       = "Instance ${aws_instance.web.id} running on ${aws_instance.web.instance_type}"
}

# Output de lista
output "subnet_ids" {
  description = "IDs de las subnets públicas"
  value       = aws_subnet.public[*].id
}

# Output de mapa
output "instance_ips" {
  description = "IPs de las instancias por nombre"
  value = {
    for instance in aws_instance.web :
    instance.tags.Name => instance.public_ip
  }
}

# Output condicional
output "load_balancer_dns" {
  description = "DNS del Load Balancer (solo si está habilitado)"
  value       = var.enable_load_balancer ? aws_lb.web[0].dns_name : null
}

# Output complejo con objeto
output "database_connection" {
  description = "Información de conexión a la base de datos"
  value = {
    endpoint = aws_db_instance.main.endpoint
    port     = aws_db_instance.main.port
    database = aws_db_instance.main.db_name
    username = aws_db_instance.main.username
  }
  sensitive = true
}
```

### **Outputs Sensibles**

```hcl
# Output sensible para contraseñas
output "database_password" {
  description = "Contraseña de la base de datos"
  value       = random_password.db_password.result
  sensitive   = true
}

# Output sensible para certificados
output "private_key" {
  description = "Clave privada del certificado"
  value       = tls_private_key.example.private_key_pem
  sensitive   = true
}

# Ver outputs sensibles
# terraform output -json | jq '.database_password.value' -r
```

### **Precondiciones en Outputs**

```hcl
output "instance_public_ip" {
  description = "IP pública de la instancia"
  value       = aws_instance.web.public_ip

  precondition {
    condition     = aws_instance.web.public_ip != ""
    error_message = "La instancia debe tener una IP pública asignada."
  }
}

output "ssl_certificate_arn" {
  description = "ARN del certificado SSL"
  value       = data.aws_acm_certificate.ssl.arn

  precondition {
    condition     = data.aws_acm_certificate.ssl.status == "ISSUED"
    error_message = "El certificado SSL debe estar en estado ISSUED."
  }
}
```

## 🏠 Variables Locales (Locals)

### **¿Qué son los Locals?**

Los **locals** permiten definir valores calculados que se pueden reutilizar:

```hcl
locals {
  # Valores calculados
  common_tags = {
    Environment = var.environment
    Project     = var.project_name
    ManagedBy   = "terraform"
    CreatedDate = formatdate("YYYY-MM-DD", timestamp())
  }

  # Nombres consistentes
  name_prefix = "${var.project_name}-${var.environment}"

  # Lógica condicional
  instance_type = var.environment == "production" ? "t3.large" : "t3.micro"

  # Configuraciones complejas
  subnets = [
    for i, cidr in var.subnet_cidrs : {
      name = "${local.name_prefix}-subnet-${i + 1}"
      cidr = cidr
      az   = data.aws_availability_zones.available.names[i]
    }
  ]

  # Valores derivados
  database_name = replace(local.name_prefix, "-", "_")

  # Configuración de monitoreo
  monitoring_config = {
    enabled = var.environment == "production"
    retention_days = var.environment == "production" ? 365 : 30
    detailed_monitoring = var.environment == "production"
  }
}

# Usar locals en recursos
resource "aws_instance" "web" {
  ami           = data.aws_ami.amazon_linux.id
  instance_type = local.instance_type

  tags = merge(local.common_tags, {
    Name = "${local.name_prefix}-web"
  })
}
```

### **Casos de Uso Comunes para Locals**

```hcl
locals {
  # Agregación de datos
  all_subnet_cidrs = concat(var.public_subnet_cidrs, var.private_subnet_cidrs)

  # Transformación de datos
  instance_configs = {
    for config in var.instance_configs :
    config.name => {
      ami           = data.aws_ami.amazon_linux.id
      instance_type = config.type
      subnet_id     = config.subnet_id
      security_groups = config.security_groups
    }
  }

  # Configuración condicional compleja
  backup_config = var.environment == "production" ? {
    enabled               = true
    backup_retention_period = 30
    backup_window          = "03:00-04:00"
    maintenance_window     = "sun:04:00-sun:05:00"
    skip_final_snapshot    = false
  } : {
    enabled               = false
    backup_retention_period = 1
    backup_window          = "03:00-04:00"
    maintenance_window     = "sun:04:00-sun:05:00"
    skip_final_snapshot    = true
  }

  # URLs y endpoints
  endpoints = {
    api      = "https://api.${var.domain_name}"
    web      = "https://www.${var.domain_name}"
    admin    = "https://admin.${var.domain_name}"
    database = aws_db_instance.main.endpoint
  }
}
```

## 📁 Archivos de Variables

### **Archivo variables.tf**

```hcl
# variables.tf - Definición de variables

# Configuración del proyecto
variable "project_name" {
  description = "Nombre del proyecto"
  type        = string

  validation {
    condition     = can(regex("^[a-z0-9-]+$", var.project_name))
    error_message = "El nombre del proyecto debe contener solo letras minúsculas, números y guiones."
  }
}

variable "environment" {
  description = "Entorno de despliegue"
  type        = string

  validation {
    condition     = contains(["dev", "staging", "prod"], var.environment)
    error_message = "El entorno debe ser: dev, staging o prod."
  }
}

# Configuración de red
variable "vpc_cidr" {
  description = "CIDR block para la VPC"
  type        = string
  default     = "10.0.0.0/16"

  validation {
    condition     = can(cidrhost(var.vpc_cidr, 0))
    error_message = "vpc_cidr debe ser un CIDR block válido."
  }
}

variable "public_subnet_cidrs" {
  description = "CIDR blocks para subnets públicas"
  type        = list(string)
  default     = ["10.0.1.0/24", "10.0.2.0/24"]

  validation {
    condition = alltrue([
      for cidr in var.public_subnet_cidrs : can(cidrhost(cidr, 0))
    ])
    error_message = "Todos los CIDRs deben ser válidos."
  }
}

# Configuración de compute
variable "instance_type" {
  description = "Tipo de instancia EC2"
  type        = string
  default     = "t3.micro"
}

variable "min_capacity" {
  description = "Capacidad mínima del ASG"
  type        = number
  default     = 1
}

variable "max_capacity" {
  description = "Capacidad máxima del ASG"
  type        = number
  default     = 3
}

# Configuración de base de datos
variable "db_instance_class" {
  description = "Clase de instancia RDS"
  type        = string
  default     = "db.t3.micro"
}

variable "db_allocated_storage" {
  description = "Almacenamiento asignado a RDS (GB)"
  type        = number
  default     = 20
}

# Variables sensibles
variable "db_password" {
  description = "Contraseña de la base de datos"
  type        = string
  sensitive   = true
}

# Variables complejas
variable "tags" {
  description = "Tags comunes para todos los recursos"
  type        = map(string)
  default = {
    ManagedBy = "terraform"
  }
}
```

### **Archivos .tfvars**

#### **terraform.tfvars (valores por defecto)**

```hcl
# terraform.tfvars - Valores por defecto

project_name = "mi-aplicacion"
environment  = "dev"

# Configuración de red
vpc_cidr = "10.0.0.0/16"
public_subnet_cidrs = [
  "10.0.1.0/24",
  "10.0.2.0/24"
]
private_subnet_cidrs = [
  "10.0.10.0/24",
  "10.0.20.0/24"
]

# Configuración de compute
instance_type = "t3.micro"
min_capacity  = 1
max_capacity  = 2

# Tags comunes
tags = {
  ManagedBy   = "terraform"
  Team        = "devops"
  CostCenter  = "engineering"
}
```

#### **dev.tfvars (desarrollo)**

```hcl
# dev.tfvars - Configuración de desarrollo

environment = "dev"

# Instancias pequeñas para desarrollo
instance_type     = "t3.micro"
db_instance_class = "db.t3.micro"

# Capacidad mínima
min_capacity = 1
max_capacity = 2

# Base de datos simple
db_allocated_storage    = 20
backup_retention_period = 1

# Tags específicos de desarrollo
tags = {
  ManagedBy   = "terraform"
  Environment = "development"
  Team        = "devops"
  AutoShutdown = "true"
}
```

#### **prod.tfvars (producción)**

```hcl
# prod.tfvars - Configuración de producción

environment = "prod"

# Instancias más grandes para producción
instance_type     = "t3.large"
db_instance_class = "db.t3.large"

# Alta disponibilidad
min_capacity = 2
max_capacity = 10

# Base de datos robusta
db_allocated_storage    = 100
backup_retention_period = 30
multi_az               = true

# Configuración de seguridad adicional
enable_encryption = true
enable_monitoring = true

# Tags de producción
tags = {
  ManagedBy   = "terraform"
  Environment = "production"
  Team        = "devops"
  Critical    = "true"
  Backup      = "required"
}
```

### **Usar Archivos de Variables**

```bash
# Usar archivo específico
terraform plan -var-file="dev.tfvars"
terraform apply -var-file="prod.tfvars"

# Combinar múltiples archivos
terraform plan \
  -var-file="common.tfvars" \
  -var-file="dev.tfvars"

# Variables desde línea de comandos
terraform plan \
  -var="environment=staging" \
  -var="instance_type=t3.small"

# Variables de entorno
export TF_VAR_environment="dev"
export TF_VAR_instance_type="t3.micro"
terraform plan
```

## 🔧 Técnicas Avanzadas

### **Variable Jerárquicas**

```hcl
# Configuración jerárquica por entorno
locals {
  environment_config = {
    dev = {
      instance_type = "t3.micro"
      min_capacity  = 1
      max_capacity  = 2
      monitoring    = false
      backup_retention = 1
    }
    staging = {
      instance_type = "t3.small"
      min_capacity  = 1
      max_capacity  = 3
      monitoring    = true
      backup_retention = 7
    }
    prod = {
      instance_type = "t3.large"
      min_capacity  = 2
      max_capacity  = 10
      monitoring    = true
      backup_retention = 30
    }
  }

  # Obtener configuración actual
  current_config = local.environment_config[var.environment]
}

# Usar configuración jerárquica
resource "aws_instance" "web" {
  ami           = data.aws_ami.amazon_linux.id
  instance_type = local.current_config.instance_type

  monitoring = local.current_config.monitoring
}
```

### **Variables Dinámicas**

```hcl
# Generar configuraciones dinámicamente
locals {
  # Crear subnets basadas en zonas disponibles
  subnet_configs = [
    for i, az in data.aws_availability_zones.available.names : {
      name = "${var.project_name}-subnet-${i + 1}"
      cidr = cidrsubnet(var.vpc_cidr, 8, i + 1)
      az   = az
      type = i < 2 ? "public" : "private"
    }
  ]

  # Separar subnets por tipo
  public_subnets = [
    for subnet in local.subnet_configs : subnet if subnet.type == "public"
  ]

  private_subnets = [
    for subnet in local.subnet_configs : subnet if subnet.type == "private"
  ]
}

# Crear subnets dinámicamente
resource "aws_subnet" "dynamic" {
  for_each = {
    for subnet in local.subnet_configs : subnet.name => subnet
  }

  vpc_id            = aws_vpc.main.id
  cidr_block        = each.value.cidr
  availability_zone = each.value.az

  map_public_ip_on_launch = each.value.type == "public"

  tags = {
    Name = each.value.name
    Type = each.value.type
  }
}
```

### **Outputs Dinámicos**

```hcl
# Output de todos los recursos creados dinámicamente
output "subnet_info" {
  description = "Información de todas las subnets"
  value = {
    for name, subnet in aws_subnet.dynamic : name => {
      id                = subnet.id
      cidr_block        = subnet.cidr_block
      availability_zone = subnet.availability_zone
      type              = subnet.tags.Type
    }
  }
}

# Output filtrado
output "web_instance_ips" {
  description = "IPs de instancias web"
  value = {
    for name, instance in aws_instance.web : name => {
      public_ip  = instance.public_ip
      private_ip = instance.private_ip
    } if instance.tags.Type == "web"
  }
}

# Output condicional complejo
output "monitoring_endpoints" {
  description = "Endpoints de monitoreo (solo si está habilitado)"
  value = var.enable_monitoring ? {
    cloudwatch = "https://console.aws.amazon.com/cloudwatch"
    logs       = aws_cloudwatch_log_group.app[*].name
    alarms     = aws_cloudwatch_metric_alarm.high_cpu[*].name
  } : null
}
```

## 📋 Ejemplo Completo: Configuración Multi-Entorno

### **Structure del Proyecto**

```
infrastructure/
├── main.tf
├── variables.tf
├── outputs.tf
├── locals.tf
└── environments/
    ├── dev.tfvars
    ├── staging.tfvars
    └── prod.tfvars
```

### **variables.tf**

```hcl
# Project configuration
variable "project_name" {
  description = "Name of the project"
  type        = string

  validation {
    condition     = length(var.project_name) > 2 && length(var.project_name) < 20
    error_message = "Project name must be between 3 and 19 characters."
  }
}

variable "environment" {
  description = "Environment name"
  type        = string

  validation {
    condition     = contains(["dev", "staging", "prod"], var.environment)
    error_message = "Environment must be dev, staging, or prod."
  }
}

# Network configuration
variable "vpc_cidr" {
  description = "CIDR block for VPC"
  type        = string
  default     = "10.0.0.0/16"
}

variable "availability_zones" {
  description = "List of availability zones"
  type        = list(string)
  default     = []
}

# Compute configuration
variable "instance_config" {
  description = "Instance configuration per environment"
  type = object({
    type         = string
    min_capacity = number
    max_capacity = number
    desired_capacity = number
  })
}

# Database configuration
variable "database_config" {
  description = "Database configuration"
  type = object({
    engine            = string
    engine_version    = string
    instance_class    = string
    allocated_storage = number
    backup_retention  = number
    multi_az         = bool
  })
}

# Feature flags
variable "feature_flags" {
  description = "Feature flags for different environments"
  type = object({
    enable_monitoring     = bool
    enable_backup        = bool
    enable_encryption    = bool
    enable_multi_az      = bool
  })
}

# Tags
variable "additional_tags" {
  description = "Additional tags to apply to resources"
  type        = map(string)
  default     = {}
}
```

### **locals.tf**

```hcl
locals {
  # Common naming
  name_prefix = "${var.project_name}-${var.environment}"

  # Determine AZs to use
  azs = length(var.availability_zones) > 0 ? var.availability_zones : slice(data.aws_availability_zones.available.names, 0, 2)

  # Generate subnet CIDRs dynamically
  subnet_cidrs = [
    for i in range(length(local.azs)) : cidrsubnet(var.vpc_cidr, 8, i + 1)
  ]

  private_subnet_cidrs = [
    for i in range(length(local.azs)) : cidrsubnet(var.vpc_cidr, 8, i + 10)
  ]

  # Common tags
  common_tags = merge({
    Project     = var.project_name
    Environment = var.environment
    ManagedBy   = "terraform"
    CreatedDate = formatdate("YYYY-MM-DD", timestamp())
  }, var.additional_tags)

  # Environment-specific configurations
  monitoring_config = {
    enabled = var.feature_flags.enable_monitoring
    retention_days = var.environment == "prod" ? 365 : 30
    detailed_monitoring = var.environment == "prod"
  }

  backup_config = {
    enabled = var.feature_flags.enable_backup
    retention_period = var.database_config.backup_retention
    window = var.environment == "prod" ? "03:00-04:00" : "07:00-08:00"
  }
}
```

### **outputs.tf**

```hcl
# Network outputs
output "vpc_id" {
  description = "ID of the VPC"
  value       = aws_vpc.main.id
}

output "public_subnet_ids" {
  description = "IDs of the public subnets"
  value       = aws_subnet.public[*].id
}

output "private_subnet_ids" {
  description = "IDs of the private subnets"
  value       = aws_subnet.private[*].id
}

# Compute outputs
output "load_balancer_dns" {
  description = "DNS name of the load balancer"
  value       = aws_lb.main.dns_name
}

output "auto_scaling_group_arn" {
  description = "ARN of the Auto Scaling Group"
  value       = aws_autoscaling_group.main.arn
}

# Database outputs
output "database_endpoint" {
  description = "RDS instance endpoint"
  value       = aws_db_instance.main.endpoint
  sensitive   = true
}

output "database_port" {
  description = "RDS instance port"
  value       = aws_db_instance.main.port
}

# Complete infrastructure summary
output "infrastructure_summary" {
  description = "Complete summary of created infrastructure"
  value = {
    project_name = var.project_name
    environment  = var.environment
    region       = data.aws_region.current.name

    network = {
      vpc_id     = aws_vpc.main.id
      vpc_cidr   = aws_vpc.main.cidr_block
      public_subnets = [
        for subnet in aws_subnet.public : {
          id   = subnet.id
          cidr = subnet.cidr_block
          az   = subnet.availability_zone
        }
      ]
      private_subnets = [
        for subnet in aws_subnet.private : {
          id   = subnet.id
          cidr = subnet.cidr_block
          az   = subnet.availability_zone
        }
      ]
    }

    compute = {
      load_balancer_dns = aws_lb.main.dns_name
      instance_type     = var.instance_config.type
      capacity = {
        min     = var.instance_config.min_capacity
        max     = var.instance_config.max_capacity
        desired = var.instance_config.desired_capacity
      }
    }

    database = {
      engine     = var.database_config.engine
      version    = var.database_config.engine_version
      class      = var.database_config.instance_class
      multi_az   = var.database_config.multi_az
    }

    features = var.feature_flags
  }
}
```

### **environments/dev.tfvars**

```hcl
# Development environment configuration

project_name = "mi-app"
environment  = "dev"

# Network
vpc_cidr = "10.0.0.0/16"

# Compute - minimal for development
instance_config = {
  type             = "t3.micro"
  min_capacity     = 1
  max_capacity     = 2
  desired_capacity = 1
}

# Database - basic configuration
database_config = {
  engine            = "mysql"
  engine_version    = "8.0"
  instance_class    = "db.t3.micro"
  allocated_storage = 20
  backup_retention  = 1
  multi_az         = false
}

# Features - minimal for development
feature_flags = {
  enable_monitoring  = false
  enable_backup      = false
  enable_encryption  = false
  enable_multi_az    = false
}

# Development-specific tags
additional_tags = {
  Team        = "development"
  AutoShutdown = "true"
  CostCenter  = "dev-ops"
}
```

### **environments/prod.tfvars**

```hcl
# Production environment configuration

project_name = "mi-app"
environment  = "prod"

# Network
vpc_cidr = "10.0.0.0/16"

# Compute - robust for production
instance_config = {
  type             = "t3.large"
  min_capacity     = 2
  max_capacity     = 10
  desired_capacity = 3
}

# Database - production-ready
database_config = {
  engine            = "mysql"
  engine_version    = "8.0"
  instance_class    = "db.t3.large"
  allocated_storage = 100
  backup_retention  = 30
  multi_az         = true
}

# Features - all enabled for production
feature_flags = {
  enable_monitoring  = true
  enable_backup      = true
  enable_encryption  = true
  enable_multi_az    = true
}

# Production tags
additional_tags = {
  Team        = "platform"
  Critical    = "true"
  Compliance  = "required"
  CostCenter  = "production"
}
```

## 🎯 Mejores Prácticas

### **Organización de Variables**

```hcl
# 1. Agrupar por categoría
# variables.tf

# === PROJECT CONFIGURATION ===
variable "project_name" { ... }
variable "environment" { ... }
variable "region" { ... }

# === NETWORK CONFIGURATION ===
variable "vpc_cidr" { ... }
variable "public_subnet_cidrs" { ... }
variable "private_subnet_cidrs" { ... }

# === COMPUTE CONFIGURATION ===
variable "instance_type" { ... }
variable "min_capacity" { ... }
variable "max_capacity" { ... }

# === DATABASE CONFIGURATION ===
variable "db_instance_class" { ... }
variable "db_allocated_storage" { ... }
```

### **Validaciones Robustas**

```hcl
# Validación combinada
variable "instance_config" {
  description = "Instance configuration"
  type = object({
    type     = string
    count    = number
    key_name = string
  })

  validation {
    # Validar tipo de instancia
    condition = contains([
      "t3.micro", "t3.small", "t3.medium", "t3.large"
    ], var.instance_config.type)
    error_message = "Instance type must be a valid t3 instance."
  }

  validation {
    # Validar cantidad
    condition = var.instance_config.count >= 1 && var.instance_config.count <= 10
    error_message = "Instance count must be between 1 and 10."
  }

  validation {
    # Validar key name
    condition     = length(var.instance_config.key_name) > 0
    error_message = "Key name cannot be empty."
  }
}
```

### **Outputs Informativos**

```hcl
# Output con información contextual
output "connection_instructions" {
  description = "Instructions to connect to the infrastructure"
  value = <<-EOT
    # Connect to the application:
    curl https://${aws_lb.main.dns_name}

    # Connect to database (from application subnet):
    mysql -h ${aws_db_instance.main.endpoint} -P ${aws_db_instance.main.port} -u admin -p

    # View logs:
    aws logs describe-log-groups --log-group-name-prefix "/aws/lambda/${var.project_name}"
  EOT
}
```

## ✅ Checklist de Mejores Prácticas

### **Variables**

- [ ] Usar descripciones claras y detalladas
- [ ] Implementar validaciones apropiadas
- [ ] Definir valores por defecto sensatos
- [ ] Marcar variables sensibles apropiadamente
- [ ] Agrupar por categorías lógicas
- [ ] Usar tipos específicos en lugar de `any`

### **Outputs**

- [ ] Incluir descripciones informativas
- [ ] Marcar outputs sensibles cuando sea necesario
- [ ] Usar nombres descriptivos y consistentes
- [ ] Incluir información contextual útil
- [ ] Implementar precondiciones cuando sea apropiado

### **Organización**

- [ ] Separar variables, locals y outputs en archivos dedicados
- [ ] Usar archivos .tfvars por entorno
- [ ] Implementar convenciones de naming consistentes
- [ ] Documentar configuraciones complejas

## 🎯 Resumen del Módulo

### **Conceptos Clave Aprendidos**

- [x] **Variables**: Definición, tipos y validaciones
- [x] **Outputs**: Exposición de información de recursos
- [x] **Locals**: Valores calculados reutilizables
- [x] **Archivos .tfvars**: Gestión de configuraciones por entorno
- [x] **Validaciones**: Controles de calidad en inputs

### **Habilidades Desarrolladas**

- 🔧 **Parametrización** de configuraciones
- 📋 **Validación** de inputs complejos
- 🏗️ **Organización** de proyectos multi-entorno
- 📊 **Exposición** de información importante
- 🔒 **Manejo** de información sensible

**Próximo módulo:** [05. Estado de Terraform](05-estado-terraform.md)
