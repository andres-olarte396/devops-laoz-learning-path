# 02. Herramientas de Automatización

**Potencia tu automatización con herramientas especializadas!** Este módulo te enseñará a usar herramientas profesionales de automatización que van más allá de scripts simples.

##  Objetivos de Aprendizaje

Al completar este módulo serás capaz de:

- **Dominar Make** para automatización de builds
- **Configurar Ansible** para gestión de infraestructura
- **Entender Puppet y Chef** para configuration management
- **Implementar automatización** declarativa vs imperativa
- **Gestionar inventarios** y configuraciones complejas
- **Crear playbooks** reutilizables y escalables

##  Contenido del Módulo

### 1. Make - Automatización de Builds

####  **Fundamentos de Make**

Make es una herramienta de automatización de build que utiliza un archivo `Makefile` para definir tareas y sus dependencias.

```makefile
# Makefile básico para proyecto web
.PHONY: help install build test deploy clean

# Variables
APP_NAME = mi-aplicacion
VERSION = 1.0.0
BUILD_DIR = dist
NODE_VERSION = 18

# Target por defecto
help: ## Mostrar ayuda
 @echo "Comandos disponibles:"
 @grep -E '^[a-zA-Z_-]+:.*?## .*$$' $(MAKEFILE_LIST) | \
  awk 'BEGIN {FS = ":.*?## "}; {printf "\033[36m%-20s\033[0m %s\n", $$1, $$2}'

install: ## Instalar dependencias
 @echo " Instalando dependencias..."
 npm ci
 @echo " Dependencias instaladas"

build: install ## Construir la aplicación
 @echo " Construyendo aplicación..."
 npm run build
 @echo " Build completado en $(BUILD_DIR)/"

test: install ## Ejecutar tests
 @echo " Ejecutando tests..."
 npm run test:unit
 npm run test:integration
 @echo " Tests completados"

lint: install ## Ejecutar linting
 @echo " Ejecutando linting..."
 npm run lint
 npm run lint:fix
 @echo " Linting completado"

# Target compuesto que ejecuta calidad completa
quality: lint test ## Ejecutar verificaciones de calidad
 @echo " Verificaciones de calidad completadas"

package: build ## Crear paquete de distribución
 @echo " Creando paquete..."
 mkdir -p packages
 tar -czf packages/$(APP_NAME)-$(VERSION).tar.gz -C $(BUILD_DIR) .
 @echo " Paquete creado: packages/$(APP_NAME)-$(VERSION).tar.gz"

deploy-staging: package ## Desplegar a staging
 @echo " Desplegando a staging..."
 ./scripts/deploy.sh staging packages/$(APP_NAME)-$(VERSION).tar.gz
 @echo " Desplegado a staging"

deploy-prod: package ## Desplegar a producción
 @echo " Desplegando a producción..."
 @read -p "¿Confirmas deployment a producción? [y/N] " confirm && \
 if [ "$$confirm" = "y" ] || [ "$$confirm" = "Y" ]; then \
  ./scripts/deploy.sh production packages/$(APP_NAME)-$(VERSION).tar.gz; \
  echo " Desplegado a producción"; \
 else \
  echo " Deployment cancelado"; \
 fi

clean: ## Limpiar archivos generados
 @echo " Limpiando archivos..."
 rm -rf $(BUILD_DIR)
 rm -rf node_modules
 rm -rf packages
 @echo " Limpieza completada"

# Targets para desarrollo
dev: install ## Iniciar servidor de desarrollo
 @echo " Iniciando servidor de desarrollo..."
 npm run dev

watch: install ## Iniciar build en modo watch
 @echo " Iniciando build en modo watch..."
 npm run build:watch

# Docker targets
docker-build: ## Construir imagen Docker
 @echo " Construyendo imagen Docker..."
 docker build -t $(APP_NAME):$(VERSION) .
 docker tag $(APP_NAME):$(VERSION) $(APP_NAME):latest
 @echo " Imagen Docker construida"

docker-run: docker-build ## Ejecutar contenedor
 @echo " Ejecutando contenedor..."
 docker run -p 8080:8080 --name $(APP_NAME) $(APP_NAME):latest

docker-clean: ## Limpiar recursos Docker
 @echo " Limpiando recursos Docker..."
 -docker stop $(APP_NAME)
 -docker rm $(APP_NAME)
 -docker rmi $(APP_NAME):$(VERSION) $(APP_NAME):latest
 @echo " Recursos Docker limpiados"

# Verificar prerequisites
check-prereqs: ## Verificar prerrequisitos
 @echo " Verificando prerrequisitos..."
 @command -v node >/dev/null 2>&1 || { echo " Node.js no está instalado"; exit 1; }
 @node -v | grep -q "v$(NODE_VERSION)" || { echo " Se requiere Node.js v$(NODE_VERSION)"; }
 @command -v npm >/dev/null 2>&1 || { echo " npm no está instalado"; exit 1; }
 @echo " Prerrequisitos verificados"

# CI/CD pipeline completo
ci: check-prereqs quality package ## Pipeline completo de CI
 @echo " Pipeline CI completado exitosamente"

cd-staging: ci deploy-staging ## Pipeline completo CD para staging
 @echo " Pipeline CD staging completado"

cd-prod: ci deploy-prod ## Pipeline completo CD para producción
 @echo " Pipeline CD producción completado"

# Backup y restore
backup: ## Crear backup de la aplicación
 @echo " Creando backup..."
 ./scripts/backup.sh
 @echo " Backup completado"

restore: ## Restaurar desde backup
 @echo " Restaurando desde backup..."
 ./scripts/restore.sh
 @echo " Restauración completada"
```

####  **Makefile Avanzado para Microservicios**

```makefile
# Makefile para arquitectura de microservicios
.PHONY: help

# Configuración
SERVICES := auth user order payment notification
DOCKER_REGISTRY := registry.company.com
VERSION ?= latest
NAMESPACE ?= default

help:
 @echo " Makefile para Microservicios"
 @echo "Servicios disponibles: $(SERVICES)"
 @echo ""
 @echo "Comandos globales:"
 @echo "  make build-all          - Construir todos los servicios"
 @echo "  make test-all           - Probar todos los servicios"
 @echo "  make deploy-all         - Desplegar todos los servicios"
 @echo ""
 @echo "Comandos por servicio:"
 @echo "  make build-<service>    - Construir servicio específico"
 @echo "  make test-<service>     - Probar servicio específico"
 @echo "  make deploy-<service>   - Desplegar servicio específico"

# Templates para targets dinámicos
define service_targets
build-$(1): ## Construir servicio $(1)
 @echo " Construyendo servicio $(1)..."
 cd services/$(1) && docker build -t $(DOCKER_REGISTRY)/$(1):$(VERSION) .
 @echo " Servicio $(1) construido"

test-$(1): ## Probar servicio $(1)
 @echo " Probando servicio $(1)..."
 cd services/$(1) && npm test
 @echo " Tests de $(1) completados"

deploy-$(1): build-$(1) ## Desplegar servicio $(1)
 @echo " Desplegando servicio $(1)..."
 docker push $(DOCKER_REGISTRY)/$(1):$(VERSION)
 kubectl set image deployment/$(1) $(1)=$(DOCKER_REGISTRY)/$(1):$(VERSION) -n $(NAMESPACE)
 kubectl rollout status deployment/$(1) -n $(NAMESPACE)
 @echo " Servicio $(1) desplegado"

logs-$(1): ## Ver logs del servicio $(1)
 kubectl logs -f deployment/$(1) -n $(NAMESPACE)

shell-$(1): ## Conectar al shell del servicio $(1)
 kubectl exec -it deployment/$(1) -n $(NAMESPACE) -- /bin/sh
endef

# Generar targets para cada servicio
$(foreach service,$(SERVICES),$(eval $(call service_targets,$(service))))

# Targets agregados
build-all: $(addprefix build-,$(SERVICES)) ## Construir todos los servicios

test-all: $(addprefix test-,$(SERVICES)) ## Probar todos los servicios

deploy-all: $(addprefix deploy-,$(SERVICES)) ## Desplegar todos los servicios

# Gestión de base de datos
db-migrate: ## Ejecutar migraciones de base de datos
 @echo " Ejecutando migraciones..."
 kubectl exec -it deployment/auth -n $(NAMESPACE) -- npm run migrate
 @echo " Migraciones completadas"

db-seed: ## Poblar base de datos con datos de prueba
 @echo " Poblando base de datos..."
 kubectl exec -it deployment/auth -n $(NAMESPACE) -- npm run seed
 @echo " Base de datos poblada"

# Monitoreo y observabilidad
status: ## Verificar estado de todos los servicios
 @echo " Estado de servicios:"
 @for service in $(SERVICES); do \
  echo -n "$$service: "; \
  kubectl get deployment/$$service -n $(NAMESPACE) -o jsonpath='{.status.readyReplicas}/{.spec.replicas}' 2>/dev/null || echo "No encontrado"; \
  echo ""; \
 done

health-check: ## Verificar health de todos los servicios
 @echo " Health check de servicios:"
 @for service in $(SERVICES); do \
  echo "Checking $$service..."; \
  kubectl exec deployment/$$service -n $(NAMESPACE) -- curl -f http://localhost:8080/health || echo " $$service no responde"; \
 done

# Limpieza
clean: ## Limpiar recursos
 @echo " Limpiando recursos..."
 docker system prune -f
 @echo " Limpieza completada"

nuke: ##  PELIGRO: Eliminar todos los servicios
 @echo " Eliminando TODOS los servicios..."
 @read -p "¿Estás seguro? Esto eliminará todo en el namespace $(NAMESPACE) [y/N] " confirm && \
 if [ "$$confirm" = "y" ] || [ "$$confirm" = "Y" ]; then \
  kubectl delete namespace $(NAMESPACE); \
  echo " Namespace $(NAMESPACE) eliminado"; \
 else \
  echo " Operación cancelada"; \
 fi
```

### 2. Ansible - Configuration Management

####  **Introducción a Ansible**

Ansible es una herramienta de automatización que utiliza un lenguaje declarativo simple para describir configuraciones de sistemas.

####  **Inventario de Ansible**

```ini
# inventory/hosts.ini
[webservers]
web1.example.com ansible_host=192.168.1.10
web2.example.com ansible_host=192.168.1.11
web3.example.com ansible_host=192.168.1.12

[databases]
db1.example.com ansible_host=192.168.1.20
db2.example.com ansible_host=192.168.1.21

[loadbalancers]
lb1.example.com ansible_host=192.168.1.30

[all:vars]
ansible_user=admin
ansible_ssh_private_key_file=~/.ssh/id_rsa
ansible_python_interpreter=/usr/bin/python3

[webservers:vars]
http_port=80
max_clients=200

[databases:vars]
mysql_port=3306
mysql_datadir=/var/lib/mysql
```

####  **Playbook Básico de Ansible**

```yaml
---
# playbooks/webserver-setup.yml
- name: Configurar servidores web
  hosts: webservers
  become: yes
  vars:
    app_name: "mi-aplicacion"
    app_version: "1.0.0"
    app_port: 8080

  tasks:
    - name: Actualizar caché de paquetes
      apt:
        update_cache: yes
        cache_valid_time: 3600
      when: ansible_os_family == "Debian"

    - name: Instalar paquetes necesarios
      package:
        name:
          - nginx
          - nodejs
          - npm
          - git
          - htop
        state: present

    - name: Crear usuario de aplicación
      user:
        name: "{{ app_name }}"
        system: yes
        shell: /bin/bash
        home: "/opt/{{ app_name }}"
        create_home: yes

    - name: Crear directorios de aplicación
      file:
        path: "{{ item }}"
        state: directory
        owner: "{{ app_name }}"
        group: "{{ app_name }}"
        mode: '0755'
      loop:
        - "/opt/{{ app_name }}/app"
        - "/opt/{{ app_name }}/logs"
        - "/opt/{{ app_name }}/backups"

    - name: Clonar repositorio de aplicación
      git:
        repo: "https://github.com/company/{{ app_name }}.git"
        dest: "/opt/{{ app_name }}/app"
        version: "v{{ app_version }}"
        force: yes
      become_user: "{{ app_name }}"
      notify: restart application

    - name: Instalar dependencias de Node.js
      npm:
        path: "/opt/{{ app_name }}/app"
        production: yes
      become_user: "{{ app_name }}"

    - name: Configurar archivo de entorno
      template:
        src: env.j2
        dest: "/opt/{{ app_name }}/app/.env"
        owner: "{{ app_name }}"
        group: "{{ app_name }}"
        mode: '0600'
      notify: restart application

    - name: Configurar servicio systemd
      template:
        src: systemd-service.j2
        dest: "/etc/systemd/system/{{ app_name }}.service"
        mode: '0644'
      notify:
        - reload systemd
        - restart application

    - name: Configurar nginx
      template:
        src: nginx-site.j2
        dest: "/etc/nginx/sites-available/{{ app_name }}"
        mode: '0644'
      notify: restart nginx

    - name: Habilitar sitio nginx
      file:
        src: "/etc/nginx/sites-available/{{ app_name }}"
        dest: "/etc/nginx/sites-enabled/{{ app_name }}"
        state: link
      notify: restart nginx

    - name: Eliminar sitio default de nginx
      file:
        path: /etc/nginx/sites-enabled/default
        state: absent
      notify: restart nginx

    - name: Configurar firewall
      ufw:
        rule: allow
        port: "{{ item }}"
        proto: tcp
      loop:
        - "22"    # SSH
        - "80"    # HTTP
        - "443"   # HTTPS

    - name: Habilitar firewall
      ufw:
        state: enabled
        policy: deny
        direction: incoming

    - name: Iniciar y habilitar servicios
      systemd:
        name: "{{ item }}"
        state: started
        enabled: yes
      loop:
        - nginx
        - "{{ app_name }}"

    - name: Verificar que la aplicación responde
      uri:
        url: "http://{{ ansible_host }}:{{ http_port }}/health"
        method: GET
        status_code: 200
      retries: 5
      delay: 10

  handlers:
    - name: reload systemd
      systemd:
        daemon_reload: yes

    - name: restart application
      systemd:
        name: "{{ app_name }}"
        state: restarted

    - name: restart nginx
      systemd:
        name: nginx
        state: restarted
```

####  **Templates de Ansible**

```jinja2
{# templates/env.j2 #}
NODE_ENV=production
PORT={{ app_port }}
DB_HOST={{ hostvars[groups['databases'][0]]['ansible_host'] }}
DB_PORT={{ mysql_port }}
DB_NAME={{ app_name }}
DB_USER={{ app_name }}
DB_PASSWORD={{ vault_db_password }}

LOG_LEVEL=info
LOG_FILE=/opt/{{ app_name }}/logs/app.log

SESSION_SECRET={{ vault_session_secret }}
JWT_SECRET={{ vault_jwt_secret }}

# Configuración específica del entorno
{% if environment == 'production' %}
DEBUG=false
CACHE_TTL=3600
{% else %}
DEBUG=true
CACHE_TTL=60
{% endif %}
```

```jinja2
{# templates/systemd-service.j2 #}
[Unit]
Description={{ app_name | title }} Application
After=network.target

[Service]
Type=simple
User={{ app_name }}
Group={{ app_name }}
WorkingDirectory=/opt/{{ app_name }}/app
ExecStart=/usr/bin/node server.js
Restart=always
RestartSec=10
Environment=NODE_ENV=production

# Logging
StandardOutput=journal
StandardError=journal
SyslogIdentifier={{ app_name }}

# Security
NoNewPrivileges=true
PrivateTmp=true
ProtectSystem=strict
ProtectHome=true
ReadWritePaths=/opt/{{ app_name }}/logs

[Install]
WantedBy=multi-user.target
```

```jinja2
{# templates/nginx-site.j2 #}
upstream {{ app_name }}_backend {
{% for host in groups['webservers'] %}
    server {{ hostvars[host]['ansible_host'] }}:{{ app_port }};
{% endfor %}
}

server {
    listen {{ http_port }};
    server_name {{ ansible_host }} {{ inventory_hostname }};

    # Security headers
    add_header X-Frame-Options DENY;
    add_header X-Content-Type-Options nosniff;
    add_header X-XSS-Protection "1; mode=block";

    # Gzip compression
    gzip on;
    gzip_types text/plain text/css application/json application/javascript text/xml application/xml;

    # Static files
    location /static/ {
        alias /opt/{{ app_name }}/app/public/;
        expires 1y;
        add_header Cache-Control "public, immutable";
    }

    # Main application
    location / {
        proxy_pass http://{{ app_name }}_backend;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_cache_bypass $http_upgrade;

        # Timeouts
        proxy_connect_timeout 60s;
        proxy_send_timeout 60s;
        proxy_read_timeout 60s;
    }

    # Health check
    location /health {
        proxy_pass http://{{ app_name }}_backend;
        access_log off;
    }
}
```

####  **Ansible Vault para Secretos**

```bash
# Crear archivo de secretos
ansible-vault create group_vars/all/vault.yml

# Contenido del vault.yml
vault_db_password: "super_secret_password"
vault_session_secret: "session_secret_key"
vault_jwt_secret: "jwt_secret_key"
vault_api_key: "api_key_value"

# Editar vault
ansible-vault edit group_vars/all/vault.yml

# Ejecutar playbook con vault
ansible-playbook -i inventory/hosts.ini playbooks/webserver-setup.yml --ask-vault-pass
```

####  **Playbook de Deployment**

```yaml
---
# playbooks/deploy-application.yml
- name: Deploy aplicación
  hosts: webservers
  become: yes
  vars:
    app_name: "mi-aplicacion"
    deploy_version: "{{ version | default('latest') }}"
    backup_dir: "/opt/{{ app_name }}/backups"

  tasks:
    - name: Verificar que la nueva versión existe
      uri:
        url: "https://github.com/company/{{ app_name }}/releases/tag/v{{ deploy_version }}"
        method: HEAD
        status_code: 200
      delegate_to: localhost
      run_once: true

    - name: Crear backup de la versión actual
      archive:
        path: "/opt/{{ app_name }}/app"
        dest: "{{ backup_dir }}/{{ app_name }}-{{ ansible_date_time.epoch }}.tar.gz"
        owner: "{{ app_name }}"
        group: "{{ app_name }}"

    - name: Parar aplicación
      systemd:
        name: "{{ app_name }}"
        state: stopped

    - name: Actualizar código
      git:
        repo: "https://github.com/company/{{ app_name }}.git"
        dest: "/opt/{{ app_name }}/app"
        version: "v{{ deploy_version }}"
        force: yes
      become_user: "{{ app_name }}"

    - name: Instalar/actualizar dependencias
      npm:
        path: "/opt/{{ app_name }}/app"
        production: yes
      become_user: "{{ app_name }}"

    - name: Ejecutar migraciones de base de datos
      command: npm run migrate
      args:
        chdir: "/opt/{{ app_name }}/app"
      become_user: "{{ app_name }}"
      run_once: true

    - name: Iniciar aplicación
      systemd:
        name: "{{ app_name }}"
        state: started

    - name: Esperar que la aplicación esté lista
      wait_for:
        port: "{{ app_port }}"
        delay: 5
        timeout: 30

    - name: Verificar health check
      uri:
        url: "http://{{ ansible_host }}:{{ http_port }}/health"
        method: GET
        status_code: 200
      retries: 5
      delay: 10

    - name: Limpiar backups antiguos (mantener últimos 5)
      shell: |
        cd {{ backup_dir }}
        ls -t {{ app_name }}-*.tar.gz | tail -n +6 | xargs -r rm --
      become_user: "{{ app_name }}"
```

### 3. Puppet - Configuration Management Declarativo

####  **Introducción a Puppet**

Puppet es una herramienta de configuration management que utiliza un lenguaje declarativo para definir el estado deseado de los sistemas.

```puppet
# manifests/webserver.pp
class webserver {
  # Variables
  $app_name = 'mi-aplicacion'
  $app_user = $app_name
  $app_port = 8080
  $app_dir = "/opt/${app_name}"

  # Paquetes necesarios
  package { ['nginx', 'nodejs', 'npm', 'git']:
    ensure => installed,
  }

  # Usuario de la aplicación
  user { $app_user:
    ensure     => present,
    system     => true,
    shell      => '/bin/bash',
    home       => $app_dir,
    managehome => true,
  }

  # Directorios de la aplicación
  file { [$app_dir, "${app_dir}/app", "${app_dir}/logs", "${app_dir}/backups"]:
    ensure  => directory,
    owner   => $app_user,
    group   => $app_user,
    mode    => '0755',
    require => User[$app_user],
  }

  # Clonar repositorio
  vcsrepo { "${app_dir}/app":
    ensure   => present,
    provider => git,
    source   => "https://github.com/company/${app_name}.git",
    revision => 'main',
    owner    => $app_user,
    group    => $app_user,
    require  => [Package['git'], File["${app_dir}/app"]],
    notify   => Exec['npm-install'],
  }

  # Instalar dependencias npm
  exec { 'npm-install':
    command     => '/usr/bin/npm install --production',
    cwd         => "${app_dir}/app",
    user        => $app_user,
    refreshonly => true,
    require     => Package['npm'],
    notify      => Service[$app_name],
  }

  # Configuración de la aplicación
  file { "${app_dir}/app/.env":
    ensure  => file,
    content => template('webserver/env.erb'),
    owner   => $app_user,
    group   => $app_user,
    mode    => '0600',
    require => File["${app_dir}/app"],
    notify  => Service[$app_name],
  }

  # Servicio systemd
  file { "/etc/systemd/system/${app_name}.service":
    ensure  => file,
    content => template('webserver/systemd.erb'),
    mode    => '0644',
    notify  => [Exec['systemd-reload'], Service[$app_name]],
  }

  exec { 'systemd-reload':
    command     => '/bin/systemctl daemon-reload',
    refreshonly => true,
  }

  # Configuración nginx
  file { "/etc/nginx/sites-available/${app_name}":
    ensure  => file,
    content => template('webserver/nginx.erb'),
    mode    => '0644',
    require => Package['nginx'],
    notify  => Service['nginx'],
  }

  file { "/etc/nginx/sites-enabled/${app_name}":
    ensure  => link,
    target  => "/etc/nginx/sites-available/${app_name}",
    require => File["/etc/nginx/sites-available/${app_name}"],
    notify  => Service['nginx'],
  }

  # Eliminar sitio default
  file { '/etc/nginx/sites-enabled/default':
    ensure => absent,
    notify => Service['nginx'],
  }

  # Servicios
  service { $app_name:
    ensure  => running,
    enable  => true,
    require => [File["/etc/systemd/system/${app_name}.service"], Exec['npm-install']],
  }

  service { 'nginx':
    ensure  => running,
    enable  => true,
    require => Package['nginx'],
  }

  # Firewall
  firewall { '100 allow ssh':
    dport  => 22,
    proto  => tcp,
    action => accept,
  }

  firewall { '200 allow http':
    dport  => 80,
    proto  => tcp,
    action => accept,
  }

  firewall { '300 allow https':
    dport  => 443,
    proto  => tcp,
    action => accept,
  }
}

# Aplicar la clase
node 'webserver.example.com' {
  include webserver
}
```

### 4. Chef - Infrastructure as Code

####  **Introducción a Chef**

Chef utiliza "recetas" (recipes) escritas en Ruby para definir configuraciones de sistema.

```ruby
# recipes/webserver.rb
# Variables
app_name = 'mi-aplicacion'
app_user = app_name
app_port = 8080
app_dir = "/opt/#{app_name}"

# Actualizar paquetes
execute 'apt-update' do
  command 'apt-get update'
  action :run
end

# Instalar paquetes
%w[nginx nodejs npm git].each do |pkg|
  package pkg do
    action :install
  end
end

# Crear usuario
user app_user do
  system true
  shell '/bin/bash'
  home app_dir
  manage_home true
  action :create
end

# Crear directorios
[app_dir, "#{app_dir}/app", "#{app_dir}/logs", "#{app_dir}/backups"].each do |dir|
  directory dir do
    owner app_user
    group app_user
    mode '0755'
    recursive true
    action :create
  end
end

# Clonar repositorio
git "#{app_dir}/app" do
  repository "https://github.com/company/#{app_name}.git"
  revision 'main'
  user app_user
  group app_user
  action :sync
  notifies :run, 'execute[npm-install]', :immediately
end

# Instalar dependencias npm
execute 'npm-install' do
  command 'npm install --production'
  cwd "#{app_dir}/app"
  user app_user
  action :nothing
end

# Configuración de aplicación
template "#{app_dir}/app/.env" do
  source 'env.erb'
  owner app_user
  group app_user
  mode '0600'
  variables(
    app_port: app_port,
    app_name: app_name
  )
  notifies :restart, "service[#{app_name}]"
end

# Servicio systemd
template "/etc/systemd/system/#{app_name}.service" do
  source 'systemd.erb'
  mode '0644'
  variables(
    app_name: app_name,
    app_user: app_user,
    app_dir: app_dir
  )
  notifies :run, 'execute[systemd-reload]', :immediately
  notifies :restart, "service[#{app_name}]"
end

execute 'systemd-reload' do
  command 'systemctl daemon-reload'
  action :nothing
end

# Configuración nginx
template "/etc/nginx/sites-available/#{app_name}" do
  source 'nginx.erb'
  mode '0644'
  variables(
    app_name: app_name,
    app_port: app_port
  )
  notifies :restart, 'service[nginx]'
end

link "/etc/nginx/sites-enabled/#{app_name}" do
  to "/etc/nginx/sites-available/#{app_name}"
  notifies :restart, 'service[nginx]'
end

file '/etc/nginx/sites-enabled/default' do
  action :delete
  notifies :restart, 'service[nginx]'
end

# Iniciar servicios
service app_name do
  action [:enable, :start]
end

service 'nginx' do
  action [:enable, :start]
end

# Configurar firewall
execute 'configure-firewall' do
  command <<-EOF
    ufw allow 22/tcp
    ufw allow 80/tcp
    ufw allow 443/tcp
    ufw --force enable
  EOF
  action :run
end
```

##  Laboratorios Prácticos

### Lab 1: Automatización con Make

**Objetivo**: Crear un Makefile completo para un proyecto web.

1. **Crear estructura del proyecto**:

```bash
mkdir -p myproject/{src,tests,docs,scripts}
cd myproject
```

2. **Crear Makefile**:

```makefile
.PHONY: help install test build deploy

help:
 @echo "Comandos disponibles:"
 @echo "  install  - Instalar dependencias"
 @echo "  test     - Ejecutar tests"
 @echo "  build    - Construir aplicación"
 @echo "  deploy   - Desplegar aplicación"

install:
 npm install

test: install
 npm test

build: test
 npm run build

deploy: build
 ./scripts/deploy.sh
```

3. **Probar automatización**:

```bash
make help
make install
make build
```

### Lab 2: Configuración con Ansible

**Objetivo**: Configurar un servidor web usando Ansible.

1. **Crear inventario**:

```ini
[webservers]
localhost ansible_connection=local
```

2. **Crear playbook simple**:

```yaml
---
- hosts: webservers
  become: yes
  tasks:
    - name: Instalar nginx
      package:
        name: nginx
        state: present

    - name: Iniciar nginx
      service:
        name: nginx
        state: started
        enabled: yes
```

3. **Ejecutar playbook**:

```bash
ansible-playbook -i inventory playbook.yml
```

### Lab 3: Comparación de Herramientas

**Objetivo**: Implementar la misma configuración con diferentes herramientas.

Tarea: Configurar un servidor web con nginx usando:

1. Script bash
2. Makefile
3. Ansible
4. Puppet o Chef

Comparar ventajas y desventajas de cada enfoque.

##  Ejercicios Prácticos

### Ejercicio 1: Pipeline de CI/CD con Make

Crear un Makefile que implemente un pipeline completo:

- Instalación de dependencias
- Linting y formateo de código
- Tests unitarios e integración
- Build de la aplicación
- Creación de paquetes
- Deployment a múltiples entornos

### Ejercicio 2: Infraestructura con Ansible

Desarrollar playbooks de Ansible para:

- Configurar un cluster de aplicación web
- Configurar base de datos con replicación
- Configurar load balancer
- Implementar monitoreo básico

### Ejercicio 3: Automatización Híbrida

Crear una solución que combine:

- Make para builds locales
- Ansible para configuración de infraestructura
- Scripts personalizados para tareas específicas

##  Troubleshooting

### Problemas Comunes con Make

1. **Tabs vs Espacios**:

```makefile
# INCORRECTO (espacios)
target:
    echo "Hola"

# CORRECTO (tab)
target:
 echo "Hola"
```

2. **Variables no definidas**:

```makefile
# Definir variable con valor por defecto
VERSION ?= latest
```

### Problemas Comunes con Ansible

1. **Permisos SSH**:

```bash
# Verificar conexión
ansible all -i inventory -m ping
```

2. **Sudo sin contraseña**:

```bash
# Configurar en /etc/sudoers
user ALL=(ALL) NOPASSWD:ALL
```

##  Recursos Adicionales

- [GNU Make Manual](https://www.gnu.org/software/make/manual/)
- [Ansible Documentation](https://docs.ansible.com/)
- [Puppet Documentation](https://puppet.com/docs/)
- [Chef Documentation](https://docs.chef.io/)

##  Checklist de Completado

- [ ] Creado Makefile básico para automatización
- [ ] Implementado Makefile avanzado para microservicios
- [ ] Configurado inventario de Ansible
- [ ] Desarrollado playbook de configuración
- [ ] Implementado deployment con Ansible
- [ ] Usado Ansible Vault para secretos
- [ ] Experimentado con Puppet o Chef
- [ ] Comparado diferentes herramientas
- [ ] Completado laboratorios prácticos

---

**Excelente!** Ahora dominas las herramientas fundamentales de automatización DevOps. Continúa con el siguiente módulo para aprender sobre CI/CD con Jenkins y plataformas modernas.
