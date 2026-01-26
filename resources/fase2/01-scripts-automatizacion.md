# 01. Scripts de Automatización

**Automatiza tareas repetitivas y mejora tu productividad!**Este módulo te enseñará a crear scripts poderosos para automatizar procesos comunes en DevOps.

## Objetivos de Aprendizaje

Al completar este módulo serás capaz de:

- **Crear scripts bash/shell**para automatización en Unix/Linux
- **Desarrollar scripts PowerShell**para entornos Windows
- **Utilizar Python**para automatización multiplataforma
- **Gestionar parámetros**y configuraciones en scripts
- **Implementar logging**y manejo de errores
- **Crear scripts reutilizables**y mantenibles

## Contenido del Módulo

### 1. Scripts Bash/Shell para Unix/Linux

#### **Fundamentos de Bash**

```bash
#!/bin/bash

# Script de ejemplo: Automatización de backup
# Autor: DevOps Learning Path
# Fecha: $(date)

# Variables de configuración
BACKUP_DIR="/backup"
SOURCE_DIR="/var/www/html"
DATE=$(date +%Y%m%d_%H%M%S)
LOG_FILE="/var/log/backup.log"

# Función para logging
log_message() {
    echo "[$(date '+%Y-%m-%d %H:%M:%S')] $1" | tee -a "$LOG_FILE"
}

# Función principal de backup
perform_backup() {
    local backup_name="website_backup_${DATE}.tar.gz"

    log_message "Iniciando backup de $SOURCE_DIR"

    # Crear directorio de backup si no existe
    mkdir -p "$BACKUP_DIR"

    # Realizar backup con compresión
    if tar -czf "${BACKUP_DIR}/${backup_name}" -C "$SOURCE_DIR" .; then
        log_message "Backup completado exitosamente: $backup_name"
        echo " Backup creado: ${BACKUP_DIR}/${backup_name}"
    else
        log_message "ERROR: Falló el backup"
        echo " Error en el backup"
        exit 1
    fi
}

# Función para limpiar backups antiguos
cleanup_old_backups() {
    log_message "Limpiando backups antiguos (>7 días)"
    find "$BACKUP_DIR" -name "website_backup_*.tar.gz" -mtime +7 -delete
    log_message "Limpieza completada"
}

# Función principal
main() {
    echo " Iniciando script de backup automatizado"

    # Verificar que el directorio fuente existe
    if [ ! -d "$SOURCE_DIR" ]; then
        echo " Error: Directorio fuente no existe: $SOURCE_DIR"
        exit 1
    fi

    perform_backup
    cleanup_old_backups

    echo " Script completado exitosamente"
}

# Ejecutar función principal
main "$@"
```

#### **Script Avanzado: Deployment Automatizado**

```bash
#!/bin/bash

# Script de deployment automatizado
# Versión: 2.0

set -euo pipefail  # Fail fast en errores

# Configuración
APP_NAME="mi-aplicacion"
DEPLOY_DIR="/opt/${APP_NAME}"
BACKUP_DIR="/opt/backups"
GIT_REPO="https://github.com/usuario/mi-aplicacion.git"
SERVICE_NAME="${APP_NAME}.service"

# Colores para output
RED='\033[0;31m'
GREEN='\033[0;32m'
YELLOW='\033[1;33m'
NC='\033[0m' # No Color

# Función para imprimir mensajes coloreados
print_status() {
    echo -e "${GREEN}[INFO]${NC} $1"
}

print_warning() {
    echo -e "${YELLOW}[WARN]${NC} $1"
}

print_error() {
    echo -e "${RED}[ERROR]${NC} $1"
}

# Función para verificar prerrequisitos
check_prerequisites() {
    print_status "Verificando prerrequisitos..."

    # Verificar que somos root o tenemos sudo
    if [[ $EUID -ne 0 ]]; then
        print_error "Este script debe ejecutarse como root o con sudo"
        exit 1
    fi

    # Verificar que git está instalado
    if ! command -v git &> /dev/null; then
        print_error "Git no está instalado"
        exit 1
    fi

    # Verificar que systemctl está disponible
    if ! command -v systemctl &> /dev/null; then
        print_error "systemctl no está disponible"
        exit 1
    fi

    print_status " Prerrequisitos verificados"
}

# Función para crear backup antes del deployment
create_backup() {
    if [ -d "$DEPLOY_DIR" ]; then
        print_status "Creando backup de la versión actual..."
        local backup_name="${APP_NAME}_backup_$(date +%Y%m%d_%H%M%S)"
        mkdir -p "$BACKUP_DIR"
        cp -r "$DEPLOY_DIR" "${BACKUP_DIR}/${backup_name}"
        print_status " Backup creado: ${BACKUP_DIR}/${backup_name}"
    fi
}

# Función para parar el servicio
stop_service() {
    print_status "Parando servicio $SERVICE_NAME..."
    if systemctl is-active --quiet "$SERVICE_NAME"; then
        systemctl stop "$SERVICE_NAME"
        print_status " Servicio parado"
    else
        print_warning "El servicio no estaba ejecutándose"
    fi
}

# Función para descargar nueva versión
download_new_version() {
    print_status "Descargando nueva versión desde $GIT_REPO..."

    # Crear directorio temporal
    local temp_dir=$(mktemp -d)

    # Clonar repositorio
    git clone "$GIT_REPO" "$temp_dir"

    # Remover directorio anterior si existe
    if [ -d "$DEPLOY_DIR" ]; then
        rm -rf "$DEPLOY_DIR"
    fi

    # Mover nueva versión al directorio de deployment
    mv "$temp_dir" "$DEPLOY_DIR"

    print_status " Nueva versión descargada"
}

# Función para instalar dependencias
install_dependencies() {
    print_status "Instalando dependencias..."

    cd "$DEPLOY_DIR"

    # Si es una aplicación Node.js
    if [ -f "package.json" ]; then
        npm install --production
    fi

    # Si es una aplicación Python
    if [ -f "requirements.txt" ]; then
        pip3 install -r requirements.txt
    fi

    print_status " Dependencias instaladas"
}

# Función para configurar permisos
set_permissions() {
    print_status "Configurando permisos..."
    chown -R www-data:www-data "$DEPLOY_DIR"
    chmod -R 755 "$DEPLOY_DIR"
    print_status " Permisos configurados"
}

# Función para iniciar el servicio
start_service() {
    print_status "Iniciando servicio $SERVICE_NAME..."
    systemctl start "$SERVICE_NAME"

    # Verificar que el servicio se inició correctamente
    sleep 5
    if systemctl is-active --quiet "$SERVICE_NAME"; then
        print_status " Servicio iniciado exitosamente"
    else
        print_error " Falló al iniciar el servicio"
        rollback
        exit 1
    fi
}

# Función para verificar que la aplicación responde
health_check() {
    print_status "Verificando health check..."
    local max_attempts=10
    local attempt=1

    while [ $attempt -le $max_attempts ]; do
        if curl -f http://localhost:8080/health &> /dev/null; then
            print_status " Aplicación respondiendo correctamente"
            return 0
        fi

        print_warning "Intento $attempt/$max_attempts fallido, reintentando en 5 segundos..."
        sleep 5
        ((attempt++))
    done

    print_error " Health check falló después de $max_attempts intentos"
    rollback
    exit 1
}

# Función de rollback
rollback() {
    print_warning "Iniciando rollback..."

    # Encontrar el backup más reciente
    local latest_backup=$(ls -t "${BACKUP_DIR}/${APP_NAME}_backup_"* 2>/dev/null | head -n1)

    if [ -n "$latest_backup" ]; then
        print_status "Restaurando desde: $latest_backup"

        # Parar servicio
        systemctl stop "$SERVICE_NAME"

        # Restaurar backup
        rm -rf "$DEPLOY_DIR"
        cp -r "$latest_backup" "$DEPLOY_DIR"

        # Reiniciar servicio
        systemctl start "$SERVICE_NAME"

        print_status " Rollback completado"
    else
        print_error "No se encontró backup para rollback"
    fi
}

# Función principal
main() {
    echo " Iniciando deployment automatizado de $APP_NAME"
    echo "======"

    check_prerequisites
    create_backup
    stop_service
    download_new_version
    install_dependencies
    set_permissions
    start_service
    health_check

    echo "======"
    echo " Deployment completado exitosamente"
    echo " $APP_NAME está ahora ejecutándose con la nueva versión"
}

# Manejar señales para cleanup
trap 'print_error "Script interrumpido"; exit 1' INT TERM

# Ejecutar si es llamado directamente
if [[ "${BASH_SOURCE[0]}" == "${0}" ]]; then
    main "$@"
fi
```

### 2. Scripts PowerShell para Windows

#### **Script PowerShell Básico**

```powershell
# Script de automatización para Windows
# Backup automatizado con PowerShell

param(
    [Parameter(Mandatory=$true)]
    [string]$SourcePath,

    [Parameter(Mandatory=$false)]
    [string]$BackupPath = "C:\Backups",

    [Parameter(Mandatory=$false)]
    [int]$RetentionDays = 7,

    [switch]$Compress
)

# Configuración
$LogPath = "$env:TEMP\backup.log"
$DateStamp = Get-Date -Format "yyyyMMdd_HHmmss"

# Función para logging
function Write-Log {
    param([string]$Message, [string]$Level = "INFO")

    $timestamp = Get-Date -Format "yyyy-MM-dd HH:mm:ss"
    $logEntry = "[$timestamp] [$Level] $Message"

    Write-Host $logEntry
    Add-Content -Path $LogPath -Value $logEntry
}

# Función para verificar prerrequisitos
function Test-Prerequisites {
    Write-Log "Verificando prerrequisitos..."

    # Verificar que el path fuente existe
    if (-not (Test-Path $SourcePath)) {
        Write-Log "Error: El path fuente no existe: $SourcePath" "ERROR"
        exit 1
    }

    # Crear directorio de backup si no existe
    if (-not (Test-Path $BackupPath)) {
        New-Item -ItemType Directory -Path $BackupPath -Force
        Write-Log "Directorio de backup creado: $BackupPath"
    }

    Write-Log " Prerrequisitos verificados"
}

# Función principal de backup
function Start-Backup {
    Write-Log "Iniciando backup de $SourcePath"

    $backupName = "backup_$DateStamp"
    $destinationPath = Join-Path $BackupPath $backupName

    try {
        if ($Compress) {
            # Crear backup comprimido
            $zipPath = "$destinationPath.zip"
            Compress-Archive -Path $SourcePath -DestinationPath $zipPath -Force
            Write-Log " Backup comprimido creado: $zipPath"
        } else {
            # Crear backup sin compresión
            Copy-Item -Path $SourcePath -Destination $destinationPath -Recurse -Force
            Write-Log " Backup creado: $destinationPath"
        }
    }
    catch {
        Write-Log " Error durante el backup: $($_.Exception.Message)" "ERROR"
        exit 1
    }
}

# Función para limpiar backups antiguos
function Remove-OldBackups {
    Write-Log "Limpiando backups antiguos (>$RetentionDays días)"

    $cutoffDate = (Get-Date).AddDays(-$RetentionDays)

    Get-ChildItem -Path $BackupPath -Filter "backup_*" |
        Where-Object { $_.CreationTime -lt $cutoffDate } |
        ForEach-Object {
            Remove-Item -Path $_.FullName -Recurse -Force
            Write-Log "Eliminado backup antiguo: $($_.Name)"
        }

    Write-Log " Limpieza completada"
}

# Script principal
function Main {
    Write-Host " Iniciando script de backup automatizado" -ForegroundColor Green
    Write-Host "Source: $SourcePath" -ForegroundColor Yellow
    Write-Host "Backup: $BackupPath" -ForegroundColor Yellow
    Write-Host "Compress: $Compress" -ForegroundColor Yellow
    Write-Host "=" * 50

    Test-Prerequisites
    Start-Backup
    Remove-OldBackups

    Write-Host "=" * 50
    Write-Host " Script completado exitosamente" -ForegroundColor Green
}

# Ejecutar script principal
Main
```

#### **Script PowerShell Avanzado: CI/CD Helper**

```powershell
# Script avanzado para CI/CD en Windows
[CmdletBinding()]
param(
    [Parameter(Mandatory=$true)]
    [ValidateSet("build", "test", "deploy", "rollback")]
    [string]$Action,

    [Parameter(Mandatory=$false)]
    [string]$Environment = "staging",

    [Parameter(Mandatory=$false)]
    [string]$Version = "latest",

    [Parameter(Mandatory=$false)]
    [string]$ConfigFile = "deploy.json"
)

# Configuración global
$ErrorActionPreference = "Stop"
$ProgressPreference = "SilentlyContinue"

# Cargar configuración
if (Test-Path $ConfigFile) {
    $Config = Get-Content $ConfigFile | ConvertFrom-Json
} else {
    Write-Error "Archivo de configuración no encontrado: $ConfigFile"
}

# Función para ejecutar comandos con logging
function Invoke-LoggedCommand {
    param(
        [string]$Command,
        [string]$Description
    )

    Write-Host " $Description..." -ForegroundColor Yellow

    try {
        Invoke-Expression $Command
        Write-Host " $Description completado" -ForegroundColor Green
    }
    catch {
        Write-Host " Error en $Description`: $($_.Exception.Message)" -ForegroundColor Red
        throw
    }
}

# Función para build
function Start-Build {
    Write-Host " Iniciando build para $Environment" -ForegroundColor Cyan

    # Restaurar dependencias NuGet
    Invoke-LoggedCommand "dotnet restore" "Restaurando dependencias"

    # Build de la aplicación
    $buildCmd = "dotnet build --configuration Release --no-restore"
    Invoke-LoggedCommand $buildCmd "Building aplicación"

    # Ejecutar tests unitarios
    $testCmd = "dotnet test --configuration Release --no-build --verbosity minimal"
    Invoke-LoggedCommand $testCmd "Ejecutando tests unitarios"

    # Publicar aplicación
    $publishCmd = "dotnet publish --configuration Release --output ./publish"
    Invoke-LoggedCommand $publishCmd "Publicando aplicación"
}

# Función para testing
function Start-Testing {
    Write-Host " Iniciando tests para $Environment" -ForegroundColor Cyan

    # Tests unitarios
    Invoke-LoggedCommand "dotnet test --logger trx --results-directory ./TestResults" "Tests unitarios"

    # Tests de integración (si existen)
    if (Test-Path "tests/Integration") {
        Invoke-LoggedCommand "dotnet test tests/Integration --logger trx" "Tests de integración"
    }

    # Análisis de código con SonarQube (si está configurado)
    if ($Config.SonarQube.Enabled) {
        $sonarCmd = "dotnet sonarscanner begin /k:`"$($Config.SonarQube.ProjectKey)`""
        Invoke-LoggedCommand $sonarCmd "Iniciando análisis SonarQube"

        Invoke-LoggedCommand "dotnet build" "Build para SonarQube"

        Invoke-LoggedCommand "dotnet sonarscanner end" "Finalizando análisis SonarQube"
    }
}

# Función para deployment
function Start-Deployment {
    Write-Host " Iniciando deployment a $Environment" -ForegroundColor Cyan

    $envConfig = $Config.Environments.$Environment

    # Parar servicio si está corriendo
    if ($envConfig.ServiceName) {
        try {
            Stop-Service -Name $envConfig.ServiceName -Force
            Write-Host " Servicio $($envConfig.ServiceName) parado" -ForegroundColor Green
        }
        catch {
            Write-Host " Servicio no estaba corriendo o no existe" -ForegroundColor Yellow
        }
    }

    # Crear backup de la versión actual
    if (Test-Path $envConfig.DeployPath) {
        $backupPath = "$($envConfig.DeployPath)_backup_$(Get-Date -Format 'yyyyMMdd_HHmmss')"
        Copy-Item -Path $envConfig.DeployPath -Destination $backupPath -Recurse
        Write-Host " Backup creado: $backupPath" -ForegroundColor Green
    }

    # Copiar nueva versión
    if (Test-Path "./publish") {
        Remove-Item -Path $envConfig.DeployPath -Recurse -Force -ErrorAction SilentlyContinue
        Copy-Item -Path "./publish" -Destination $envConfig.DeployPath -Recurse
        Write-Host " Nueva versión desplegada" -ForegroundColor Green
    }

    # Actualizar configuración específica del entorno
    $configSource = "./config/$Environment.appsettings.json"
    if (Test-Path $configSource) {
        Copy-Item -Path $configSource -Destination "$($envConfig.DeployPath)/appsettings.json" -Force
        Write-Host " Configuración de $Environment aplicada" -ForegroundColor Green
    }

    # Iniciar servicio
    if ($envConfig.ServiceName) {
        Start-Service -Name $envConfig.ServiceName
        Start-Sleep -Seconds 5

        # Verificar que el servicio está corriendo
        $service = Get-Service -Name $envConfig.ServiceName
        if ($service.Status -eq "Running") {
            Write-Host " Servicio $($envConfig.ServiceName) iniciado correctamente" -ForegroundColor Green
        } else {
            throw " Error: El servicio no se pudo iniciar"
        }
    }

    # Health check
    if ($envConfig.HealthCheckUrl) {
        Start-HealthCheck -Url $envConfig.HealthCheckUrl
    }
}

# Función para health check
function Start-HealthCheck {
    param([string]$Url)

    Write-Host " Verificando health check: $Url" -ForegroundColor Yellow

    $maxAttempts = 10
    $attempt = 1

    do {
        try {
            $response = Invoke-WebRequest -Uri $Url -TimeoutSec 10
            if ($response.StatusCode -eq 200) {
                Write-Host " Health check exitoso" -ForegroundColor Green
                return
            }
        }
        catch {
            Write-Host " Intento $attempt/$maxAttempts fallido, reintentando..." -ForegroundColor Yellow
            Start-Sleep -Seconds 5
        }
        $attempt++
    } while ($attempt -le $maxAttempts)

    throw " Health check falló después de $maxAttempts intentos"
}

# Función para rollback
function Start-Rollback {
    Write-Host " Iniciando rollback en $Environment" -ForegroundColor Magenta

    $envConfig = $Config.Environments.$Environment

    # Encontrar el backup más reciente
    $backupPattern = "$($envConfig.DeployPath)_backup_*"
    $latestBackup = Get-ChildItem -Path (Split-Path $envConfig.DeployPath) -Filter (Split-Path $backupPattern -Leaf) |
                   Sort-Object CreationTime -Descending |
                   Select-Object -First 1

    if ($latestBackup) {
        # Parar servicio
        if ($envConfig.ServiceName) {
            Stop-Service -Name $envConfig.ServiceName -Force
        }

        # Restaurar backup
        Remove-Item -Path $envConfig.DeployPath -Recurse -Force
        Copy-Item -Path $latestBackup.FullName -Destination $envConfig.DeployPath -Recurse

        # Iniciar servicio
        if ($envConfig.ServiceName) {
            Start-Service -Name $envConfig.ServiceName
        }

        Write-Host " Rollback completado desde: $($latestBackup.Name)" -ForegroundColor Green
    } else {
        throw " No se encontró backup para rollback"
    }
}

# Script principal
try {
    Write-Host "=" * 60 -ForegroundColor Cyan
    Write-Host " DevOps Automation Script" -ForegroundColor Cyan
    Write-Host "Action: $Action | Environment: $Environment | Version: $Version" -ForegroundColor Yellow
    Write-Host "=" * 60 -ForegroundColor Cyan

    switch ($Action) {
        "build" { Start-Build }
        "test" { Start-Testing }
        "deploy" { Start-Deployment }
        "rollback" { Start-Rollback }
    }

    Write-Host "=" * 60 -ForegroundColor Green
    Write-Host " $Action completado exitosamente" -ForegroundColor Green
    Write-Host "=" * 60 -ForegroundColor Green
}
catch {
    Write-Host "=" * 60 -ForegroundColor Red
    Write-Host " Error en $Action`: $($_.Exception.Message)" -ForegroundColor Red
    Write-Host "=" * 60 -ForegroundColor Red
    exit 1
}
```

### 3. Python para Automatización Multiplataforma

#### **Script Python Básico**

```python
#!/usr/bin/env python3
"""
Script de automatización básico en Python
Gestión de deployments y configuraciones
"""

import os
import sys
import json
import logging
import argparse
import subprocess
import shutil
from datetime import datetime
from pathlib import Path
from typing import Dict, List, Optional

# Configuración de logging
logging.basicConfig(
    level=logging.INFO,
    format='%(asctime)s - %(levelname)s - %(message)s',
    handlers=[
        logging.FileHandler('automation.log'),
        logging.StreamHandler(sys.stdout)
    ]
)

logger = logging.getLogger(__name__)

class AutomationHelper:
    """Clase principal para automatización DevOps"""

    def __init__(self, config_file: str = "config.json"):
        self.config_file = config_file
        self.config = self.load_config()

    def load_config(self) -> Dict:
        """Cargar configuración desde archivo JSON"""
        try:
            with open(self.config_file, 'r') as f:
                return json.load(f)
        except FileNotFoundError:
            logger.error(f"Archivo de configuración no encontrado: {self.config_file}")
            sys.exit(1)
        except json.JSONDecodeError as e:
            logger.error(f"Error al parsear configuración JSON: {e}")
            sys.exit(1)

    def run_command(self, command: str, check: bool = True) -> subprocess.CompletedProcess:
        """Ejecutar comando shell con logging"""
        logger.info(f"Ejecutando: {command}")

        try:
            result = subprocess.run(
                command,
                shell=True,
                capture_output=True,
                text=True,
                check=check
            )

            if result.stdout:
                logger.info(f"STDOUT: {result.stdout.strip()}")
            if result.stderr:
                logger.warning(f"STDERR: {result.stderr.strip()}")

            return result

        except subprocess.CalledProcessError as e:
            logger.error(f"Comando falló: {e}")
            logger.error(f"STDOUT: {e.stdout}")
            logger.error(f"STDERR: {e.stderr}")
            raise

    def create_backup(self, source_path: str, backup_dir: str) -> str:
        """Crear backup de directorio"""
        timestamp = datetime.now().strftime("%Y%m%d_%H%M%S")
        backup_name = f"backup_{timestamp}"
        backup_path = os.path.join(backup_dir, backup_name)

        logger.info(f"Creando backup: {source_path} -> {backup_path}")

        # Crear directorio de backup si no existe
        Path(backup_dir).mkdir(parents=True, exist_ok=True)

        # Crear backup
        shutil.copytree(source_path, backup_path)

        logger.info(f" Backup creado: {backup_path}")
        return backup_path

    def cleanup_old_backups(self, backup_dir: str, retention_days: int = 7):
        """Limpiar backups antiguos"""
        logger.info(f"Limpiando backups antiguos en {backup_dir}")

        backup_path = Path(backup_dir)
        if not backup_path.exists():
            return

        cutoff_time = datetime.now().timestamp() - (retention_days * 24 * 3600)

        for backup in backup_path.glob("backup_*"):
            if backup.stat().st_mtime < cutoff_time:
                logger.info(f"Eliminando backup antiguo: {backup}")
                shutil.rmtree(backup)

        logger.info(" Limpieza de backups completada")

    def build_application(self) -> bool:
        """Build de la aplicación"""
        logger.info(" Iniciando build de la aplicación")

        build_config = self.config.get('build', {})

        try:
            # Instalar dependencias
            if 'install_command' in build_config:
                self.run_command(build_config['install_command'])

            # Ejecutar build
            if 'build_command' in build_config:
                self.run_command(build_config['build_command'])

            # Ejecutar tests
            if 'test_command' in build_config:
                self.run_command(build_config['test_command'])

            logger.info(" Build completado exitosamente")
            return True

        except subprocess.CalledProcessError:
            logger.error(" Build falló")
            return False

    def deploy_application(self, environment: str) -> bool:
        """Deploy de la aplicación"""
        logger.info(f" Iniciando deployment a {environment}")

        env_config = self.config['environments'].get(environment)
        if not env_config:
            logger.error(f"Configuración no encontrada para entorno: {environment}")
            return False

        try:
            # Crear backup si existe versión anterior
            if os.path.exists(env_config['deploy_path']):
                self.create_backup(
                    env_config['deploy_path'],
                    env_config['backup_path']
                )

            # Parar servicio si está configurado
            if 'service_name' in env_config:
                self.stop_service(env_config['service_name'])

            # Copiar nueva versión
            self.deploy_files(env_config)

            # Aplicar configuración específica del entorno
            self.apply_environment_config(environment, env_config)

            # Iniciar servicio
            if 'service_name' in env_config:
                self.start_service(env_config['service_name'])

            # Health check
            if 'health_check_url' in env_config:
                if not self.health_check(env_config['health_check_url']):
                    logger.error("Health check falló")
                    return False

            logger.info(" Deployment completado exitosamente")
            return True

        except Exception as e:
            logger.error(f" Deployment falló: {e}")
            return False

    def deploy_files(self, env_config: Dict):
        """Copiar archivos de deployment"""
        source_path = env_config['source_path']
        deploy_path = env_config['deploy_path']

        logger.info(f"Copiando archivos: {source_path} -> {deploy_path}")

        # Remover versión anterior
        if os.path.exists(deploy_path):
            shutil.rmtree(deploy_path)

        # Copiar nueva versión
        shutil.copytree(source_path, deploy_path)

        logger.info(" Archivos copiados")

    def apply_environment_config(self, environment: str, env_config: Dict):
        """Aplicar configuración específica del entorno"""
        config_source = f"config/{environment}.json"
        config_target = os.path.join(env_config['deploy_path'], 'config.json')

        if os.path.exists(config_source):
            shutil.copy2(config_source, config_target)
            logger.info(f" Configuración de {environment} aplicada")

    def stop_service(self, service_name: str):
        """Parar servicio del sistema"""
        logger.info(f"Parando servicio: {service_name}")

        if sys.platform.startswith('linux'):
            self.run_command(f"sudo systemctl stop {service_name}", check=False)
        elif sys.platform.startswith('win'):
            self.run_command(f"net stop {service_name}", check=False)

    def start_service(self, service_name: str):
        """Iniciar servicio del sistema"""
        logger.info(f"Iniciando servicio: {service_name}")

        if sys.platform.startswith('linux'):
            self.run_command(f"sudo systemctl start {service_name}")
        elif sys.platform.startswith('win'):
            self.run_command(f"net start {service_name}")

        logger.info(" Servicio iniciado")

    def health_check(self, url: str, max_attempts: int = 10) -> bool:
        """Verificar health check de la aplicación"""
        import requests
        import time

        logger.info(f"Verificando health check: {url}")

        for attempt in range(1, max_attempts + 1):
            try:
                response = requests.get(url, timeout=10)
                if response.status_code == 200:
                    logger.info(" Health check exitoso")
                    return True
            except requests.RequestException:
                pass

            logger.warning(f"Intento {attempt}/{max_attempts} fallido, reintentando...")
            time.sleep(5)

        logger.error(" Health check falló")
        return False

def main():
    """Función principal"""
    parser = argparse.ArgumentParser(description='Script de automatización DevOps')
    parser.add_argument('action', choices=['build', 'deploy', 'backup', 'cleanup'],
                       help='Acción a ejecutar')
    parser.add_argument('--environment', '-e', default='staging',
                       help='Entorno de deployment')
    parser.add_argument('--config', '-c', default='config.json',
                       help='Archivo de configuración')

    args = parser.parse_args()

    # Crear instancia del helper
    automation = AutomationHelper(args.config)

    try:
        if args.action == 'build':
            success = automation.build_application()
        elif args.action == 'deploy':
            success = automation.deploy_application(args.environment)
        elif args.action == 'backup':
            env_config = automation.config['environments'][args.environment]
            automation.create_backup(
                env_config['deploy_path'],
                env_config['backup_path']
            )
            success = True
        elif args.action == 'cleanup':
            env_config = automation.config['environments'][args.environment]
            automation.cleanup_old_backups(env_config['backup_path'])
            success = True

        if success:
            logger.info(" Operación completada exitosamente")
            sys.exit(0)
        else:
            logger.error(" Operación falló")
            sys.exit(1)

    except KeyboardInterrupt:
        logger.warning("Operación cancelada por el usuario")
        sys.exit(1)
    except Exception as e:
        logger.error(f"Error inesperado: {e}")
        sys.exit(1)

if __name__ == "__main__":
    main()
```

## Laboratorios Prácticos

### Lab 1: Script de Backup Básico

**Objetivo**: Crear un script que haga backup automático de directorios.

```bash
# Crear estructura de prueba
mkdir -p /tmp/test-backup/{source,backup}
echo "Archivo de prueba" > /tmp/test-backup/source/test.txt

# Ejecutar script de backup
./backup.sh /tmp/test-backup/source /tmp/test-backup/backup

# Verificar resultado
ls -la /tmp/test-backup/backup/
```

### Lab 2: Automatización de CI/CD

**Objetivo**: Crear pipeline automatizado con scripts.

1. **Setup del proyecto**:

```bash
mkdir -p myapp/{src,tests,scripts}
cd myapp
git init
```

2. **Crear script de CI**:

```bash
#!/bin/bash
# ci.sh - Script de integración continua

echo " Iniciando CI Pipeline"

# Instalar dependencias
npm install

# Ejecutar linting
npm run lint

# Ejecutar tests
npm test

# Build de la aplicación
npm run build

echo " CI Pipeline completado"
```

3. **Automatizar con Git hooks**:

```bash
# Crear pre-commit hook
cat > .git/hooks/pre-commit << 'EOF'
#!/bin/bash
./scripts/ci.sh
EOF
chmod +x .git/hooks/pre-commit
```

### Lab 3: Script de Monitoreo

**Objetivo**: Crear script de monitoreo de servicios.

```python
#!/usr/bin/env python3
import requests
import json
import time
from datetime import datetime

def check_service(url, service_name):
    try:
        response = requests.get(url, timeout=5)
        status = "UP" if response.status_code == 200 else "DOWN"
        response_time = response.elapsed.total_seconds()

        log_entry = {
            "timestamp": datetime.now().isoformat(),
            "service": service_name,
            "status": status,
            "response_time": response_time,
            "status_code": response.status_code
        }

        print(json.dumps(log_entry))
        return status == "UP"

    except Exception as e:
        log_entry = {
            "timestamp": datetime.now().isoformat(),
            "service": service_name,
            "status": "ERROR",
            "error": str(e)
        }
        print(json.dumps(log_entry))
        return False

# Lista de servicios a monitorear
services = [
    {"name": "web-app", "url": "http://localhost:8080/health"},
    {"name": "api", "url": "http://localhost:3000/api/health"},
    {"name": "database", "url": "http://localhost:5432/health"}
]

# Monitoreo continuo
while True:
    for service in services:
        check_service(service["url"], service["name"])

    time.sleep(30)  # Check cada 30 segundos
```

## Ejercicios Prácticos

### Ejercicio 1: Script de Deployment Multi-Entorno

Crear un script que pueda deployar a diferentes entornos (dev, staging, prod) con configuraciones específicas.

**Requisitos**:

- Backup automático antes del deployment
- Configuración por entorno
- Rollback en caso de fallo
- Health check post-deployment

### Ejercicio 2: Automatización de Base de Datos

Crear scripts para:

- Backup de base de datos
- Aplicación de migraciones
- Restauración desde backup
- Monitoreo de performance

### Ejercicio 3: Script de Limpieza del Sistema

Desarrollar un script que:

- Limpie logs antiguos
- Elimine archivos temporales
- Optimice espacio en disco
- Genere reporte de limpieza

## Troubleshooting

### Problemas Comunes

1. **Permisos de ejecución**:

```bash
chmod +x script.sh
```

2. **Variables de entorno**:

```bash
export MY_VAR="valor"
```

3. **Paths relativos vs absolutos**:

```bash
# Usar path absoluto
SCRIPT_DIR="$(cd "$(dirname "${BASH_SOURCE[0]}")" && pwd)"
```

4. **Manejo de errores**:

```bash
set -euo pipefail  # Fail fast
```

## Recursos Adicionales

- [Bash Scripting Guide](https://tldp.org/LDP/Bash-Beginners-Guide/html/)
- [PowerShell Documentation](https://docs.microsoft.com/en-us/powershell/)
- [Python Automation](https://automatetheboringstuff.com/)
- [Shell Check](https://www.shellcheck.net/) - Validador de scripts bash

## Checklist de Completado

- [ ] Creado script bash básico de backup
- [ ] Desarrollado script PowerShell para Windows
- [ ] Implementado script Python multiplataforma
- [ ] Aplicado manejo de errores robusto
- [ ] Configurado logging apropiado
- [ ] Probado scripts en diferentes entornos
- [ ] Documentado scripts creados
- [ ] Implementado al menos 2 laboratorios

---

**Felicitaciones!**Has dominado los fundamentos de scripting para automatización DevOps. Continúa con el siguiente módulo para aprender sobre herramientas de automatización más avanzadas.
