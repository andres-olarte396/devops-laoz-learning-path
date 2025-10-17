# Módulo 08: Seguridad en Infraestructura como Código (IaC)

## 🎯 Objetivos

- Comprender la importancia de asegurar la IaC para prevenir configuraciones vulnerables en la nube.
- Aprender a escanear plantillas de Terraform y Ansible en busca de malas prácticas de seguridad.
- Integrar el escaneo de IaC en los pipelines de CI/CD.

## 📜 Contenido

### 1. ¿Qué es la Seguridad en IaC?

- **Definición**: Es la práctica de analizar el código que define tu infraestructura (ej. archivos `.tf` de Terraform, playbooks de Ansible, plantillas ARM/Bicep) en busca de configuraciones que podrían llevar a vulnerabilidades de seguridad en el entorno desplegado.
- **Importancia (Shift-Left para la Infraestructura)**: Al igual que SAST para el código de la aplicación, el escaneo de IaC permite detectar problemas de seguridad **antes** de que la infraestructura sea provisionada. Corregir una mala configuración en el código es mucho más rápido y barato que arreglarla en producción.

### 2. Vectores de Ataque Comunes a través de IaC

- **Permisos Excesivos**: Roles de IAM/RBAC con permisos de `*.*` (wildcard).
- **Puertos de Red Expuestos**: Grupos de seguridad que exponen puertos sensibles (como el 22 para SSH o el 3389 para RDP) a todo Internet (`0.0.0.0/0`).
- **Almacenamiento no Cifrado**: Buckets de S3 o discos de máquinas virtuales sin el cifrado en reposo habilitado.
- **Logging y Monitoreo Deshabilitados**: No habilitar logs de auditoría (como CloudTrail en AWS) para recursos críticos.
- **Secretos Hardcodeados**: Incluir contraseñas, tokens o claves API directamente en el código de Terraform.

### 3. Herramientas para Escanear IaC

Estas herramientas funcionan de manera similar a los linters, pero con un enfoque en la seguridad.

- **Checkov**: Una herramienta de código abierto muy popular de Bridgecrew (Palo Alto Networks). Escanea Terraform, CloudFormation, Kubernetes, Ansible y más. Tiene cientos de políticas predefinidas basadas en benchmarks de la industria (CIS).
- **tfsec**: Una herramienta de código abierto específicamente diseñada para Terraform. Es muy rápida y se enfoca en la facilidad de uso.
- **Terrascan**: Otra herramienta de código abierto que escanea múltiples formatos de IaC en busca de incumplimientos de políticas.
- **KICS (Keeping Infrastructure as Code Secure)**: Proyecto de código abierto de Checkmarx que soporta una amplia gama de tecnologías de IaC.

### 4. Integración en el Pipeline de CI/CD

El escaneo de IaC debe ser un paso obligatorio en tu pipeline cada vez que se modifica el código de infraestructura.

```yaml
name: IaC Security Scan
on:
  pull_request:
    paths:
      - 'infrastructure/**'
jobs:
  checkov-scan:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v3

      - name: Run Checkov action
        uses: bridgecrewio/checkov-action@master
        with:
          directory: ./infrastructure
          framework: terraform
          soft_fail: true # No falla el build, solo reporta. Cambiar a 'false' para bloquear.
```

## 🏢 Ejemplo Práctico: Escaneo con Checkov

1. **Código Terraform Vulnerable (`main.tf`)**:
   Este código crea un bucket de S3 público y sin cifrado.

   ```terraform
   resource "aws_s3_bucket" "my_bucket" {
     bucket = "my-super-secret-data-bucket"
     acl    = "public-read" # ¡Vulnerabilidad! El bucket es público.

     # Falta el bloque de cifrado
   }
   ```

2. **Ejecutar Checkov**:
   Instala Checkov (`pip install checkov`) y ejecútalo en el directorio:

   ```bash
   checkov -d .
   ```

3. **Resultados de Checkov**:
   Checkov producirá una salida como esta, indicando las políticas que fallaron y cómo solucionarlas.

   ```plaintext
   Check: S3_BUCKET_ACL_PUBLIC_READ
   FAILED for resource: aws_s3_bucket.my_bucket
   File: /main.tf:1-6
   Guide: https://docs.bridgecrew.io/docs/s3_13-s3-acls

   Check: S3_BUCKET_ENCRYPTION
   FAILED for resource: aws_s3_bucket.my_bucket
   File: /main.tf:1-6
   Guide: https://docs.bridgecrew.io/docs/s3_14-enable-s3-bucket-encryption
   ```

## ✍️ Ejercicio

1. Instala [Checkov](https://www.checkov.io/2.Basics/Installing%20Checkov.html) en tu máquina.
2. Crea un archivo `main.tf` con el código vulnerable del ejemplo anterior (el bucket de S3 público).
3. Ejecuta `checkov -d .` y verifica que detecta las dos vulnerabilidades (ACL pública y falta de cifrado).
4. Sigue las guías de la documentación de Checkov (o busca en la documentación de Terraform) para corregir el código.
5. Vuelve a ejecutar Checkov y confirma que ahora todos los checks pasan.
