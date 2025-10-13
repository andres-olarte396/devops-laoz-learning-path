# 🧪 Evaluación Final - Fase 4: Infrastructure as Code

[![Infrastructure as Code](https://img.shields.io/badge/Infrastructure%20as%20Code-007ACC?style=for-the-badge&logo=terraform&logoColor=white)](https://www.terraform.io/)
[![Evaluation](https://img.shields.io/badge/Evaluation-FF6B35?style=for-the-badge&logo=checkmarx&logoColor=white)](https://checkmarx.com/)

## 🎯 Objetivos de la Evaluación

Esta evaluación integral está diseñada para validar tu dominio completo de Infrastructure as Code utilizando Terraform y Ansible. Los ejercicios prácticos cubren todos los aspectos críticos aprendidos en los 11 módulos de la Fase 4.

### 📊 Criterios de Evaluación

- **Terraform (40%)**: Sintaxis, módulos, estado, providers
- **Ansible (30%)**: Playbooks, roles, inventarios, templates  
- **Integración (15%)**: Terraform + Ansible workflows
- **Seguridad (10%)**: Mejores prácticas de seguridad
- **CI/CD (5%)**: Testing y automatización

**Puntuación mínima para aprobar: 75%**

## 📋 Estructura de la Evaluación

### Parte 1: Preguntas Teóricas (20 puntos)

### Parte 2: Ejercicios Prácticos (60 puntos)

### Parte 3: Proyecto Integrador (20 puntos)

---

## 📝 PARTE 1: PREGUNTAS TEÓRICAS (20 puntos)

### Pregunta 1 (2 puntos)

**Explica la diferencia entre infraestructura inmutable y mutable. ¿Cuándo usarías Terraform vs Ansible?**

```
Tu respuesta aquí:
_________________________________________________
_________________________________________________
_________________________________________________
_________________________________________________
```

### Pregunta 2 (2 puntos)

**¿Qué es el estado de Terraform y por qué es crítico para su funcionamiento? Menciona 3 mejores prácticas para su gestión.**

```
Tu respuesta aquí:
_________________________________________________
_________________________________________________
_________________________________________________
_________________________________________________
```

### Pregunta 3 (2 puntos)

**Describe el concepto de "idempotencia" en Ansible y cómo se garantiza en los módulos.**

```
Tu respuesta aquí:
_________________________________________________
_________________________________________________
_________________________________________________
_________________________________________________
```

### Pregunta 4 (2 puntos)

**Explica qué son los "data sources" en Terraform y proporciona un ejemplo de uso práctico.**

```
Tu respuesta aquí:
_________________________________________________
_________________________________________________
_________________________________________________
_________________________________________________
```

### Pregunta 5 (2 puntos)

**¿Cuáles son las diferencias entre `vars`, `locals`, y `outputs` en Terraform? Proporciona un ejemplo de cada uno.**

```
Tu respuesta aquí:
_________________________________________________
_________________________________________________
_________________________________________________
_________________________________________________
```

### Pregunta 6 (2 puntos)

**Describe los diferentes tipos de inventarios en Ansible y cuándo usar cada uno.**

```
Tu respuesta aquí:
_________________________________________________
_________________________________________________
_________________________________________________
_________________________________________________
```

### Pregunta 7 (2 puntos)

**¿Qué son los handlers en Ansible y cómo se relacionan con las tareas que los notifican?**

```
Tu respuesta aquí:
_________________________________________________
_________________________________________________
_________________________________________________
_________________________________________________
```

### Pregunta 8 (2 puntos)

**Explica el concepto de "Policy as Code" y cómo se implementa con herramientas como OPA (Open Policy Agent).**

```
Tu respuesta aquí:
_________________________________________________
_________________________________________________
_________________________________________________
_________________________________________________
```

### Pregunta 9 (2 puntos)

**¿Cuáles son los beneficios de usar módulos en Terraform? Menciona al menos 4 ventajas.**

```
Tu respuesta aquí:
_________________________________________________
_________________________________________________
_________________________________________________
_________________________________________________
```

### Pregunta 10 (2 puntos)

**Describe qué es GitOps y cómo se implementa un workflow GitOps para Infrastructure as Code.**

```
Tu respuesta aquí:
_________________________________________________
_________________________________________________
_________________________________________________
_________________________________________________
```

---

## 💻 PARTE 2: EJERCICIOS PRÁCTICOS (60 puntos)

### Ejercicio 1: Módulo de Terraform para VPC (15 puntos)

**Objetivo**: Crear un módulo reutilizable de Terraform para desplegar una VPC en AWS.

**Requerimientos**:

1. Crear la estructura del módulo `modules/vpc/`
2. Implementar VPC con subnets públicas y privadas
3. Configurar Internet Gateway y NAT Gateway
4. Implementar outputs necesarios
5. Crear variables con validaciones
6. Incluir documentación con terraform-docs

**Entregables**:

- `modules/vpc/main.tf`
- `modules/vpc/variables.tf`
- `modules/vpc/outputs.tf`
- `modules/vpc/README.md`

**Criterios de evaluación**:

- [x] Estructura correcta del módulo (3 puntos)
- [x] Recursos de red bien configurados (4 puntos)
- [x] Variables con validaciones apropiadas (3 puntos)
- [x] Outputs útiles y bien documentados (2 puntos)
- [x] Documentación clara y completa (3 puntos)

```hcl
# Tu código aquí
```

### Ejercicio 2: Role de Ansible para Servidor Web (15 puntos)

**Objetivo**: Crear un role completo de Ansible para configurar un servidor web nginx.

**Requerimientos**:

1. Estructura de role con todas las carpetas necesarias
2. Instalar y configurar nginx
3. Crear templates dinámicos para configuración
4. Implementar handlers para restart/reload
5. Configurar firewall básico
6. Incluir tests con Molecule

**Entregables**:

- `roles/webserver/` (estructura completa)
- `roles/webserver/molecule/default/molecule.yml`
- `roles/webserver/README.md`

**Criterios de evaluación**:

- [x] Estructura de role correcta (3 puntos)
- [x] Tasks bien organizadas y funcionales (4 puntos)
- [x] Templates dinámicos con Jinja2 (3 puntos)
- [x] Handlers apropiados (2 puntos)
- [x] Tests con Molecule funcionales (3 puntos)

```yaml
# Tu código aquí
```

### Ejercicio 3: Integración Terraform-Ansible (15 puntos)

**Objetivo**: Crear un workflow que use Terraform para provisionar infraestructura y Ansible para configurarla.

**Requerimientos**:

1. Terraform crea instancias EC2 con security groups
2. Generar inventario dinámico para Ansible
3. Provisioner que ejecute Ansible automáticamente
4. Configurar aplicación web con base de datos
5. Implementar outputs para verificación

**Entregables**:

- `main.tf` (infraestructura principal)
- `templates/inventory.tpl` (template de inventario)
- `playbooks/site.yml` (playbook principal)
- `README.md` (instrucciones de uso)

**Criterios de evaluación**:

- [x] Infraestructura bien definida (4 puntos)
- [x] Inventario dinámico funcional (3 puntos)
- [x] Integración con provisioners (4 puntos)
- [x] Configuración completa de aplicación (4 puntos)

```hcl
# Tu código Terraform aquí
```

```yaml
# Tu código Ansible aquí
```

### Ejercicio 4: Pipeline de CI/CD (15 puntos)

**Objetivo**: Crear un pipeline completo de CI/CD para Infrastructure as Code.

**Requerimientos**:

1. Workflow de GitHub Actions para Terraform
2. Workflow de GitHub Actions para Ansible
3. Gates de calidad y seguridad (linting, security scan)
4. Tests automatizados
5. Deployment con aprobación manual para producción

**Entregables**:

- `.github/workflows/terraform-ci.yml`
- `.github/workflows/ansible-ci.yml`
- `.github/workflows/deploy.yml`
- Configuración de tools (checkov, ansible-lint, etc.)

**Criterios de evaluación**:

- [x] Workflows bien estructurados (4 puntos)
- [x] Gates de calidad implementados (4 puntos)
- [x] Tests automatizados funcionales (3 puntos)
- [x] Proceso de deployment seguro (4 puntos)

```yaml
# Tu pipeline de GitHub Actions aquí
```

---

## 🚀 PARTE 3: PROYECTO INTEGRADOR (20 puntos)

### Proyecto: Infraestructura Completa para Aplicación Web

**Scenario**: Eres el arquitecto de infraestructura de una startup que necesita desplegar una aplicación web de e-commerce en AWS. La aplicación requiere alta disponibilidad, seguridad, y capacidad de escalamiento.

**Requerimientos Funcionales**:

1. **Red y Seguridad (5 puntos)**:
   - VPC con subnets públicas y privadas en múltiples AZ
   - Security groups restrictivos
   - NACLs para defensa en profundidad
   - Bastion host para acceso administrativo

2. **Compute y Aplicación (5 puntos)**:
   - Auto Scaling Group con Launch Template
   - Application Load Balancer
   - Configuración automática de servidores web con Ansible
   - SSL/TLS terminado en el ALB

3. **Base de Datos (3 puntos)**:
   - RDS Multi-AZ para alta disponibilidad
   - Subnet group privado
   - Secrets Manager para credenciales

4. **Monitoreo y Logging (3 puntos)**:
   - CloudWatch para métricas y logs
   - SNS para alertas
   - Configuración de logs centralizados

5. **CI/CD y Automatización (4 puntos)**:
   - Pipeline completo de deployment
   - Tests de infraestructura
   - Rollback automático en caso de fallo

**Entregables del Proyecto**:

```
proyecto-final/
├── terraform/
│   ├── modules/
│   │   ├── vpc/
│   │   ├── compute/
│   │   ├── database/
│   │   └── monitoring/
│   ├── environments/
│   │   ├── staging.tfvars
│   │   └── production.tfvars
│   ├── main.tf
│   ├── variables.tf
│   ├── outputs.tf
│   └── README.md
├── ansible/
│   ├── roles/
│   │   ├── common/
│   │   ├── webserver/
│   │   └── monitoring/
│   ├── playbooks/
│   │   ├── site.yml
│   │   └── rolling-update.yml
│   ├── inventory/
│   │   └── aws_ec2.yml
│   └── group_vars/
├── .github/workflows/
│   ├── infrastructure.yml
│   └── application.yml
├── tests/
│   ├── terraform/
│   └── ansible/
├── docs/
│   ├── architecture.md
│   └── runbook.md
└── README.md
```

**Criterios de Evaluación del Proyecto**:

- [x] **Arquitectura (5 puntos)**: Diseño bien pensado, alta disponibilidad, seguridad
- [x] **Código de Calidad (5 puntos)**: Modular, reutilizable, bien documentado
- [x] **Seguridad (3 puntos)**: Mejores prácticas implementadas
- [x] **Automatización (4 puntos)**: CI/CD funcional, tests automatizados
- [x] **Documentación (3 puntos)**: Clara, completa, útil para operaciones

**Tiempo Estimado**: 8-12 horas

---

## 🧪 LABORATORIO PRÁCTICO

### Setup del Ambiente de Testing

```bash
#!/bin/bash
# setup-test-environment.sh

echo "🚀 Configurando ambiente de testing para Fase 4..."

# Crear estructura de proyecto
mkdir -p {terraform/{modules/{vpc,compute,database},environments},ansible/{roles,playbooks,inventory},tests}

# Instalar herramientas necesarias
pip install terraform-compliance ansible-lint molecule[docker]

# Configurar AWS CLI (usuario debe tener credenciales)
aws configure list

# Instalar Terraform
if ! command -v terraform &> /dev/null; then
    wget -O- https://apt.releases.hashicorp.com/gpg | gpg --dearmor | sudo tee /usr/share/keyrings/hashicorp-archive-keyring.gpg
    echo "deb [signed-by=/usr/share/keyrings/hashicorp-archive-keyring.gpg] https://apt.releases.hashicorp.com $(lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/hashicorp.list
    sudo apt update && sudo apt install terraform
fi

# Instalar herramientas de testing
go install github.com/gruntwork-io/terratest@latest

echo "✅ Ambiente configurado correctamente"
echo "📝 Puedes comenzar con los ejercicios de evaluación"
```

### Scripts de Validación

```python
#!/usr/bin/env python3
# validate-exercise.py

import os
import json
import yaml
import subprocess
from pathlib import Path

def validate_terraform_module(module_path):
    """Valida estructura y sintaxis de módulo Terraform"""
    required_files = ['main.tf', 'variables.tf', 'outputs.tf', 'README.md']
    
    for file in required_files:
        if not os.path.exists(f"{module_path}/{file}"):
            return False, f"Missing required file: {file}"
    
    # Validar sintaxis
    result = subprocess.run(['terraform', 'validate'], 
                          cwd=module_path, 
                          capture_output=True)
    
    if result.returncode != 0:
        return False, f"Terraform validation failed: {result.stderr.decode()}"
    
    return True, "Module validation passed"

def validate_ansible_role(role_path):
    """Valida estructura y sintaxis de role Ansible"""
    required_dirs = ['tasks', 'handlers', 'defaults', 'meta']
    
    for dir in required_dirs:
        if not os.path.exists(f"{role_path}/{dir}"):
            return False, f"Missing required directory: {dir}"
    
    # Validar sintaxis con ansible-lint
    result = subprocess.run(['ansible-lint', role_path], 
                          capture_output=True)
    
    if result.returncode != 0:
        return False, f"Ansible lint failed: {result.stdout.decode()}"
    
    return True, "Role validation passed"

def run_validation():
    """Ejecuta todas las validaciones"""
    results = {}
    
    # Validar módulos Terraform
    terraform_modules = ['modules/vpc', 'modules/compute']
    for module in terraform_modules:
        if os.path.exists(module):
            valid, message = validate_terraform_module(module)
            results[module] = {'valid': valid, 'message': message}
    
    # Validar roles Ansible  
    ansible_roles = ['roles/webserver', 'roles/common']
    for role in ansible_roles:
        if os.path.exists(role):
            valid, message = validate_ansible_role(role)
            results[role] = {'valid': valid, 'message': message}
    
    return results

if __name__ == "__main__":
    results = run_validation()
    
    print("🔍 Resultados de Validación:")
    print("=" * 50)
    
    all_passed = True
    for component, result in results.items():
        status = "✅ PASS" if result['valid'] else "❌ FAIL"
        print(f"{status} {component}: {result['message']}")
        if not result['valid']:
            all_passed = False
    
    print("=" * 50)
    if all_passed:
        print("🎉 Todas las validaciones pasaron correctamente!")
    else:
        print("⚠️  Algunas validaciones fallaron. Revisa los errores arriba.")
```

---

## 📊 RÚBRICA DE EVALUACIÓN

### Excelente (90-100 puntos)

- Dominio completo de Terraform y Ansible
- Código modular, reutilizable y bien documentado
- Implementación de todas las mejores prácticas
- Pipeline de CI/CD completamente funcional
- Seguridad integrada desde el diseño
- Proyecto integrador con arquitectura avanzada

### Competente (75-89 puntos)

- Buen conocimiento de las herramientas
- Código funcional con estructura adecuada
- Implementación de la mayoría de mejores prácticas
- Pipeline básico pero funcional
- Consideraciones básicas de seguridad
- Proyecto integrador funcional

### En Desarrollo (60-74 puntos)

- Conocimiento básico de las herramientas
- Código funciona pero necesita mejoras
- Algunas mejores prácticas implementadas
- Pipeline incompleto o con errores
- Seguridad básica o faltante
- Proyecto integrador parcialmente completo

### Insuficiente (< 60 puntos)

- Conocimiento limitado de las herramientas
- Código con errores o no funcional
- Pocas o ninguna mejor práctica implementada
- Sin pipeline o completamente roto
- Sin consideraciones de seguridad
- Proyecto integrador incompleto o no funcional

---

## 🎯 INSTRUCCIONES DE ENTREGA

### Formato de Entrega

1. **Repositorio Git**: Crear un repositorio con toda la solución
2. **Documentación**: README.md detallado con instrucciones
3. **Video Demo**: 10-15 minutos demostrando la solución funcionando
4. **Reporte**: Documento explicando decisiones de arquitectura

### Checklist de Entrega

- [ ] Todas las preguntas teóricas respondidas
- [ ] Todos los ejercicios prácticos completados
- [ ] Proyecto integrador funcional
- [ ] Código versionado en Git
- [ ] Documentación completa
- [ ] Tests automatizados funcionando
- [ ] Video de demostración grabado
- [ ] Reporte de arquitectura incluido

### Fecha Límite

**Plazo**: 15 días calendario desde la asignación
**Entrega**: Enviar link del repositorio y video por email

---

## 🚀 RECURSOS ADICIONALES

### Documentación Oficial

- [Terraform Documentation](https://www.terraform.io/docs/)
- [Ansible Documentation](https://docs.ansible.com/)
- [AWS Provider Documentation](https://registry.terraform.io/providers/hashicorp/aws/latest/docs)

### Herramientas de Testing

- [Terratest](https://terratest.gruntwork.io/)
- [Molecule](https://molecule.readthedocs.io/)
- [Kitchen-Terraform](https://newcontext-oss.github.io/kitchen-terraform/)

### Ejemplos y Referencias

- [Terraform Best Practices](https://www.terraform.io/docs/cloud/guides/recommended-practices/index.html)
- [Ansible Best Practices](https://docs.ansible.com/ansible/latest/user_guide/playbooks_best_practices.html)
- [AWS Well-Architected Framework](https://aws.amazon.com/architecture/well-architected/)

---

## 🎉 CERTIFICACIÓN

Al completar exitosamente esta evaluación (≥75 puntos), recibirás:

- [x] **Certificado Digital**: "Infrastructure as Code Specialist"
- [x] **Badge Digital**: Para LinkedIn y redes sociales  
- [x] **Recomendación**: Carta de recomendación para empleadores
- [x] **Acceso VIP**: Comunidad exclusiva de IaC practitioners

### Próximos Pasos Sugeridos

1. **HashiCorp Certified Terraform Associate**
2. **Red Hat Certified Specialist in Ansible Automation**
3. **AWS Certified DevOps Engineer**
4. **Certified Kubernetes Administrator (CKA)**

---

**¡Buena suerte con tu evaluación! 🚀**

*💡 Recuerda: Esta evaluación no solo mide tu conocimiento técnico, sino también tu capacidad para aplicar Infrastructure as Code en escenarios reales. Toma tu tiempo, planifica bien, y demuestra tu expertise en cada ejercicio.*
