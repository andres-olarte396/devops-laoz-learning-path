# Modelo CIA: Confidencialidad, Integridad, Disponibilidad

## Objetivos
Al completar este módulo serás capaz de:
- Comprender los tres pilares fundamentales de la seguridad de la información
- Aplicar el modelo CIA en el diseño de aplicaciones
- Identificar amenazas contra cada pilar
- Implementar controles de seguridad apropiados

## El Modelo CIA

El modelo CIA es el fundamento de la seguridad de la información, compuesto por tres principios esenciales:

### Confidencialidad (Confidentiality)

**Definición**: Asegurar que la información sea accesible solo a personas autorizadas.

#### Amenazas comunes:
- Acceso no autorizado a datos
- Interceptación de comunicaciones
- Exposición accidental de información
- Ataques de ingeniería social

#### Controles implementables:

**1. Cifrado**
```csharp
// Cifrado de datos sensibles en .NET
public class EncryptionService
{
    private readonly byte[] _key;
    
    public string Encrypt(string plaintext)
    {
        using (var aes = Aes.Create())
        {
            aes.Key = _key;
            var encryptor = aes.CreateEncryptor();
            
            using (var msEncrypt = new MemoryStream())
            {
                using (var csEncrypt = new CryptoStream(msEncrypt, encryptor, CryptoStreamMode.Write))
                using (var swEncrypt = new StreamWriter(csEncrypt))
                {
                    swEncrypt.Write(plaintext);
                }
                return Convert.ToBase64String(msEncrypt.ToArray());
            }
        }
    }
}
```

**2. Control de acceso**
```csharp
// Autorización basada en roles
[Authorize(Roles = "Admin,Manager")]
public class SensitiveDataController : ControllerBase
{
    [HttpGet]
    [Authorize(Policy = "CanViewSensitiveData")]
    public async Task<IActionResult> GetSensitiveData()
    {
        // Solo usuarios autorizados pueden acceder
        return Ok(sensitiveData);
    }
}
```

### Integridad (Integrity)

**Definición**: Garantizar que la información no sea alterada de manera no autorizada.

#### Amenazas comunes:
- Modificación maliciosa de datos
- Corrupción de datos
- Ataques man-in-the-middle
- Errores en la transmisión

#### Controles implementables:

**1. Hashing y checksums**
```csharp
// Verificación de integridad con hash
public class IntegrityService
{
    public string ComputeHash(string data)
    {
        using (var sha256 = SHA256.Create())
        {
            var hashedBytes = sha256.ComputeHash(Encoding.UTF8.GetBytes(data));
            return Convert.ToBase64String(hashedBytes);
        }
    }
    
    public bool VerifyIntegrity(string data, string expectedHash)
    {
        var computedHash = ComputeHash(data);
        return computedHash.Equals(expectedHash, StringComparison.Ordinal);
    }
}
```

**2. Firmas digitales**
```csharp
// Firma digital para validar autenticidad
public class DigitalSignatureService
{
    public byte[] SignData(byte[] data, RSA privateKey)
    {
        return privateKey.SignData(data, HashAlgorithmName.SHA256, RSASignaturePadding.Pkcs1);
    }
    
    public bool VerifySignature(byte[] data, byte[] signature, RSA publicKey)
    {
        return publicKey.VerifyData(data, signature, HashAlgorithmName.SHA256, RSASignaturePadding.Pkcs1);
    }
}
```

### Disponibilidad (Availability)

**Definición**: Asegurar que la información y servicios estén disponibles cuando se necesiten.

#### Amenazas comunes:
- Ataques de denegación de servicio (DoS/DDoS)
- Fallos de hardware/software
- Desastres naturales
- Sobrecarga del sistema

#### Controles implementables:

**1. Redundancia y alta disponibilidad**
```yaml
# Kubernetes Deployment con alta disponibilidad
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mi-aplicacion
spec:
  replicas: 3  # Múltiples instancias
  selector:
    matchLabels:
      app: mi-aplicacion
  template:
    metadata:
      labels:
        app: mi-aplicacion
    spec:
      containers:
      - name: app
        image: mi-app:latest
        resources:
          requests:
            memory: "256Mi"
            cpu: "250m"
          limits:
            memory: "512Mi"
            cpu: "500m"
        livenessProbe:
          httpGet:
            path: /health
            port: 80
          initialDelaySeconds: 30
          periodSeconds: 10
```

**2. Rate limiting**
```csharp
// Implementación de rate limiting
public class RateLimitingMiddleware
{
    private readonly RequestDelegate _next;
    private readonly IMemoryCache _cache;
    
    public async Task InvokeAsync(HttpContext context)
    {
        var clientId = GetClientIdentifier(context);
        var key = $"rate_limit_{clientId}";
        
        if (_cache.TryGetValue(key, out int requestCount))
        {
            if (requestCount >= 100) // 100 requests per minute
            {
                context.Response.StatusCode = 429; // Too Many Requests
                await context.Response.WriteAsync("Rate limit exceeded");
                return;
            }
            _cache.Set(key, requestCount + 1, TimeSpan.FromMinutes(1));
        }
        else
        {
            _cache.Set(key, 1, TimeSpan.FromMinutes(1));
        }
        
        await _next(context);
    }
}
```

## Implementación Práctica del Modelo CIA

### Ejemplo: Sistema de Gestión de Documentos

```csharp
public class DocumentService
{
    private readonly IEncryptionService _encryption;
    private readonly IHashService _hash;
    private readonly IAccessControlService _accessControl;
    
    // CONFIDENCIALIDAD: Cifrar documento antes de almacenar
    public async Task<string> StoreDocumentAsync(Document document, string userId)
    {
        // Verificar autorización
        if (!await _accessControl.CanCreateDocument(userId))
            throw new UnauthorizedAccessException();
            
        // Cifrar contenido sensible
        document.Content = _encryption.Encrypt(document.Content);
        
        // Calcular hash para integridad
        document.ContentHash = _hash.ComputeHash(document.Content);
        
        return await _repository.SaveAsync(document);
    }
    
    // INTEGRIDAD: Verificar que el documento no ha sido alterado
    public async Task<Document> GetDocumentAsync(string documentId, string userId)
    {
        // Verificar autorización (CONFIDENCIALIDAD)
        if (!await _accessControl.CanReadDocument(userId, documentId))
            throw new UnauthorizedAccessException();
            
        var document = await _repository.GetAsync(documentId);
        
        // Verificar integridad
        var currentHash = _hash.ComputeHash(document.Content);
        if (currentHash != document.ContentHash)
            throw new SecurityException("Document integrity compromised");
            
        // Descifrar contenido
        document.Content = _encryption.Decrypt(document.Content);
        
        return document;
    }
}
```

## Métricas y Monitoreo

### KPIs de Seguridad CIA

**Confidencialidad:**
- Número de intentos de acceso no autorizado
- Tiempo de detección de brechas de datos
- Porcentaje de datos cifrados

**Integridad:**
- Número de modificaciones no autorizadas detectadas
- Tiempo de detección de alteraciones
- Frecuencia de verificaciones de integridad

**Disponibilidad:**
- Uptime del sistema (99.9% target)
- Tiempo de respuesta promedio
- Número de incidentes de DoS

```csharp
// Métricas de seguridad
public class SecurityMetrics
{
    private readonly IMetricsCollector _metrics;
    
    public void RecordAccessAttempt(string userId, string resource, bool authorized)
    {
        _metrics.Increment("access_attempts_total", new { 
            user = userId, 
            resource = resource, 
            authorized = authorized.ToString() 
        });
    }
    
    public void RecordIntegrityCheck(string resource, bool passed)
    {
        _metrics.Increment("integrity_checks_total", new { 
            resource = resource, 
            result = passed ? "pass" : "fail" 
        });
    }
    
    public void RecordAvailabilityEvent(string service, double responseTime)
    {
        _metrics.RecordValue("response_time_seconds", responseTime, new { 
            service = service 
        });
    }
}
```

## Ejercicios Prácticos

### Ejercicio 1: Análisis CIA
Analiza una aplicación web típica e identifica:
1. Qué datos requieren confidencialidad
2. Qué información necesita protección de integridad
3. Qué servicios son críticos para la disponibilidad

### Ejercicio 2: Implementación de controles
Implementa los siguientes controles en una aplicación .NET:
1. Cifrado de datos personales
2. Verificación de integridad de archivos subidos
3. Rate limiting para prevenir DoS

### Ejercicio 3: Evaluación de riesgos
Para cada pilar del CIA, identifica:
1. Las principales amenazas
2. El impacto potencial
3. Controles de mitigación apropiados

## Checklist de Implementación CIA

### Confidencialidad ✓
- [ ] Datos sensibles identificados y clasificados
- [ ] Cifrado en tránsito (HTTPS/TLS)
- [ ] Cifrado en reposo para datos críticos
- [ ] Control de acceso implementado
- [ ] Autenticación fuerte habilitada
- [ ] Logs de acceso configurados

### Integridad ✓
- [ ] Checksums implementados para datos críticos
- [ ] Firmas digitales para documentos importantes
- [ ] Validación de entrada robusta
- [ ] Logs de modificación habilitados
- [ ] Backup y versionado implementado
- [ ] Detección de alteraciones automática

### Disponibilidad ✓
- [ ] Redundancia implementada
- [ ] Load balancing configurado
- [ ] Rate limiting implementado
- [ ] Monitoreo de salud configurado
- [ ] Plan de recuperación de desastres
- [ ] Alertas de disponibilidad configuradas

## Recursos Adicionales
- [NIST Cybersecurity Framework](https://www.nist.gov/cyberframework)
- [ISO 27001 Information Security](https://www.iso.org/isoiec-27001-information-security.html)
- [OWASP Security Guidelines](https://owasp.org/)

## Siguiente Paso
[Autenticación y autorización](./02-autenticacion-autorizacion.md)