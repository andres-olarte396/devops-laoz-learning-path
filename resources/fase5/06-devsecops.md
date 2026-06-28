# Módulo 06: DevSecOps - Integración de Seguridad en el Ciclo de Vida

## 📋 Introducción

DevSecOps es la práctica de integrar la seguridad en cada fase del ciclo de vida de desarrollo de software (SDLC). En lugar de tratar la seguridad como una verificación posterior, DevSecOps hace que la seguridad sea responsabilidad compartida de todos los miembros del equipo de desarrollo y operaciones.

### ¿Qué es DevSecOps?

DevSecOps combina:

- **Development** (Desarrollo): Prácticas de desarrollo ágil
- **Security** (Seguridad): Controles y pruebas de seguridad
- **Operations** (Operaciones): Despliegue y monitoreo continuo

### Principios Fundamentales

- **Shift Left**: Mover la seguridad hacia la izquierda en el SDLC
- **Automatización**: Integrar controles de seguridad automatizados
- **Colaboración**: Fomentar la cultura de seguridad compartida
- **Feedback continuo**: Retroalimentación rápida sobre problemas de seguridad
- **Monitoreo constante**: Vigilancia continua de la seguridad en producción

---

## 🎯 Objetivos de Aprendizaje

Al completar este módulo, serás capaz de:

1. **Implementar** controles de seguridad en pipelines CI/CD
2. **Configurar** herramientas de análisis de seguridad estático (SAST)
3. **Integrar** análisis de seguridad dinámico (DAST)
4. **Gestionar** vulnerabilidades de contenedores
5. **Implementar** escaneo de dependencias
6. **Configurar** políticas de seguridad como código
7. **Monitorear** seguridad en tiempo real
8. **Crear** cultura de seguridad compartida

---

## 🏗️ Fundamentos de DevSecOps

### 1. El Modelo de Seguridad Shift-Left

```yaml
# Tradicional Security Model
phases:
  1_development: "Código sin controles de seguridad"
  2_testing: "Pruebas funcionales únicamente"
  3_deployment: "Despliegue sin validaciones"
  4_security_review: "🚨 Revisión de seguridad AL FINAL"
  5_production: "Problemas de seguridad descubiertos en producción"

# DevSecOps Shift-Left Model
phases:
  1_planning: "Modelado de amenazas y requisitos de seguridad"
  2_development: "Análisis estático, revisión de código seguro"
  3_build: "Escaneo de dependencias, análisis de contenedores"
  4_testing: "Pruebas de seguridad automatizadas (DAST)"
  5_deployment: "Validaciones de configuración de seguridad"
  6_monitoring: "Monitoreo continuo de seguridad"
```

### 2. Pipeline DevSecOps Completo

```yaml
# .github/workflows/devsecops-pipeline.yml
name: DevSecOps Pipeline

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

env:
  REGISTRY: ghcr.io
  IMAGE_NAME: ${{ github.repository }}

jobs:
  # 1. Análisis de código estático (SAST)
  sast-analysis:
    name: Static Application Security Testing
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v4
        with:
          fetch-depth: 0

      - name: Setup .NET
        uses: actions/setup-dotnet@v3
        with:
          dotnet-version: '8.0.x'

      # CodeQL Analysis
      - name: Initialize CodeQL
        uses: github/codeql-action/init@v2
        with:
          languages: csharp
          queries: security-and-quality

      - name: Build application
        run: dotnet build --configuration Release

      - name: Perform CodeQL Analysis
        uses: github/codeql-action/analyze@v2

      # SonarQube Security Scan
      - name: SonarQube Scan
        uses: sonarqube-quality-gate-action@master
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
          SONAR_TOKEN: ${{ secrets.SONAR_TOKEN }}
        with:
          scanMetadataReportFile: target/sonar/report-task.txt

      # Semgrep Security Analysis
      - name: Run Semgrep
        uses: returntocorp/semgrep-action@v1
        with:
          config: >-
            p/security-audit
            p/secrets
            p/owasp-top-ten
        env:
          SEMGREP_APP_TOKEN: ${{ secrets.SEMGREP_APP_TOKEN }}

  # 2. Análisis de dependencias
  dependency-analysis:
    name: Dependency Security Scan
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      # .NET Dependencies
      - name: Setup .NET
        uses: actions/setup-dotnet@v3
        with:
          dotnet-version: '8.0.x'

      - name: Restore dependencies
        run: dotnet restore

      # Snyk Dependency Scan
      - name: Run Snyk to check for vulnerabilities
        uses: snyk/actions/dotnet@master
        env:
          SNYK_TOKEN: ${{ secrets.SNYK_TOKEN }}
        with:
          args: --severity-threshold=high --fail-on=upgradable

      # OWASP Dependency Check
      - name: Run OWASP Dependency Check
        uses: dependency-check/Dependency-Check_Action@main
        with:
          project: 'DevSecOps-Demo'
          path: '.'
          format: 'HTML,JSON,XML'
          args: >
            --enableRetired
            --enableExperimental
            --scan '**/*.csproj'

      - name: Upload OWASP Dependency Check results
        uses: actions/upload-artifact@v3
        with:
          name: dependency-check-report
          path: reports/

  # 3. Container Security Scan
  container-security:
    name: Container Security Analysis
    runs-on: ubuntu-latest
    needs: [sast-analysis]
    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Build Docker image
        run: |
          docker build -t ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}:${{ github.sha }} .

      # Trivy Container Scan
      - name: Run Trivy vulnerability scanner
        uses: aquasecurity/trivy-action@master
        with:
          image-ref: '${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}:${{ github.sha }}'
          format: 'sarif'
          output: 'trivy-results.sarif'
          severity: 'CRITICAL,HIGH'
          exit-code: '1'

      - name: Upload Trivy scan results
        uses: github/codeql-action/upload-sarif@v2
        if: always()
        with:
          sarif_file: 'trivy-results.sarif'

      # Anchore Container Scan
      - name: Anchore Container Scan
        uses: anchore/scan-action@v3
        with:
          image: '${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}:${{ github.sha }}'
          fail-build: true
          severity-cutoff: high

      # Docker Scout
      - name: Docker Scout CVEs
        uses: docker/scout-action@v1
        with:
          command: cves
          image: '${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}:${{ github.sha }}'
          sarif-file: scout-report.sarif
          summary: true

  # 4. Infrastructure as Code Security
  iac-security:
    name: Infrastructure Security Scan
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      # Checkov IaC Security
      - name: Run Checkov action
        uses: bridgecrewio/checkov-action@master
        with:
          directory: ./infrastructure
          framework: terraform,dockerfile,kubernetes
          output_format: sarif
          output_file_path: checkov-report.sarif
          quiet: true
          soft_fail: false

      # Terrascan
      - name: Run Terrascan
        uses: tenable/terrascan-action@main
        with:
          iac_type: 'terraform'
          iac_version: 'v1.0'
          policy_type: 'aws'
          only_warn: false
          sarif_upload: true

      # tfsec
      - name: Run tfsec
        uses: aquasecurity/tfsec-action@v1.0.0
        with:
          soft_fail: false

  # 5. Dynamic Application Security Testing (DAST)
  dast-testing:
    name: Dynamic Security Testing
    runs-on: ubuntu-latest
    needs: [container-security]
    services:
      app:
        image: ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}:${{ github.sha }}
        ports:
          - 8080:80
        options: >-
          --health-cmd "curl -f http://localhost/health || exit 1"
          --health-interval 30s
          --health-timeout 10s
          --health-retries 3

    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      # OWASP ZAP Security Scan
      - name: ZAP Baseline Scan
        uses: zaproxy/action-baseline@v0.10.0
        with:
          target: 'http://localhost:8080'
          rules_file_name: '.zap/rules.tsv'
          cmd_options: '-a'

      - name: ZAP Full Scan
        uses: zaproxy/action-full-scan@v0.8.0
        with:
          target: 'http://localhost:8080'
          rules_file_name: '.zap/rules.tsv'
          cmd_options: '-a'

      # Nuclei Security Scanner
      - name: Run Nuclei Scanner
        uses: projectdiscovery/nuclei-action@main
        with:
          target: 'http://localhost:8080'
          templates: 'cves,vulnerabilities,exposed-panels'
          output: 'nuclei-report.txt'

  # 6. Security Policy Enforcement
  policy-enforcement:
    name: Security Policy as Code
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      # Open Policy Agent (OPA)
      - name: Install OPA
        run: |
          curl -L -o opa https://openpolicyagent.org/downloads/v0.58.0/opa_linux_amd64_static
          chmod 755 ./opa

      - name: Run OPA Policy Tests
        run: |
          ./opa test policies/

      # Falco Rules Validation
      - name: Validate Falco Rules
        run: |
          docker run --rm -v $(pwd)/security/falco:/rules falcosecurity/falco:latest \
            falco --validate /rules/security_rules.yaml

      # Conftest Policy Testing
      - name: Run Conftest
        uses: instrumenta/conftest-action@master
        with:
          files: 'k8s/*.yaml'
          policy: 'policies/kubernetes'

  # 7. Compliance and Reporting
  compliance-reporting:
    name: Compliance and Security Reporting
    runs-on: ubuntu-latest
    needs: [sast-analysis, dependency-analysis, container-security, dast-testing]
    if: always()
    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      # Aggregate Security Reports
      - name: Download all artifacts
        uses: actions/download-artifact@v3

      - name: Generate Security Dashboard
        run: |
          python scripts/security-dashboard.py \
            --sast-report codeql-results.sarif \
            --dependency-report dependency-check-report.json \
            --container-report trivy-results.sarif \
            --dast-report zap-report.json \
            --output security-dashboard.html

      - name: Upload Security Dashboard
        uses: actions/upload-artifact@v3
        with:
          name: security-dashboard
          path: security-dashboard.html

      # Send Notifications
      - name: Send Security Report to Slack
        uses: 8398a7/action-slack@v3
        with:
          status: ${{ job.status }}
          custom_payload: |
            {
              attachments: [{
                color: '${{ job.status }}' === 'success' ? 'good' : 'danger',
                title: 'DevSecOps Pipeline Results',
                fields: [{
                  title: 'Project',
                  value: '${{ github.repository }}',
                  short: true
                }, {
                  title: 'Branch',
                  value: '${{ github.ref }}',
                  short: true
                }]
              }]
            }
        env:
          SLACK_WEBHOOK_URL: ${{ secrets.SLACK_WEBHOOK }}

  # 8. Deploy with Security Validation
  secure-deployment:
    name: Secure Deployment
    runs-on: ubuntu-latest
    needs: [policy-enforcement, compliance-reporting]
    if: github.ref == 'refs/heads/main' && success()
    environment: production
    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Deploy to staging for security validation
        run: |
          # Deploy to staging environment
          kubectl apply -f k8s/staging/ --dry-run=server

      - name: Run Runtime Security Validation
        run: |
          # Runtime security checks
          python scripts/runtime-security-check.py --environment staging

      - name: Deploy to production
        run: |
          kubectl apply -f k8s/production/
          
      - name: Enable Runtime Security Monitoring
        run: |
          # Enable Falco rules for production
          kubectl apply -f security/falco/production-rules.yaml
```

---

## 🔒 Herramientas de Análisis de Seguridad

### 1. Static Application Security Testing (SAST)

#### CodeQL Configuration

```yaml
# .github/codeql/codeql-config.yml
name: "CodeQL Config"

queries:
  - uses: security-and-quality
  - uses: security-experimental

paths-ignore:
  - "**/bin/**"
  - "**/obj/**"
  - "**/packages/**"
  - "**/node_modules/**"
  - "**/*.Designer.cs"
  - "**/*.generated.cs"

paths:
  - "src/**"
  - "api/**"

query-filters:
  - exclude:
      id: cs/web/xss
      problem.severity: warning
```

#### SonarQube Configuration

```yaml
# sonar-project.properties
sonar.projectKey=devsecops-demo
sonar.organization=your-org
sonar.host.url=https://sonarcloud.io

# Source and test directories
sonar.sources=src
sonar.tests=tests
sonar.exclusions=**/bin/**,**/obj/**,**/*.Designer.cs

# Security-specific settings
sonar.security.hotspots.inheritFromParent=true
sonar.security.review.rating=A

# Quality Gates
sonar.qualitygate.wait=true
```

#### Semgrep Rules

```yaml
# .semgrep/custom-rules.yml
rules:
  - id: hardcoded-secrets
    pattern-either:
      - pattern: |
          $PASSWORD = "..."
      - pattern: |
          $API_KEY = "..."
      - pattern: |
          $SECRET = "..."
    message: Hardcoded secret detected
    languages: [csharp]
    severity: ERROR
    metadata:
      category: security
      subcategory: [vuln]
      cwe: "CWE-798: Use of Hard-coded Credentials"
      
  - id: sql-injection-risk
    pattern: |
      $QUERY = $"SELECT * FROM ... WHERE ... = {$INPUT}"
    message: Potential SQL injection vulnerability
    languages: [csharp]
    severity: WARNING
    metadata:
      category: security
      subcategory: [vuln]
      cwe: "CWE-89: SQL Injection"

  - id: unsafe-deserialization
    pattern-either:
      - pattern: |
          JsonSerializer.Deserialize<$TYPE>($INPUT)
      - pattern: |
          BinaryFormatter.Deserialize($STREAM)
    message: Unsafe deserialization detected
    languages: [csharp]
    severity: ERROR
```

### 2. Dynamic Application Security Testing (DAST)

#### OWASP ZAP Configuration

```yaml
# .zap/rules.tsv
10016 IGNORE (Cookie No HttpOnly Flag)
10017 IGNORE (Cookie Without Secure Flag)
10020 IGNORE (X-Frame-Options Header Not Set)
10021 IGNORE (X-Content-Type-Options Header Missing)
40018 IGNORE (SQL Injection - Hypersonic SQL)
90022 IGNORE (Application Error Disclosure)

# Custom ZAP script
# .zap/custom-scan.py
#!/usr/bin/env python3

import time
from zapv2 import ZAPv2

# ZAP Configuration
zap = ZAPv2(proxies={'http': 'http://127.0.0.1:8080', 'https': 'http://127.0.0.1:8080'})

# Target application
target = 'http://localhost:5000'

# Start ZAP session
print('Starting ZAP session...')
zap.core.new_session('DevSecOps-DAST-Session')

# Configure authentication
print('Configuring authentication...')
zap.authentication.set_authentication_method('1', 'formBasedAuthentication', 
    'loginUrl=http://localhost:5000/login&loginRequestData=username%3D{%USERNAME%}%26password%3D{%PASSWORD%}')

# Spider scan
print('Starting spider scan...')
zap.spider.scan(target)
time.sleep(10)

while int(zap.spider.status()) < 100:
    print(f'Spider progress: {zap.spider.status()}%')
    time.sleep(5)

print('Spider scan completed')

# Active security scan
print('Starting active security scan...')
zap.ascan.scan(target)

while int(zap.ascan.status()) < 100:
    print(f'Active scan progress: {zap.ascan.status()}%')
    time.sleep(10)

print('Active scan completed')

# Generate reports
print('Generating security reports...')
with open('zap-report.html', 'w') as f:
    f.write(zap.core.htmlreport())

with open('zap-report.json', 'w') as f:
    f.write(zap.core.jsonreport())

print('DAST scan completed successfully!')
```

### 3. Container Security Scanning

#### Trivy Configuration

```yaml
# trivy.yaml
format: sarif
output: trivy-results.sarif
severity: 
  - CRITICAL
  - HIGH
  - MEDIUM
exit-code: 1
timeout: 15m

# Security checks
security-checks:
  - vuln
  - config
  - secret

# Ignore specific vulnerabilities (with justification)
ignore:
  - CVE-2019-12345  # Fixed in next release, low risk in our context
  - CVE-2020-67890  # False positive, not applicable to our use case

# Custom policies
custom-policies:
  - policy/docker-best-practices.rego
  - policy/kubernetes-security.rego
```

#### Custom Docker Security Policy

```rego
# policy/docker-best-practices.rego
package docker.security

# Deny running as root
deny[msg] {
    input[i].Cmd == "user"
    val := input[i].Value
    val[0] == "root"
    msg := "Container should not run as root user"
}

# Require health check
deny[msg] {
    not has_healthcheck
    msg := "Container must include a HEALTHCHECK instruction"
}

has_healthcheck {
    input[_].Cmd == "healthcheck"
}

# Deny privileged containers
deny[msg] {
    input[i].Cmd == "run"
    contains(input[i].Value[0], "--privileged")
    msg := "Container should not run in privileged mode"
}

# Require non-root user
deny[msg] {
    not has_user_instruction
    msg := "Dockerfile must specify a non-root USER"
}

has_user_instruction {
    input[_].Cmd == "user"
    input[_].Value[0] != "root"
    input[_].Value[0] != "0"
}
```

---

## 🛡️ Configuración de Seguridad en Kubernetes

### 1. Pod Security Standards

```yaml
# security/pod-security-policy.yaml
apiVersion: v1
kind: Namespace
metadata:
  name: secure-namespace
  labels:
    pod-security.kubernetes.io/enforce: restricted
    pod-security.kubernetes.io/audit: restricted
    pod-security.kubernetes.io/warn: restricted

---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: secure-app
  namespace: secure-namespace
spec:
  replicas: 3
  selector:
    matchLabels:
      app: secure-app
  template:
    metadata:
      labels:
        app: secure-app
    spec:
      serviceAccountName: secure-app-sa
      securityContext:
        runAsNonRoot: true
        runAsUser: 10001
        runAsGroup: 10001
        fsGroup: 10001
        seccompProfile:
          type: RuntimeDefault
      containers:
      - name: app
        image: myapp:secure
        ports:
        - containerPort: 8080
        securityContext:
          allowPrivilegeEscalation: false
          readOnlyRootFilesystem: true
          runAsNonRoot: true
          runAsUser: 10001
          capabilities:
            drop:
            - ALL
            add:
            - NET_BIND_SERVICE
        resources:
          limits:
            memory: "256Mi"
            cpu: "500m"
          requests:
            memory: "128Mi"
            cpu: "250m"
        livenessProbe:
          httpGet:
            path: /health
            port: 8080
          initialDelaySeconds: 30
          periodSeconds: 10
        readinessProbe:
          httpGet:
            path: /ready
            port: 8080
          initialDelaySeconds: 5
          periodSeconds: 5
        env:
        - name: DB_PASSWORD
          valueFrom:
            secretKeyRef:
              name: db-secret
              key: password
        volumeMounts:
        - name: tmp-volume
          mountPath: /tmp
        - name: app-config
          mountPath: /app/config
          readOnly: true
      volumes:
      - name: tmp-volume
        emptyDir: {}
      - name: app-config
        configMap:
          name: app-config

---
apiVersion: v1
kind: ServiceAccount
metadata:
  name: secure-app-sa
  namespace: secure-namespace
automountServiceAccountToken: false

---
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: secure-namespace
  name: secure-app-role
rules:
- apiGroups: [""]
  resources: ["configmaps"]
  verbs: ["get", "list"]

---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: secure-app-binding
  namespace: secure-namespace
subjects:
- kind: ServiceAccount
  name: secure-app-sa
  namespace: secure-namespace
roleRef:
  kind: Role
  name: secure-app-role
  apiGroup: rbac.authorization.k8s.io
```

### 2. Network Policies

```yaml
# security/network-policies.yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: default-deny-all
  namespace: secure-namespace
spec:
  podSelector: {}
  policyTypes:
  - Ingress
  - Egress

---
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-app-ingress
  namespace: secure-namespace
spec:
  podSelector:
    matchLabels:
      app: secure-app
  policyTypes:
  - Ingress
  ingress:
  - from:
    - namespaceSelector:
        matchLabels:
          name: ingress-nginx
    - podSelector:
        matchLabels:
          app: load-balancer
    ports:
    - protocol: TCP
      port: 8080

---
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-app-egress
  namespace: secure-namespace
spec:
  podSelector:
    matchLabels:
      app: secure-app
  policyTypes:
  - Egress
  egress:
  # Allow DNS
  - to: []
    ports:
    - protocol: UDP
      port: 53
  # Allow database connection
  - to:
    - namespaceSelector:
        matchLabels:
          name: database
    ports:
    - protocol: TCP
      port: 5432
  # Allow HTTPS outbound for external APIs
  - to: []
    ports:
    - protocol: TCP
      port: 443
```

### 3. Falco Security Rules

```yaml
# security/falco/security-rules.yaml
- rule: Unexpected outbound connection
  desc: Detect unexpected outbound connections
  condition: >
    outbound and not (
      (fd.sport in (http_port, https_port)) or
      (fd.dport in (http_port, https_port)) or
      (fd.sport in (mysql_port, postgres_port)) or
      (proc.name in (allowed_processes))
    )
  output: >
    Unexpected outbound connection 
    (command=%proc.cmdline pid=%proc.pid connection=%fd.name)
  priority: WARNING
  tags: [network, mitre_command_and_control]

- rule: Privilege escalation attempt
  desc: Detect attempts to escalate privileges
  condition: >
    spawned_process and (
      proc.name in (su, sudo, doas) or
      proc.args contains "sudo" or
      proc.args contains "su -"
    )
  output: >
    Privilege escalation attempt 
    (user=%user.name command=%proc.cmdline pid=%proc.pid)
  priority: HIGH
  tags: [privilege_escalation, mitre_privilege_escalation]

- rule: Sensitive file access
  desc: Detect access to sensitive files
  condition: >
    open_read and (
      fd.name startswith /etc/shadow or
      fd.name startswith /etc/passwd or
      fd.name startswith /etc/sudoers or
      fd.name contains "id_rsa" or
      fd.name contains "id_dsa"
    )
  output: >
    Sensitive file accessed 
    (file=%fd.name command=%proc.cmdline pid=%proc.pid)
  priority: HIGH
  tags: [file_access, mitre_credential_access]

- rule: Container runtime modification
  desc: Detect modifications to container runtime
  condition: >
    spawned_process and (
      proc.name in (docker, runc, containerd) or
      proc.pname in (docker, runc, containerd)
    ) and not (
      proc.args contains "ps" or
      proc.args contains "inspect" or
      proc.args contains "logs"
    )
  output: >
    Container runtime modification detected 
    (command=%proc.cmdline pid=%proc.pid)
  priority: CRITICAL
  tags: [container, mitre_execution]
```

---

## 📊 Monitoreo de Seguridad en Tiempo Real

### 1. Security Dashboard Script

```python
#!/usr/bin/env python3
# scripts/security-dashboard.py

import json
import argparse
from datetime import datetime
from jinja2 import Template
import matplotlib.pyplot as plt
import base64
from io import BytesIO

class SecurityDashboard:
    def __init__(self):
        self.metrics = {
            'sast': {'critical': 0, 'high': 0, 'medium': 0, 'low': 0},
            'dast': {'critical': 0, 'high': 0, 'medium': 0, 'low': 0},
            'dependency': {'critical': 0, 'high': 0, 'medium': 0, 'low': 0},
            'container': {'critical': 0, 'high': 0, 'medium': 0, 'low': 0}
        }
        self.total_vulnerabilities = 0
        self.security_score = 0

    def parse_sarif_report(self, report_path, category):
        """Parse SARIF format security reports"""
        try:
            with open(report_path, 'r') as f:
                sarif_data = json.load(f)
            
            for run in sarif_data.get('runs', []):
                for result in run.get('results', []):
                    level = result.get('level', 'note').lower()
                    severity = self.map_severity(level)
                    self.metrics[category][severity] += 1
                    self.total_vulnerabilities += 1
        except Exception as e:
            print(f"Error parsing {report_path}: {e}")

    def parse_dependency_report(self, report_path):
        """Parse OWASP Dependency Check JSON report"""
        try:
            with open(report_path, 'r') as f:
                data = json.load(f)
            
            for dependency in data.get('dependencies', []):
                for vulnerability in dependency.get('vulnerabilities', []):
                    severity = vulnerability.get('severity', 'LOW').lower()
                    if severity == 'critical':
                        self.metrics['dependency']['critical'] += 1
                    elif severity == 'high':
                        self.metrics['dependency']['high'] += 1
                    elif severity == 'medium':
                        self.metrics['dependency']['medium'] += 1
                    else:
                        self.metrics['dependency']['low'] += 1
                    self.total_vulnerabilities += 1
        except Exception as e:
            print(f"Error parsing dependency report: {e}")

    def map_severity(self, level):
        """Map different severity levels to standard format"""
        level = level.lower()
        if level in ['critical', 'error']:
            return 'critical'
        elif level in ['high', 'warning']:
            return 'high'
        elif level in ['medium', 'note']:
            return 'medium'
        else:
            return 'low'

    def calculate_security_score(self):
        """Calculate overall security score (0-100)"""
        # Weight different severity levels
        weights = {'critical': 10, 'high': 5, 'medium': 2, 'low': 1}
        
        total_weighted_issues = 0
        for category in self.metrics:
            for severity, count in self.metrics[category].items():
                total_weighted_issues += count * weights[severity]
        
        # Calculate score (100 = perfect, 0 = maximum issues)
        max_score = 100
        if total_weighted_issues == 0:
            self.security_score = max_score
        else:
            # Logarithmic scale for scoring
            import math
            self.security_score = max(0, max_score - (math.log(total_weighted_issues + 1) * 15))
        
        return round(self.security_score, 1)

    def generate_charts(self):
        """Generate security metrics charts"""
        charts = {}
        
        # Vulnerability by category chart
        categories = list(self.metrics.keys())
        critical_counts = [self.metrics[cat]['critical'] for cat in categories]
        high_counts = [self.metrics[cat]['high'] for cat in categories]
        medium_counts = [self.metrics[cat]['medium'] for cat in categories]
        low_counts = [self.metrics[cat]['low'] for cat in categories]
        
        fig, ax = plt.subplots(figsize=(10, 6))
        width = 0.6
        
        x = range(len(categories))
        ax.bar(x, critical_counts, width, label='Critical', color='#dc3545')
        ax.bar(x, high_counts, width, bottom=critical_counts, label='High', color='#fd7e14')
        ax.bar(x, medium_counts, width, 
               bottom=[critical_counts[i] + high_counts[i] for i in range(len(categories))], 
               label='Medium', color='#ffc107')
        ax.bar(x, low_counts, width,
               bottom=[critical_counts[i] + high_counts[i] + medium_counts[i] for i in range(len(categories))],
               label='Low', color='#28a745')
        
        ax.set_xlabel('Security Categories')
        ax.set_ylabel('Number of Issues')
        ax.set_title('Security Vulnerabilities by Category and Severity')
        ax.set_xticks(x)
        ax.set_xticklabels([cat.upper() for cat in categories])
        ax.legend()
        
        # Save to base64 string
        buffer = BytesIO()
        plt.savefig(buffer, format='png', dpi=150, bbox_inches='tight')
        buffer.seek(0)
        charts['vulnerabilities_chart'] = base64.b64encode(buffer.getvalue()).decode()
        plt.close()
        
        # Security score gauge
        fig, ax = plt.subplots(figsize=(8, 6), subplot_kw=dict(projection='polar'))
        
        # Create gauge chart
        theta = [0, self.security_score/100 * 2 * 3.14159]
        r = [0.8, 0.8]
        
        colors = ['#dc3545' if self.security_score < 50 else '#ffc107' if self.security_score < 80 else '#28a745']
        ax.plot(theta, r, linewidth=20, color=colors[0])
        ax.set_ylim(0, 1)
        ax.set_title(f'Security Score: {self.security_score}/100', size=16, weight='bold')
        ax.grid(True)
        
        buffer = BytesIO()
        plt.savefig(buffer, format='png', dpi=150, bbox_inches='tight')
        buffer.seek(0)
        charts['security_score_chart'] = base64.b64encode(buffer.getvalue()).decode()
        plt.close()
        
        return charts

    def generate_html_report(self, output_path):
        """Generate comprehensive HTML security dashboard"""
        charts = self.generate_charts()
        security_score = self.calculate_security_score()
        
        template = Template('''
        <!DOCTYPE html>
        <html>
        <head>
            <title>DevSecOps Security Dashboard</title>
            <meta charset="utf-8">
            <meta name="viewport" content="width=device-width, initial-scale=1">
            <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.1.3/dist/css/bootstrap.min.css" rel="stylesheet">
            <style>
                .metric-card { transition: transform 0.2s; }
                .metric-card:hover { transform: translateY(-5px); }
                .severity-critical { border-left: 5px solid #dc3545; }
                .severity-high { border-left: 5px solid #fd7e14; }
                .severity-medium { border-left: 5px solid #ffc107; }
                .severity-low { border-left: 5px solid #28a745; }
                .security-score { font-size: 3rem; font-weight: bold; }
                .score-excellent { color: #28a745; }
                .score-good { color: #ffc107; }
                .score-poor { color: #dc3545; }
            </style>
        </head>
        <body>
            <div class="container-fluid py-4">
                <div class="row">
                    <div class="col-12">
                        <h1 class="text-center mb-4">🛡️ DevSecOps Security Dashboard</h1>
                        <p class="text-center text-muted">Generated on {{ timestamp }}</p>
                    </div>
                </div>
                
                <!-- Security Score -->
                <div class="row mb-4">
                    <div class="col-md-6 mx-auto">
                        <div class="card text-center">
                            <div class="card-body">
                                <h3>Overall Security Score</h3>
                                <div class="security-score {{ score_class }}">{{ security_score }}/100</div>
                                <img src="data:image/png;base64,{{ charts.security_score_chart }}" class="img-fluid mt-3">
                            </div>
                        </div>
                    </div>
                </div>
                
                <!-- Metrics Cards -->
                <div class="row mb-4">
                    {% for category, metrics in vulnerability_metrics.items() %}
                    <div class="col-md-3 mb-3">
                        <div class="card metric-card h-100">
                            <div class="card-body">
                                <h5 class="card-title">{{ category.upper() }}</h5>
                                <div class="row">
                                    <div class="col-6">
                                        <div class="severity-critical p-2 mb-2 rounded">
                                            <strong>Critical:</strong> {{ metrics.critical }}
                                        </div>
                                        <div class="severity-high p-2 mb-2 rounded">
                                            <strong>High:</strong> {{ metrics.high }}
                                        </div>
                                    </div>
                                    <div class="col-6">
                                        <div class="severity-medium p-2 mb-2 rounded">
                                            <strong>Medium:</strong> {{ metrics.medium }}
                                        </div>
                                        <div class="severity-low p-2 mb-2 rounded">
                                            <strong>Low:</strong> {{ metrics.low }}
                                        </div>
                                    </div>
                                </div>
                                <hr>
                                <strong>Total: {{ metrics.critical + metrics.high + metrics.medium + metrics.low }}</strong>
                            </div>
                        </div>
                    </div>
                    {% endfor %}
                </div>
                
                <!-- Charts -->
                <div class="row mb-4">
                    <div class="col-12">
                        <div class="card">
                            <div class="card-body text-center">
                                <h5 class="card-title">Vulnerability Distribution</h5>
                                <img src="data:image/png;base64,{{ charts.vulnerabilities_chart }}" class="img-fluid">
                            </div>
                        </div>
                    </div>
                </div>
                
                <!-- Summary Statistics -->
                <div class="row">
                    <div class="col-md-6">
                        <div class="card">
                            <div class="card-body">
                                <h5 class="card-title">Security Summary</h5>
                                <ul class="list-group list-group-flush">
                                    <li class="list-group-item d-flex justify-content-between">
                                        <span>Total Vulnerabilities:</span>
                                        <strong>{{ total_vulnerabilities }}</strong>
                                    </li>
                                    <li class="list-group-item d-flex justify-content-between">
                                        <span>Critical Issues:</span>
                                        <strong class="text-danger">{{ critical_total }}</strong>
                                    </li>
                                    <li class="list-group-item d-flex justify-content-between">
                                        <span>High Priority Issues:</span>
                                        <strong class="text-warning">{{ high_total }}</strong>
                                    </li>
                                    <li class="list-group-item d-flex justify-content-between">
                                        <span>Security Score:</span>
                                        <strong class="{{ score_class }}">{{ security_score }}/100</strong>
                                    </li>
                                </ul>
                            </div>
                        </div>
                    </div>
                    <div class="col-md-6">
                        <div class="card">
                            <div class="card-body">
                                <h5 class="card-title">Recommendations</h5>
                                <div class="alert alert-info">
                                    {% if security_score >= 80 %}
                                    <strong>🎉 Excellent Security Posture!</strong><br>
                                    Your application has a strong security posture. Continue monitoring and keep dependencies updated.
                                    {% elif security_score >= 60 %}
                                    <strong>⚠️ Good Security, Some Issues</strong><br>
                                    Address critical and high-priority vulnerabilities. Review security policies and automated scans.
                                    {% else %}
                                    <strong>🚨 Security Needs Attention</strong><br>
                                    Immediate action required! Address critical vulnerabilities and implement additional security measures.
                                    {% endif %}
                                </div>
                                
                                {% if critical_total > 0 %}
                                <div class="alert alert-danger">
                                    <strong>Critical Action Required:</strong><br>
                                    {{ critical_total }} critical vulnerabilities detected. Address immediately before deployment.
                                </div>
                                {% endif %}
                            </div>
                        </div>
                    </div>
                </div>
            </div>
            
            <script src="https://cdn.jsdelivr.net/npm/bootstrap@5.1.3/dist/js/bootstrap.bundle.min.js"></script>
        </body>
        </html>
        ''')
        
        # Calculate totals
        critical_total = sum(self.metrics[cat]['critical'] for cat in self.metrics)
        high_total = sum(self.metrics[cat]['high'] for cat in self.metrics)
        
        # Determine score class for styling
        if security_score >= 80:
            score_class = 'score-excellent'
        elif security_score >= 60:
            score_class = 'score-good'
        else:
            score_class = 'score-poor'
        
        html_content = template.render(
            timestamp=datetime.now().strftime("%Y-%m-%d %H:%M:%S"),
            vulnerability_metrics=self.metrics,
            security_score=security_score,
            total_vulnerabilities=self.total_vulnerabilities,
            critical_total=critical_total,
            high_total=high_total,
            charts=charts,
            score_class=score_class
        )
        
        with open(output_path, 'w') as f:
            f.write(html_content)
        
        print(f"Security dashboard generated: {output_path}")
        print(f"Security Score: {security_score}/100")
        print(f"Total Vulnerabilities: {self.total_vulnerabilities}")

def main():
    parser = argparse.ArgumentParser(description='Generate DevSecOps Security Dashboard')
    parser.add_argument('--sast-report', help='SAST SARIF report file')
    parser.add_argument('--dependency-report', help='Dependency check JSON report')
    parser.add_argument('--container-report', help='Container security SARIF report')
    parser.add_argument('--dast-report', help='DAST JSON report')
    parser.add_argument('--output', required=True, help='Output HTML file')
    
    args = parser.parse_args()
    
    dashboard = SecurityDashboard()
    
    if args.sast_report:
        dashboard.parse_sarif_report(args.sast_report, 'sast')
    
    if args.dependency_report:
        dashboard.parse_dependency_report(args.dependency_report)
    
    if args.container_report:
        dashboard.parse_sarif_report(args.container_report, 'container')
    
    if args.dast_report:
        # Implement DAST report parsing based on your tool
        pass
    
    dashboard.generate_html_report(args.output)

if __name__ == '__main__':
    main()
```

---

## 📋 Ejercicios Prácticos

### Ejercicio 1: Configurar Pipeline DevSecOps Básico

**Objetivo**: Implementar controles de seguridad en un pipeline CI/CD.

**Tareas**:

1. Configurar análisis SAST con CodeQL
2. Implementar escaneo de dependencias con Snyk
3. Añadir análisis de contenedores con Trivy
4. Crear gates de calidad de seguridad

### Ejercicio 2: Implementar Políticas de Seguridad

**Objetivo**: Crear políticas de seguridad como código.

**Componentes**:

- Políticas OPA para Kubernetes
- Reglas Falco para runtime security
- Configuración de Pod Security Standards
- Network Policies restrictivas

### Ejercicio 3: Security Dashboard

**Objetivo**: Crear dashboard de seguridad centralizado.

**Tareas**:

- Agregar métricas de múltiples herramientas
- Generar reportes HTML automatizados
- Configurar alertas de seguridad
- Implementar scoring de seguridad

---

## 🧪 Laboratorio

### Lab 1: DevSecOps Pipeline Completo

1. **Configurar herramientas de seguridad**:

   ```bash
   # Instalar herramientas localmente
   curl -s https://raw.githubusercontent.com/aquasecurity/trivy/main/contrib/install.sh | sh
   docker pull owasp/zap2docker-stable
   ```

2. **Ejecutar análisis de seguridad**:
   - SAST con múltiples herramientas
   - DAST con OWASP ZAP
   - Container scanning con Trivy

3. **Configurar políticas**:
   - Pod Security Standards
   - Network Policies
   - Falco rules

### Lab 2: Incident Response

1. **Simular vulnerabilidades**
2. **Respuesta automatizada**
3. **Forensics y análisis**
4. **Remediación y lessons learned**

---

## 📖 Best Practices

### 1. Principios de DevSecOps

```yaml
principles:
  shift_left:
    - "Integrar seguridad desde el inicio"
    - "Automatizar controles de seguridad"
    - "Feedback rápido sobre issues"
  
  shared_responsibility:
    - "Desarrolladores responsables de código seguro"
    - "Operations responsable de infraestructura segura"
    - "Security como facilitador, no gatekeeper"
  
  continuous_monitoring:
    - "Monitoreo de seguridad 24/7"
    - "Alertas automáticas en tiempo real"
    - "Respuesta rápida a incidentes"
```

### 2. Gestión de Vulnerabilidades

```yaml
vulnerability_management:
  severity_levels:
    critical: "Fix within 24 hours"
    high: "Fix within 1 week"
    medium: "Fix within 1 month"
    low: "Fix in next release cycle"
  
  exceptions:
    process: "Formal risk acceptance required"
    approval: "Security team + Engineering manager"
    documentation: "Risk assessment + mitigation plan"
    review: "Quarterly re-evaluation"
```

---

## ✅ Resumen del Módulo

En este módulo has aprendido:

1. **Principios de DevSecOps** y shift-left security
2. **Pipeline de seguridad** con múltiples herramientas (SAST, DAST, SCA)
3. **Container security** con Trivy y políticas de seguridad
4. **Kubernetes security** con Pod Security Standards y Network Policies
5. **Infrastructure security** con IaC scanning
6. **Runtime security** con Falco y monitoreo
7. **Security dashboard** para visibilidad centralizada
8. **Compliance y reporting** automatizado
9. **Incident response** y gestión de vulnerabilidades

### Próximas etapas

- **Módulo 07**: Secret Management con HashiCorp Vault y AWS Secrets Manager
- **Módulo 08**: Auditorías y cumplimiento normativo
- **Módulo 09**: Vulnerability scanning avanzado

¡Continúa con el siguiente módulo para dominar la gestión de secretos!
