# Módulo 07: Secret Management - HashiCorp Vault y AWS Secrets Manager

## 📋 Introducción

La gestión segura de secretos es fundamental en cualquier arquitectura DevOps moderna. Los secretos incluyen contraseñas, claves API, certificados, tokens de acceso y cualquier información sensible que las aplicaciones necesiten para funcionar. Una gestión inadecuada puede llevar a brechas de seguridad críticas.

### ¿Qué es Secret Management?

Secret Management es el conjunto de herramientas, procesos y políticas para:

- **Almacenar** secretos de forma segura y encriptada
- **Rotar** secretos automáticamente para reducir riesgos
- **Auditar** el acceso y uso de secretos
- **Controlar** quién puede acceder a qué secretos
- **Distribuir** secretos de forma segura a aplicaciones

### Problemas Comunes sin Secret Management

- Secretos hardcodeados en código fuente
- Contraseñas compartidas por email o chat
- Falta de rotación de credenciales
- Sin auditoría de acceso a secretos
- Credenciales almacenadas en texto plano

---

## 🎯 Objetivos de Aprendizaje

Al completar este módulo, serás capaz de:

1. **Instalar y configurar** HashiCorp Vault
2. **Integrar** AWS Secrets Manager en aplicaciones
3. **Implementar** rotación automática de secretos
4. **Configurar** políticas de acceso granulares
5. **Integrar** secret management en pipelines CI/CD
6. **Monitorear** el uso de secretos y detectar anomalías
7. **Implementar** dynamic secrets para bases de datos
8. **Configurar** encryption as a service

---

## 🏗️ HashiCorp Vault

### 1. Instalación y Configuración

#### Docker Compose Setup

```yaml
# docker-compose.vault.yml
version: '3.8'

services:
  vault:
    image: hashicorp/vault:1.15.2
    container_name: vault
    restart: unless-stopped
    ports:
      - "8200:8200"
    environment:
      VAULT_DEV_ROOT_TOKEN_ID: "dev-root-token"
      VAULT_DEV_LISTEN_ADDRESS: "0.0.0.0:8200"
      VAULT_LOCAL_CONFIG: |
        {
          "backend": {
            "consul": {
              "address": "consul:8500",
              "path": "vault/"
            }
          },
          "listener": {
            "tcp": {
              "address": "0.0.0.0:8200",
              "tls_disable": true
            }
          },
          "ui": true,
          "disable_mlock": true,
          "api_addr": "http://127.0.0.1:8200"
        }
    cap_add:
      - IPC_LOCK
    volumes:
      - vault_data:/vault/data
      - vault_logs:/vault/logs
      - ./vault/config:/vault/config
      - ./vault/policies:/vault/policies
    command: ["vault", "server", "-config=/vault/config/vault.hcl"]
    networks:
      - vault_network
    depends_on:
      - consul

  consul:
    image: hashicorp/consul:1.16.1
    container_name: consul
    restart: unless-stopped
    ports:
      - "8500:8500"
    environment:
      CONSUL_BIND_INTERFACE: eth0
    volumes:
      - consul_data:/consul/data
    command: [
      "consul", "agent",
      "-server",
      "-bootstrap-expect=1",
      "-datacenter=dc1",
      "-data-dir=/consul/data",
      "-node=consul-server",
      "-bind={{ GetInterfaceIP \"eth0\" }}",
      "-client=0.0.0.0",
      "-ui"
    ]
    networks:
      - vault_network

  # Vault Agent para auto-auth
  vault-agent:
    image: hashicorp/vault:1.15.2
    container_name: vault-agent
    restart: unless-stopped
    volumes:
      - ./vault/agent:/vault/config
      - vault_agent_data:/vault/data
    command: ["vault", "agent", "-config=/vault/config/agent.hcl"]
    networks:
      - vault_network
    depends_on:
      - vault

volumes:
  vault_data:
  vault_logs:
  vault_agent_data:
  consul_data:

networks:
  vault_network:
    driver: bridge
```

#### Configuración de Vault

```hcl
# vault/config/vault.hcl
ui = true
disable_mlock = true

storage "consul" {
  address = "consul:8500"
  path    = "vault/"
}

listener "tcp" {
  address     = "0.0.0.0:8200"
  tls_disable = true
}

api_addr = "http://127.0.0.1:8200"
cluster_addr = "https://127.0.0.1:8201"

# Telemetry
telemetry {
  prometheus_retention_time = "30s"
  disable_hostname = true
}

# Seal configuration (Auto-unseal with cloud KMS in production)
# seal "awskms" {
#   region     = "us-east-1"
#   kms_key_id = "alias/vault-unseal-key"
# }

# Entropy Augmentation (Enterprise feature)
# entropy "seal" {
#   mode = "augmentation"
# }
```

### 2. Inicialización y Configuración Básica

```bash
#!/bin/bash
# scripts/vault-setup.sh

set -e

VAULT_ADDR="http://localhost:8200"
export VAULT_ADDR

# Función para esperar que Vault esté listo
wait_for_vault() {
    echo "Waiting for Vault to be ready..."
    while ! curl -s "$VAULT_ADDR/v1/sys/health" > /dev/null; do
        sleep 2
    done
    echo "Vault is ready!"
}

# Inicializar Vault
init_vault() {
    echo "Initializing Vault..."
    
    # Verificar si ya está inicializado
    if vault status | grep -q "Initialized.*true"; then
        echo "Vault is already initialized"
        return 0
    fi
    
    # Inicializar con 5 key shares y threshold de 3
    vault operator init \
        -key-shares=5 \
        -key-threshold=3 \
        -format=json > /tmp/vault-init.json
    
    echo "Vault initialized successfully!"
    echo "Please securely store the unseal keys and root token from /tmp/vault-init.json"
    
    # Extraer las claves de unseal
    for i in {0..4}; do
        echo "Unseal Key $((i+1)): $(cat /tmp/vault-init.json | jq -r ".unseal_keys_b64[$i]")"
    done
    
    echo "Root Token: $(cat /tmp/vault-init.json | jq -r '.root_token')"
}

# Desbloquear Vault
unseal_vault() {
    echo "Unsealing Vault..."
    
    if [ ! -f /tmp/vault-init.json ]; then
        echo "Error: Vault initialization file not found!"
        exit 1
    fi
    
    # Usar 3 claves para desbloquear
    for i in {0..2}; do
        KEY=$(cat /tmp/vault-init.json | jq -r ".unseal_keys_b64[$i]")
        vault operator unseal "$KEY"
    done
    
    echo "Vault unsealed successfully!"
}

# Configurar autenticación
setup_auth() {
    echo "Setting up authentication methods..."
    
    # Usar root token para configuración inicial
    ROOT_TOKEN=$(cat /tmp/vault-init.json | jq -r '.root_token')
    export VAULT_TOKEN="$ROOT_TOKEN"
    
    # Habilitar métodos de autenticación
    vault auth enable userpass
    vault auth enable approle
    vault auth enable aws
    vault auth enable kubernetes
    
    echo "Authentication methods enabled!"
}

# Configurar secret engines
setup_secret_engines() {
    echo "Setting up secret engines..."
    
    # Key-Value v2
    vault secrets enable -path=secret kv-v2
    vault secrets enable -path=app-secrets kv-v2
    
    # Database secrets engine
    vault secrets enable database
    
    # PKI para certificados
    vault secrets enable pki
    vault secrets tune -max-lease-ttl=87600h pki
    
    # AWS secrets engine
    vault secrets enable aws
    
    # Transit para encryption as a service
    vault secrets enable transit
    
    echo "Secret engines configured!"
}

# Configurar políticas básicas
setup_policies() {
    echo "Setting up policies..."
    
    # Política para desarrolladores
    vault policy write dev-policy - <<EOF
# Permitir lectura/escritura en secret/dev/*
path "secret/data/dev/*" {
  capabilities = ["create", "read", "update", "delete", "list"]
}

# Permitir lectura en secret/shared/*
path "secret/data/shared/*" {
  capabilities = ["read", "list"]
}

# Permitir uso del transit engine para encriptación
path "transit/encrypt/dev-key" {
  capabilities = ["update"]
}

path "transit/decrypt/dev-key" {
  capabilities = ["update"]
}
EOF

    # Política para aplicaciones de producción
    vault policy write prod-app-policy - <<EOF
# Solo lectura en secretos de producción
path "secret/data/prod/*" {
  capabilities = ["read"]
}

# Acceso a base de datos dinámica
path "database/creds/prod-db-role" {
  capabilities = ["read"]
}

# Encriptación con clave de producción
path "transit/encrypt/prod-key" {
  capabilities = ["update"]
}

path "transit/decrypt/prod-key" {
  capabilities = ["update"]
}
EOF

    # Política para CI/CD
    vault policy write cicd-policy - <<EOF
# Acceso limitado para deployments
path "secret/data/cicd/*" {
  capabilities = ["read"]
}

path "secret/data/staging/*" {
  capabilities = ["create", "read", "update", "delete"]
}

# Generar certificados para staging
path "pki/issue/staging-cert" {
  capabilities = ["update"]
}
EOF

    echo "Policies configured!"
}

# Configurar usuarios y roles
setup_users_roles() {
    echo "Setting up users and roles..."
    
    # Usuario desarrollador
    vault write auth/userpass/users/developer \
        password="dev-password" \
        policies="dev-policy"
    
    # AppRole para aplicaciones
    vault write auth/approle/role/prod-app \
        bind_secret_id=true \
        token_policies="prod-app-policy" \
        token_ttl=1h \
        token_max_ttl=4h \
        secret_id_ttl=24h
    
    # Obtener role-id para AppRole
    ROLE_ID=$(vault read -field=role_id auth/approle/role/prod-app/role-id)
    SECRET_ID=$(vault write -field=secret_id auth/approle/role/prod-app/secret-id)
    
    echo "AppRole credentials:"
    echo "Role ID: $ROLE_ID"
    echo "Secret ID: $SECRET_ID"
    
    echo "Users and roles configured!"
}

# Configurar dynamic secrets para base de datos
setup_database_secrets() {
    echo "Setting up database dynamic secrets..."
    
    # Configurar conexión a PostgreSQL
    vault write database/config/postgresql \
        plugin_name=postgresql-database-plugin \
        connection_url="postgresql://{{username}}:{{password}}@postgres:5432/myapp?sslmode=disable" \
        allowed_roles="dev-db-role,prod-db-role" \
        username="vault" \
        password="vault-password"
    
    # Crear rol para desarrollo
    vault write database/roles/dev-db-role \
        db_name=postgresql \
        creation_statements="CREATE ROLE \"{{name}}\" WITH LOGIN PASSWORD '{{password}}' VALID UNTIL '{{expiration}}'; \
                            GRANT SELECT, INSERT, UPDATE, DELETE ON ALL TABLES IN SCHEMA public TO \"{{name}}\";" \
        default_ttl="1h" \
        max_ttl="24h"
    
    # Crear rol para producción
    vault write database/roles/prod-db-role \
        db_name=postgresql \
        creation_statements="CREATE ROLE \"{{name}}\" WITH LOGIN PASSWORD '{{password}}' VALID UNTIL '{{expiration}}'; \
                            GRANT SELECT ON ALL TABLES IN SCHEMA public TO \"{{name}}\";" \
        default_ttl="30m" \
        max_ttl="2h"
    
    echo "Database secrets configured!"
}

# Configurar PKI para certificados
setup_pki() {
    echo "Setting up PKI..."
    
    # Generar CA root
    vault write pki/root/generate/internal \
        common_name="My Company Root CA" \
        ttl=87600h
    
    # Configurar URLs
    vault write pki/config/urls \
        issuing_certificates="http://vault:8200/v1/pki/ca" \
        crl_distribution_points="http://vault:8200/v1/pki/crl"
    
    # Crear rol para certificados
    vault write pki/roles/staging-cert \
        allowed_domains="staging.mycompany.com" \
        allow_subdomains=true \
        max_ttl="720h"
    
    vault write pki/roles/prod-cert \
        allowed_domains="mycompany.com" \
        allow_subdomains=true \
        max_ttl="168h"
    
    echo "PKI configured!"
}

# Configurar encryption as a service
setup_transit() {
    echo "Setting up Transit engine..."
    
    # Crear claves de encriptación
    vault write transit/keys/dev-key type=aes256-gcm96
    vault write transit/keys/prod-key type=aes256-gcm96
    vault write transit/keys/pii-key type=aes256-gcm96
    
    echo "Transit keys created!"
}

# Función principal
main() {
    wait_for_vault
    
    if [ "$1" = "init" ]; then
        init_vault
        unseal_vault
        setup_auth
        setup_secret_engines
        setup_policies
        setup_users_roles
        setup_database_secrets
        setup_pki
        setup_transit
        
        echo "Vault setup completed successfully!"
        echo "Web UI available at: $VAULT_ADDR"
    elif [ "$1" = "unseal" ]; then
        unseal_vault
    else
        echo "Usage: $0 {init|unseal}"
        exit 1
    fi
}

main "$@"
```

### 3. Integración con Aplicaciones

#### .NET Core Integration

```csharp
// Vault/VaultService.cs
using VaultSharp;
using VaultSharp.V1.AuthMethods.AppRole;
using VaultSharp.V1.Commons;
using Microsoft.Extensions.Options;

public class VaultConfiguration
{
    public string Address { get; set; } = string.Empty;
    public string RoleId { get; set; } = string.Empty;
    public string SecretId { get; set; } = string.Empty;
    public string MountPoint { get; set; } = "secret";
}

public interface IVaultService
{
    Task<string> GetSecretAsync(string path, string key);
    Task<Dictionary<string, object>> GetSecretsAsync(string path);
    Task SetSecretAsync(string path, string key, string value);
    Task<string> EncryptAsync(string keyName, string plaintext);
    Task<string> DecryptAsync(string keyName, string ciphertext);
    Task<DatabaseCredentials> GetDatabaseCredentialsAsync(string role);
}

public class VaultService : IVaultService
{
    private readonly IVaultClient _vaultClient;
    private readonly VaultConfiguration _config;
    private readonly ILogger<VaultService> _logger;

    public VaultService(IOptions<VaultConfiguration> config, ILogger<VaultService> logger)
    {
        _config = config.Value;
        _logger = logger;
        _vaultClient = CreateVaultClient();
    }

    private IVaultClient CreateVaultClient()
    {
        var appRoleAuthMethod = new AppRoleAuthMethodInfo(_config.RoleId, _config.SecretId);
        var vaultClientSettings = new VaultClientSettings(_config.Address, appRoleAuthMethod);
        
        return new VaultClient(vaultClientSettings);
    }

    public async Task<string> GetSecretAsync(string path, string key)
    {
        try
        {
            _logger.LogInformation("Retrieving secret from path: {Path}, key: {Key}", path, key);
            
            var secret = await _vaultClient.V1.Secrets.KeyValue.V2
                .ReadSecretAsync(path, mountPoint: _config.MountPoint);
            
            if (secret?.Data?.Data?.TryGetValue(key, out var value) == true)
            {
                return value.ToString() ?? string.Empty;
            }
            
            throw new KeyNotFoundException($"Secret key '{key}' not found in path '{path}'");
        }
        catch (Exception ex)
        {
            _logger.LogError(ex, "Error retrieving secret from path: {Path}, key: {Key}", path, key);
            throw;
        }
    }

    public async Task<Dictionary<string, object>> GetSecretsAsync(string path)
    {
        try
        {
            var secret = await _vaultClient.V1.Secrets.KeyValue.V2
                .ReadSecretAsync(path, mountPoint: _config.MountPoint);
            
            return secret?.Data?.Data ?? new Dictionary<string, object>();
        }
        catch (Exception ex)
        {
            _logger.LogError(ex, "Error retrieving secrets from path: {Path}", path);
            throw;
        }
    }

    public async Task SetSecretAsync(string path, string key, string value)
    {
        try
        {
            var secrets = new Dictionary<string, object> { { key, value } };
            
            await _vaultClient.V1.Secrets.KeyValue.V2
                .WriteSecretAsync(path, secrets, mountPoint: _config.MountPoint);
            
            _logger.LogInformation("Secret written to path: {Path}, key: {Key}", path, key);
        }
        catch (Exception ex)
        {
            _logger.LogError(ex, "Error writing secret to path: {Path}, key: {Key}", path, key);
            throw;
        }
    }

    public async Task<string> EncryptAsync(string keyName, string plaintext)
    {
        try
        {
            var encryptionRequest = new EncryptRequestOptions
            {
                Base64EncodedPlainText = Convert.ToBase64String(System.Text.Encoding.UTF8.GetBytes(plaintext))
            };

            var result = await _vaultClient.V1.Secrets.Transit
                .EncryptAsync(keyName, encryptionRequest);

            return result.Data.CipherText;
        }
        catch (Exception ex)
        {
            _logger.LogError(ex, "Error encrypting data with key: {KeyName}", keyName);
            throw;
        }
    }

    public async Task<string> DecryptAsync(string keyName, string ciphertext)
    {
        try
        {
            var decryptionRequest = new DecryptRequestOptions
            {
                CipherText = ciphertext
            };

            var result = await _vaultClient.V1.Secrets.Transit
                .DecryptAsync(keyName, decryptionRequest);

            return System.Text.Encoding.UTF8.GetString(
                Convert.FromBase64String(result.Data.Base64EncodedPlainText));
        }
        catch (Exception ex)
        {
            _logger.LogError(ex, "Error decrypting data with key: {KeyName}", keyName);
            throw;
        }
    }

    public async Task<DatabaseCredentials> GetDatabaseCredentialsAsync(string role)
    {
        try
        {
            var credentials = await _vaultClient.V1.Secrets.Database
                .GetCredentialsAsync(role);

            return new DatabaseCredentials
            {
                Username = credentials.Data.Username,
                Password = credentials.Data.Password,
                LeaseId = credentials.LeaseId,
                LeaseDuration = credentials.LeaseDurationSeconds
            };
        }
        catch (Exception ex)
        {
            _logger.LogError(ex, "Error getting database credentials for role: {Role}", role);
            throw;
        }
    }
}

public class DatabaseCredentials
{
    public string Username { get; set; } = string.Empty;
    public string Password { get; set; } = string.Empty;
    public string LeaseId { get; set; } = string.Empty;
    public int LeaseDuration { get; set; }
}

// Program.cs
public class Program
{
    public static void Main(string[] args)
    {
        var builder = WebApplication.CreateBuilder(args);

        // Configurar Vault
        builder.Services.Configure<VaultConfiguration>(
            builder.Configuration.GetSection("Vault"));
        
        builder.Services.AddSingleton<IVaultService, VaultService>();
        
        // Configurar servicios
        builder.Services.AddControllers();
        builder.Services.AddHealthChecks()
            .AddCheck<VaultHealthCheck>("vault");

        var app = builder.Build();

        app.UseHealthChecks("/health");
        app.MapControllers();

        app.Run();
    }
}

// Health Check para Vault
public class VaultHealthCheck : IHealthCheck
{
    private readonly IVaultService _vaultService;

    public VaultHealthCheck(IVaultService vaultService)
    {
        _vaultService = vaultService;
    }

    public async Task<HealthCheckResult> CheckHealthAsync(
        HealthCheckContext context,
        CancellationToken cancellationToken = default)
    {
        try
        {
            // Intentar leer un secreto de prueba
            await _vaultService.GetSecretAsync("health", "check");
            return HealthCheckResult.Healthy("Vault is accessible");
        }
        catch (Exception ex)
        {
            return HealthCheckResult.Unhealthy("Vault is not accessible", ex);
        }
    }
}
```

---

## ☁️ AWS Secrets Manager

### 1. Configuración con Terraform

```hcl
# terraform/secrets-manager.tf
resource "aws_secretsmanager_secret" "database_credentials" {
  name                    = "prod/database/credentials"
  description             = "Database credentials for production"
  recovery_window_in_days = 7

  replica {
    region = "us-west-2"
  }

  tags = {
    Environment = "production"
    Application = "myapp"
    Terraform   = "true"
  }
}

resource "aws_secretsmanager_secret_version" "database_credentials" {
  secret_id = aws_secretsmanager_secret.database_credentials.id
  secret_string = jsonencode({
    username = "admin"
    password = random_password.db_password.result
    engine   = "mysql"
    host     = aws_rds_instance.main.endpoint
    port     = 3306
    dbname   = aws_db_instance.main.db_name
  })
}

resource "random_password" "db_password" {
  length  = 32
  special = true
}

# API Keys para aplicación
resource "aws_secretsmanager_secret" "api_keys" {
  name        = "prod/api/keys"
  description = "API keys for external services"

  tags = {
    Environment = "production"
    Application = "myapp"
  }
}

resource "aws_secretsmanager_secret_version" "api_keys" {
  secret_id = aws_secretsmanager_secret.api_keys.id
  secret_string = jsonencode({
    stripe_api_key     = var.stripe_api_key
    sendgrid_api_key   = var.sendgrid_api_key
    twilio_auth_token  = var.twilio_auth_token
    oauth_client_secret = var.oauth_client_secret
  })
}

# Rotación automática
resource "aws_secretsmanager_secret_rotation" "database_rotation" {
  secret_id           = aws_secretsmanager_secret.database_credentials.id
  rotation_lambda_arn = aws_lambda_function.rotate_secret.arn

  rotation_rules {
    automatically_after_days = 30
  }
}

# Lambda para rotación de secretos
resource "aws_lambda_function" "rotate_secret" {
  filename         = "rotate_secret.zip"
  function_name    = "rotate-database-secret"
  role            = aws_iam_role.lambda_rotation_role.arn
  handler         = "lambda_function.lambda_handler"
  runtime         = "python3.9"
  timeout         = 60

  environment {
    variables = {
      SECRETS_MANAGER_ENDPOINT = "https://secretsmanager.${data.aws_region.current.name}.amazonaws.com"
    }
  }
}

# IAM Role para Lambda de rotación
resource "aws_iam_role" "lambda_rotation_role" {
  name = "lambda-rotation-role"

  assume_role_policy = jsonencode({
    Version = "2012-10-17"
    Statement = [
      {
        Action = "sts:AssumeRole"
        Effect = "Allow"
        Principal = {
          Service = "lambda.amazonaws.com"
        }
      }
    ]
  })
}

resource "aws_iam_role_policy" "lambda_rotation_policy" {
  name = "lambda-rotation-policy"
  role = aws_iam_role.lambda_rotation_role.id

  policy = jsonencode({
    Version = "2012-10-17"
    Statement = [
      {
        Effect = "Allow"
        Action = [
          "secretsmanager:DescribeSecret",
          "secretsmanager:GetSecretValue",
          "secretsmanager:PutSecretValue",
          "secretsmanager:UpdateSecretVersionStage"
        ]
        Resource = aws_secretsmanager_secret.database_credentials.arn
      },
      {
        Effect = "Allow"
        Action = [
          "rds:ModifyDBInstance",
          "rds:DescribeDBInstances"
        ]
        Resource = "*"
      },
      {
        Effect = "Allow"
        Action = [
          "logs:CreateLogGroup",
          "logs:CreateLogStream",
          "logs:PutLogEvents"
        ]
        Resource = "arn:aws:logs:*:*:*"
      }
    ]
  })
}

# Política para acceso de aplicaciones
data "aws_iam_policy_document" "app_secrets_policy" {
  statement {
    effect = "Allow"
    actions = [
      "secretsmanager:GetSecretValue"
    ]
    resources = [
      aws_secretsmanager_secret.database_credentials.arn,
      aws_secretsmanager_secret.api_keys.arn
    ]
    condition {
      test     = "StringEquals"
      variable = "secretsmanager:ResourceTag/Application"
      values   = ["myapp"]
    }
  }
}

resource "aws_iam_policy" "app_secrets_policy" {
  name        = "app-secrets-policy"
  description = "Policy for application to access secrets"
  policy      = data.aws_iam_policy_document.app_secrets_policy.json
}

# Role para aplicación EC2
resource "aws_iam_role" "app_role" {
  name = "app-role"

  assume_role_policy = jsonencode({
    Version = "2012-10-17"
    Statement = [
      {
        Action = "sts:AssumeRole"
        Effect = "Allow"
        Principal = {
          Service = "ec2.amazonaws.com"
        }
      }
    ]
  })
}

resource "aws_iam_role_policy_attachment" "app_secrets_attachment" {
  role       = aws_iam_role.app_role.name
  policy_arn = aws_iam_policy.app_secrets_policy.arn
}

resource "aws_iam_instance_profile" "app_profile" {
  name = "app-profile"
  role = aws_iam_role.app_role.name
}
```

### 2. Lambda Function para Rotación

```python
# lambda/rotate_secret.py
import json
import boto3
import logging
import os
from typing import Dict, Any

logger = logging.getLogger()
logger.setLevel(logging.INFO)

def lambda_handler(event: Dict[str, Any], context: Any) -> Dict[str, Any]:
    """
    Lambda function para rotar secretos de base de datos en AWS Secrets Manager
    """
    try:
        secret_arn = event['Step1']['SecretArn']
        token = event['Step1']['ClientRequestToken']
        step = event['Step1']['Step']
        
        logger.info(f"Processing step: {step} for secret: {secret_arn}")
        
        # Crear clientes de AWS
        secrets_client = boto3.client('secretsmanager')
        rds_client = boto3.client('rds')
        
        # Procesar según el paso
        if step == "createSecret":
            create_secret(secrets_client, secret_arn, token)
        elif step == "setSecret":
            set_secret(secrets_client, rds_client, secret_arn, token)
        elif step == "testSecret":
            test_secret(secrets_client, secret_arn, token)
        elif step == "finishSecret":
            finish_secret(secrets_client, secret_arn, token)
        else:
            raise ValueError(f"Invalid step: {step}")
        
        return {"statusCode": 200, "body": f"Successfully processed step: {step}"}
        
    except Exception as e:
        logger.error(f"Error in rotation: {str(e)}")
        raise

def create_secret(secrets_client: Any, secret_arn: str, token: str) -> None:
    """
    Crear nueva versión del secreto con nueva contraseña
    """
    logger.info("Creating new secret version")
    
    # Obtener secreto actual
    current_secret = secrets_client.get_secret_value(SecretArn=secret_arn, VersionStage="AWSCURRENT")
    current_credentials = json.loads(current_secret['SecretString'])
    
    # Generar nueva contraseña
    new_password = generate_password()
    
    # Crear nuevas credenciales
    new_credentials = current_credentials.copy()
    new_credentials['password'] = new_password
    
    # Almacenar nueva versión
    secrets_client.put_secret_value(
        SecretArn=secret_arn,
        VersionId=token,
        SecretString=json.dumps(new_credentials),
        VersionStages=['AWSPENDING']
    )
    
    logger.info("New secret version created successfully")

def set_secret(secrets_client: Any, rds_client: Any, secret_arn: str, token: str) -> None:
    """
    Actualizar la contraseña en la base de datos
    """
    logger.info("Setting new password in database")
    
    # Obtener credenciales pendientes
    pending_secret = secrets_client.get_secret_value(
        SecretArn=secret_arn, 
        VersionId=token, 
        VersionStage="AWSPENDING"
    )
    pending_credentials = json.loads(pending_secret['SecretString'])
    
    # Obtener credenciales actuales para conexión
    current_secret = secrets_client.get_secret_value(SecretArn=secret_arn, VersionStage="AWSCURRENT")
    current_credentials = json.loads(current_secret['SecretString'])
    
    # Actualizar contraseña en RDS
    try:
        rds_client.modify_db_instance(
            DBInstanceIdentifier=get_db_instance_id(current_credentials['host']),
            MasterUserPassword=pending_credentials['password'],
            ApplyImmediately=True
        )
        logger.info("Database password updated successfully")
        
    except Exception as e:
        logger.error(f"Failed to update database password: {str(e)}")
        raise

def test_secret(secrets_client: Any, secret_arn: str, token: str) -> None:
    """
    Probar la nueva contraseña conectándose a la base de datos
    """
    logger.info("Testing new secret")
    
    # Obtener credenciales pendientes
    pending_secret = secrets_client.get_secret_value(
        SecretArn=secret_arn, 
        VersionId=token, 
        VersionStage="AWSPENDING"
    )
    pending_credentials = json.loads(pending_secret['SecretString'])
    
    # Probar conexión
    if not test_database_connection(pending_credentials):
        raise Exception("Failed to connect with new credentials")
    
    logger.info("New secret tested successfully")

def finish_secret(secrets_client: Any, secret_arn: str, token: str) -> None:
    """
    Finalizar rotación moviendo versiones
    """
    logger.info("Finishing secret rotation")
    
    # Mover AWSPENDING a AWSCURRENT
    secrets_client.update_secret_version_stage(
        SecretArn=secret_arn,
        VersionStage="AWSCURRENT",
        MoveToVersionId=token,
        RemoveFromVersionId=get_current_version_id(secrets_client, secret_arn)
    )
    
    logger.info("Secret rotation completed successfully")

def generate_password(length: int = 32) -> str:
    """
    Generar contraseña segura
    """
    import secrets
    import string
    
    alphabet = string.ascii_letters + string.digits + "!@#$%^&*"
    password = ''.join(secrets.choice(alphabet) for _ in range(length))
    return password

def get_db_instance_id(endpoint: str) -> str:
    """
    Extraer ID de instancia RDS del endpoint
    """
    return endpoint.split('.')[0]

def get_current_version_id(secrets_client: Any, secret_arn: str) -> str:
    """
    Obtener ID de versión actual
    """
    response = secrets_client.describe_secret(SecretArn=secret_arn)
    for version_id, stages in response['VersionIdsToStages'].items():
        if 'AWSCURRENT' in stages:
            return version_id
    raise Exception("No current version found")

def test_database_connection(credentials: Dict[str, str]) -> bool:
    """
    Probar conexión a base de datos
    """
    try:
        import pymysql
        
        connection = pymysql.connect(
            host=credentials['host'],
            user=credentials['username'],
            password=credentials['password'],
            database=credentials['dbname'],
            port=int(credentials['port']),
            connect_timeout=5
        )
        
        with connection.cursor() as cursor:
            cursor.execute("SELECT 1")
            result = cursor.fetchone()
        
        connection.close()
        return result is not None
        
    except Exception as e:
        logger.error(f"Database connection test failed: {str(e)}")
        return False
```

### 3. Integración con .NET

```csharp
// AWS/SecretsManagerService.cs
using Amazon.SecretsManager;
using Amazon.SecretsManager.Model;
using System.Text.Json;

public class SecretsManagerConfiguration
{
    public string Region { get; set; } = "us-east-1";
    public int CacheTimeout { get; set; } = 300; // 5 minutos
}

public interface ISecretsManagerService
{
    Task<T> GetSecretAsync<T>(string secretName) where T : class;
    Task<string> GetSecretStringAsync(string secretName);
    Task UpdateSecretAsync(string secretName, object value);
    Task<DatabaseCredentials> GetDatabaseCredentialsAsync(string secretName);
}

public class SecretsManagerService : ISecretsManagerService
{
    private readonly IAmazonSecretsManager _secretsManager;
    private readonly IMemoryCache _cache;
    private readonly SecretsManagerConfiguration _config;
    private readonly ILogger<SecretsManagerService> _logger;

    public SecretsManagerService(
        IAmazonSecretsManager secretsManager,
        IMemoryCache cache,
        IOptions<SecretsManagerConfiguration> config,
        ILogger<SecretsManagerService> logger)
    {
        _secretsManager = secretsManager;
        _cache = cache;
        _config = config.Value;
        _logger = logger;
    }

    public async Task<T> GetSecretAsync<T>(string secretName) where T : class
    {
        var cacheKey = $"secret:{secretName}";
        
        if (_cache.TryGetValue(cacheKey, out T? cachedValue) && cachedValue != null)
        {
            _logger.LogDebug("Retrieved secret {SecretName} from cache", secretName);
            return cachedValue;
        }

        try
        {
            _logger.LogInformation("Retrieving secret {SecretName} from AWS Secrets Manager", secretName);
            
            var request = new GetSecretValueRequest
            {
                SecretId = secretName,
                VersionStage = "AWSCURRENT"
            };

            var response = await _secretsManager.GetSecretValueAsync(request);
            
            var secretValue = JsonSerializer.Deserialize<T>(response.SecretString);
            
            if (secretValue != null)
            {
                // Cache por tiempo configurado
                var cacheOptions = new MemoryCacheEntryOptions
                {
                    AbsoluteExpirationRelativeToNow = TimeSpan.FromSeconds(_config.CacheTimeout)
                };
                _cache.Set(cacheKey, secretValue, cacheOptions);
            }

            return secretValue ?? throw new InvalidOperationException($"Failed to deserialize secret {secretName}");
        }
        catch (Exception ex)
        {
            _logger.LogError(ex, "Error retrieving secret {SecretName}", secretName);
            throw;
        }
    }

    public async Task<string> GetSecretStringAsync(string secretName)
    {
        try
        {
            var request = new GetSecretValueRequest
            {
                SecretId = secretName,
                VersionStage = "AWSCURRENT"
            };

            var response = await _secretsManager.GetSecretValueAsync(request);
            return response.SecretString;
        }
        catch (Exception ex)
        {
            _logger.LogError(ex, "Error retrieving secret string {SecretName}", secretName);
            throw;
        }
    }

    public async Task UpdateSecretAsync(string secretName, object value)
    {
        try
        {
            var secretString = JsonSerializer.Serialize(value);
            
            var request = new PutSecretValueRequest
            {
                SecretId = secretName,
                SecretString = secretString
            };

            await _secretsManager.PutSecretValueAsync(request);
            
            // Invalidar cache
            _cache.Remove($"secret:{secretName}");
            
            _logger.LogInformation("Updated secret {SecretName}", secretName);
        }
        catch (Exception ex)
        {
            _logger.LogError(ex, "Error updating secret {SecretName}", secretName);
            throw;
        }
    }

    public async Task<DatabaseCredentials> GetDatabaseCredentialsAsync(string secretName)
    {
        var credentials = await GetSecretAsync<DatabaseCredentials>(secretName);
        return credentials;
    }
}

public class DatabaseCredentials
{
    public string Username { get; set; } = string.Empty;
    public string Password { get; set; } = string.Empty;
    public string Engine { get; set; } = string.Empty;
    public string Host { get; set; } = string.Empty;
    public int Port { get; set; }
    public string DbName { get; set; } = string.Empty;
    
    public string GetConnectionString()
    {
        return Engine.ToLower() switch
        {
            "mysql" => $"Server={Host};Port={Port};Database={DbName};Uid={Username};Pwd={Password};",
            "postgres" => $"Host={Host};Port={Port};Database={DbName};Username={Username};Password={Password};",
            _ => throw new NotSupportedException($"Database engine {Engine} not supported")
        };
    }
}

// Startup configuration
public class Program
{
    public static void Main(string[] args)
    {
        var builder = WebApplication.CreateBuilder(args);

        // AWS Services
        builder.Services.AddAWSService<IAmazonSecretsManager>();
        
        // Secret Management
        builder.Services.Configure<SecretsManagerConfiguration>(
            builder.Configuration.GetSection("SecretsManager"));
        
        builder.Services.AddSingleton<ISecretsManagerService, SecretsManagerService>();
        builder.Services.AddMemoryCache();

        // Database context con credenciales dinámicas
        builder.Services.AddDbContext<ApplicationDbContext>((serviceProvider, options) =>
        {
            var secretsService = serviceProvider.GetRequiredService<ISecretsManagerService>();
            
            // Obtener credenciales de forma asíncrona
            var credentials = secretsService.GetDatabaseCredentialsAsync("prod/database/credentials").Result;
            var connectionString = credentials.GetConnectionString();
            
            options.UseMySql(connectionString, ServerVersion.AutoDetect(connectionString));
        });

        var app = builder.Build();
        
        app.Run();
    }
}
```

---

## 🔄 CI/CD Integration

### 1. GitHub Actions con Vault

```yaml
# .github/workflows/vault-integration.yml
name: CI/CD with Vault Integration

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

env:
  VAULT_ADDR: ${{ secrets.VAULT_ADDR }}
  VAULT_NAMESPACE: ${{ secrets.VAULT_NAMESPACE }}

jobs:
  # Job para obtener secretos de Vault
  vault-secrets:
    runs-on: ubuntu-latest
    outputs:
      database-url: ${{ steps.secrets.outputs.database-url }}
      api-key: ${{ steps.secrets.outputs.api-key }}
    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Import secrets from Vault
        uses: hashicorp/vault-action@v2
        id: secrets
        with:
          url: ${{ env.VAULT_ADDR }}
          method: approle
          roleId: ${{ secrets.VAULT_ROLE_ID }}
          secretId: ${{ secrets.VAULT_SECRET_ID }}
          secrets: |
            secret/data/prod/database url | database-url ;
            secret/data/prod/api stripe_key | api-key

      - name: Verify secrets retrieved
        run: |
          echo "Database URL length: ${#DATABASE_URL}"
          echo "API Key length: ${#API_KEY}"
        env:
          DATABASE_URL: ${{ steps.secrets.outputs.database-url }}
          API_KEY: ${{ steps.secrets.outputs.api-key }}

  # Test con secretos
  test:
    needs: vault-secrets
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Setup .NET
        uses: actions/setup-dotnet@v3
        with:
          dotnet-version: '8.0.x'

      - name: Run tests
        run: dotnet test
        env:
          DATABASE_URL: ${{ needs.vault-secrets.outputs.database-url }}
          API_KEY: ${{ needs.vault-secrets.outputs.api-key }}

  # Deploy con dynamic secrets
  deploy:
    needs: [vault-secrets, test]
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main'
    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Get dynamic database credentials
        uses: hashicorp/vault-action@v2
        id: db-creds
        with:
          url: ${{ env.VAULT_ADDR }}
          method: approle
          roleId: ${{ secrets.VAULT_ROLE_ID }}
          secretId: ${{ secrets.VAULT_SECRET_ID }}
          secrets: |
            database/creds/prod-db-role username | db-username ;
            database/creds/prod-db-role password | db-password

      - name: Deploy application
        run: |
          echo "Deploying with dynamic credentials..."
          # Deploy logic here
        env:
          DB_USERNAME: ${{ steps.db-creds.outputs.db-username }}
          DB_PASSWORD: ${{ steps.db-creds.outputs.db-password }}

      - name: Revoke dynamic credentials
        if: always()
        run: |
          curl -X POST "${{ env.VAULT_ADDR }}/v1/sys/leases/revoke" \
               -H "X-Vault-Token: ${{ steps.db-creds.outputs.vault-token }}" \
               -d '{"lease_id": "${{ steps.db-creds.outputs.lease-id }}"}'
```

### 2. ArgoCD con External Secrets Operator

```yaml
# argocd/external-secrets-operator.yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: external-secrets-operator
  namespace: argocd
spec:
  project: default
  source:
    repoURL: https://charts.external-secrets.io
    chart: external-secrets
    targetRevision: 0.9.9
    helm:
      values: |
        installCRDs: true
        controllerClass: external-secrets.io/external-secrets
  destination:
    server: https://kubernetes.default.svc
    namespace: external-secrets-system
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
    syncOptions:
    - CreateNamespace=true

---
# Vault SecretStore
apiVersion: external-secrets.io/v1beta1
kind: SecretStore
metadata:
  name: vault-secret-store
  namespace: production
spec:
  provider:
    vault:
      server: "https://vault.company.com"
      path: "secret"
      version: "v2"
      auth:
        kubernetes:
          mountPath: "kubernetes"
          role: "external-secrets"
          serviceAccountRef:
            name: external-secrets-sa

---
# AWS Secrets Manager SecretStore
apiVersion: external-secrets.io/v1beta1
kind: SecretStore
metadata:
  name: aws-secrets-store
  namespace: production
spec:
  provider:
    aws:
      service: SecretsManager
      region: us-east-1
      auth:
        serviceAccount:
          name: external-secrets-sa

---
# ExternalSecret para base de datos
apiVersion: external-secrets.io/v1beta1
kind: ExternalSecret
metadata:
  name: database-secret
  namespace: production
spec:
  refreshInterval: 300s # 5 minutes
  secretStoreRef:
    name: vault-secret-store
    kind: SecretStore
  target:
    name: database-credentials
    creationPolicy: Owner
    template:
      type: Opaque
      data:
        connection-string: |
          Server={{ .host }};Port={{ .port }};Database={{ .database }};
          User Id={{ .username }};Password={{ .password }};
  data:
  - secretKey: host
    remoteRef:
      key: prod/database
      property: host
  - secretKey: port
    remoteRef:
      key: prod/database
      property: port
  - secretKey: database
    remoteRef:
      key: prod/database
      property: database
  - secretKey: username
    remoteRef:
      key: prod/database
      property: username
  - secretKey: password
    remoteRef:
      key: prod/database
      property: password

---
# Dynamic database credentials
apiVersion: external-secrets.io/v1beta1
kind: ExternalSecret
metadata:
  name: dynamic-db-secret
  namespace: production
spec:
  refreshInterval: 1800s # 30 minutes
  secretStoreRef:
    name: vault-secret-store
    kind: SecretStore
  target:
    name: dynamic-db-credentials
    creationPolicy: Owner
  data:
  - secretKey: username
    remoteRef:
      key: database/creds/prod-db-role
      property: username
  - secretKey: password
    remoteRef:
      key: database/creds/prod-db-role
      property: password
```

---

## 📊 Monitoreo y Auditoría

### 1. Vault Telemetry

```yaml
# monitoring/vault-monitoring.yml
apiVersion: v1
kind: ConfigMap
metadata:
  name: vault-exporter-config
data:
  config.yml: |
    vault:
      address: http://vault:8200
      token_path: /var/secrets/vault-token
    metrics:
      enabled: true
      path: /metrics
    log_level: info

---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: vault-exporter
spec:
  replicas: 1
  selector:
    matchLabels:
      app: vault-exporter
  template:
    metadata:
      labels:
        app: vault-exporter
    spec:
      containers:
      - name: vault-exporter
        image: grapeshot/vault_exporter:latest
        ports:
        - containerPort: 9410
        env:
        - name: VAULT_ADDR
          value: "http://vault:8200"
        - name: VAULT_TOKEN
          valueFrom:
            secretKeyRef:
              name: vault-token
              key: token
        resources:
          requests:
            memory: "64Mi"
            cpu: "50m"
          limits:
            memory: "128Mi"
            cpu: "100m"

---
apiVersion: v1
kind: Service
metadata:
  name: vault-exporter
  labels:
    app: vault-exporter
spec:
  ports:
  - port: 9410
    targetPort: 9410
    name: metrics
  selector:
    app: vault-exporter

---
# ServiceMonitor para Prometheus
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  name: vault-exporter
spec:
  selector:
    matchLabels:
      app: vault-exporter
  endpoints:
  - port: metrics
    interval: 30s
    path: /metrics
```

### 2. Alertas de Seguridad

```yaml
# monitoring/vault-alerts.yml
groups:
- name: vault.rules
  rules:
  # Vault down
  - alert: VaultDown
    expr: up{job="vault"} == 0
    for: 5m
    labels:
      severity: critical
      service: vault
    annotations:
      summary: "Vault server is down"
      description: "Vault server has been down for more than 5 minutes"

  # Vault sealed
  - alert: VaultSealed
    expr: vault_core_sealed == 1
    for: 1m
    labels:
      severity: critical
      service: vault
    annotations:
      summary: "Vault is sealed"
      description: "Vault is currently sealed and cannot serve requests"

  # High number of failed authentication attempts
  - alert: VaultHighAuthFailures
    expr: rate(vault_audit_log_request_failure_total[5m]) > 0.1
    for: 5m
    labels:
      severity: warning
      service: vault
    annotations:
      summary: "High number of Vault authentication failures"
      description: "Vault is experiencing {{ $value }} authentication failures per second"

  # Secret accessed by unusual client
  - alert: VaultUnusualSecretAccess
    expr: increase(vault_audit_log_request_total{path=~"secret/.*"}[1h]) > 100
    for: 5m
    labels:
      severity: warning
      service: vault
    annotations:
      summary: "Unusual secret access pattern detected"
      description: "More than 100 secret requests in the last hour from path {{ $labels.path }}"

  # Token about to expire
  - alert: VaultTokenExpiringSoon
    expr: vault_token_ttl < 3600 # 1 hour
    for: 5m
    labels:
      severity: warning
      service: vault
    annotations:
      summary: "Vault token expiring soon"
      description: "Vault token will expire in less than 1 hour"

  # AWS Secrets Manager alerts
  - alert: SecretsManagerRotationFailed
    expr: aws_secretsmanager_rotation_failed_total > 0
    for: 1m
    labels:
      severity: critical
      service: secrets-manager
    annotations:
      summary: "AWS Secrets Manager rotation failed"
      description: "Secret rotation failed for {{ $labels.secret_name }}"
```

---

## 📋 Ejercicios Prácticos

### Ejercicio 1: Configuración Básica de Vault

**Objetivo**: Instalar y configurar Vault con múltiples secret engines.

**Tareas**:

1. Desplegar Vault con Docker Compose
2. Configurar KV, Database y Transit engines
3. Crear políticas de acceso
4. Configurar usuarios y AppRoles

### Ejercicio 2: Integración con AWS Secrets Manager

**Objetivo**: Implementar rotación automática de secretos.

**Componentes**:

- Terraform para provisionar secretos
- Lambda para rotación personalizada
- Integración con RDS
- Monitoreo de rotaciones

### Ejercicio 3: CI/CD con Secret Management

**Objetivo**: Integrar secrets en pipeline de deployment.

**Tareas**:

- GitHub Actions con Vault integration
- External Secrets Operator en Kubernetes
- Dynamic secrets para deployments
- Auditoría de acceso a secretos

---

## 🧪 Laboratorio

### Lab 1: Vault Setup Completo

1. **Desplegar infraestructura**:

   ```bash
   docker-compose -f docker-compose.vault.yml up -d
   ./scripts/vault-setup.sh init
   ```

2. **Configurar aplicación**:
   - Integrar VaultService en .NET
   - Configurar dynamic database credentials
   - Implementar encryption as a service

3. **Testing y validación**:
   - Probar rotación de secretos
   - Validar políticas de acceso
   - Monitorear métricas

### Lab 2: AWS Secrets Manager

1. **Provisionar con Terraform**
2. **Configurar rotación automática**
3. **Integrar con aplicaciones**
4. **Configurar alertas y monitoreo**

---

## 📖 Best Practices

### 1. Principios de Secret Management

```yaml
principles:
  least_privilege:
    - "Mínimo acceso necesario por rol"
    - "Políticas granulares por aplicación"
    - "Revisión periódica de permisos"
  
  rotation:
    - "Rotación automática de secretos críticos"
    - "Frecuencia basada en sensibilidad"
    - "Testing automatizado post-rotación"
  
  auditing:
    - "Log completo de acceso a secretos"
    - "Alertas en patrones anómalos"
    - "Retención adecuada de logs"
```

### 2. Gestión del Ciclo de Vida

```yaml
lifecycle_management:
  creation:
    - "Secrets generados automáticamente"
    - "Complejidad adecuada"
    - "Almacenamiento encriptado"
  
  distribution:
    - "Just-in-time access"
    - "Canales seguros únicamente"
    - "Tiempo de vida limitado"
  
  retirement:
    - "Revocación inmediata"
    - "Limpieza de caches"
    - "Validación de no uso"
```

---

## ✅ Resumen del Módulo

En este módulo has aprendido:

1. **HashiCorp Vault** - Instalación, configuración y secret engines
2. **AWS Secrets Manager** - Rotación automática y integración
3. **Dynamic Secrets** - Credenciales temporales para bases de datos
4. **Encryption as a Service** - Protección de datos con Transit engine
5. **CI/CD Integration** - Secrets en pipelines de deployment
6. **Kubernetes Integration** - External Secrets Operator
7. **Monitoreo y Auditoría** - Métricas y alertas de seguridad
8. **Best Practices** - Principios de gestión segura de secretos

### Próximas etapas

- **Módulo 08**: Auditorías y cumplimiento normativo
- **Módulo 09**: Vulnerability scanning avanzado
- **Módulo 10**: Distributed tracing con Jaeger

¡Continúa con el siguiente módulo para dominar auditorías y compliance!
