# Notas de Ciberseguridad en DevOps

## Principios Fundamentales de Seguridad

### Modelo CIA
- **Confidencialidad**: Proteger información sensible del acceso no autorizado
- **Integridad**: Garantizar que los datos no sean alterados sin autorización
- **Disponibilidad**: Asegurar que los servicios estén disponibles cuando se necesiten

### Defensa en Profundidad
Múltiples capas de seguridad:
1. **Perímetro**: Firewalls, WAF
2. **Red**: Segmentación, VLANs
3. **Host**: Antivirus, hardening
4. **Aplicación**: Validación de entrada, autenticación
5. **Datos**: Cifrado, clasificación

## OWASP Top 10 - Conceptos Clave

### 1. Injection Attacks
```sql
-- Ejemplo vulnerable
string query = "SELECT * FROM users WHERE username = '" + userInput + "'";

-- Versión segura con parámetros
string query = "SELECT * FROM users WHERE username = @username";
```

### 2. Cross-Site Scripting (XSS)
```html
<!-- Vulnerable -->
<div>Hello <%= user.name %></div>

<!-- Seguro -->
<div>Hello <%= HttpUtility.HtmlEncode(user.name) %></div>
```

### 3. Cross-Site Request Forgery (CSRF)
```csharp
// Protección CSRF en .NET Core
[ValidateAntiForgeryToken]
public IActionResult Transfer(TransferModel model)
{
    // Procesar transferencia
}
```

## Seguridad en la Nube

### Azure Security Checklist
- [ ] Configurar Azure AD con MFA
- [ ] Usar Key Vault para secretos
- [ ] Implementar Network Security Groups
- [ ] Habilitar Azure Security Center
- [ ] Configurar logging y monitoring
- [ ] Aplicar RBAC (Role-Based Access Control)

### AWS Security Essentials
- [ ] Configurar IAM policies
- [ ] Usar AWS Secrets Manager
- [ ] Implementar VPC security groups
- [ ] Habilitar CloudTrail
- [ ] Configurar GuardDuty
- [ ] Aplicar principio de menor privilegio

## Seguridad de APIs

### JWT Token Security
```csharp
// Configuración segura de JWT
services.AddAuthentication(JwtBearerDefaults.AuthenticationScheme)
    .AddJwtBearer(options =>
    {
        options.TokenValidationParameters = new TokenValidationParameters
        {
            ValidateIssuer = true,
            ValidateAudience = true,
            ValidateLifetime = true,
            ValidateIssuerSigningKey = true,
            ClockSkew = TimeSpan.Zero
        };
    });
```

### Rate Limiting
```csharp
// Rate limiting en ASP.NET Core
services.AddRateLimiter(options =>
{
    options.AddFixedWindowLimiter("api", opt =>
    {
        opt.PermitLimit = 100;
        opt.Window = TimeSpan.FromMinutes(1);
    });
});
```

## DevSecOps Pipeline

### Security Gates en CI/CD
1. **SAST**(Static Application Security Testing)
   - SonarQube
   - Checkmarx
   - Veracode

2. **DAST**(Dynamic Application Security Testing)
   - OWASP ZAP
   - Burp Suite
   - Nessus

3. **Container Security**
   - Trivy
   - Aqua Security
   - Twistlock

4. **Infrastructure Scanning**
   - Checkov
   - Terraform Compliance
   - AWS Config

### Ejemplo de Pipeline Seguro
```yaml
# GitHub Actions Security Pipeline
name: Security Pipeline
on: [push, pull_request]

jobs:
  security-scan:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2

      - name: Run SAST with SonarQube
        uses: sonarqube-quality-gate-action@master

      - name: Container Security Scan
        run: |
          docker run --rm -v "$PWD:/workspace" \
            aquasec/trivy filesystem --exit-code 1 /workspace

      - name: Infrastructure Security
        run: |
          checkov -d ./terraform --framework terraform
```

## Herramientas Recomendadas

### Desarrollo Seguro
- **IDE Plugins**: SonarLint, Checkmarx CxSAST
- **Code Review**: GitHub Security Advisories, CodeQL
- **Dependency Check**: Snyk, OWASP Dependency Check

### Runtime Security
- **SIEM**: Splunk, Azure Sentinel
- **EDR**: CrowdStrike, Carbon Black
- **Network Monitoring**: Wireshark, Zeek

### Cloud Security
- **CSPM**: Prisma Cloud, CloudSploit
- **CWPP**: Aqua, Sysdig Secure
- **Secret Management**: HashiCorp Vault, Azure Key Vault

## Best Practices

### Secure Coding
1. **Validar todas las entradas**
2. **Usar bibliotecas criptográficas probadas**
3. **Implementar logging de seguridad**
4. **Aplicar principio de menor privilegio**
5. **Mantener dependencias actualizadas**

### Infrastructure Security
1. **Usar IaC para configuración consistente**
2. **Implementar network segmentation**
3. **Cifrar datos en tránsito y reposo**
4. **Configurar monitoring y alertas**
5. **Realizar auditorías regulares**

### Incident Response
1. **Tener un plan de respuesta definido**
2. **Configurar alertas automáticas**
3. **Mantener backups seguros**
4. **Practicar escenarios de breach**
5. **Documentar lecciones aprendidas**

## Recursos Adicionales

- [OWASP Application Security Verification Standard](https://owasp.org/www-project-application-security-verification-standard/)
- [NIST Cybersecurity Framework](https://www.nist.gov/cyberframework)
- [CIS Controls](https://www.cisecurity.org/controls/)
- [SANS Top 25 Software Errors](https://www.sans.org/top25-software-errors/)