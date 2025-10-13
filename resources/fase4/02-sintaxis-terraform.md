# 02. Sintaxis y Configuración de Terraform

![HCL Badge](https://img.shields.io/badge/HCL-623CE4?style=for-the-badge&logo=terraform&logoColor=white)

> **"HCL es un lenguaje de configuración estructurado que es tanto legible por humanos como procesable por máquinas"**

## 🎯 Objetivos de Aprendizaje

Al completar este módulo serás capaz de:

- [x] Dominar la sintaxis HCL (HashiCorp Configuration Language)
- [x] Escribir bloques de configuración complejos
- [x] Usar expresiones, funciones y operadores
- [x] Implementar lógica condicional y bucles
- [x] Configurar interpolación de cadenas
- [x] Validar y formatear código Terraform

## 📚 Fundamentos de HCL

### **¿Qué es HCL?**

**HCL (HashiCorp Configuration Language)** es un lenguaje de configuración diseñado para ser:

- **Legible**: Sintaxis clara y expresiva
- **Declarativo**: Describes el estado deseado, no cómo conseguirlo
- **Jerárquico**: Soporta estructuras anidadas
- **Tipado**: Tipos de datos explícitos

### **Estructura Básica**

```hcl
# Comentario de línea

/*
  Comentario
  de múltiples
  líneas
*/

# Bloque básico
block_type "block_label" "block_name" {
  # Argumentos del bloque
  argument_name = "value"
  
  # Sub-bloque
  sub_block {
    sub_argument = "sub_value"
  }
}
```

## 🏗️ Tipos de Bloques en Terraform

### **1. Bloque Terraform**

```hcl
terraform {
  # Versión mínima requerida
  required_version = ">= 1.0"
  
  # Providers requeridos
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
    random = {
      source  = "hashicorp/random"
      version = "~> 3.1"
    }
  }
  
  # Backend para estado remoto
  backend "s3" {
    bucket = "mi-terraform-state"
    key    = "proyecto/terraform.tfstate"
    region = "us-west-2"
  }
}
```

### **2. Bloque Provider**

```hcl
# Provider AWS con configuración explícita
provider "aws" {
  region  = "us-west-2"
  profile = "desarrollo"
  
  # Tags por defecto para todos los recursos
  default_tags {
    tags = {
      Environment = "desarrollo"
      Project     = "mi-proyecto"
      ManagedBy   = "terraform"
    }
  }
}

# Provider AWS con alias (múltiples regiones)
provider "aws" {
  alias  = "east"
  region = "us-east-1"
}
```

### **3. Bloque Resource**

```hcl
# Recurso básico
resource "aws_instance" "web" {
  ami           = "ami-0c02fb55956c7d316"
  instance_type = "t3.micro"
  
  tags = {
    Name = "Servidor Web"
  }
}

# Recurso con dependencias y configuración avanzada
resource "aws_security_group" "web" {
  name_prefix = "web-sg"
  description = "Security group para servidor web"
  
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
  
  tags = {
    Name = "web-security-group"
  }
}
```

### **4. Bloque Data**

```hcl
# Obtener datos de recursos existentes
data "aws_ami" "ubuntu" {
  most_recent = true
  owners      = ["099720109477"] # Canonical
  
  filter {
    name   = "name"
    values = ["ubuntu/images/hvm-ssd/ubuntu-focal-20.04-amd64-server-*"]
  }
  
  filter {
    name   = "virtualization-type"
    values = ["hvm"]
  }
}

# Usar datos en recursos
resource "aws_instance" "example" {
  ami           = data.aws_ami.ubuntu.id
  instance_type = "t3.micro"
}
```

### **5. Bloque Variable**

```hcl
# Variable simple
variable "instance_type" {
  description = "Tipo de instancia EC2"
  type        = string
  default     = "t3.micro"
}

# Variable compleja con validación
variable "allowed_regions" {
  description = "Lista de regiones permitidas"
  type        = list(string)
  default     = ["us-west-2", "us-east-1"]
  
  validation {
    condition = length(var.allowed_regions) > 0
    error_message = "Debe especificar al menos una región."
  }
}

# Variable sensible
variable "database_password" {
  description = "Contraseña de la base de datos"
  type        = string
  sensitive   = true
}

# Variable con múltiples tipos
variable "tags" {
  description = "Tags para aplicar a recursos"
  type        = map(string)
  default = {
    Environment = "desarrollo"
    Project     = "mi-proyecto"
  }
}
```

### **6. Bloque Output**

```hcl
# Output simple
output "instance_ip" {
  description = "IP pública de la instancia"
  value       = aws_instance.web.public_ip
}

# Output sensible
output "database_endpoint" {
  description = "Endpoint de la base de datos"
  value       = aws_db_instance.main.endpoint
  sensitive   = true
}

# Output con dependencias
output "complete_url" {
  description = "URL completa de la aplicación"
  value       = "https://${aws_instance.web.public_dns}:443"
  depends_on  = [aws_instance.web]
}
```

### **7. Bloque Local**

```hcl
locals {
  # Variables locales calculadas
  environment = "production"
  project     = "mi-app"
  
  # Tags comunes
  common_tags = {
    Environment = local.environment
    Project     = local.project
    ManagedBy   = "terraform"
    CreatedDate = formatdate("YYYY-MM-DD", timestamp())
  }
  
  # Nombres de recursos
  instance_name = "${local.project}-${local.environment}-web"
  
  # Configuración condicional
  instance_type = local.environment == "production" ? "t3.large" : "t3.micro"
}
```

## 📊 Tipos de Datos en HCL

### **Tipos Primitivos**

```hcl
# String
variable "name" {
  type    = string
  default = "mi-servidor"
}

# Number (int o float)
variable "instance_count" {
  type    = number
  default = 3
}

variable "cpu_utilization" {
  type    = number
  default = 80.5
}

# Boolean
variable "enable_monitoring" {
  type    = bool
  default = true
}
```

### **Tipos de Colección**

```hcl
# List - Secuencia ordenada de valores del mismo tipo
variable "availability_zones" {
  type    = list(string)
  default = ["us-west-2a", "us-west-2b", "us-west-2c"]
}

# Set - Colección no ordenada de valores únicos
variable "security_groups" {
  type    = set(string)
  default = ["sg-web", "sg-database"]
}

# Map - Colección de pares clave-valor
variable "instance_types" {
  type = map(string)
  default = {
    web      = "t3.micro"
    database = "t3.small"
    cache    = "t3.nano"
  }
}
```

### **Tipos Estructurales**

```hcl
# Object - Estructura con tipos de atributos definidos
variable "server_config" {
  type = object({
    name         = string
    instance_type = string
    disk_size    = number
    monitoring   = bool
    tags         = map(string)
  })
  
  default = {
    name         = "web-server"
    instance_type = "t3.micro"
    disk_size    = 20
    monitoring   = true
    tags = {
      Environment = "development"
    }
  }
}

# Tuple - Secuencia de elementos de diferentes tipos
variable "mixed_list" {
  type    = tuple([string, number, bool])
  default = ["server-1", 8080, true]
}
```

## 🔧 Expresiones y Funciones

### **Referencias a Valores**

```hcl
# Referencia a variable
var.instance_type

# Referencia a local
local.common_tags

# Referencia a recurso
aws_instance.web.public_ip

# Referencia a data source
data.aws_ami.ubuntu.id

# Referencia a output (de otro módulo)
module.network.vpc_id
```

### **Operadores**

```hcl
locals {
  # Operadores aritméticos
  total_instances = var.web_instances + var.app_instances
  half_capacity   = var.max_capacity / 2
  
  # Operadores de comparación
  is_production = var.environment == "production"
  has_capacity  = var.current_load < var.max_capacity
  
  # Operadores lógicos
  deploy_monitoring = var.environment == "production" && var.enable_monitoring
  
  # Operador ternario (condicional)
  instance_type = var.environment == "production" ? "t3.large" : "t3.micro"
}
```

### **Funciones Útiles**

#### **Funciones de Cadenas**

```hcl
locals {
  # Formateo de cadenas
  server_name = format("%s-%s-%03d", var.project, var.environment, var.instance_number)
  
  # Manipulación de cadenas
  uppercase_env = upper(var.environment)
  lowercase_name = lower(var.server_name)
  
  # División y unión
  name_parts = split("-", var.server_name)
  full_name  = join("-", [var.project, var.environment, "web"])
  
  # Reemplazo
  sanitized_name = replace(var.user_input, " ", "-")
  
  # Subcadenas
  short_name = substr(var.long_name, 0, 10)
}
```

#### **Funciones de Colecciones**

```hcl
locals {
  # Lista de zonas de disponibilidad
  azs = ["us-west-2a", "us-west-2b", "us-west-2c"]
  
  # Longitud de lista
  az_count = length(local.azs)
  
  # Concatenar listas
  all_subnets = concat(var.public_subnets, var.private_subnets)
  
  # Elementos únicos
  unique_tags = distinct(var.all_tags)
  
  # Ordenar lista
  sorted_azs = sort(local.azs)
  
  # Filtrar lista
  production_instances = [
    for instance in var.instances : instance
    if instance.environment == "production"
  ]
  
  # Transformar lista a mapa
  instance_map = {
    for idx, instance in var.instances : idx => instance
  }
}
```

#### **Funciones de Mapas**

```hcl
locals {
  tags = {
    Environment = "production"
    Project     = "mi-app"
    Owner       = "devops-team"
  }
  
  # Obtener claves
  tag_keys = keys(local.tags)
  
  # Obtener valores
  tag_values = values(local.tags)
  
  # Combinar mapas
  all_tags = merge(local.tags, var.additional_tags)
  
  # Buscar en mapa
  environment = lookup(local.tags, "Environment", "desarrollo")
}
```

#### **Funciones de Fecha y Tiempo**

```hcl
locals {
  # Timestamp actual
  created_at = timestamp()
  
  # Formatear fecha
  date_tag = formatdate("YYYY-MM-DD", timestamp())
  
  # Agregar tiempo
  expiry_date = timeadd(timestamp(), "24h")
}
```

#### **Funciones de Archivos**

```hcl
locals {
  # Leer archivo
  user_data = file("${path.module}/scripts/user-data.sh")
  
  # Leer archivo JSON
  config = jsondecode(file("${path.module}/config.json"))
  
  # Leer archivo YAML
  settings = yamldecode(file("${path.module}/settings.yaml"))
  
  # Template de archivo
  startup_script = templatefile("${path.module}/templates/startup.tpl", {
    database_url = var.database_url
    app_version  = var.app_version
  })
}
```

## 🔄 Expresiones Dinámicas

### **For Expressions**

```hcl
locals {
  # Lista de instancias
  instances = [
    { name = "web-1", type = "t3.micro" },
    { name = "web-2", type = "t3.small" },
    { name = "db-1", type = "t3.medium" }
  ]
  
  # Transformar lista a lista
  instance_names = [for instance in local.instances : instance.name]
  
  # Transformar lista a mapa
  instance_types = {
    for instance in local.instances : instance.name => instance.type
  }
  
  # Filtrar y transformar
  web_instances = [
    for instance in local.instances : instance.name
    if startswith(instance.name, "web-")
  ]
  
  # For con índice
  numbered_instances = {
    for idx, instance in local.instances : idx => instance.name
  }
}
```

### **Conditional Expressions**

```hcl
locals {
  # Condicional simple
  instance_type = var.environment == "production" ? "t3.large" : "t3.micro"
  
  # Condicional anidado
  storage_type = (
    var.environment == "production" ? "gp3" :
    var.environment == "staging" ? "gp2" :
    "standard"
  )
  
  # Condicional con listas
  security_groups = var.environment == "production" ? [
    aws_security_group.web.id,
    aws_security_group.monitoring.id
  ] : [aws_security_group.web.id]
}
```

### **Splat Expressions**

```hcl
# Lista de instancias
resource "aws_instance" "web" {
  count         = 3
  ami           = data.aws_ami.amazon_linux.id
  instance_type = "t3.micro"
  
  tags = {
    Name = "web-${count.index}"
  }
}

locals {
  # Obtener todos los IDs usando splat
  instance_ids = aws_instance.web[*].id
  
  # Obtener IPs públicas
  public_ips = aws_instance.web[*].public_ip
  
  # Splat con atributos anidados
  subnet_ids = data.aws_subnet.private[*].id
}
```

## 🎛️ Meta-argumentos

### **count**

```hcl
# Crear múltiples instancias
resource "aws_instance" "web" {
  count         = var.instance_count
  ami           = data.aws_ami.amazon_linux.id
  instance_type = "t3.micro"
  
  tags = {
    Name = "web-server-${count.index + 1}"
  }
}

# Condicional con count
resource "aws_s3_bucket" "logs" {
  count  = var.enable_logging ? 1 : 0
  bucket = "${var.project}-logs"
}
```

### **for_each**

```hcl
# Usando for_each con set
resource "aws_instance" "web" {
  for_each = toset(var.server_names)
  
  ami           = data.aws_ami.amazon_linux.id
  instance_type = "t3.micro"
  
  tags = {
    Name = each.value
  }
}

# Usando for_each con map
resource "aws_instance" "servers" {
  for_each = var.server_config
  
  ami           = data.aws_ami.amazon_linux.id
  instance_type = each.value.instance_type
  
  tags = {
    Name        = each.key
    Environment = each.value.environment
  }
}
```

### **depends_on**

```hcl
resource "aws_instance" "web" {
  ami           = data.aws_ami.amazon_linux.id
  instance_type = "t3.micro"
  
  # Dependencia explícita
  depends_on = [
    aws_security_group.web,
    aws_key_pair.deployer
  ]
}
```

### **lifecycle**

```hcl
resource "aws_instance" "web" {
  ami           = data.aws_ami.amazon_linux.id
  instance_type = var.instance_type
  
  lifecycle {
    # Crear antes de destruir
    create_before_destroy = true
    
    # Prevenir destrucción accidental
    prevent_destroy = true
    
    # Ignorar cambios en ciertos atributos
    ignore_changes = [
      tags,
      user_data
    ]
  }
}
```

## 🧪 Configuración Avanzada

### **Ejemplo Completo: Infraestructura Web**

```hcl
# variables.tf
variable "environment" {
  description = "Entorno de despliegue"
  type        = string
  default     = "desarrollo"
  
  validation {
    condition = contains([
      "desarrollo", 
      "pruebas", 
      "produccion"
    ], var.environment)
    error_message = "Environment debe ser desarrollo, pruebas, o produccion."
  }
}

variable "instance_config" {
  description = "Configuración de instancias por entorno"
  type = map(object({
    instance_type = string
    min_size     = number
    max_size     = number
  }))
  
  default = {
    desarrollo = {
      instance_type = "t3.micro"
      min_size     = 1
      max_size     = 2
    }
    pruebas = {
      instance_type = "t3.small"
      min_size     = 2
      max_size     = 4
    }
    produccion = {
      instance_type = "t3.medium"
      min_size     = 3
      max_size     = 10
    }
  }
}

# locals.tf
locals {
  # Configuración basada en entorno
  config = var.instance_config[var.environment]
  
  # Tags comunes
  common_tags = {
    Environment = var.environment
    Project     = "mi-aplicacion-web"
    ManagedBy   = "terraform"
    CreatedDate = formatdate("YYYY-MM-DD", timestamp())
  }
  
  # Nombres de recursos
  name_prefix = "${var.environment}-webapp"
  
  # Configuración de red
  vpc_cidr = var.environment == "produccion" ? "10.0.0.0/16" : "10.1.0.0/16"
  azs      = data.aws_availability_zones.available.names
  
  # Subnets públicas y privadas
  public_subnets = [
    for idx, az in local.azs : cidrsubnet(local.vpc_cidr, 8, idx)
  ]
  
  private_subnets = [
    for idx, az in local.azs : cidrsubnet(local.vpc_cidr, 8, idx + 10)
  ]
}

# main.tf
data "aws_availability_zones" "available" {
  state = "available"
}

data "aws_ami" "amazon_linux" {
  most_recent = true
  owners      = ["amazon"]
  
  filter {
    name   = "name"
    values = ["amzn2-ami-hvm-*-x86_64-gp2"]
  }
}

# VPC
resource "aws_vpc" "main" {
  cidr_block           = local.vpc_cidr
  enable_dns_hostnames = true
  enable_dns_support   = true
  
  tags = merge(local.common_tags, {
    Name = "${local.name_prefix}-vpc"
  })
}

# Internet Gateway
resource "aws_internet_gateway" "main" {
  vpc_id = aws_vpc.main.id
  
  tags = merge(local.common_tags, {
    Name = "${local.name_prefix}-igw"
  })
}

# Subnets públicas
resource "aws_subnet" "public" {
  count = length(local.public_subnets)
  
  vpc_id                  = aws_vpc.main.id
  cidr_block              = local.public_subnets[count.index]
  availability_zone       = local.azs[count.index]
  map_public_ip_on_launch = true
  
  tags = merge(local.common_tags, {
    Name = "${local.name_prefix}-public-${count.index + 1}"
    Type = "public"
  })
}

# Launch Template para Auto Scaling
resource "aws_launch_template" "web" {
  name_prefix   = "${local.name_prefix}-"
  image_id      = data.aws_ami.amazon_linux.id
  instance_type = local.config.instance_type
  
  vpc_security_group_ids = [aws_security_group.web.id]
  
  user_data = base64encode(templatefile("${path.module}/user-data.sh", {
    environment = var.environment
  }))
  
  tag_specifications {
    resource_type = "instance"
    tags = merge(local.common_tags, {
      Name = "${local.name_prefix}-instance"
    })
  }
  
  lifecycle {
    create_before_destroy = true
  }
}

# Auto Scaling Group
resource "aws_autoscaling_group" "web" {
  name             = "${local.name_prefix}-asg"
  vpc_zone_identifier = aws_subnet.public[*].id
  
  min_size         = local.config.min_size
  max_size         = local.config.max_size
  desired_capacity = local.config.min_size
  
  launch_template {
    id      = aws_launch_template.web.id
    version = "$Latest"
  }
  
  tag {
    key                 = "Name"
    value               = "${local.name_prefix}-asg"
    propagate_at_launch = false
  }
  
  dynamic "tag" {
    for_each = local.common_tags
    content {
      key                 = tag.key
      value               = tag.value
      propagate_at_launch = true
    }
  }
}

# Security Group
resource "aws_security_group" "web" {
  name_prefix = "${local.name_prefix}-web-"
  description = "Security group para servidores web"
  vpc_id      = aws_vpc.main.id
  
  dynamic "ingress" {
    for_each = var.environment == "produccion" ? [80, 443] : [80]
    content {
      from_port   = ingress.value
      to_port     = ingress.value
      protocol    = "tcp"
      cidr_blocks = ["0.0.0.0/0"]
    }
  }
  
  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }
  
  tags = merge(local.common_tags, {
    Name = "${local.name_prefix}-web-sg"
  })
  
  lifecycle {
    create_before_destroy = true
  }
}

# outputs.tf
output "vpc_id" {
  description = "ID de la VPC"
  value       = aws_vpc.main.id
}

output "public_subnet_ids" {
  description = "IDs de las subnets públicas"
  value       = aws_subnet.public[*].id
}

output "autoscaling_group_arn" {
  description = "ARN del Auto Scaling Group"
  value       = aws_autoscaling_group.web.arn
}

output "security_group_id" {
  description = "ID del Security Group"
  value       = aws_security_group.web.id
}
```

## 🔧 Herramientas de Desarrollo

### **Formateo y Validación**

```bash
# Formatear archivos Terraform
terraform fmt -recursive

# Validar configuración
terraform validate

# Validar con información detallada
terraform validate -json
```

### **Linting con TFLint**

```bash
# Instalar TFLint (ejemplo para Linux)
curl -s https://raw.githubusercontent.com/terraform-linters/tflint/master/install_linux.sh | bash

# Ejecutar linting
tflint

# Con configuración personalizada
tflint --config=.tflint.hcl
```

### **Configuración de TFLint (`.tflint.hcl`)**

```hcl
rule "terraform_deprecated_interpolation" {
  enabled = true
}

rule "terraform_unused_declarations" {
  enabled = true
}

rule "terraform_comment_syntax" {
  enabled = true
}

rule "terraform_documented_outputs" {
  enabled = true
}

rule "terraform_documented_variables" {
  enabled = true
}

rule "terraform_typed_variables" {
  enabled = true
}

rule "terraform_module_pinned_source" {
  enabled = true
}

rule "terraform_naming_convention" {
  enabled = true
  format  = "snake_case"
}

rule "terraform_standard_module_structure" {
  enabled = true
}
```

## 🧪 Laboratorio Práctico

### **Ejercicio 1: Configuración Dinámica**

**Objetivo:** Crear una configuración que se adapte según el entorno

**Implementar:**

1. **Variables con validación**
2. **Configuración condicional usando locals**
3. **For expressions para recursos dinámicos**
4. **Bloques dynamic para configuración flexible**

### **Ejercicio 2: Funciones Avanzadas**

**Objetivo:** Usar múltiples funciones HCL

**Tareas:**

1. **Leer configuración desde archivo JSON**
2. **Usar templatefile para user-data**
3. **Implementar transformaciones con for**
4. **Crear tags dinámicos con merge**

### **Ejercicio 3: Meta-argumentos**

**Objetivo:** Dominar count y for_each

**Implementar:**

1. **Recursos con count condicional**
2. **Recursos con for_each usando mapa**
3. **Dependencias explícitas**
4. **Configuración de lifecycle**

## ✅ Checklist de Mejores Prácticas

### **Sintaxis y Estilo**

- [ ] Usar `terraform fmt` regularmente
- [ ] Mantener líneas menores a 120 caracteres
- [ ] Usar comentarios descriptivos
- [ ] Nombres de variables en snake_case
- [ ] Bloques ordenados lógicamente

### **Estructuras de Datos**

- [ ] Usar tipos explícitos en variables
- [ ] Implementar validación en variables críticas
- [ ] Usar locals para cálculos complejos
- [ ] Preferir maps sobre listas cuando sea apropiado

### **Expresiones**

- [ ] Evitar expresiones muy complejas
- [ ] Usar for expressions en lugar de funciones complejas
- [ ] Implementar condicionales claros
- [ ] Documentar lógica compleja

### **Meta-argumentos**

- [ ] Preferir for_each sobre count
- [ ] Usar depends_on solo cuando sea necesario
- [ ] Configurar lifecycle apropiadamente
- [ ] Evitar meta-argumentos anidados

## 📚 Recursos Adicionales

### **Documentación Oficial**

- **[Configuration Language](https://www.terraform.io/docs/language/index.html)**
- **[Built-in Functions](https://www.terraform.io/docs/language/functions/index.html)**
- **[Expression Types and Values](https://www.terraform.io/docs/language/expressions/types.html)**

### **Herramientas y Plugins**

- **[Terraform VS Code Extension](https://marketplace.visualstudio.com/items?itemName=HashiCorp.terraform)**
- **[TFLint](https://github.com/terraform-linters/tflint)**
- **[TFSec](https://github.com/aquasecurity/tfsec)**

## 🎯 Resumen del Módulo

### **Conceptos Clave Aprendidos**

- [x] **Sintaxis HCL**: Bloques, argumentos, expresiones
- [x] **Tipos de datos**: Primitivos, colecciones, estructurales
- [x] **Funciones**: Cadenas, colecciones, fechas, archivos
- [x] **Expresiones dinámicas**: For, condicionales, splat
- [x] **Meta-argumentos**: count, for_each, depends_on, lifecycle

### **Habilidades Desarrolladas**

- 📝 **Escritura de configuraciones complejas**
- 🔧 **Uso de funciones y expresiones**
- 🎛️ **Implementación de lógica condicional**
- 🔄 **Creación de recursos dinámicos**
- [x] **Validación y formateo de código**

**Próximo módulo:** [03. Providers y Recursos](03-providers-recursos.md)
