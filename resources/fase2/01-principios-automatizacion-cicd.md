# Automatización de Tareas

La automatización es el **DNA de DevOps**. En este módulo aprenderás a transformar procesos manuales repetitivos en workflows automatizados que liberan tiempo para tareas más estratégicas y reducen errores humanos.

---

## **¿Por qué automatizar tareas en DevOps?**

La automatización en DevOps no es solo una conveniencia; es un **imperativo estratégico**que permite:

- **Velocidad**: Ejecutar tareas en segundos que toman horas manualmente
- **Consistencia**: Eliminar variaciones humanas en procesos críticos
- **Confiabilidad**: Reducir errores y aumentar la predictibilidad
- **Escalabilidad**: Manejar volúmenes crecientes sin aumentar personal
- **Liberación de tiempo**: Enfocar esfuerzos en innovación y resolución de problemas complejos

En DevOps, **"si lo haces más de dos veces, automatízalo"**es más que un eslogan; es una filosofía operativa.

---

## **1. Fundamentos de la Automatización**

### **El espectro de la automatización**

La automatización en DevOps existe en diferentes niveles:

```
Manual → Scripted → Automated → Orchestrated → Self-Healing

```

- **Manual**: Procesos ejecutados completamente a mano
- **Scripted**: Comandos automatizados mediante scripts
- **Automated**: Workflows que se ejecutan sin intervención
- **Orchestrated**: Coordinación automática de múltiples sistemas
- **Self-Healing**: Sistemas que se auto-diagnostican y reparan

### **Principios de automatización efectiva**

#### **1. Idempotencia**

```bash
#  No idempotente - crea duplicados
echo "export PATH=$PATH:/usr/local/bin" >> ~/.bashrc

#  Idempotente - solo agrega si no existe
if ! grep -q "/usr/local/bin" ~/.bashrc; then
    echo "export PATH=$PATH:/usr/local/bin" >> ~/.bashrc
fi
```

#### **2. Atomicidad**

```bash
#  Operación atómica - todo o nada
deploy_application() {
    local temp_dir=$(mktemp -d)

    # Si cualquier paso falla, hacer rollback
    trap 'cleanup_and_rollback' ERR

    download_artifact "$temp_dir"
    validate_artifact "$temp_dir"
    backup_current_version
    deploy_new_version "$temp_dir"
    run_health_checks

    echo " Deployment successful"
}
```

#### **3. Observabilidad**

```bash
#  Log detallado para debugging
log() {
    echo "[$(date +'%Y-%m-%d %H:%M:%S')] $1" | tee -a deployment.log
}

deploy_step() {
    log "Starting deployment step: $1"
    # ... lógica del step ...
    log "Completed deployment step: $1"
}
```

---

## **2. Scripts de Shell/Bash**

### **Automatización básica con Bash**

Bash es el lenguaje de automatización más ubicuo en entornos Linux/Unix y la base para muchas tareas DevOps.

#### **Script básico de deployment**

```bash
#!/bin/bash
set -euo pipefail  # Salir en error, variables no definidas, pipes fallidos

# Configuración
APP_NAME="my-web-app"
DEPLOY_DIR="/opt/${APP_NAME}"
BACKUP_DIR="/backup/${APP_NAME}"
LOG_FILE="/var/log/${APP_NAME}-deploy.log"

# Función de logging
log() {
    echo "[$(date +'%Y-%m-%d %H:%M:%S')] $1" | tee -a "$LOG_FILE"
}

# Función de backup
backup_current_version() {
    if [ -d "$DEPLOY_DIR" ]; then
        local backup_name="${APP_NAME}-$(date +'%Y%m%d-%H%M%S')"
        log "Creating backup: $backup_name"
        sudo cp -r "$DEPLOY_DIR" "${BACKUP_DIR}/${backup_name}"
    fi
}

# Función de health check
health_check() {
    log "Running health check..."
    local retries=5
    local count=0

    while [ $count -lt $retries ]; do
        if curl -f -s http://localhost:8080/health; then
            log " Health check passed"
            return 0
        fi

        count=$((count + 1))
        log "Health check attempt $count failed, retrying..."
        sleep 10
    done

    log " Health check failed after $retries attempts"
    return 1
}

# Función principal de deployment
deploy() {
    log "Starting deployment of $APP_NAME"

    # Validar prerrequisitos
    command -v curl >/dev/null 2>&1 || { log "curl is required"; exit 1; }

    # Crear directorios si no existen
    sudo mkdir -p "$DEPLOY_DIR" "$BACKUP_DIR"

    # Backup de versión actual
    backup_current_version

    # Download y deployment (simplificado)
    log "Downloading latest version..."
    # wget -O /tmp/app.tar.gz https://releases.example.com/latest.tar.gz

    log "Extracting application..."
    # sudo tar -xzf /tmp/app.tar.gz -C "$DEPLOY_DIR"

    # Reiniciar servicio
    log "Restarting application service..."
    sudo systemctl restart "$APP_NAME" || {
        log " Failed to restart service, rolling back..."
        rollback_last_backup
        exit 1
    }

    # Health check
    if health_check; then
        log " Deployment completed successfully"
    else
        log " Health check failed, rolling back..."
        rollback_last_backup
        exit 1
    fi
}

# Función de rollback
rollback_last_backup() {
    local latest_backup=$(ls -t "${BACKUP_DIR}" | head -n 1)
    if [ -n "$latest_backup" ]; then
        log "Rolling back to: $latest_backup"
        sudo rm -rf "$DEPLOY_DIR"
        sudo cp -r "${BACKUP_DIR}/${latest_backup}" "$DEPLOY_DIR"
        sudo systemctl restart "$APP_NAME"
    fi
}

# Ejecución principal
main() {
    case "${1:-deploy}" in
        deploy)
            deploy
            ;;
        rollback)
            rollback_last_backup
            ;;
        health)
            health_check
            ;;
        *)
            echo "Usage: $0 {deploy|rollback|health}"
            exit 1
            ;;
    esac
}

main "$@"
```

#### **Script de configuración de entorno de desarrollo**

```bash
#!/bin/bash
# dev-setup.sh - Configuración automatizada de entorno de desarrollo

set -euo pipefail

# Colores para output
RED='\033[0;31m'
GREEN='\033[0;32m'
YELLOW='\033[1;33m'
NC='\033[0m' # No Color

log_info() {
    echo -e "${GREEN}[INFO]${NC} $1"
}

log_warn() {
    echo -e "${YELLOW}[WARN]${NC} $1"
}

log_error() {
    echo -e "${RED}[ERROR]${NC} $1"
}

# Detectar sistema operativo
detect_os() {
    if [[ "$OSTYPE" == "linux-gnu"* ]]; then
        echo "linux"
    elif [[ "$OSTYPE" == "darwin"* ]]; then
        echo "macos"
    elif [[ "$OSTYPE" == "msys" ]] || [[ "$OSTYPE" == "win32" ]]; then
        echo "windows"
    else
        echo "unknown"
    fi
}

# Instalar Node.js
install_nodejs() {
    log_info "Installing Node.js..."

    case $(detect_os) in
        linux)
            curl -fsSL https://deb.nodesource.com/setup_18.x | sudo -E bash -
            sudo apt-get install -y nodejs
            ;;
        macos)
            if command -v brew >/dev/null 2>&1; then
                brew install node
            else
                log_error "Homebrew not found. Please install it first."
                return 1
            fi
            ;;
        windows)
            log_info "Please download Node.js from https://nodejs.org"
            ;;
        *)
            log_error "Unsupported OS for automatic Node.js installation"
            return 1
            ;;
    esac
}

# Instalar Docker
install_docker() {
    log_info "Installing Docker..."

    case $(detect_os) in
        linux)
            curl -fsSL https://get.docker.com -o get-docker.sh
            sh get-docker.sh
            sudo usermod -aG docker $USER
            ;;
        macos)
            log_info "Please download Docker Desktop from https://www.docker.com/products/docker-desktop"
            ;;
        windows)
            log_info "Please download Docker Desktop from https://www.docker.com/products/docker-desktop"
            ;;
    esac
}

# Configurar Git
setup_git() {
    log_info "Configuring Git..."

    read -p "Enter your Git username: " git_username
    read -p "Enter your Git email: " git_email

    git config --global user.name "$git_username"
    git config --global user.email "$git_email"
    git config --global init.defaultBranch main
    git config --global core.editor "code --wait"

    log_info "Git configured successfully"
}

# Instalar VS Code extensions
install_vscode_extensions() {
    log_info "Installing VS Code extensions..."

    local extensions=(
        "ms-vscode.vscode-typescript-next"
        "esbenp.prettier-vscode"
        "ms-python.python"
        "ms-vscode.vscode-docker"
        "gitpod.gitpod-desktop"
        "hashicorp.terraform"
    )

    for ext in "${extensions[@]}"; do
        if command -v code >/dev/null 2>&1; then
            code --install-extension "$ext"
        else
            log_warn "VS Code not found, skipping extension: $ext"
        fi
    done
}

# Crear estructura de proyecto
create_project_structure() {
    log_info "Creating project structure..."

    mkdir -p ~/Development/{personal,work,learning}
    mkdir -p ~/Development/learning/{tutorials,experiments,courses}

    cat > ~/Development/README.md << EOF
# Development Environment

This directory contains all development projects organized by category:

- **personal/**: Personal projects and side projects
- **work/**: Work-related projects and repositories
- **learning/**: Educational content, tutorials, and experiments

## Quick Start

\`\`\`bash
# Navigate to learning directory
cd ~/Development/learning

# Create a new project
mkdir new-project && cd new-project
git init
\`\`\`
EOF

    log_info "Project structure created in ~/Development"
}

# Función principal
main() {
    log_info " Starting development environment setup..."

    # Verificar prerrequisitos
    command -v git >/dev/null 2>&1 || { log_error "Git is required"; exit 1; }
    command -v curl >/dev/null 2>&1 || { log_error "curl is required"; exit 1; }

    # Menu de opciones
    echo "Select components to install:"
    echo "1) Node.js"
    echo "2) Docker"
    echo "3) Configure Git"
    echo "4) VS Code Extensions"
    echo "5) Create Project Structure"
    echo "6) All of the above"
    echo "q) Quit"

    read -p "Enter your choice [1-6,q]: " choice

    case $choice in
        1) install_nodejs ;;
        2) install_docker ;;
        3) setup_git ;;
        4) install_vscode_extensions ;;
        5) create_project_structure ;;
        6)
            install_nodejs
            install_docker
            setup_git
            install_vscode_extensions
            create_project_structure
            ;;
        q) exit 0 ;;
        *) log_error "Invalid option" ;;
    esac

    log_info " Setup completed!"
    log_warn "Note: You may need to restart your terminal for some changes to take effect."
}

main "$@"
```

### **Mejores prácticas para scripts Bash**

#### **1. Manejo de errores robusto**

```bash
#!/bin/bash
set -euo pipefail  # Configuración estricta

# Trap para cleanup en caso de error
cleanup() {
    local exit_code=$?
    log "Cleaning up temporary files..."
    rm -rf "$TEMP_DIR"
    exit $exit_code
}

trap cleanup EXIT ERR

# Validación de argumentos
validate_args() {
    if [ $# -eq 0 ]; then
        echo "Error: No arguments provided"
        echo "Usage: $0 <environment> <version>"
        exit 1
    fi
}

# Función con manejo de errores
safe_execute() {
    local cmd="$1"
    local description="$2"

    log "Executing: $description"
    if ! eval "$cmd"; then
        log " Failed to execute: $description"
        return 1
    fi
    log " Successfully executed: $description"
}
```

#### **2. Configuración externa**

```bash
# config.env
APP_NAME="my-application"
DEPLOY_ENV="production"
DATABASE_URL="postgresql://localhost:5432/myapp"
LOG_LEVEL="info"
BACKUP_RETENTION_DAYS="7"

# deploy.sh
#!/bin/bash
# Cargar configuración
source "$(dirname "$0")/config.env"

# Usar variables de configuración
deploy_to_environment() {
    log "Deploying $APP_NAME to $DEPLOY_ENV"
    # ... lógica de deployment usando variables del config
}
```

#### **3. Testing de scripts**

```bash
#!/bin/bash
# test_deploy.sh - Tests para script de deployment

# Función de test
run_test() {
    local test_name="$1"
    local expected="$2"
    local actual="$3"

    if [ "$expected" = "$actual" ]; then
        echo " $test_name: PASS"
    else
        echo " $test_name: FAIL (expected: $expected, got: $actual)"
        return 1
    fi
}

# Test de validación de argumentos
test_argument_validation() {
    local output
    output=$(./deploy.sh 2>&1 || true)

    if echo "$output" | grep -q "Usage:"; then
        echo " Argument validation: PASS"
    else
        echo " Argument validation: FAIL"
        return 1
    fi
}

# Ejecutar tests
main() {
    echo "Running deployment script tests..."

    test_argument_validation
    # Agregar más tests aquí

    echo "All tests completed"
}

main "$@"
```

---

## **3. Herramientas de Automatización Avanzadas**

### **GNU Make: Automatización de builds**

Make no es solo para compilar código C; es una herramienta poderosa para automatizar cualquier flujo de trabajo basado en dependencias.

#### **Makefile básico para proyecto web**

```makefile
# Variables
APP_NAME := my-web-app
VERSION := $(shell git describe --tags --abbrev=0 2>/dev/null || echo "v0.1.0")
BUILD_DIR := build
DIST_DIR := dist

# Configuración por defecto
.DEFAULT_GOAL := help
.PHONY: help install build test deploy clean

# Target de ayuda
help: ## Mostrar esta ayuda
 @echo "Available targets:"
 @awk 'BEGIN {FS = ":.*?## "} /^[a-zA-Z_-]+:.*?## / {printf "\033[36m%-20s\033[0m %s\n", $$1, $$2}' $(MAKEFILE_LIST)

# Instalación de dependencias
install: ## Instalar dependencias del proyecto
 @echo " Installing dependencies..."
 npm ci
 @echo " Dependencies installed"

# Build del proyecto
build: install ## Compilar el proyecto
 @echo " Building $(APP_NAME) version $(VERSION)..."
 mkdir -p $(BUILD_DIR)
 npm run build
 @echo " Build completed"

# Ejecutar tests
test: install ## Ejecutar suite de tests
 @echo " Running tests..."
 npm run test:unit
 npm run test:integration
 @echo " All tests passed"

# Linting y formateo
lint: install ## Ejecutar linting y formateo
 @echo " Running linter..."
 npm run lint
 npm run format:check
 @echo " Code style validated"

# Build de producción
dist: test lint ## Crear distribución para producción
 @echo " Creating production build..."
 mkdir -p $(DIST_DIR)
 npm run build:prod
 tar -czf $(DIST_DIR)/$(APP_NAME)-$(VERSION).tar.gz -C $(BUILD_DIR) .
 @echo " Distribution created: $(DIST_DIR)/$(APP_NAME)-$(VERSION).tar.gz"

# Deployment local
deploy-local: dist ## Desplegar localmente
 @echo " Deploying locally..."
 docker build -t $(APP_NAME):$(VERSION) .
 docker run -d --name $(APP_NAME) -p 8080:80 $(APP_NAME):$(VERSION)
 @echo " Local deployment completed on http://localhost:8080"

# Deployment a staging
deploy-staging: dist ## Desplegar a staging
 @echo " Deploying to staging..."
 scp $(DIST_DIR)/$(APP_NAME)-$(VERSION).tar.gz staging-server:/tmp/
 ssh staging-server "cd /opt/$(APP_NAME) && tar -xzf /tmp/$(APP_NAME)-$(VERSION).tar.gz && sudo systemctl restart $(APP_NAME)"
 @echo " Staging deployment completed"

# Health check
health-check: ## Verificar salud de la aplicación
 @echo " Running health check..."
 curl -f http://localhost:8080/health || (echo " Health check failed" && exit 1)
 @echo " Application is healthy"

# Limpieza
clean: ## Limpiar archivos generados
 @echo " Cleaning up..."
 rm -rf $(BUILD_DIR) $(DIST_DIR) node_modules/.cache
 docker system prune -f
 @echo " Cleanup completed"

# Target condicional para CI/CD
ci: ## Target para CI/CD pipeline
ifeq ($(CI),true)
 @echo " Running in CI environment"
 $(MAKE) test lint dist
else
 @echo "  Not in CI environment, running local build"
 $(MAKE) build test
endif
```

#### **Makefile avanzado con Docker**

```makefile
# Variables de Docker
DOCKER_REGISTRY := your-registry.com
DOCKER_IMAGE := $(DOCKER_REGISTRY)/$(APP_NAME)
DOCKER_TAG := $(VERSION)

# Targets de Docker
docker-build: ## Construir imagen Docker
 @echo " Building Docker image..."
 docker build \
  --build-arg VERSION=$(VERSION) \
  --build-arg BUILD_DATE=$(shell date -u +'%Y-%m-%dT%H:%M:%SZ') \
  -t $(DOCKER_IMAGE):$(DOCKER_TAG) \
  -t $(DOCKER_IMAGE):latest \
  .
 @echo " Docker image built: $(DOCKER_IMAGE):$(DOCKER_TAG)"

docker-push: docker-build ## Subir imagen a registry
 @echo " Pushing Docker image..."
 docker push $(DOCKER_IMAGE):$(DOCKER_TAG)
 docker push $(DOCKER_IMAGE):latest
 @echo " Docker image pushed"

docker-run: docker-build ## Ejecutar contenedor localmente
 @echo " Starting Docker container..."
 docker run -d \
  --name $(APP_NAME)-dev \
  -p 8080:80 \
  -e NODE_ENV=development \
  $(DOCKER_IMAGE):$(DOCKER_TAG)
 @echo " Container started on http://localhost:8080"

docker-stop: ## Detener y remover contenedor
 @echo " Stopping Docker container..."
 docker stop $(APP_NAME)-dev || true
 docker rm $(APP_NAME)-dev || true
 @echo " Container stopped"
```

---

## **4. Ansible: Automatización de Configuración**

Ansible es una herramienta de automatización que permite gestionar configuraciones, deployments y tareas administrativas de forma declarativa.

### **Conceptos básicos de Ansible**

#### **Inventario (inventory.yml)**

```yaml
# inventory.yml
all:
  children:
    webservers:
      hosts:
        web1.example.com:
          ansible_user: deploy
        web2.example.com:
          ansible_user: deploy

    databases:
      hosts:
        db1.example.com:
          ansible_user: admin
      vars:
        db_port: 5432

    staging:
      hosts:
        staging.example.com:
          ansible_user: deploy
          ansible_ssh_private_key_file: ~/.ssh/staging_key

  vars:
    ansible_ssh_common_args: '-o StrictHostKeyChecking=no'
```

#### **Playbook básico de configuración**

```yaml
# setup-webserver.yml
---
- name: Configure Web Servers
  hosts: webservers
  become: yes
  vars:
    app_name: my-web-app
    app_user: www-data
    nginx_port: 80

  tasks:
    - name: Update package cache
      apt:
        update_cache: yes
        cache_valid_time: 3600
      when: ansible_os_family == "Debian"

    - name: Install required packages
      apt:
        name:
          - nginx
          - nodejs
          - npm
          - git
          - curl
        state: present

    - name: Create application user
      user:
        name: "{{ app_user }}"
        system: yes
        shell: /bin/bash
        home: "/opt/{{ app_name }}"
        createhome: yes

    - name: Create application directories
      file:
        path: "{{ item }}"
        state: directory
        owner: "{{ app_user }}"
        group: "{{ app_user }}"
        mode: '0755'
      loop:
        - "/opt/{{ app_name }}"
        - "/opt/{{ app_name }}/releases"
        - "/opt/{{ app_name }}/shared"
        - "/var/log/{{ app_name }}"

    - name: Configure Nginx
      template:
        src: nginx.conf.j2
        dest: /etc/nginx/sites-available/{{ app_name }}
        backup: yes
      notify:
        - restart nginx

    - name: Enable Nginx site
      file:
        src: /etc/nginx/sites-available/{{ app_name }}
        dest: /etc/nginx/sites-enabled/{{ app_name }}
        state: link
      notify:
        - restart nginx

    - name: Remove default Nginx site
      file:
        path: /etc/nginx/sites-enabled/default
        state: absent
      notify:
        - restart nginx

    - name: Start and enable services
      systemd:
        name: "{{ item }}"
        state: started
        enabled: yes
      loop:
        - nginx
        - ufw

    - name: Configure firewall
      ufw:
        rule: allow
        port: "{{ item }}"
        proto: tcp
      loop:
        - "22"
        - "80"
        - "443"

  handlers:
    - name: restart nginx
      systemd:
        name: nginx
        state: restarted
```

#### **Template de Nginx (nginx.conf.j2)**

```nginx
# templates/nginx.conf.j2
server {
    listen {{ nginx_port }};
    server_name {{ inventory_hostname }};

    root /opt/{{ app_name }}/current/public;
    index index.html index.htm;

    # Logs
    access_log /var/log/{{ app_name }}/nginx_access.log;
    error_log /var/log/{{ app_name }}/nginx_error.log;

    # Gzip compression
    gzip on;
    gzip_types text/plain text/css application/json application/javascript text/xml application/xml application/xml+rss text/javascript;

    location / {
        try_files $uri $uri/ @nodejs;
    }

    location @nodejs {
        proxy_pass http://127.0.0.1:3000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_cache_bypass $http_upgrade;
    }

    # Health check endpoint
    location /health {
        proxy_pass http://127.0.0.1:3000/health;
    }

    # Static assets caching
    location ~* \.(jpg|jpeg|png|gif|ico|css|js)$ {
        expires 1y;
        add_header Cache-Control "public, immutable";
    }
}
```

#### **Playbook de deployment**

```yaml
# deploy-app.yml
---
- name: Deploy Application
  hosts: webservers
  become: yes
  vars:
    app_name: my-web-app
    app_user: www-data
    app_repo: https://github.com/company/my-web-app.git
    app_version: "{{ version | default('main') }}"
    releases_to_keep: 3

  tasks:
    - name: Create release directory
      set_fact:
        release_dir: "/opt/{{ app_name }}/releases/{{ ansible_date_time.epoch }}"

    - name: Clone repository
      git:
        repo: "{{ app_repo }}"
        dest: "{{ release_dir }}"
        version: "{{ app_version }}"
        depth: 1
      become_user: "{{ app_user }}"

    - name: Install Node.js dependencies
      npm:
        path: "{{ release_dir }}"
        production: yes
      become_user: "{{ app_user }}"

    - name: Build application
      command: npm run build
      args:
        chdir: "{{ release_dir }}"
      become_user: "{{ app_user }}"

    - name: Copy environment configuration
      template:
        src: .env.j2
        dest: "{{ release_dir }}/.env"
        owner: "{{ app_user }}"
        group: "{{ app_user }}"
        mode: '0600'

    - name: Create systemd service file
      template:
        src: app.service.j2
        dest: /etc/systemd/system/{{ app_name }}.service
      notify:
        - reload systemd
        - restart app

    - name: Update current symlink
      file:
        src: "{{ release_dir }}"
        dest: "/opt/{{ app_name }}/current"
        state: link
        force: yes
      notify:
        - restart app

    - name: Start and enable application service
      systemd:
        name: "{{ app_name }}"
        state: started
        enabled: yes
        daemon_reload: yes

    - name: Wait for application to be ready
      uri:
        url: "http://{{ inventory_hostname }}/health"
        method: GET
        status_code: 200
      register: health_check
      until: health_check.status == 200
      retries: 30
      delay: 2

    - name: Clean up old releases
      shell: |
        cd /opt/{{ app_name }}/releases
        ls -t | tail -n +{{ releases_to_keep + 1 }} | xargs rm -rf
      become_user: "{{ app_user }}"

  handlers:
    - name: reload systemd
      systemd:
        daemon_reload: yes

    - name: restart app
      systemd:
        name: "{{ app_name }}"
        state: restarted
```

#### **Ejecución de playbooks**

```bash
# Configurar servidores web
ansible-playbook -i inventory.yml setup-webserver.yml

# Deployment con versión específica
ansible-playbook -i inventory.yml deploy-app.yml -e "version=v2.1.0"

# Deployment solo a staging
ansible-playbook -i inventory.yml deploy-app.yml --limit staging

# Dry run para verificar cambios
ansible-playbook -i inventory.yml deploy-app.yml --check --diff

# Ejecutar con verbosidad para debugging
ansible-playbook -i inventory.yml deploy-app.yml -vvv
```

---

## **5. Otras Herramientas de Automatización**

### **Puppet: Gestión Declarativa de Configuración**

Puppet utiliza un enfoque declarativo donde defines el estado deseado del sistema.

#### **Manifest básico de Puppet**

```puppet
# manifests/webserver.pp
class webserver {
  # Asegurar que Nginx esté instalado
  package { 'nginx':
    ensure => present,
  }

  # Configurar archivo de Nginx
  file { '/etc/nginx/nginx.conf':
    ensure  => file,
    content => template('nginx/nginx.conf.erb'),
    require => Package['nginx'],
    notify  => Service['nginx'],
  }

  # Asegurar que el servicio esté corriendo
  service { 'nginx':
    ensure  => running,
    enable  => true,
    require => Package['nginx'],
  }

  # Crear usuario de aplicación
  user { 'webappuser':
    ensure => present,
    home   => '/opt/webapp',
    shell  => '/bin/bash',
  }

  # Crear directorio de aplicación
  file { '/opt/webapp':
    ensure => directory,
    owner  => 'webappuser',
    group  => 'webappuser',
    mode   => '0755',
  }
}

# Aplicar la clase
include webserver
```

### **Chef: Automatización Basada en Ruby**

Chef utiliza "recipes" escritas en Ruby para automatizar configuración.

#### **Recipe básica de Chef**

```ruby
# cookbooks/webapp/recipes/default.rb

# Instalar paquetes necesarios
%w[nginx nodejs npm git].each do |pkg|
  package pkg do
    action :install
  end
end

# Crear usuario de aplicación
user 'webapp' do
  comment 'Web Application User'
  home '/opt/webapp'
  shell '/bin/bash'
  action :create
end

# Crear directorios
directory '/opt/webapp' do
  owner 'webapp'
  group 'webapp'
  mode '0755'
  action :create
end

# Configurar Nginx
template '/etc/nginx/sites-available/webapp' do
  source 'nginx-site.erb'
  variables({
    server_name: node['webapp']['server_name'],
    document_root: '/opt/webapp/current/public'
  })
  notifies :restart, 'service[nginx]', :delayed
end

# Habilitar sitio de Nginx
link '/etc/nginx/sites-enabled/webapp' do
  to '/etc/nginx/sites-available/webapp'
  action :create
  notifies :restart, 'service[nginx]', :delayed
end

# Asegurar que Nginx esté corriendo
service 'nginx' do
  action [:enable, :start]
end
```

---

## **6. Casos de Uso Prácticos**

### **Automatización de Database Backups**

```bash
#!/bin/bash
# database-backup.sh - Backup automatizado de base de datos

set -euo pipefail

# Configuración
DB_NAME="production_db"
DB_USER="backup_user"
BACKUP_DIR="/backup/database"
S3_BUCKET="my-company-backups"
RETENTION_DAYS=7

# Función de logging
log() {
    echo "[$(date +'%Y-%m-%d %H:%M:%S')] $1" | tee -a /var/log/db-backup.log
}

# Crear backup
create_backup() {
    local timestamp=$(date +'%Y%m%d_%H%M%S')
    local backup_file="${BACKUP_DIR}/${DB_NAME}_${timestamp}.sql.gz"

    log "Starting backup of database: $DB_NAME"

    # Crear directorio si no existe
    mkdir -p "$BACKUP_DIR"

    # Ejecutar backup con compresión
    if pg_dump -U "$DB_USER" "$DB_NAME" | gzip > "$backup_file"; then
        log " Backup created successfully: $backup_file"
    else
        log " Backup failed"
        exit 1
    fi

    # Subir a S3
    if aws s3 cp "$backup_file" "s3://${S3_BUCKET}/database/"; then
        log " Backup uploaded to S3"
    else
        log " Failed to upload backup to S3"
    fi
}

# Limpiar backups antiguos
cleanup_old_backups() {
    log "Cleaning up backups older than $RETENTION_DAYS days"

    # Limpiar backups locales
    find "$BACKUP_DIR" -name "${DB_NAME}_*.sql.gz" -mtime +$RETENTION_DAYS -delete

    # Limpiar backups en S3
    aws s3 ls "s3://${S3_BUCKET}/database/" | \
    awk -v retention="$RETENTION_DAYS" '
    {
        cmd="date -d \"" $1 " " $2 "\" +%s"
        cmd | getline file_date
        close(cmd)

        cmd="date +%s"
        cmd | getline current_date
        close(cmd)

        days_old = (current_date - file_date) / 86400
        if (days_old > retention) {
            print "s3://'${S3_BUCKET}'/database/" $4
        }
    }' | xargs -r aws s3 rm

    log " Cleanup completed"
}

# Verificar backup
verify_backup() {
    local backup_file="$1"

    log "Verifying backup integrity..."

    # Verificar que el archivo no esté vacío
    if [ ! -s "$backup_file" ]; then
        log " Backup file is empty"
        return 1
    fi

    # Verificar que se pueda descomprimir
    if ! gzip -t "$backup_file"; then
        log " Backup file is corrupted"
        return 1
    fi

    log " Backup verification passed"
}

# Función principal
main() {
    log "Starting database backup process"

    create_backup

    # Verificar el último backup creado
    local latest_backup=$(ls -t ${BACKUP_DIR}/${DB_NAME}_*.sql.gz | head -n 1)
    verify_backup "$latest_backup"

    cleanup_old_backups

    log " Database backup process completed successfully"
}

main "$@"
```

### **Script de Monitoreo de Sistema**

```bash
#!/bin/bash
# system-monitor.sh - Monitoreo automatizado de sistema

set -euo pipefail

# Configuración
ALERT_EMAIL="admin@company.com"
CPU_THRESHOLD=80
MEMORY_THRESHOLD=85
DISK_THRESHOLD=90
LOG_FILE="/var/log/system-monitor.log"

# Función de logging
log() {
    echo "[$(date +'%Y-%m-%d %H:%M:%S')] $1" | tee -a "$LOG_FILE"
}

# Enviar alerta
send_alert() {
    local subject="$1"
    local message="$2"

    log "ALERT: $subject"

    # Enviar email (requiere configuración de sendmail/postfix)
    echo "$message" | mail -s "[$HOSTNAME] $subject" "$ALERT_EMAIL"

    # También puede enviar a Slack, Discord, etc.
    # curl -X POST -H 'Content-type: application/json' \
    #     --data "{\"text\":\"$subject\\n$message\"}" \
    #     "$SLACK_WEBHOOK_URL"
}

# Verificar CPU
check_cpu() {
    local cpu_usage=$(top -bn1 | grep "Cpu(s)" | awk '{print $2}' | sed 's/%us,//')
    cpu_usage=${cpu_usage%.*}  # Remover decimales

    if [ "$cpu_usage" -gt "$CPU_THRESHOLD" ]; then
        send_alert "High CPU Usage Alert" \
            "CPU usage is at ${cpu_usage}% (threshold: ${CPU_THRESHOLD}%)"
    fi

    log "CPU Usage: ${cpu_usage}%"
}

# Verificar memoria
check_memory() {
    local memory_info=$(free | grep Mem)
    local total=$(echo $memory_info | awk '{print $2}')
    local used=$(echo $memory_info | awk '{print $3}')
    local memory_usage=$((used * 100 / total))

    if [ "$memory_usage" -gt "$MEMORY_THRESHOLD" ]; then
        send_alert "High Memory Usage Alert" \
            "Memory usage is at ${memory_usage}% (threshold: ${MEMORY_THRESHOLD}%)"
    fi

    log "Memory Usage: ${memory_usage}%"
}

# Verificar disco
check_disk() {
    df -h | grep -vE '^Filesystem|tmpfs|cdrom' | awk '{ print $5 " " $1 }' | \
    while read output; do
        usage=$(echo $output | awk '{ print $1}' | sed 's/%//g')
        partition=$(echo $output | awk '{ print $2 }')

        if [ $usage -gt $DISK_THRESHOLD ]; then
            send_alert "High Disk Usage Alert" \
                "Partition $partition is at ${usage}% (threshold: ${DISK_THRESHOLD}%)"
        fi

        log "Disk Usage $partition: ${usage}%"
    done
}

# Verificar servicios críticos
check_services() {
    local critical_services=("nginx" "postgresql" "redis")

    for service in "${critical_services[@]}"; do
        if ! systemctl is-active --quiet "$service"; then
            send_alert "Service Down Alert" \
                "Critical service '$service' is not running on $HOSTNAME"
        else
            log "Service $service: Running"
        fi
    done
}

# Verificar conectividad
check_connectivity() {
    local endpoints=("google.com" "github.com")

    for endpoint in "${endpoints[@]}"; do
        if ! ping -c 1 "$endpoint" >/dev/null 2>&1; then
            send_alert "Connectivity Alert" \
                "Unable to reach $endpoint from $HOSTNAME"
        else
            log "Connectivity to $endpoint: OK"
        fi
    done
}

# Generar reporte
generate_report() {
    local report_file="/tmp/system-report-$(date +'%Y%m%d').txt"

    {
        echo "System Health Report for $HOSTNAME"
        echo "Generated: $(date)"
        echo "===="
        echo

        echo "CPU Information:"
        lscpu | grep "Model name\|CPU(s)\|Thread(s)"
        echo

        echo "Memory Information:"
        free -h
        echo

        echo "Disk Usage:"
        df -h
        echo

        echo "Top Processes by CPU:"
        ps aux --sort=-%cpu | head -10
        echo

        echo "Top Processes by Memory:"
        ps aux --sort=-%mem | head -10
        echo

        echo "Network Connections:"
        ss -tuln | head -20

    } > "$report_file"

    log "System report generated: $report_file"
}

# Función principal
main() {
    log "Starting system monitoring check"

    check_cpu
    check_memory
    check_disk
    check_services
    check_connectivity

    # Generar reporte diario
    if [ "$(date +%H)" -eq 8 ]; then  # Solo a las 8 AM
        generate_report
    fi

    log "System monitoring check completed"
}

main "$@"
```

---

## **7. Integración con CI/CD**

### **Script para Pipeline de GitHub Actions**

```yaml
# .github/workflows/automation-pipeline.yml
name: Automation Pipeline

on:
  push:
    branches: [ main, develop ]
  pull_request:
    branches: [ main ]

jobs:
  automated-tests:
    runs-on: ubuntu-latest

    steps:
    - uses: actions/checkout@v3

    - name: Setup Node.js
      uses: actions/setup-node@v3
      with:
        node-version: '18'
        cache: 'npm'

    - name: Install dependencies
      run: npm ci

    - name: Run automation scripts validation
      run: |
        # Validar scripts de bash
        find scripts/ -name "*.sh" -exec shellcheck {} \;

        # Validar playbooks de Ansible
        find ansible/ -name "*.yml" -exec ansible-lint {} \;

        # Validar Makefiles
        make --dry-run --directory=.

    - name: Test automation scripts
      run: |
        # Ejecutar tests de scripts
        bash test/test_deployment.sh
        bash test/test_monitoring.sh

    - name: Security scan
      run: |
        # Escanear secrets en scripts
        docker run --rm -v "$PWD:/code" trufflesecurity/trufflehog:latest filesystem /code
```

---

## **8. Mejores Prácticas de Automatización**

### **Principios fundamentales**

1. **Idempotencia**: Los scripts deben poder ejecutarse múltiples veces con el mismo resultado
2. **Atomicidad**: Las operaciones deben ser todo-o-nada
3. **Observabilidad**: Logs detallados para debugging y auditoría
4. **Seguridad**: Nunca hardcodear credenciales, usar gestores de secretos
5. **Testing**: Todos los scripts de automatización deben tener tests
6. **Documentación**: Cada script debe tener documentación clara de uso

### **Checklist de automatización**

```markdown
## Checklist para Scripts de Automatización

### Funcionalidad
- [ ] El script es idempotente
- [ ] Maneja errores gracefully
- [ ] Tiene rollback en caso de fallo
- [ ] Valida prerrequisitos antes de ejecutar

### Seguridad
- [ ] No contiene credenciales hardcodeadas
- [ ] Usa variables de entorno para secretos
- [ ] Valida inputs del usuario
- [ ] Ejecuta con privilegios mínimos necesarios

### Observabilidad
- [ ] Genera logs detallados
- [ ] Incluye timestamps en logs
- [ ] Reporta métricas de ejecución
- [ ] Envía notificaciones en caso de error

### Mantenibilidad
- [ ] Código bien documentado
- [ ] Funciones modulares y reutilizables
- [ ] Configuración externalizada
- [ ] Tests automatizados incluidos

### Performance
- [ ] Optimizado para tiempo de ejecución
- [ ] Maneja recursos eficientemente
- [ ] Incluye timeouts apropiados
- [ ] Limpia recursos temporales
```

---

## **Próximos Pasos**

Una vez que domines la automatización de tareas, estarás listo para avanzar a:

- **Integración Continua (CI)**: Automatizar builds, tests y validaciones
- **Entrega/Despliegue Continuo (CD)**: Automatizar deployments y releases
- **Pruebas Automatizadas**: Integrar testing en pipelines automatizados

La automatización es el **fundamento**sobre el cual se construyen todas las prácticas avanzadas de DevOps. Cada hora invertida en automatización se multiplica en eficiencia y confiabilidad a largo plazo.

---

## **Recursos y Referencias**

### **Recursos Multimedia**

Para profundizar en las herramientas de Infrastructure as Code (IaC) y gestión de configuración, hemos incluido recursos multimedia especializados:

#### **Video: Infrastructure as Code - Puppet vs Chef vs Ansible**

<video width="100%" controls>
  <source src="../../assets/IaC__Puppet_vs_Chef_vs_Ansible.mp4" type="video/mp4">
  Tu navegador no soporta la reproducción de video HTML5.
</video>

**Duración**: Aproximadamente 15-20 minutos
**Contenido**: Comparación detallada entre las tres herramientas principales de gestión de configuración, incluyendo:

- Arquitectura y filosofía de cada herramienta
- Casos de uso recomendados
- Ventajas y desventajas
- Ejemplos prácticos de configuración
- Criterios para elegir la herramienta adecuada

#### **Audio: Puppet vs Chef vs Ansible - Análisis Profundo**

<audio controls style="width: 100%;">
  <source src="../../assets/Puppet_vs._Chef_vs - 1758859501151.m4a" type="audio/mp4">
  Tu navegador no soporta la reproducción de audio HTML5.
</audio>

**Formato**: Podcast/Conferencia
**Enfoque**: Análisis técnico comparativo que complementa el video con:

- Experiencias reales de implementación
- Consideraciones de adopción empresarial
- Mejores prácticas específicas por herramienta
- Tendencias del mercado y futuro de IaC

> **Tip de aprendizaje**: Te recomendamos ver el video primero para obtener una comprensión visual de las herramientas, y luego escuchar el audio para profundizar en los aspectos técnicos y estratégicos.

### **Documentación oficial**

- [GNU Bash Manual](https://www.gnu.org/software/bash/manual/)
- [GNU Make Documentation](https://www.gnu.org/software/make/manual/)
- [Ansible Documentation](https://docs.ansible.com/)
- [Puppet Documentation](https://puppet.com/docs/)
- [Chef Documentation](https://docs.chef.io/)

### **Herramientas de validación**

- [ShellCheck](https://shellcheck.net/) - Linter para scripts bash
- [Ansible Lint](https://ansible-lint.readthedocs.io/) - Linter para playbooks
- [YAML Lint](https://yamllint.readthedocs.io/) - Validador de sintaxis YAML

### **Ejemplos y templates**

- Scripts de ejemplo en el repositorio del curso
- Templates de Ansible playbooks
- Makefiles para diferentes tipos de proyecto

Felicidades! Has completado el módulo de automatización de tareas. Ahora tienes las bases para automatizar procesos y estás listo para integrar estas habilidades con pipelines de CI/CD.
