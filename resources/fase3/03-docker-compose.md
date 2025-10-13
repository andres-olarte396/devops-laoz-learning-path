# Docker Compose: Aplicaciones Multi-Contenedor

## 🎯 Objetivos de Aprendizaje

Al finalizar este tema serás capaz de:
- Comprender qué es Docker Compose y cuándo usarlo
- Escribir archivos docker-compose.yml efectivos
- Gestionar aplicaciones multi-contenedor
- Configurar redes y volúmenes con Compose
- Implementar entornos de desarrollo completos

## 📚 ¿Qué es Docker Compose?

**Docker Compose** es una herramienta para definir y ejecutar aplicaciones Docker multi-contenedor usando un archivo YAML declarativo.

### 🔍 Ventajas de Docker Compose

1. **Simplicidad**: Un solo archivo para toda la aplicación
2. **Reproducibilidad**: Entornos idénticos en cualquier lugar
3. **Gestión integrada**: Redes, volúmenes y servicios juntos
4. **Desarrollo ágil**: Levantar/tumbar entornos rápidamente
5. **Escalabilidad**: Fácil escalado de servicios

### 🆚 Docker Run vs Docker Compose

```bash
# ❌ Con docker run (complejo y propenso a errores)
docker network create app-network
docker volume create db-data
docker run -d --name database --network app-network -v db-data:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=secret mysql:8.0
docker run -d --name webapp --network app-network -p 8080:80 -e DB_HOST=database nginx

# ✅ Con docker-compose (simple y declarativo)
docker-compose up -d
```

## 📄 Estructura del archivo docker-compose.yml

### Versión y Estructura Básica

```yaml
version: '3.8'

services:
  web:
    # Configuración del servicio web
  
  database:
    # Configuración de la base de datos

volumes:
  # Definición de volúmenes

networks:
  # Definición de redes
```

### Ejemplo Completo: Aplicación Web + Base de Datos

```yaml
version: '3.8'

services:
  # Servicio de base de datos
  database:
    image: mysql:8.0
    container_name: app-mysql
    restart: unless-stopped
    environment:
      MYSQL_ROOT_PASSWORD: rootpassword
      MYSQL_DATABASE: myapp
      MYSQL_USER: appuser
      MYSQL_PASSWORD: apppassword
    volumes:
      - db-data:/var/lib/mysql
      - ./init.sql:/docker-entrypoint-initdb.d/init.sql:ro
    networks:
      - app-network
    healthcheck:
      test: ["CMD", "mysqladmin", "ping", "-h", "localhost"]
      timeout: 20s
      retries: 10

  # Servicio de aplicación web
  webapp:
    build: 
      context: .
      dockerfile: Dockerfile
    container_name: app-web
    restart: unless-stopped
    ports:
      - "8080:80"
    environment:
      - DB_HOST=database
      - DB_NAME=myapp
      - DB_USER=appuser
      - DB_PASSWORD=apppassword
    volumes:
      - ./logs:/var/log/app
      - ./uploads:/app/uploads
    networks:
      - app-network
    depends_on:
      database:
        condition: service_healthy

  # Servicio de cache Redis
  redis:
    image: redis:7-alpine
    container_name: app-redis
    restart: unless-stopped
    command: redis-server --appendonly yes
    volumes:
      - redis-data:/data
    networks:
      - app-network

volumes:
  db-data:
    driver: local
  redis-data:
    driver: local

networks:
  app-network:
    driver: bridge
```

## 🔧 Configuración de Servicios

### Usando Imágenes Existentes

```yaml
services:
  nginx:
    image: nginx:alpine
    ports:
      - "80:80"
    
  postgres:
    image: postgres:14
    environment:
      POSTGRES_DB: myapp
      POSTGRES_USER: user
      POSTGRES_PASSWORD: password
```

### Construyendo desde Dockerfile

```yaml
services:
  webapp:
    build:
      context: .
      dockerfile: Dockerfile
      args:
        - BUILD_ENV=production
        - VERSION=1.0.0
    ports:
      - "8080:8080"
```

### Build Avanzado

```yaml
services:
  frontend:
    build:
      context: ./frontend
      dockerfile: Dockerfile.prod
      target: production
      cache_from:
        - node:16-alpine
      args:
        NODE_ENV: production
```

## 🌐 Redes en Docker Compose

### Red por Defecto

```yaml
# Docker Compose crea automáticamente una red
services:
  web:
    image: nginx
  api:
    image: node:16
  # web puede comunicarse con api usando el nombre del servicio
```

### Redes Personalizadas

```yaml
version: '3.8'

services:
  frontend:
    image: nginx
    networks:
      - frontend-network
      
  backend:
    image: node:16
    networks:
      - frontend-network
      - backend-network
      
  database:
    image: postgres:14
    networks:
      - backend-network

networks:
  frontend-network:
    driver: bridge
  backend-network:
    driver: bridge
    internal: true  # Red interna, sin acceso a internet
```

### Red Externa (Existente)

```yaml
services:
  app:
    image: myapp
    networks:
      - existing-network

networks:
  existing-network:
    external: true
```

## 💾 Volúmenes en Docker Compose

### Tipos de Volúmenes

```yaml
services:
  webapp:
    image: nginx
    volumes:
      # Volumen con nombre
      - html-data:/usr/share/nginx/html
      
      # Bind mount (directorio host)
      - ./html:/usr/share/nginx/html
      
      # Bind mount con opciones
      - ./config:/etc/nginx:ro
      
      # Volumen temporal
      - /tmp
      
      # Volumen anónimo
      - /var/log

volumes:
  html-data:
    driver: local
    driver_opts:
      type: none
      o: bind
      device: /host/path/html
```

### Volúmenes Externos

```yaml
services:
  database:
    image: postgres:14
    volumes:
      - postgres-data:/var/lib/postgresql/data

volumes:
  postgres-data:
    external: true
```

## 🔒 Variables de Entorno

### Métodos de Configuración

```yaml
services:
  webapp:
    image: myapp
    environment:
      # Directamente en el archivo
      - NODE_ENV=production
      - PORT=3000
      
      # Desde variables del host
      - SECRET_KEY=${SECRET_KEY}
      
      # Con valor por defecto
      - DEBUG=${DEBUG:-false}
    
    # Desde archivo .env
    env_file:
      - .env
      - .env.production
```

### Archivo .env

```bash
# .env
NODE_ENV=development
DATABASE_URL=postgresql://user:pass@localhost/db
SECRET_KEY=mi-secreto-super-seguro
DEBUG=true
REDIS_URL=redis://localhost:6379
```

### Variables por Servicio

```yaml
services:
  web:
    image: nginx
    env_file: ./config/web.env
    
  api:
    image: node:16
    env_file: ./config/api.env
```

## 🔄 Dependencias entre Servicios

### depends_on Simple

```yaml
services:
  webapp:
    image: myapp
    depends_on:
      - database
      - redis
      
  database:
    image: postgres:14
    
  redis:
    image: redis:alpine
```

### depends_on con Condiciones

```yaml
services:
  webapp:
    image: myapp
    depends_on:
      database:
        condition: service_healthy
      redis:
        condition: service_started
        
  database:
    image: postgres:14
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U postgres"]
      interval: 10s
      timeout: 5s
      retries: 5
```

## 📊 Ejemplos Prácticos Completos

### Stack LAMP (Linux, Apache, MySQL, PHP)

```yaml
version: '3.8'

services:
  # Servidor web Apache + PHP
  webserver:
    build:
      context: .
      dockerfile: Dockerfile.php
    container_name: lamp-web
    restart: unless-stopped
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./src:/var/www/html
      - ./config/apache:/etc/apache2/sites-available
    environment:
      - APACHE_DOCUMENT_ROOT=/var/www/html
    networks:
      - lamp-network
    depends_on:
      database:
        condition: service_healthy

  # Base de datos MySQL
  database:
    image: mysql:8.0
    container_name: lamp-mysql
    restart: unless-stopped
    environment:
      MYSQL_ROOT_PASSWORD: root123
      MYSQL_DATABASE: lampdb
      MYSQL_USER: lampuser
      MYSQL_PASSWORD: lamppass
    volumes:
      - mysql-data:/var/lib/mysql
      - ./sql:/docker-entrypoint-initdb.d
    networks:
      - lamp-network
    healthcheck:
      test: ["CMD", "mysqladmin", "ping", "-h", "localhost"]
      interval: 10s
      timeout: 5s
      retries: 3

  # phpMyAdmin para gestión de BD
  phpmyadmin:
    image: phpmyadmin/phpmyadmin:latest
    container_name: lamp-phpmyadmin
    restart: unless-stopped
    ports:
      - "8080:80"
    environment:
      PMA_HOST: database
      PMA_PORT: 3306
      PMA_USER: lampuser
      PMA_PASSWORD: lamppass
    networks:
      - lamp-network
    depends_on:
      - database

volumes:
  mysql-data:

networks:
  lamp-network:
    driver: bridge
```

### Stack MEAN (MongoDB, Express, Angular, Node.js)

```yaml
version: '3.8'

services:
  # Base de datos MongoDB
  mongodb:
    image: mongo:6
    container_name: mean-mongo
    restart: unless-stopped
    environment:
      MONGO_INITDB_ROOT_USERNAME: root
      MONGO_INITDB_ROOT_PASSWORD: rootpass
      MONGO_INITDB_DATABASE: meanapp
    volumes:
      - mongo-data:/data/db
      - ./mongo-init:/docker-entrypoint-initdb.d
    networks:
      - mean-network
    ports:
      - "27017:27017"

  # API Backend (Node.js + Express)
  backend:
    build:
      context: ./backend
      dockerfile: Dockerfile
    container_name: mean-backend
    restart: unless-stopped
    environment:
      - NODE_ENV=development
      - MONGODB_URI=mongodb://root:rootpass@mongodb:27017/meanapp?authSource=admin
      - JWT_SECRET=mi-jwt-secreto
    volumes:
      - ./backend:/app
      - /app/node_modules
    networks:
      - mean-network
    ports:
      - "3000:3000"
    depends_on:
      - mongodb

  # Frontend (Angular)
  frontend:
    build:
      context: ./frontend
      dockerfile: Dockerfile.dev
    container_name: mean-frontend
    restart: unless-stopped
    environment:
      - API_URL=http://localhost:3000/api
    volumes:
      - ./frontend:/app
      - /app/node_modules
    networks:
      - mean-network
    ports:
      - "4200:4200"
    depends_on:
      - backend

  # Servidor web Nginx (Proxy reverso)
  nginx:
    image: nginx:alpine
    container_name: mean-nginx
    restart: unless-stopped
    ports:
      - "80:80"
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf:ro
    networks:
      - mean-network
    depends_on:
      - frontend
      - backend

volumes:
  mongo-data:

networks:
  mean-network:
    driver: bridge
```

### Stack de Microservicios

```yaml
version: '3.8'

services:
  # API Gateway
  gateway:
    image: nginx:alpine
    container_name: api-gateway
    ports:
      - "80:80"
    volumes:
      - ./gateway/nginx.conf:/etc/nginx/nginx.conf:ro
    networks:
      - microservices
    depends_on:
      - user-service
      - product-service
      - order-service

  # Servicio de usuarios
  user-service:
    build: ./services/user
    container_name: user-service
    environment:
      - DB_HOST=user-db
      - DB_NAME=users
      - REDIS_HOST=redis
    networks:
      - microservices
    depends_on:
      - user-db
      - redis

  # Base de datos de usuarios
  user-db:
    image: postgres:14
    container_name: user-postgres
    environment:
      POSTGRES_DB: users
      POSTGRES_USER: userservice
      POSTGRES_PASSWORD: userpass
    volumes:
      - user-db-data:/var/lib/postgresql/data
    networks:
      - microservices

  # Servicio de productos
  product-service:
    build: ./services/product
    container_name: product-service
    environment:
      - DB_HOST=product-db
      - DB_NAME=products
    networks:
      - microservices
    depends_on:
      - product-db

  # Base de datos de productos
  product-db:
    image: mysql:8.0
    container_name: product-mysql
    environment:
      MYSQL_DATABASE: products
      MYSQL_USER: productservice
      MYSQL_PASSWORD: productpass
      MYSQL_ROOT_PASSWORD: rootpass
    volumes:
      - product-db-data:/var/lib/mysql
    networks:
      - microservices

  # Servicio de pedidos
  order-service:
    build: ./services/order
    container_name: order-service
    environment:
      - MONGO_URI=mongodb://order-db:27017/orders
      - USER_SERVICE_URL=http://user-service:3000
      - PRODUCT_SERVICE_URL=http://product-service:3000
    networks:
      - microservices
    depends_on:
      - order-db

  # Base de datos de pedidos
  order-db:
    image: mongo:6
    container_name: order-mongo
    volumes:
      - order-db-data:/data/db
    networks:
      - microservices

  # Cache Redis compartido
  redis:
    image: redis:7-alpine
    container_name: shared-redis
    command: redis-server --appendonly yes
    volumes:
      - redis-data:/data
    networks:
      - microservices

  # Monitoreo con Prometheus
  prometheus:
    image: prom/prometheus:latest
    container_name: prometheus
    ports:
      - "9090:9090"
    volumes:
      - ./monitoring/prometheus.yml:/etc/prometheus/prometheus.yml:ro
      - prometheus-data:/prometheus
    networks:
      - microservices

  # Dashboard con Grafana
  grafana:
    image: grafana/grafana:latest
    container_name: grafana
    ports:
      - "3000:3000"
    environment:
      - GF_SECURITY_ADMIN_PASSWORD=admin
    volumes:
      - grafana-data:/var/lib/grafana
    networks:
      - microservices

volumes:
  user-db-data:
  product-db-data:
  order-db-data:
  redis-data:
  prometheus-data:
  grafana-data:

networks:
  microservices:
    driver: bridge
```

## 🎮 Comandos de Docker Compose

### Comandos Básicos

```bash
# Levantar servicios
docker-compose up

# En segundo plano
docker-compose up -d

# Construir imágenes antes de levantar
docker-compose up --build

# Levantar servicios específicos
docker-compose up web database

# Parar servicios
docker-compose stop

# Parar y eliminar contenedores
docker-compose down

# Parar, eliminar contenedores y volúmenes
docker-compose down -v

# Parar, eliminar todo incluyendo imágenes
docker-compose down --rmi all
```

### Gestión de Servicios

```bash
# Ver estado de servicios
docker-compose ps

# Ver logs de todos los servicios
docker-compose logs

# Logs de un servicio específico
docker-compose logs webapp

# Seguir logs en tiempo real
docker-compose logs -f webapp

# Ejecutar comando en servicio
docker-compose exec webapp bash

# Escalar servicios
docker-compose up --scale webapp=3

# Reiniciar servicios
docker-compose restart webapp
```

### Build y Configuración

```bash
# Construir/reconstruir servicios
docker-compose build

# Construir sin cache
docker-compose build --no-cache

# Validar archivo docker-compose.yml
docker-compose config

# Ver configuración procesada
docker-compose config --services

# Usar archivo específico
docker-compose -f docker-compose.prod.yml up
```

## 🔧 Archivos de Configuración Múltiples

### Desarrollo y Producción

```yaml
# docker-compose.yml (base)
version: '3.8'

services:
  webapp:
    build: .
    environment:
      - NODE_ENV=development
    
  database:
    image: postgres:14
```

```yaml
# docker-compose.override.yml (desarrollo)
version: '3.8'

services:
  webapp:
    volumes:
      - .:/app
    ports:
      - "3000:3000"
    command: npm run dev
```

```yaml
# docker-compose.prod.yml (producción)
version: '3.8'

services:
  webapp:
    environment:
      - NODE_ENV=production
    restart: unless-stopped
    
  nginx:
    image: nginx:alpine
    ports:
      - "80:80"
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf:ro
```

```bash
# Usar configuración específica
docker-compose -f docker-compose.yml -f docker-compose.prod.yml up -d
```

## 🛠️ Ejercicio Práctico Completo

### WordPress + MySQL + Redis

```yaml
version: '3.8'

services:
  # Base de datos MySQL
  mysql:
    image: mysql:8.0
    container_name: wp-mysql
    restart: unless-stopped
    environment:
      MYSQL_ROOT_PASSWORD: rootpassword
      MYSQL_DATABASE: wordpress
      MYSQL_USER: wpuser
      MYSQL_PASSWORD: wppassword
    volumes:
      - mysql-data:/var/lib/mysql
    networks:
      - wordpress-network

  # WordPress
  wordpress:
    image: wordpress:latest
    container_name: wp-site
    restart: unless-stopped
    ports:
      - "8080:80"
    environment:
      WORDPRESS_DB_HOST: mysql:3306
      WORDPRESS_DB_USER: wpuser
      WORDPRESS_DB_PASSWORD: wppassword
      WORDPRESS_DB_NAME: wordpress
      # Redis cache
      WORDPRESS_CONFIG_EXTRA: |
        define('WP_REDIS_HOST', 'redis');
        define('WP_REDIS_PORT', 6379);
    volumes:
      - wordpress-data:/var/www/html
      - ./themes:/var/www/html/wp-content/themes
      - ./plugins:/var/www/html/wp-content/plugins
    networks:
      - wordpress-network
    depends_on:
      - mysql
      - redis

  # Cache Redis
  redis:
    image: redis:7-alpine
    container_name: wp-redis
    restart: unless-stopped
    command: redis-server --maxmemory 128mb --maxmemory-policy allkeys-lru
    networks:
      - wordpress-network

  # Servidor web Nginx
  nginx:
    image: nginx:alpine
    container_name: wp-nginx
    restart: unless-stopped
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./nginx/nginx.conf:/etc/nginx/nginx.conf:ro
      - ./nginx/ssl:/etc/nginx/ssl:ro
      - wordpress-data:/var/www/html:ro
    networks:
      - wordpress-network
    depends_on:
      - wordpress

  # Backup automático
  backup:
    image: mysql:8.0
    container_name: wp-backup
    restart: unless-stopped
    environment:
      MYSQL_HOST: mysql
      MYSQL_USER: wpuser
      MYSQL_PASSWORD: wppassword
      MYSQL_DATABASE: wordpress
    volumes:
      - ./backups:/backups
      - ./scripts/backup.sh:/backup.sh:ro
    networks:
      - wordpress-network
    command: >
      sh -c "
        while true; do
          mysqldump -h mysql -u wpuser -pwppassword wordpress > /backups/wordpress_$$(date +%Y%m%d_%H%M%S).sql
          sleep 86400
        done
      "
    depends_on:
      - mysql

volumes:
  mysql-data:
  wordpress-data:

networks:
  wordpress-network:
    driver: bridge
```

## 📝 Mejores Prácticas

1. **Usa versiones específicas** de imágenes
2. **Implementa health checks** para servicios críticos
3. **Configura restart policies** apropiadas
4. **Usa redes personalizadas** para aislamiento
5. **Organiza volúmenes** para persistencia adecuada
6. **Documenta variables de entorno** requeridas
7. **Separa configuraciones** por entorno
8. **Implementa monitoreo** y logging
9. **Usa .dockerignore** para builds eficientes
10. **Valida archivos** con `docker-compose config`

## 🎯 Resumen

Has aprendido:
- [x] Conceptos fundamentales de Docker Compose
- [x] Sintaxis y estructura de archivos YAML
- [x] Configuración de servicios, redes y volúmenes
- [x] Gestión de dependencias entre servicios
- [x] Implementación de stacks completos
- [x] Comandos esenciales de Compose

## 📚 Recursos Adicionales

- [Docker Compose Documentation](https://docs.docker.com/compose/)
- [Compose File Reference](https://docs.docker.com/compose/compose-file/)
- [Awesome Docker Compose](https://github.com/docker/awesome-compose)
- [Docker Compose Examples](https://github.com/docker/awesome-compose)

---

**Próximo tema:** [Optimización de imágenes](./04-optimizacion-imagenes.md)