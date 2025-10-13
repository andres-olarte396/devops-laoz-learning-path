# 🛡️ Módulo 10: Seguridad y Mejores Prácticas en IaC

[![Security](https://img.shields.io/badge/Security-FF0000?style=for-the-badge&logo=security&logoColor=white)](https://owasp.org/)
[![Terraform](https://img.shields.io/badge/Terraform-7B42BC?style=for-the-badge&logo=terraform&logoColor=white)](https://www.terraform.io/)
[![Ansible](https://img.shields.io/badge/Ansible-EE0000?style=for-the-badge&logo=ansible&logoColor=white)](https://www.ansible.com/)
[![Vault](https://img.shields.io/badge/HashiCorp%20Vault-000000?style=for-the-badge&logo=vault&logoColor=white)](https://www.vaultproject.io/)

## 🎯 Objetivos de Aprendizaje

Al completar este módulo, podrás:

- [x] Implementar manejo seguro de secretos y credenciales
- [x] Aplicar principios de seguridad en Infrastructure as Code
- [x] Configurar políticas de seguridad y compliance
- [x] Usar herramientas de análisis estático y dinámico
- [x] Implementar controles de acceso y autenticación
- [x] Configurar auditoría y logging de seguridad
- [x] Aplicar principios de defensa en profundidad
- [x] Crear políticas de respuesta a incidentes

## 📋 Prerrequisitos

- [x] Conocimiento avanzado de Terraform y Ansible
- [x] Comprensión de conceptos de seguridad básicos
- [x] Experiencia con providers de cloud
- [x] Familiaridad con conceptos de compliance

## 🏗️ Marco de Seguridad para IaC

```mermaid
graph TB
    A[Security Framework] --> B[Identity & Access Management]
    A --> C[Secrets Management]
    A --> D[Network Security]
    A --> E[Data Protection]
    A --> F[Compliance & Governance]
    A --> G[Monitoring & Auditing]
    
    B --> B1[IAM Roles & Policies]
    B --> B2[Service Accounts]
    B --> B3[Multi-Factor Authentication]
    
    C --> C1[HashiCorp Vault]
    C --> C2[AWS Secrets Manager]
    C --> C3[Ansible Vault]
    
    D --> D1[Security Groups]
    D --> D2[Network ACLs]
    D --> D3[VPN & Private Networks]
    
    E --> E1[Encryption at Rest]
    E --> E2[Encryption in Transit]
    E --> E3[Key Management]
    
    F --> F1[Policy as Code]
    F --> F2[Compliance Scanning]
    F --> F3[Risk Assessment]
    
    G --> G1[CloudTrail]
    G --> G2[Security Information Event Management]
    G --> G3[Incident Response]
```

## 🔐 Gestión de Secretos

### HashiCorp Vault con Terraform

```hcl
# terraform/vault-setup.tf
resource "vault_auth_backend" "aws" {
  type = "aws"
  path = "aws"
  
  tune {
    default_lease_ttl  = "1h"
    max_lease_ttl      = "24h"
  }
}

resource "vault_aws_auth_backend_role" "terraform" {
  backend                        = vault_auth_backend.aws.path
  role                          = "terraform-role"
  auth_type                     = "iam"
  bound_iam_principal_arns      = [aws_iam_role.terraform.arn]
  token_ttl                     = 3600
  token_max_ttl                 = 7200
  token_policies                = ["terraform-policy"]
}

resource "vault_policy" "terraform" {
  name = "terraform-policy"
  
  policy = <<EOT
# Permitir lectura de secretos de aplicación
path "secret/data/applications/*" {
  capabilities = ["read"]
}

# Permitir gestión de secrets de infraestructura
path "secret/data/infrastructure/*" {
  capabilities = ["read", "create", "update"]
}

# Permitir gestión de certificados
path "pki/issue/infrastructure" {
  capabilities = ["create", "update"]
}
EOT
}

# Configurar motor de secretos
resource "vault_mount" "secret" {
  path        = "secret"
  type        = "kv"
  options     = { version = "2" }
  description = "Secretos de aplicaciones e infraestructura"
}

# Secretos de base de datos
resource "vault_kv_secret_v2" "database" {
  mount = vault_mount.secret.path
  name  = "infrastructure/database"
  
  data_json = jsonencode({
    username = var.db_username
    password = random_password.db_password.result
    host     = aws_db_instance.main.endpoint
    port     = aws_db_instance.main.port
  })
}

# Secretos de aplicación
resource "vault_kv_secret_v2" "app_secrets" {
  mount = vault_mount.secret.path
  name  = "applications/${var.app_name}"
  
  data_json = jsonencode({
    api_key        = random_password.api_key.result
    jwt_secret     = random_password.jwt_secret.result
    encryption_key = random_password.encryption_key.result
  })
}

# PKI para certificados SSL
resource "vault_mount" "pki" {
  path                      = "pki"
  type                      = "pki"
  default_lease_ttl_seconds = 86400   # 1 día
  max_lease_ttl_seconds     = 2592000 # 30 días
}

resource "vault_pki_secret_backend_root_cert" "root" {
  backend     = vault_mount.pki.path
  type        = "internal"
  common_name = "Internal Root CA"
  ttl         = "8760h" # 1 año
}

resource "vault_pki_secret_backend_role" "infrastructure" {
  backend        = vault_mount.pki.path
  name           = "infrastructure"
  ttl            = "24h"
  max_ttl        = "72h"
  allow_localhost = true
  allowed_domains = [
    "internal.company.com",
    "*.internal.company.com",
    "localhost"
  ]
  allow_subdomains = true
}
```

### Integración de Ansible con Vault

```yaml
---
# ansible/playbooks/vault-integration.yml
- name: Configuración segura con HashiCorp Vault
  hosts: all
  vars:
    vault_addr: "{{ vault_server_url }}"
    vault_token: "{{ vault_auth_token }}"
    
  tasks:
    - name: Autenticarse con Vault usando AWS IAM
      uri:
        url: "{{ vault_addr }}/v1/auth/aws/login"
        method: POST
        body_format: json
        body:
          role: "terraform-role"
          iam_http_request_method: "POST"
          iam_request_url: "{{ vault_aws_sts_endpoint }}"
          iam_request_body: "Action=GetCallerIdentity&Version=2011-06-15"
          iam_request_headers: "{{ vault_aws_headers | to_json }}"
      register: vault_auth
      delegate_to: localhost
      run_once: true
      
    - name: Obtener secretos de base de datos
      uri:
        url: "{{ vault_addr }}/v1/secret/data/infrastructure/database"
        method: GET
        headers:
          X-Vault-Token: "{{ vault_auth.json.auth.client_token }}"
      register: db_secrets
      delegate_to: localhost
      run_once: true
      no_log: true
      
    - name: Obtener secretos de aplicación
      uri:
        url: "{{ vault_addr }}/v1/secret/data/applications/{{ app_name }}"
        method: GET
        headers:
          X-Vault-Token: "{{ vault_auth.json.auth.client_token }}"
      register: app_secrets
      delegate_to: localhost
      run_once: true
      no_log: true
      
    - name: Generar certificado SSL
      uri:
        url: "{{ vault_addr }}/v1/pki/issue/infrastructure"
        method: POST
        headers:
          X-Vault-Token: "{{ vault_auth.json.auth.client_token }}"
        body_format: json
        body:
          common_name: "{{ inventory_hostname }}.internal.company.com"
          ttl: "24h"
      register: ssl_cert
      delegate_to: localhost
      no_log: true
      
    - name: Crear archivo de configuración de aplicación
      template:
        src: app-config-secure.j2
        dest: /etc/myapp/config.yml
        owner: myapp
        group: myapp
        mode: '0600'
      vars:
        database_config: "{{ db_secrets.json.data.data }}"
        app_config: "{{ app_secrets.json.data.data }}"
      notify: restart application
      
    - name: Instalar certificado SSL
      copy:
        content: "{{ ssl_cert.json.data.certificate }}"
        dest: /etc/ssl/certs/server.crt
        owner: root
        group: root
        mode: '0644'
      notify: reload nginx
      
    - name: Instalar clave privada SSL
      copy:
        content: "{{ ssl_cert.json.data.private_key }}"
        dest: /etc/ssl/private/server.key
        owner: root
        group: ssl-cert
        mode: '0640'
      notify: reload nginx
```

### Ansible Vault para Secretos Locales

```yaml
# ansible/group_vars/production/vault.yml (encrypted)
$ANSIBLE_VAULT;1.1;AES256
66386439653262323761393931323833393039653264303537336265663437373663323336663864
3737663533646438343635323630386230633731653664350a636131643831616332306564363463
34656434333938323437303034333131623236656330623061626235333933663836323930653465
3733376565613438310a393130393738343633663035373562663563613264626431353834613631
38313564356233353130326334343832666330656132336163643563316533653030376364663663
3662663565393030363137346562303765393636373030613332

# Variables encriptadas para producción
vault_db_password: supersecretpassword
vault_api_keys:
  stripe: sk_live_xxxxxxxxxxxx
  sendgrid: SG.xxxxxxxxxxxx
vault_certificates:
  ssl_cert: |
    -----BEGIN CERTIFICATE-----
    MIIDXTCCAkWgAwIBAgIJAxxxxxxxxxxxxxxx
    -----END CERTIFICATE-----
  ssl_key: |
    -----BEGIN PRIVATE KEY-----
    MIIEvQIBADANBgkqhkiG9w0BAQEFAASCBKcwggSjAgEAAoIBAQC
    -----END PRIVATE KEY-----
```

```yaml
# ansible/playbooks/secure-deployment.yml  
---
- name: Despliegue seguro con Ansible Vault
  hosts: production
  vars_files:
    - group_vars/production/vault.yml
    
  tasks:
    - name: Configurar base de datos con credenciales encriptadas
      postgresql_user:
        name: "{{ app_db_user }}"
        password: "{{ vault_db_password }}"
        priv: "{{ app_db_name }}:ALL"
        state: present
      no_log: true
      
    - name: Configurar certificados SSL
      block:
        - name: Instalar certificado
          copy:
            content: "{{ vault_certificates.ssl_cert }}"
            dest: /etc/ssl/certs/app.crt
            owner: root
            group: root
            mode: '0644'
            
        - name: Instalar clave privada
          copy:
            content: "{{ vault_certificates.ssl_key }}"
            dest: /etc/ssl/private/app.key
            owner: root
            group: ssl-cert
            mode: '0640'
      notify: restart nginx
      
    - name: Configurar variables de entorno de aplicación
      template:
        src: app.env.j2
        dest: /etc/myapp/.env
        owner: myapp
        group: myapp
        mode: '0600'
      vars:
        api_keys: "{{ vault_api_keys }}"
      notify: restart application
```

## 🔒 Políticas de Seguridad como Código

### Open Policy Agent (OPA) con Terraform

```hcl
# terraform/opa-policies.tf
resource "aws_s3_bucket" "policy_storage" {
  bucket = "${var.company}-opa-policies-${var.environment}"
  
  tags = {
    Purpose = "OPA Policy Storage"
    Environment = var.environment
  }
}

resource "aws_s3_bucket_versioning" "policy_storage" {
  bucket = aws_s3_bucket.policy_storage.id
  versioning_configuration {
    status = "Enabled"
  }
}

resource "aws_s3_object" "terraform_policies" {
  for_each = fileset("${path.module}/policies/terraform/", "*.rego")
  
  bucket = aws_s3_bucket.policy_storage.id
  key    = "terraform/${each.value}"
  source = "${path.module}/policies/terraform/${each.value}"
  etag   = filemd5("${path.module}/policies/terraform/${each.value}")
}

resource "aws_s3_object" "ansible_policies" {
  for_each = fileset("${path.module}/policies/ansible/", "*.rego")
  
  bucket = aws_s3_bucket.policy_storage.id
  key    = "ansible/${each.value}"
  source = "${path.module}/policies/ansible/${each.value}"
  etag   = filemd5("${path.module}/policies/ansible/${each.value}")
}
```

### Políticas OPA para Terraform

```rego
# policies/terraform/security.rego
package terraform.security

import future.keywords

# Negar instancias sin encriptación
deny[msg] {
    input.resource_changes[_].type == "aws_instance"
    resource := input.resource_changes[_]
    not resource.change.after.root_block_device[_].encrypted
    msg := sprintf("EC2 instance '%s' must have encrypted root volume", [resource.address])
}

# Requiere tags obligatorios
required_tags := ["Environment", "Owner", "Project"]

deny[msg] {
    input.resource_changes[_].type in ["aws_instance", "aws_s3_bucket", "aws_rds_cluster"]
    resource := input.resource_changes[_]
    missing_tags := required_tags - object.keys(resource.change.after.tags)
    count(missing_tags) > 0
    msg := sprintf("Resource '%s' is missing required tags: %v", [resource.address, missing_tags])
}

# Prohibir acceso público a S3
deny[msg] {
    input.resource_changes[_].type == "aws_s3_bucket_public_access_block"
    resource := input.resource_changes[_]
    resource.change.after.block_public_acls == false
    msg := sprintf("S3 bucket '%s' must block public ACLs", [resource.address])
}

# Requiere HTTPS para load balancers
deny[msg] {
    input.resource_changes[_].type == "aws_lb_listener"
    resource := input.resource_changes[_]
    resource.change.after.protocol != "HTTPS"
    resource.change.after.port in [80, 443]
    msg := sprintf("Load balancer listener '%s' must use HTTPS", [resource.address])
}

# Validar configuración de RDS
deny[msg] {
    input.resource_changes[_].type == "aws_db_instance"
    resource := input.resource_changes[_]
    not resource.change.after.storage_encrypted
    msg := sprintf("RDS instance '%s' must have encrypted storage", [resource.address])
}

deny[msg] {
    input.resource_changes[_].type == "aws_db_instance"
    resource := input.resource_changes[_]
    resource.change.after.publicly_accessible == true
    msg := sprintf("RDS instance '%s' must not be publicly accessible", [resource.address])
}
```

### Políticas OPA para Ansible

```rego
# policies/ansible/security.rego
package ansible.security

import future.keywords

# Validar configuración SSH
deny[msg] {
    input.tasks[_].name == "Configure SSH"
    task := input.tasks[_]
    task.lineinfile.line contains "PermitRootLogin yes"
    msg := "SSH root login must be disabled"
}

deny[msg] {
    input.tasks[_].name == "Configure SSH"
    task := input.tasks[_]
    task.lineinfile.line contains "PasswordAuthentication yes"
    msg := "SSH password authentication should be disabled"
}

# Validar gestión de usuarios
deny[msg] {
    input.tasks[_].user
    task := input.tasks[_]
    task.user.password
    not task.no_log
    msg := sprintf("Task '%s' handles passwords and must use 'no_log: true'", [task.name])
}

# Requiere uso de become para tareas privilegiadas
privileged_modules := [
    "package", "apt", "yum", "service", "systemd",
    "firewalld", "iptables", "user", "group"
]

deny[msg] {
    task := input.tasks[_]
    module_name := object.keys(task)[_]
    module_name in privileged_modules
    not task.become
    not input.become
    msg := sprintf("Task '%s' requires privilege escalation", [task.name])
}

# Validar configuración de firewall
deny[msg] {
    input.tasks[_].firewalld
    task := input.tasks[_]
    task.firewalld.port
    task.firewalld.source == "0.0.0.0/0"
    msg := sprintf("Firewall rule in task '%s' allows access from anywhere", [task.name])
}
```

## 🔍 Herramientas de Análisis de Seguridad

### Checkov para Terraform

```yaml
# .checkov.yml
framework: 
  - terraform
  - ansible

output: junitxml
output-file-path: reports/checkov-report.xml

check:
  # Incluir checks específicos
  - CKV_AWS_20  # S3 Bucket encrypted
  - CKV_AWS_21  # S3 Bucket versioning
  - CKV_AWS_61  # RDS encrypted
  - CKV_AWS_16  # EBS encrypted
  - CKV_AWS_79  # Instance metadata service v2

skip-check:
  # Saltar checks no aplicables
  - CKV_AWS_8   # LaunchConfiguration sin metadatos IMDSv2

quiet: false
compact: false
verbose: 2

severity: CRITICAL,HIGH,MEDIUM

# Custom policies directory
external-checks-dir: ./custom-policies/

# Baseline para ignorar hallazgos existentes
baseline: .checkov.baseline
```

```bash
#!/bin/bash
# scripts/security-scan.sh

echo "🔍 Ejecutando análisis de seguridad..."

# Checkov para Terraform
echo "📊 Analizando código Terraform con Checkov..."
checkov -d terraform/ \
    --config-file .checkov.yml \
    --output junitxml \
    --output-file-path reports/terraform-security.xml

# tfsec para análisis adicional
echo "🔐 Ejecutando tfsec..."
tfsec terraform/ \
    --format json \
    --out reports/tfsec-results.json

# ansible-lint para playbooks
echo "📋 Analizando playbooks con ansible-lint..."
ansible-lint ansible/playbooks/ \
    --format parseable \
    > reports/ansible-lint.txt

# Semgrep para análisis de código
echo "🔎 Ejecutando Semgrep..."
semgrep --config=auto \
    --json \
    --output reports/semgrep-results.json \
    terraform/ ansible/

echo "✅ Análisis de seguridad completado"
echo "📄 Reportes disponibles en el directorio reports/"
```

### Pipeline de CI/CD con Security Gates

```yaml
# .github/workflows/security-pipeline.yml
name: Security Pipeline

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

jobs:
  security-scan:
    name: Security Analysis
    runs-on: ubuntu-latest
    
    steps:
      - name: Checkout code
        uses: actions/checkout@v4
        
      - name: Setup Python
        uses: actions/setup-python@v4
        with:
          python-version: '3.11'
          
      - name: Install security tools
        run: |
          pip install checkov ansible-lint
          curl -s https://raw.githubusercontent.com/aquasec/tfsec/master/scripts/install_linux.sh | bash
          curl -s https://get.semgrep.dev | bash
          
      - name: Run Checkov
        run: |
          checkov -d terraform/ --config-file .checkov.yml
          checkov -d ansible/ --framework ansible
          
      - name: Run tfsec
        run: tfsec terraform/ --soft-fail
        
      - name: Run ansible-lint
        run: ansible-lint ansible/playbooks/
        
      - name: Run Semgrep
        run: semgrep --config=auto terraform/ ansible/
        
      - name: Upload security results
        uses: actions/upload-artifact@v3
        if: always()
        with:
          name: security-reports
          path: reports/

  policy-validation:
    name: Policy Validation
    runs-on: ubuntu-latest
    needs: security-scan
    
    steps:
      - name: Checkout code
        uses: actions/checkout@v4
        
      - name: Setup OPA
        run: |
          curl -L -o opa https://openpolicyagent.org/downloads/v0.57.0/opa_linux_amd64_static
          chmod +x opa
          sudo mv opa /usr/local/bin/
          
      - name: Validate Terraform policies
        run: |
          terraform plan -out=tfplan.binary
          terraform show -json tfplan.binary > tfplan.json
          opa eval -d policies/terraform/ -i tfplan.json "data.terraform.security.deny[x]"
          
      - name: Validate Ansible policies
        run: |
          for playbook in ansible/playbooks/*.yml; do
            opa eval -d policies/ansible/ -i "$playbook" "data.ansible.security.deny[x]"
          done

  vulnerability-scan:
    name: Vulnerability Scanning
    runs-on: ubuntu-latest
    
    steps:
      - name: Checkout code
        uses: actions/checkout@v4
        
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
```

## 🛡️ Hardening de Infraestructura

### Configuración Segura de AWS

```hcl
# terraform/aws-security-baseline.tf
# Configuración de CloudTrail
resource "aws_cloudtrail" "main" {
  name           = "${var.company}-cloudtrail-${var.environment}"
  s3_bucket_name = aws_s3_bucket.cloudtrail.bucket
  
  include_global_service_events = true
  is_multi_region_trail        = true
  enable_logging               = true
  
  event_selector {
    read_write_type                 = "All"
    include_management_events       = true
    exclude_management_event_sources = []
    
    data_resource {
      type   = "AWS::S3::Object"
      values = ["${aws_s3_bucket.app_data.arn}/*"]
    }
  }
  
  tags = {
    Name        = "Main CloudTrail"
    Environment = var.environment
  }
}

# Configuración de GuardDuty
resource "aws_guardduty_detector" "main" {
  enable = true
  
  finding_publishing_frequency = "FIFTEEN_MINUTES"
  
  datasources {
    s3_logs {
      enable = true
    }
    kubernetes {
      audit_logs {
        enable = true
      }
    }
    malware_protection {
      scan_ec2_instance_with_findings {
        ebs_volumes {
          enable = true
        }
      }
    }
  }
  
  tags = {
    Name        = "Main GuardDuty"
    Environment = var.environment
  }
}

# Configuración de Config
resource "aws_config_configuration_recorder" "main" {
  name     = "${var.company}-config-recorder"
  role_arn = aws_iam_role.config.arn
  
  recording_group {
    all_supported                 = true
    include_global_resource_types = true
  }
}

# Reglas de Config para compliance
resource "aws_config_config_rule" "s3_bucket_public_read_prohibited" {
  name = "s3-bucket-public-read-prohibited"
  
  source {
    owner             = "AWS"
    source_identifier = "S3_BUCKET_PUBLIC_READ_PROHIBITED"
  }
  
  depends_on = [aws_config_configuration_recorder.main]
}

resource "aws_config_config_rule" "rds_instance_public_access_check" {
  name = "rds-instance-public-access-check"
  
  source {
    owner             = "AWS"
    source_identifier = "RDS_INSTANCE_PUBLIC_ACCESS_CHECK"
  }
  
  depends_on = [aws_config_configuration_recorder.main]
}

# Security Hub
resource "aws_securityhub_account" "main" {
  enable_default_standards = true
}

# Habilitar estándares de Security Hub
resource "aws_securityhub_standards_subscription" "aws_foundational" {
  standards_arn = "arn:aws:securityhub:::ruleset/finding-format/aws-foundational-security-standard/v/1.0.0"
  depends_on    = [aws_securityhub_account.main]
}

resource "aws_securityhub_standards_subscription" "cis" {
  standards_arn = "arn:aws:securityhub:::ruleset/finding-format/cis-aws-foundations-benchmark/v/1.2.0"
  depends_on    = [aws_securityhub_account.main]
}
```

### Hardening con Ansible

```yaml
---
# ansible/roles/security-hardening/tasks/main.yml
- name: Aplicar hardening de sistema
  include_tasks: "{{ item }}"
  loop:
    - system-hardening.yml
    - ssh-hardening.yml
    - firewall-hardening.yml
    - audit-hardening.yml
    - service-hardening.yml

# system-hardening.yml
- name: Configurar parámetros del kernel
  sysctl:
    name: "{{ item.name }}"
    value: "{{ item.value }}"
    state: present
    reload: yes
  loop:
    # Network security
    - { name: 'net.ipv4.ip_forward', value: '0' }
    - { name: 'net.ipv4.conf.all.send_redirects', value: '0' }
    - { name: 'net.ipv4.conf.default.send_redirects', value: '0' }
    - { name: 'net.ipv4.conf.all.accept_redirects', value: '0' }
    - { name: 'net.ipv4.conf.default.accept_redirects', value: '0' }
    - { name: 'net.ipv4.conf.all.secure_redirects', value: '0' }
    - { name: 'net.ipv4.conf.default.secure_redirects', value: '0' }
    - { name: 'net.ipv4.conf.all.log_martians', value: '1' }
    - { name: 'net.ipv4.conf.default.log_martians', value: '1' }
    - { name: 'net.ipv4.icmp_echo_ignore_broadcasts', value: '1' }
    - { name: 'net.ipv4.icmp_ignore_bogus_error_responses', value: '1' }
    - { name: 'net.ipv4.conf.all.rp_filter', value: '1' }
    - { name: 'net.ipv4.conf.default.rp_filter', value: '1' }
    - { name: 'net.ipv4.tcp_syncookies', value: '1' }
    # Memory protection
    - { name: 'kernel.randomize_va_space', value: '2' }
    - { name: 'kernel.exec-shield', value: '1' }
    - { name: 'kernel.dmesg_restrict', value: '1' }

- name: Configurar límites de sistema
  pam_limits:
    domain: "{{ item.domain }}"
    limit_type: "{{ item.type }}"
    limit_item: "{{ item.item }}"
    value: "{{ item.value }}"
  loop:
    - { domain: '*', type: 'hard', item: 'core', value: '0' }
    - { domain: '*', type: 'soft', item: 'nproc', value: '65536' }
    - { domain: '*', type: 'hard', item: 'nproc', value: '65536' }
    - { domain: '*', type: 'soft', item: 'nofile', value: '65536' }
    - { domain: '*', type: 'hard', item: 'nofile', value: '65536' }

- name: Desactivar servicios innecesarios
  service:
    name: "{{ item }}"
    state: stopped
    enabled: no
  loop:
    - avahi-daemon
    - cups
    - nfs-common
    - rpcbind
  ignore_errors: yes

- name: Configurar umask seguro
  lineinfile:
    path: /etc/login.defs
    regexp: '^UMASK'
    line: 'UMASK 027'
    backup: yes

# ssh-hardening.yml
- name: Configurar SSH de forma segura
  lineinfile:
    path: /etc/ssh/sshd_config
    regexp: "{{ item.regexp }}"
    line: "{{ item.line }}"
    backup: yes
  loop:
    - { regexp: '^#?Protocol', line: 'Protocol 2' }
    - { regexp: '^#?PermitRootLogin', line: 'PermitRootLogin no' }
    - { regexp: '^#?PasswordAuthentication', line: 'PasswordAuthentication no' }
    - { regexp: '^#?PubkeyAuthentication', line: 'PubkeyAuthentication yes' }
    - { regexp: '^#?AuthorizedKeysFile', line: 'AuthorizedKeysFile .ssh/authorized_keys' }
    - { regexp: '^#?PermitEmptyPasswords', line: 'PermitEmptyPasswords no' }
    - { regexp: '^#?ChallengeResponseAuthentication', line: 'ChallengeResponseAuthentication no' }
    - { regexp: '^#?UsePAM', line: 'UsePAM yes' }
    - { regexp: '^#?X11Forwarding', line: 'X11Forwarding no' }
    - { regexp: '^#?MaxAuthTries', line: 'MaxAuthTries 3' }
    - { regexp: '^#?ClientAliveInterval', line: 'ClientAliveInterval 300' }
    - { regexp: '^#?ClientAliveCountMax', line: 'ClientAliveCountMax 0' }
    - { regexp: '^#?LoginGraceTime', line: 'LoginGraceTime 60' }
    - { regexp: '^#?MaxStartups', line: 'MaxStartups 10:30:100' }
  notify: restart ssh

- name: Configurar banners SSH
  copy:
    content: |
      ******************************************************************
      * This system is for authorized users only.                     *
      * All activities on this system are logged and monitored.       *
      * Unauthorized access is prohibited and will be prosecuted.     *
      ******************************************************************
    dest: /etc/issue.net
    owner: root
    group: root
    mode: '0644'

- name: Activar banner SSH
  lineinfile:
    path: /etc/ssh/sshd_config
    regexp: '^#?Banner'
    line: 'Banner /etc/issue.net'
    backup: yes
  notify: restart ssh

# firewall-hardening.yml
- name: Instalar y configurar UFW
  block:
    - name: Instalar UFW
      package:
        name: ufw
        state: present
        
    - name: Configurar política por defecto
      ufw:
        policy: "{{ item.policy }}"
        direction: "{{ item.direction }}"
      loop:
        - { policy: 'deny', direction: 'incoming' }
        - { policy: 'allow', direction: 'outgoing' }
        
    - name: Permitir SSH
      ufw:
        rule: allow
        port: '22'
        proto: tcp
        
    - name: Permitir puertos de aplicación
      ufw:
        rule: allow
        port: "{{ item }}"
        proto: tcp
      loop: "{{ allowed_ports | default([]) }}"
      
    - name: Habilitar UFW
      ufw:
        state: enabled
```

## 📊 Monitoreo y Auditoría

### Configuración de Logging Centralizado

```yaml
---
# ansible/roles/logging/tasks/main.yml
- name: Configurar rsyslog para envío centralizado
  blockinfile:
    path: /etc/rsyslog.conf
    block: |
      # Configuración para envío a SIEM
      *.* @@{{ siem_server }}:514
      
      # Log de seguridad local
      auth,authpriv.* /var/log/auth.log
      *.info;mail.none;authpriv.none;cron.none /var/log/messages
      
      # Rotación de logs
      $WorkDirectory /var/spool/rsyslog
      $ActionFileDefaultTemplate RSYSLOG_TraditionalFileFormat
      $FileOwner root
      $FileGroup adm
      $FileCreateMode 0640
      $DirCreateMode 0755
    backup: yes
  notify: restart rsyslog

- name: Instalar y configurar auditd
  block:
    - name: Instalar auditd
      package:
        name: auditd
        state: present
        
    - name: Configurar reglas de auditoría
      template:
        src: audit.rules.j2
        dest: /etc/audit/rules.d/audit.rules
        owner: root
        group: root
        mode: '0640'
      notify: restart auditd
      
    - name: Configurar auditd
      template:
        src: auditd.conf.j2
        dest: /etc/audit/auditd.conf
        owner: root
        group: root
        mode: '0640'
      notify: restart auditd

# templates/audit.rules.j2
# Reglas de auditoría de seguridad

# Eliminar reglas existentes
-D

# Buffer size
-b 8192

# Fallo de auditoría = continuar
-f 1

# Auditar cambios de archivos de configuración del sistema
-w /etc/passwd -p wa -k identity
-w /etc/group -p wa -k identity
-w /etc/shadow -p wa -k identity
-w /etc/sudoers -p wa -k identity
-w /etc/ssh/sshd_config -p wa -k sshd

# Auditar acceso a archivos sensibles
-w /etc/hosts -p wa -k network
-w /etc/network/ -p wa -k network
-w /etc/iptables/ -p wa -k network

# Auditar cambios en crontab
-w /etc/crontab -p wa -k cron
-w /etc/cron.d/ -p wa -k cron
-w /var/spool/cron/ -p wa -k cron

# Auditar intentos de login
-w /var/log/faillog -p wa -k logins
-w /var/log/lastlog -p wa -k logins
-w /var/log/tallylog -p wa -k logins

# Auditar cambios de privilegios
-w /bin/su -p x -k privilege_escalation
-w /usr/bin/sudo -p x -k privilege_escalation
-w /etc/sudoers -p rw -k privilege_escalation

# Auditar modificaciones del kernel
-w /sbin/insmod -p x -k modules
-w /sbin/rmmod -p x -k modules
-w /sbin/modprobe -p x -k modules
-a always,exit -F arch=b64 -S init_module -S delete_module -k modules

# Hacer las reglas inmutables
-e 2
```

### Dashboard de Seguridad con Prometheus

```yaml
# ansible/roles/monitoring/tasks/security-metrics.yml
---
- name: Instalar node_exporter para métricas de sistema
  get_url:
    url: "https://github.com/prometheus/node_exporter/releases/download/v1.6.1/node_exporter-1.6.1.linux-amd64.tar.gz"
    dest: /tmp/node_exporter.tar.gz
    
- name: Extraer node_exporter
  unarchive:
    src: /tmp/node_exporter.tar.gz
    dest: /tmp/
    remote_src: yes
    
- name: Instalar node_exporter
  copy:
    src: /tmp/node_exporter-1.6.1.linux-amd64/node_exporter
    dest: /usr/local/bin/node_exporter
    owner: root
    group: root
    mode: '0755'
    remote_src: yes

- name: Crear servicio systemd para node_exporter
  template:
    src: node_exporter.service.j2
    dest: /etc/systemd/system/node_exporter.service
  notify: 
    - reload systemd
    - restart node_exporter

- name: Instalar textfile collector para métricas customizadas
  file:
    path: /var/lib/node_exporter/textfile_collector
    state: directory
    owner: prometheus
    group: prometheus
    mode: '0755'

- name: Script para métricas de seguridad
  copy:
    content: |
      #!/bin/bash
      # Script para recopilar métricas de seguridad
      
      METRICS_FILE="/var/lib/node_exporter/textfile_collector/security.prom"
      
      # Contar intentos de login fallidos
      failed_logins=$(grep "Failed password" /var/log/auth.log | wc -l)
      echo "security_failed_logins_total $failed_logins" > $METRICS_FILE
      
      # Contar conexiones SSH activas
      ssh_connections=$(ss -tn state established '( dport = :22 or sport = :22 )' | wc -l)
      echo "security_ssh_connections_active $ssh_connections" >> $METRICS_FILE
      
      # Estado del firewall
      ufw_status=$(ufw status | grep -q "Status: active" && echo 1 || echo 0)
      echo "security_firewall_active $ufw_status" >> $METRICS_FILE
      
      # Actualizaciones de seguridad pendientes
      security_updates=$(apt list --upgradable 2>/dev/null | grep -c security)
      echo "security_updates_pending $security_updates" >> $METRICS_FILE
      
      # Servicios críticos
      for service in ssh nginx postgresql; do
        status=$(systemctl is-active $service | grep -q active && echo 1 || echo 0)
        echo "security_service_active{service=\"$service\"} $status" >> $METRICS_FILE
      done
    dest: /usr/local/bin/security-metrics.sh
    owner: root
    group: root
    mode: '0755'

- name: Cron job para métricas de seguridad
  cron:
    name: "security metrics collection"
    minute: "*/5"
    job: "/usr/local/bin/security-metrics.sh"
```

## 🚨 Respuesta a Incidentes

### Playbook de Respuesta Automatizada

```yaml
---
# ansible/playbooks/incident-response.yml
- name: Respuesta automática a incidentes de seguridad
  hosts: "{{ target_hosts | default('all') }}"
  vars:
    incident_id: "{{ incident_id | default('UNKNOWN') }}"
    incident_type: "{{ incident_type | default('security_breach') }}"
    
  tasks:
    - name: Crear directorio para evidencia
      file:
        path: "/var/log/incidents/{{ incident_id }}"
        state: directory
        owner: root
        group: root
        mode: '0700'
        
    - name: Recopilar información del sistema
      block:
        - name: Procesos activos
          shell: ps aux > "/var/log/incidents/{{ incident_id }}/processes.log"
          
        - name: Conexiones de red
          shell: netstat -tulpn > "/var/log/incidents/{{ incident_id }}/network.log"
          
        - name: Logs del sistema
          shell: |
            tail -n 1000 /var/log/syslog > "/var/log/incidents/{{ incident_id }}/syslog.log"
            tail -n 1000 /var/log/auth.log > "/var/log/incidents/{{ incident_id }}/auth.log"
            
        - name: Información de memoria
          shell: free -h > "/var/log/incidents/{{ incident_id }}/memory.log"
          
        - name: Uso de disco
          shell: df -h > "/var/log/incidents/{{ incident_id }}/disk.log"
          
    - name: Aislar sistema comprometido
      block:
        - name: Bloquear tráfico de red sospechoso
          ufw:
            rule: deny
            from_ip: "{{ item }}"
          loop: "{{ suspicious_ips | default([]) }}"
          
        - name: Terminar procesos sospechosos
          command: "pkill -f {{ item }}"
          loop: "{{ suspicious_processes | default([]) }}"
          ignore_errors: yes
          
        - name: Desactivar usuarios comprometidos
          user:
            name: "{{ item }}"
            password_lock: yes
          loop: "{{ compromised_users | default([]) }}"
          
      when: incident_type == "security_breach"
      
    - name: Notificar al equipo de seguridad
      uri:
        url: "{{ security_webhook_url }}"
        method: POST
        body_format: json
        body:
          incident_id: "{{ incident_id }}"
          host: "{{ inventory_hostname }}"
          incident_type: "{{ incident_type }}"
          timestamp: "{{ ansible_date_time.iso8601 }}"
          status: "contained"
      delegate_to: localhost
      
    - name: Crear snapshot de evidencia
      command: |
        tar -czf "/var/log/incidents/{{ incident_id }}/evidence-{{ inventory_hostname }}.tar.gz" \
        "/var/log/incidents/{{ incident_id }}/"
      
    - name: Enviar evidencia a sistema central
      synchronize:
        src: "/var/log/incidents/{{ incident_id }}/"
        dest: "{{ evidence_storage_path }}/{{ incident_id }}/{{ inventory_hostname }}/"
        mode: push
      delegate_to: "{{ evidence_server }}"
```

## 🎓 Conclusiones

La seguridad en Infrastructure as Code requiere:

- [x] **Gestión robusta de secretos**: Usar Vault y encriptación
- [x] **Políticas como código**: OPA y compliance automatizado
- [x] **Análisis continuo**: Herramientas de seguridad integradas
- [x] **Monitoreo proactivo**: Métricas y alertas de seguridad
- [x] **Respuesta automatizada**: Playbooks de contención
- [x] **Cultura de seguridad**: Security by design

### Próximos Pasos

1. **Módulo 11**: Testing y CI/CD
2. **Implementar**: Security pipeline en tu organización
3. **Explorar**: Zero Trust Architecture
4. **Certificación**: Certified Information Security Manager (CISM)

---

**📚 Recursos Adicionales:**

- [OWASP Infrastructure as Code Security](https://owasp.org/www-project-devsecops-guideline/)
- [NIST Cybersecurity Framework](https://www.nist.gov/cyberframework)
- [CIS Controls](https://www.cisecurity.org/controls/)
- [SANS Security Policies](https://www.sans.org/information-security-policy/)

---

*💡 **Tip del día**: La seguridad no es un producto, es un proceso. Implementa controles en capas y automatiza todo lo posible. La seguridad debe estar integrada desde el diseño, no añadida como una reflexión tardía.*