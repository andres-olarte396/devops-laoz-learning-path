# Práctica: Aplicación Multi-contenedor

## 🎯 Objetivos de la Práctica

Al completar esta práctica serás capaz de:

- Diseñar una arquitectura de microservicios con múltiples contenedores
- Implementar comunicación entre servicios
- Gestionar bases de datos y cache distribuidos
- Configurar balanceadores de carga y service discovery
- Implementar monitoreo y logging centralizado

## 📋 Descripción del Proyecto

Crearemos una **aplicación de E-commerce** completa con arquitectura de microservicios que incluye:

- **Frontend**: React.js (SPA)
- **API Gateway**: NGINX
- **Servicios Backend**:
  - Servicio de Usuarios (Node.js)
  - Servicio de Productos (Python/FastAPI)
  - Servicio de Pedidos (Java/Spring Boot)
  - Servicio de Pagos (Go)
- **Bases de Datos**:
  - PostgreSQL (usuarios, productos)
  - MongoDB (pedidos, logs)
  - Redis (cache, sesiones)
- **Infraestructura**:
  - RabbitMQ (mensajería)
  - Elasticsearch + Kibana (logging)
  - Prometheus + Grafana (métricas)

## 🏗️ Arquitectura del Sistema

```
┌─────────────────────────────────────────────────────────────┐
│                        Load Balancer                        │
│                          (NGINX)                           │
└──────────────────────┬──────────────────────────────────────┘
                       │
┌─────────────────────────────────────────────────────────────┐
│                     API Gateway                            │
│                       (NGINX)                              │
└────┬────────────┬────────────┬────────────┬─────────────────┘
     │            │            │            │
┌────▼────┐  ┌───▼────┐  ┌────▼────┐  ┌───▼────┐
│Frontend │  │ Users  │  │Products │  │ Orders │
│(React)  │  │(Node.js│  │(Python) │  │(Java)  │
└─────────┘  └────────┘  └─────────┘  └────────┘
                  │           │           │
             ┌────▼────┐ ┌───▼────┐ ┌────▼────┐
             │PostgreSQL│ │PostgreSQL│ │MongoDB │
             └─────────┘ └────────┘ └─────────┘
                       │
                  ┌────▼────┐
                  │  Redis  │
                  └─────────┘
```

## 📂 Estructura del Proyecto

```
ecommerce-microservices/
├── docker-compose.yml
├── docker-compose.override.yml
├── .env
├── nginx/
│   ├── Dockerfile
│   ├── nginx.conf
│   └── api-gateway.conf
├── frontend/
│   ├── Dockerfile
│   ├── package.json
│   ├── src/
│   └── public/
├── services/
│   ├── users-service/
│   │   ├── Dockerfile
│   │   ├── package.json
│   │   └── src/
│   ├── products-service/
│   │   ├── Dockerfile
│   │   ├── requirements.txt
│   │   └── app/
│   ├── orders-service/
│   │   ├── Dockerfile
│   │   ├── pom.xml
│   │   └── src/
│   └── payments-service/
│       ├── Dockerfile
│       ├── go.mod
│       └── main.go
├── databases/
│   ├── postgres/
│   │   └── init/
│   └── mongodb/
│       └── init/
├── monitoring/
│   ├── prometheus/
│   │   └── prometheus.yml
│   ├── grafana/
│   │   └── dashboards/
│   └── elasticsearch/
└── scripts/
    ├── init-db.sh
    ├── seed-data.sh
    └── health-check.sh
```

## 🐳 Docker Compose Principal

### docker-compose.yml

```yaml
version: '3.8'

services:
  # ================================
  # Load Balancer & API Gateway
  # ================================
  nginx-lb:
    build:
      context: ./nginx
      dockerfile: Dockerfile
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./nginx/nginx.conf:/etc/nginx/nginx.conf
      - ./nginx/api-gateway.conf:/etc/nginx/conf.d/default.conf
      - ./ssl:/etc/nginx/ssl
    depends_on:
      - frontend
      - users-service
      - products-service
      - orders-service
    networks:
      - frontend-network
      - backend-network
    restart: unless-stopped

  # ================================
  # Frontend
  # ================================
  frontend:
    build:
      context: ./frontend
      dockerfile: Dockerfile
      target: production
    expose:
      - "3000"
    environment:
      - REACT_APP_API_URL=http://nginx-lb/api
      - REACT_APP_ENVIRONMENT=production
    networks:
      - frontend-network
    restart: unless-stopped

  # ================================
  # Microservices
  # ================================
  users-service:
    build:
      context: ./services/users-service
      dockerfile: Dockerfile
    expose:
      - "3001"
    environment:
      - NODE_ENV=production
      - DATABASE_URL=postgresql://postgres:${POSTGRES_PASSWORD}@postgres:5432/users_db
      - REDIS_URL=redis://redis:6379/0
      - JWT_SECRET=${JWT_SECRET}
      - RABBITMQ_URL=amqp://guest:guest@rabbitmq:5672
    depends_on:
      postgres:
        condition: service_healthy
      redis:
        condition: service_healthy
      rabbitmq:
        condition: service_healthy
    networks:
      - backend-network
    restart: unless-stopped
    deploy:
      replicas: 2
      resources:
        limits:
          memory: 512M
          cpus: '0.5'

  products-service:
    build:
      context: ./services/products-service
      dockerfile: Dockerfile
    expose:
      - "8000"
    environment:
      - DATABASE_URL=postgresql://postgres:${POSTGRES_PASSWORD}@postgres:5432/products_db
      - REDIS_URL=redis://redis:6379/1
      - ELASTICSEARCH_URL=http://elasticsearch:9200
    depends_on:
      postgres:
        condition: service_healthy
      redis:
        condition: service_healthy
      elasticsearch:
        condition: service_healthy
    networks:
      - backend-network
    restart: unless-stopped
    deploy:
      replicas: 2
      resources:
        limits:
          memory: 1G
          cpus: '0.7'

  orders-service:
    build:
      context: ./services/orders-service
      dockerfile: Dockerfile
    expose:
      - "8080"
    environment:
      - SPRING_PROFILES_ACTIVE=production
      - MONGODB_URI=mongodb://mongodb:27017/orders_db
      - REDIS_URL=redis://redis:6379/2
      - RABBITMQ_URL=amqp://guest:guest@rabbitmq:5672
      - USERS_SERVICE_URL=http://users-service:3001
      - PRODUCTS_SERVICE_URL=http://products-service:8000
      - PAYMENTS_SERVICE_URL=http://payments-service:8081
    depends_on:
      mongodb:
        condition: service_healthy
      redis:
        condition: service_healthy
      rabbitmq:
        condition: service_healthy
    networks:
      - backend-network
    restart: unless-stopped
    deploy:
      replicas: 2
      resources:
        limits:
          memory: 1G
          cpus: '0.8'

  payments-service:
    build:
      context: ./services/payments-service
      dockerfile: Dockerfile
    expose:
      - "8081"
    environment:
      - REDIS_URL=redis://redis:6379/3
      - RABBITMQ_URL=amqp://guest:guest@rabbitmq:5672
      - STRIPE_SECRET_KEY=${STRIPE_SECRET_KEY}
      - PAYPAL_CLIENT_ID=${PAYPAL_CLIENT_ID}
      - PAYPAL_CLIENT_SECRET=${PAYPAL_CLIENT_SECRET}
    depends_on:
      redis:
        condition: service_healthy
      rabbitmq:
        condition: service_healthy
    networks:
      - backend-network
    restart: unless-stopped
    deploy:
      replicas: 1
      resources:
        limits:
          memory: 256M
          cpus: '0.3'

  # ================================
  # Databases
  # ================================
  postgres:
    image: postgres:15-alpine
    environment:
      - POSTGRES_USER=postgres
      - POSTGRES_PASSWORD=${POSTGRES_PASSWORD}
      - POSTGRES_MULTIPLE_DATABASES=users_db,products_db
    volumes:
      - postgres-data:/var/lib/postgresql/data
      - ./databases/postgres/init:/docker-entrypoint-initdb.d
    ports:
      - "5432:5432"
    networks:
      - backend-network
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U postgres"]
      interval: 10s
      timeout: 5s
      retries: 5
    restart: unless-stopped

  mongodb:
    image: mongo:6.0
    environment:
      - MONGO_INITDB_ROOT_USERNAME=admin
      - MONGO_INITDB_ROOT_PASSWORD=${MONGO_PASSWORD}
      - MONGO_INITDB_DATABASE=orders_db
    volumes:
      - mongodb-data:/data/db
      - ./databases/mongodb/init:/docker-entrypoint-initdb.d
    ports:
      - "27017:27017"
    networks:
      - backend-network
    healthcheck:
      test: ["CMD", "mongo", "--eval", "db.adminCommand('ping')"]
      interval: 10s
      timeout: 5s
      retries: 5
    restart: unless-stopped

  redis:
    image: redis:7-alpine
    command: redis-server --appendonly yes --maxmemory 256mb --maxmemory-policy allkeys-lru
    volumes:
      - redis-data:/data
    ports:
      - "6379:6379"
    networks:
      - backend-network
    healthcheck:
      test: ["CMD", "redis-cli", "ping"]
      interval: 10s
      timeout: 5s
      retries: 5
    restart: unless-stopped

  # ================================
  # Message Queue
  # ================================
  rabbitmq:
    image: rabbitmq:3.11-management-alpine
    environment:
      - RABBITMQ_DEFAULT_USER=admin
      - RABBITMQ_DEFAULT_PASS=${RABBITMQ_PASSWORD}
    volumes:
      - rabbitmq-data:/var/lib/rabbitmq
    ports:
      - "5672:5672"
      - "15672:15672"
    networks:
      - backend-network
    healthcheck:
      test: ["CMD", "rabbitmq-diagnostics", "ping"]
      interval: 10s
      timeout: 5s
      retries: 5
    restart: unless-stopped

  # ================================
  # Monitoring Stack
  # ================================
  elasticsearch:
    image: docker.elastic.co/elasticsearch/elasticsearch:8.9.0
    environment:
      - discovery.type=single-node
      - xpack.security.enabled=false
      - "ES_JAVA_OPTS=-Xms512m -Xmx512m"
    volumes:
      - elasticsearch-data:/usr/share/elasticsearch/data
    ports:
      - "9200:9200"
    networks:
      - monitoring-network
      - backend-network
    healthcheck:
      test: ["CMD-SHELL", "curl -f http://localhost:9200/_cluster/health || exit 1"]
      interval: 30s
      timeout: 10s
      retries: 5
    restart: unless-stopped

  kibana:
    image: docker.elastic.co/kibana/kibana:8.9.0
    environment:
      - ELASTICSEARCH_HOSTS=http://elasticsearch:9200
    ports:
      - "5601:5601"
    depends_on:
      elasticsearch:
        condition: service_healthy
    networks:
      - monitoring-network
    restart: unless-stopped

  prometheus:
    image: prom/prometheus:latest
    command:
      - '--config.file=/etc/prometheus/prometheus.yml'
      - '--storage.tsdb.path=/prometheus'
      - '--web.console.libraries=/etc/prometheus/console_libraries'
      - '--web.console.templates=/etc/prometheus/consoles'
      - '--storage.tsdb.retention.time=200h'
      - '--web.enable-lifecycle'
    volumes:
      - ./monitoring/prometheus/prometheus.yml:/etc/prometheus/prometheus.yml
      - prometheus-data:/prometheus
    ports:
      - "9090:9090"
    networks:
      - monitoring-network
      - backend-network
    restart: unless-stopped

  grafana:
    image: grafana/grafana:latest
    environment:
      - GF_SECURITY_ADMIN_PASSWORD=${GRAFANA_PASSWORD}
      - GF_INSTALL_PLUGINS=grafana-piechart-panel
    volumes:
      - grafana-data:/var/lib/grafana
      - ./monitoring/grafana/dashboards:/var/lib/grafana/dashboards
      - ./monitoring/grafana/provisioning:/etc/grafana/provisioning
    ports:
      - "3000:3000"
    depends_on:
      - prometheus
    networks:
      - monitoring-network
    restart: unless-stopped

  # ================================
  # Cache and Session Store
  # ================================
  redis-cluster:
    image: redis:7-alpine
    command: redis-server --cluster-enabled yes --cluster-config-file nodes.conf --cluster-node-timeout 5000 --appendonly yes
    deploy:
      replicas: 3
    networks:
      - backend-network
    restart: unless-stopped

# ================================
# Networks
# ================================
networks:
  frontend-network:
    driver: bridge
  backend-network:
    driver: bridge
  monitoring-network:
    driver: bridge

# ================================
# Volumes
# ================================
volumes:
  postgres-data:
    driver: local
  mongodb-data:
    driver: local
  redis-data:
    driver: local
  rabbitmq-data:
    driver: local
  elasticsearch-data:
    driver: local
  prometheus-data:
    driver: local
  grafana-data:
    driver: local
```

### .env

```env
# Database Passwords
POSTGRES_PASSWORD=postgres_super_secret
MONGO_PASSWORD=mongo_super_secret

# Application Secrets
JWT_SECRET=your_jwt_secret_key_here
RABBITMQ_PASSWORD=rabbitmq_super_secret

# External Services
STRIPE_SECRET_KEY=sk_test_your_stripe_secret_key
PAYPAL_CLIENT_ID=your_paypal_client_id
PAYPAL_CLIENT_SECRET=your_paypal_client_secret

# Monitoring
GRAFANA_PASSWORD=admin_password

# Environment
NODE_ENV=production
ENVIRONMENT=production
```

## 🌐 Configuración de NGINX

### nginx/Dockerfile

```dockerfile
FROM nginx:alpine

# Install additional modules
RUN apk add --no-cache \
    nginx-mod-http-headers-more \
    nginx-mod-http-geoip2

# Copy configuration files
COPY nginx.conf /etc/nginx/nginx.conf
COPY api-gateway.conf /etc/nginx/conf.d/default.conf

# Create SSL directory
RUN mkdir -p /etc/nginx/ssl

# Health check
HEALTHCHECK --interval=30s --timeout=10s --start-period=5s --retries=3 \
  CMD curl -f http://localhost/health || exit 1

EXPOSE 80 443
```

### nginx/nginx.conf

```nginx
user nginx;
worker_processes auto;
error_log /var/log/nginx/error.log notice;
pid /var/run/nginx.pid;

events {
    worker_connections 1024;
    use epoll;
    multi_accept on;
}

http {
    include /etc/nginx/mime.types;
    default_type application/octet-stream;

    # Logging format
    log_format main '$remote_addr - $remote_user [$time_local] "$request" '
                   '$status $body_bytes_sent "$http_referer" '
                   '"$http_user_agent" "$http_x_forwarded_for" '
                   'rt=$request_time uct="$upstream_connect_time" '
                   'uht="$upstream_header_time" urt="$upstream_response_time"';

    access_log /var/log/nginx/access.log main;

    # Performance settings
    sendfile on;
    tcp_nopush on;
    tcp_nodelay on;
    keepalive_timeout 65;
    types_hash_max_size 2048;
    client_max_body_size 50M;

    # Gzip compression
    gzip on;
    gzip_vary on;
    gzip_min_length 1024;
    gzip_proxied any;
    gzip_comp_level 6;
    gzip_types
        application/atom+xml
        application/geo+json
        application/javascript
        application/x-javascript
        application/json
        application/ld+json
        application/manifest+json
        application/rdf+xml
        application/rss+xml
        application/xhtml+xml
        application/xml
        font/eot
        font/otf
        font/ttf
        image/svg+xml
        text/css
        text/javascript
        text/plain
        text/xml;

    # Rate limiting
    limit_req_zone $binary_remote_addr zone=api:10m rate=10r/s;
    limit_req_zone $binary_remote_addr zone=frontend:10m rate=50r/s;

    # Upstream definitions
    upstream frontend {
        least_conn;
        server frontend:3000 max_fails=3 fail_timeout=30s;
    }

    upstream users-service {
        least_conn;
        server users-service:3001 max_fails=3 fail_timeout=30s;
    }

    upstream products-service {
        least_conn;
        server products-service:8000 max_fails=3 fail_timeout=30s;
    }

    upstream orders-service {
        least_conn;
        server orders-service:8080 max_fails=3 fail_timeout=30s;
    }

    upstream payments-service {
        least_conn;
        server payments-service:8081 max_fails=3 fail_timeout=30s;
    }

    # Include additional configurations
    include /etc/nginx/conf.d/*.conf;
}
```

### nginx/api-gateway.conf

```nginx
# Health check endpoint
server {
    listen 80;
    server_name _;

    location /health {
        access_log off;
        return 200 "healthy\n";
        add_header Content-Type text/plain;
    }
}

# Main application server
server {
    listen 80;
    server_name localhost;

    # Security headers
    add_header X-Frame-Options "SAMEORIGIN" always;
    add_header X-XSS-Protection "1; mode=block" always;
    add_header X-Content-Type-Options "nosniff" always;
    add_header Referrer-Policy "no-referrer-when-downgrade" always;
    add_header Content-Security-Policy "default-src 'self' http: https: data: blob: 'unsafe-inline'" always;

    # Frontend application
    location / {
        limit_req zone=frontend burst=20 nodelay;
        proxy_pass http://frontend;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;

        # Enable WebSocket support
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";

        # Timeouts
        proxy_connect_timeout 30s;
        proxy_send_timeout 30s;
        proxy_read_timeout 30s;
    }

    # API Gateway routes
    location /api/ {
        limit_req zone=api burst=10 nodelay;
        
        # Common proxy headers
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        
        # Remove /api prefix
        rewrite ^/api/(.*)$ /$1 break;
    }

    # Users service
    location /api/users/ {
        limit_req zone=api burst=10 nodelay;
        proxy_pass http://users-service/;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }

    # Products service
    location /api/products/ {
        limit_req zone=api burst=15 nodelay;
        proxy_pass http://products-service/;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        
        # Cache for GET requests
        location ~* ^/api/products/.+\.(jpg|jpeg|png|gif|ico|svg)$ {
            proxy_pass http://products-service;
            expires 1y;
            add_header Cache-Control "public, immutable";
        }
    }

    # Orders service
    location /api/orders/ {
        limit_req zone=api burst=5 nodelay;
        proxy_pass http://orders-service/;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }

    # Payments service
    location /api/payments/ {
        limit_req zone=api burst=3 nodelay;
        proxy_pass http://payments-service/;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        
        # Additional security for payments
        add_header Strict-Transport-Security "max-age=31536000; includeSubDomains" always;
    }

    # WebSocket support for real-time features
    location /api/ws/ {
        proxy_pass http://orders-service/ws/;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }

    # Static assets caching
    location ~* \.(js|css|png|jpg|jpeg|gif|ico|svg|woff|woff2|ttf|eot)$ {
        expires 1y;
        add_header Cache-Control "public, immutable";
        add_header X-Content-Type-Options nosniff;
    }

    # Error pages
    error_page 404 /404.html;
    error_page 500 502 503 504 /50x.html;

    location = /404.html {
        root /usr/share/nginx/html;
    }

    location = /50x.html {
        root /usr/share/nginx/html;
    }
}
```

## 🎨 Frontend (React)

### frontend/Dockerfile

```dockerfile
# Multi-stage build for React frontend
FROM node:18-alpine AS build

WORKDIR /app

# Copy package files
COPY package*.json ./
RUN npm ci --only=production

# Copy source code
COPY src/ ./src/
COPY public/ ./public/

# Build application
RUN npm run build

# Production stage
FROM nginx:alpine AS production

# Copy built application
COPY --from=build /app/build /usr/share/nginx/html

# Copy nginx configuration
COPY nginx.conf /etc/nginx/conf.d/default.conf

# Health check
HEALTHCHECK --interval=30s --timeout=10s --start-period=5s --retries=3 \
  CMD curl -f http://localhost:3000/health || exit 1

EXPOSE 3000

CMD ["nginx", "-g", "daemon off;"]
```

### frontend/package.json

```json
{
  "name": "ecommerce-frontend",
  "version": "1.0.0",
  "private": true,
  "dependencies": {
    "@reduxjs/toolkit": "^1.9.5",
    "react": "^18.2.0",
    "react-dom": "^18.2.0",
    "react-redux": "^8.1.1",
    "react-router-dom": "^6.14.1",
    "axios": "^1.4.0",
    "styled-components": "^6.0.5",
    "@mui/material": "^5.14.0",
    "@mui/icons-material": "^5.14.0",
    "@emotion/react": "^11.11.1",
    "@emotion/styled": "^11.11.0",
    "socket.io-client": "^4.7.2"
  },
  "scripts": {
    "start": "react-scripts start",
    "build": "react-scripts build",
    "test": "react-scripts test",
    "eject": "react-scripts eject"
  },
  "devDependencies": {
    "react-scripts": "5.0.1",
    "@types/react": "^18.2.15",
    "@types/react-dom": "^18.2.7"
  },
  "browserslist": {
    "production": [
      ">0.2%",
      "not dead",
      "not op_mini all"
    ],
    "development": [
      "last 1 chrome version",
      "last 1 firefox version",
      "last 1 safari version"
    ]
  }
}
```

## 🔧 Servicios Backend

### Users Service (Node.js)

```dockerfile
# services/users-service/Dockerfile
FROM node:18-alpine

WORKDIR /app

# Create non-root user
RUN addgroup -g 1001 -S nodejs && \
    adduser -S nodejs -u 1001

# Copy package files
COPY package*.json ./
RUN npm ci --only=production && \
    npm cache clean --force

# Copy source code
COPY src/ ./src/

# Set ownership
RUN chown -R nodejs:nodejs /app
USER nodejs

# Health check
HEALTHCHECK --interval=30s --timeout=10s --start-period=5s --retries=3 \
  CMD curl -f http://localhost:3001/health || exit 1

EXPOSE 3001

CMD ["node", "src/index.js"]
```

```javascript
// services/users-service/src/index.js
const express = require('express');
const { Pool } = require('pg');
const Redis = require('ioredis');
const amqp = require('amqplib');
const jwt = require('jsonwebtoken');
const bcrypt = require('bcrypt');
const prometheus = require('prom-client');

const app = express();
app.use(express.json());

// Database connections
const postgres = new Pool({
  connectionString: process.env.DATABASE_URL,
  max: 20,
  idleTimeoutMillis: 30000,
  connectionTimeoutMillis: 2000,
});

const redis = new Redis(process.env.REDIS_URL);

// Metrics
const httpRequestDuration = new prometheus.Histogram({
  name: 'http_request_duration_seconds',
  help: 'Duration of HTTP requests in seconds',
  labelNames: ['method', 'route', 'status'],
});

// Middleware
app.use((req, res, next) => {
  const start = Date.now();
  res.on('finish', () => {
    const duration = (Date.now() - start) / 1000;
    httpRequestDuration
      .labels(req.method, req.route?.path || req.path, res.statusCode)
      .observe(duration);
  });
  next();
});

// Routes
app.get('/health', (req, res) => {
  res.json({ status: 'healthy', service: 'users-service' });
});

app.get('/metrics', async (req, res) => {
  res.set('Content-Type', prometheus.register.contentType);
  res.end(await prometheus.register.metrics());
});

app.post('/auth/register', async (req, res) => {
  try {
    const { email, password, firstName, lastName } = req.body;
    
    // Hash password
    const hashedPassword = await bcrypt.hash(password, 10);
    
    // Insert user
    const result = await postgres.query(
      'INSERT INTO users (email, password, first_name, last_name) VALUES ($1, $2, $3, $4) RETURNING id, email, first_name, last_name',
      [email, hashedPassword, firstName, lastName]
    );
    
    const user = result.rows[0];
    
    // Generate JWT
    const token = jwt.sign(
      { userId: user.id, email: user.email },
      process.env.JWT_SECRET,
      { expiresIn: '24h' }
    );
    
    // Cache user session
    await redis.setex(`user:${user.id}`, 86400, JSON.stringify(user));
    
    res.status(201).json({ user, token });
  } catch (error) {
    console.error('Registration error:', error);
    res.status(500).json({ error: 'Registration failed' });
  }
});

app.post('/auth/login', async (req, res) => {
  try {
    const { email, password } = req.body;
    
    // Get user from database
    const result = await postgres.query(
      'SELECT * FROM users WHERE email = $1',
      [email]
    );
    
    if (result.rows.length === 0) {
      return res.status(401).json({ error: 'Invalid credentials' });
    }
    
    const user = result.rows[0];
    
    // Verify password
    const validPassword = await bcrypt.compare(password, user.password);
    if (!validPassword) {
      return res.status(401).json({ error: 'Invalid credentials' });
    }
    
    // Generate JWT
    const token = jwt.sign(
      { userId: user.id, email: user.email },
      process.env.JWT_SECRET,
      { expiresIn: '24h' }
    );
    
    // Cache user session
    await redis.setex(`user:${user.id}`, 86400, JSON.stringify({
      id: user.id,
      email: user.email,
      firstName: user.first_name,
      lastName: user.last_name
    }));
    
    res.json({ 
      user: {
        id: user.id,
        email: user.email,
        firstName: user.first_name,
        lastName: user.last_name
      },
      token 
    });
  } catch (error) {
    console.error('Login error:', error);
    res.status(500).json({ error: 'Login failed' });
  }
});

app.get('/users/:id', async (req, res) => {
  try {
    const { id } = req.params;
    
    // Check cache first
    const cached = await redis.get(`user:${id}`);
    if (cached) {
      return res.json(JSON.parse(cached));
    }
    
    // Get from database
    const result = await postgres.query(
      'SELECT id, email, first_name, last_name, created_at FROM users WHERE id = $1',
      [id]
    );
    
    if (result.rows.length === 0) {
      return res.status(404).json({ error: 'User not found' });
    }
    
    const user = result.rows[0];
    
    // Cache result
    await redis.setex(`user:${id}`, 3600, JSON.stringify(user));
    
    res.json(user);
  } catch (error) {
    console.error('Get user error:', error);
    res.status(500).json({ error: 'Failed to get user' });
  }
});

const PORT = process.env.PORT || 3001;
app.listen(PORT, () => {
  console.log(`Users service running on port ${PORT}`);
});
```

### Products Service (Python/FastAPI)

```dockerfile
# services/products-service/Dockerfile
FROM python:3.11-slim

WORKDIR /app

# Install system dependencies
RUN apt-get update && apt-get install -y \
    gcc \
    libpq-dev \
    curl \
    && rm -rf /var/lib/apt/lists/*

# Copy requirements first for better caching
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# Create non-root user
RUN useradd --create-home --shell /bin/bash app
USER app

# Copy application code
COPY app/ ./app/

# Health check
HEALTHCHECK --interval=30s --timeout=10s --start-period=5s --retries=3 \
  CMD curl -f http://localhost:8000/health || exit 1

EXPOSE 8000

CMD ["uvicorn", "app.main:app", "--host", "0.0.0.0", "--port", "8000"]
```

```python
# services/products-service/app/main.py
from fastapi import FastAPI, HTTPException, Depends
from sqlalchemy import create_engine, Column, Integer, String, Float, DateTime, Text
from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy.orm import sessionmaker, Session
from pydantic import BaseModel
import redis
import json
import os
from datetime import datetime
from prometheus_client import Counter, Histogram, generate_latest
import logging

# Setup logging
logging.basicConfig(level=logging.INFO)
logger = logging.getLogger(__name__)

app = FastAPI(title="Products Service", version="1.0.0")

# Database setup
DATABASE_URL = os.getenv("DATABASE_URL")
engine = create_engine(DATABASE_URL)
SessionLocal = sessionmaker(autocommit=False, autoflush=False, bind=engine)
Base = declarative_base()

# Redis setup
redis_client = redis.from_url(os.getenv("REDIS_URL"))

# Metrics
REQUEST_COUNT = Counter('http_requests_total', 'Total HTTP requests', ['method', 'endpoint'])
REQUEST_LATENCY = Histogram('http_request_duration_seconds', 'HTTP request latency')

# Models
class Product(Base):
    __tablename__ = "products"
    
    id = Column(Integer, primary_key=True, index=True)
    name = Column(String, index=True)
    description = Column(Text)
    price = Column(Float)
    category = Column(String, index=True)
    inventory = Column(Integer)
    created_at = Column(DateTime, default=datetime.utcnow)

Base.metadata.create_all(bind=engine)

# Pydantic schemas
class ProductCreate(BaseModel):
    name: str
    description: str
    price: float
    category: str
    inventory: int

class ProductResponse(BaseModel):
    id: int
    name: str
    description: str
    price: float
    category: str
    inventory: int
    created_at: datetime

    class Config:
        from_attributes = True

# Dependency
def get_db():
    db = SessionLocal()
    try:
        yield db
    finally:
        db.close()

# Middleware
@app.middleware("http")
async def metrics_middleware(request, call_next):
    start_time = time.time()
    response = await call_next(request)
    process_time = time.time() - start_time
    
    REQUEST_COUNT.labels(method=request.method, endpoint=request.url.path).inc()
    REQUEST_LATENCY.observe(process_time)
    
    return response

# Routes
@app.get("/health")
async def health():
    return {"status": "healthy", "service": "products-service"}

@app.get("/metrics")
async def metrics():
    return Response(generate_latest(), media_type="text/plain")

@app.get("/products/", response_model=list[ProductResponse])
async def get_products(category: str = None, db: Session = Depends(get_db)):
    # Try cache first
    cache_key = f"products:{category or 'all'}"
    cached = redis_client.get(cache_key)
    
    if cached:
        return json.loads(cached)
    
    # Query database
    query = db.query(Product)
    if category:
        query = query.filter(Product.category == category)
    
    products = query.all()
    
    # Cache results
    redis_client.setex(cache_key, 300, json.dumps([
        {
            "id": p.id,
            "name": p.name,
            "description": p.description,
            "price": p.price,
            "category": p.category,
            "inventory": p.inventory,
            "created_at": p.created_at.isoformat()
        } for p in products
    ]))
    
    return products

@app.get("/products/{product_id}", response_model=ProductResponse)
async def get_product(product_id: int, db: Session = Depends(get_db)):
    # Try cache first
    cache_key = f"product:{product_id}"
    cached = redis_client.get(cache_key)
    
    if cached:
        return json.loads(cached)
    
    product = db.query(Product).filter(Product.id == product_id).first()
    if not product:
        raise HTTPException(status_code=404, detail="Product not found")
    
    # Cache result
    product_data = {
        "id": product.id,
        "name": product.name,
        "description": product.description,
        "price": product.price,
        "category": product.category,
        "inventory": product.inventory,
        "created_at": product.created_at.isoformat()
    }
    redis_client.setex(cache_key, 600, json.dumps(product_data))
    
    return product

@app.post("/products/", response_model=ProductResponse)
async def create_product(product: ProductCreate, db: Session = Depends(get_db)):
    db_product = Product(**product.dict())
    db.add(db_product)
    db.commit()
    db.refresh(db_product)
    
    # Invalidate cache
    redis_client.delete("products:all")
    redis_client.delete(f"products:{product.category}")
    
    return db_product

@app.put("/products/{product_id}/inventory")
async def update_inventory(product_id: int, inventory: int, db: Session = Depends(get_db)):
    product = db.query(Product).filter(Product.id == product_id).first()
    if not product:
        raise HTTPException(status_code=404, detail="Product not found")
    
    product.inventory = inventory
    db.commit()
    
    # Invalidate cache
    redis_client.delete(f"product:{product_id}")
    redis_client.delete("products:all")
    redis_client.delete(f"products:{product.category}")
    
    return {"message": "Inventory updated successfully"}

if __name__ == "__main__":
    import uvicorn
    uvicorn.run(app, host="0.0.0.0", port=8000)
```

## 📊 Configuración de Monitoreo

### monitoring/prometheus/prometheus.yml

```yaml
global:
  scrape_interval: 15s
  evaluation_interval: 15s

rule_files:
  - "alert_rules.yml"

alerting:
  alertmanagers:
    - static_configs:
        - targets:
          - alertmanager:9093

scrape_configs:
  # Prometheus itself
  - job_name: 'prometheus'
    static_configs:
      - targets: ['localhost:9090']

  # Application services
  - job_name: 'users-service'
    static_configs:
      - targets: ['users-service:3001']
    metrics_path: '/metrics'
    scrape_interval: 10s

  - job_name: 'products-service'
    static_configs:
      - targets: ['products-service:8000']
    metrics_path: '/metrics'
    scrape_interval: 10s

  - job_name: 'orders-service'
    static_configs:
      - targets: ['orders-service:8080']
    metrics_path: '/actuator/prometheus'
    scrape_interval: 10s

  - job_name: 'payments-service'
    static_configs:
      - targets: ['payments-service:8081']
    metrics_path: '/metrics'
    scrape_interval: 10s

  # Infrastructure services
  - job_name: 'nginx'
    static_configs:
      - targets: ['nginx-lb:9113']
    scrape_interval: 30s

  - job_name: 'postgres'
    static_configs:
      - targets: ['postgres-exporter:9187']
    scrape_interval: 30s

  - job_name: 'redis'
    static_configs:
      - targets: ['redis-exporter:9121']
    scrape_interval: 30s

  - job_name: 'rabbitmq'
    static_configs:
      - targets: ['rabbitmq:15692']
    scrape_interval: 30s

  # Node exporter for system metrics
  - job_name: 'node-exporter'
    static_configs:
      - targets: ['node-exporter:9100']
    scrape_interval: 30s
```

## 🚀 Scripts de Despliegue

### scripts/init-db.sh

```bash
#!/bin/bash

set -e

echo "🚀 Inicializando bases de datos..."

# Wait for PostgreSQL
echo "⏳ Esperando PostgreSQL..."
until docker-compose exec postgres pg_isready -U postgres; do
  sleep 2
done

# Create databases and tables
echo "📊 Creando esquemas de base de datos..."

# Users database
docker-compose exec postgres psql -U postgres -d users_db -c "
CREATE TABLE IF NOT EXISTS users (
    id SERIAL PRIMARY KEY,
    email VARCHAR(255) UNIQUE NOT NULL,
    password VARCHAR(255) NOT NULL,
    first_name VARCHAR(100) NOT NULL,
    last_name VARCHAR(100) NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_users_email ON users(email);
CREATE INDEX idx_users_created_at ON users(created_at);
"

# Products database
docker-compose exec postgres psql -U postgres -d products_db -c "
CREATE TABLE IF NOT EXISTS products (
    id SERIAL PRIMARY KEY,
    name VARCHAR(255) NOT NULL,
    description TEXT,
    price DECIMAL(10,2) NOT NULL,
    category VARCHAR(100) NOT NULL,
    inventory INTEGER NOT NULL DEFAULT 0,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_products_category ON products(category);
CREATE INDEX idx_products_name ON products(name);
CREATE INDEX idx_products_price ON products(price);
"

# Wait for MongoDB
echo "⏳ Esperando MongoDB..."
until docker-compose exec mongodb mongo --eval "db.adminCommand('ping')"; do
  sleep 2
done

# Initialize MongoDB collections
echo "📊 Inicializando colecciones de MongoDB..."
docker-compose exec mongodb mongo orders_db --eval "
db.createCollection('orders');
db.createCollection('order_items');
db.createCollection('payments');

db.orders.createIndex({ 'user_id': 1 });
db.orders.createIndex({ 'status': 1 });
db.orders.createIndex({ 'created_at': -1 });
db.order_items.createIndex({ 'order_id': 1 });
db.payments.createIndex({ 'order_id': 1 });
"

echo "✅ Bases de datos inicializadas correctamente!"
```

### scripts/seed-data.sh

```bash
#!/bin/bash

set -e

echo "🌱 Cargando datos de prueba..."

# Seed products
echo "📦 Cargando productos..."
docker-compose exec postgres psql -U postgres -d products_db -c "
INSERT INTO products (name, description, price, category, inventory) VALUES
('Laptop Gaming', 'Laptop para gaming de alta performance', 1299.99, 'Electronics', 50),
('Smartphone Pro', 'Smartphone con cámara profesional', 899.99, 'Electronics', 100),
('Auriculares Bluetooth', 'Auriculares inalámbricos con cancelación de ruido', 199.99, 'Electronics', 200),
('Mesa de Oficina', 'Mesa ergonómica para oficina', 299.99, 'Furniture', 30),
('Silla Gaming', 'Silla ergonómica para gaming', 399.99, 'Furniture', 25),
('Monitor 4K', 'Monitor 27 pulgadas 4K', 449.99, 'Electronics', 40),
('Teclado Mecánico', 'Teclado mecánico RGB', 129.99, 'Electronics', 75),
('Mouse Inalámbrico', 'Mouse ergonómico inalámbrico', 59.99, 'Electronics', 150),
('Webcam HD', 'Webcam 1080p para streaming', 89.99, 'Electronics', 60),
('Escritorio Standing', 'Escritorio ajustable en altura', 599.99, 'Furniture', 15)
ON CONFLICT DO NOTHING;
"

# Create test user
echo "👤 Creando usuario de prueba..."
docker-compose exec users-service curl -X POST http://localhost:3001/auth/register \
  -H "Content-Type: application/json" \
  -d '{
    "email": "test@example.com",
    "password": "password123",
    "firstName": "Test",
    "lastName": "User"
  }' || true

echo "✅ Datos de prueba cargados correctamente!"
```

### scripts/health-check.sh

```bash
#!/bin/bash

echo "🔍 Verificando salud de servicios..."

services=(
    "nginx-lb:80/health"
    "users-service:3001/health"
    "products-service:8000/health"
    "orders-service:8080/health"
    "payments-service:8081/health"
)

all_healthy=true

for service in "${services[@]}"; do
    echo -n "Verificando $service... "
    if docker-compose exec nginx-lb curl -f http://$service > /dev/null 2>&1; then
        echo "✅ OK"
    else
        echo "❌ FAIL"
        all_healthy=false
    fi
done

# Check databases
echo -n "Verificando PostgreSQL... "
if docker-compose exec postgres pg_isready -U postgres > /dev/null 2>&1; then
    echo "✅ OK"
else
    echo "❌ FAIL"
    all_healthy=false
fi

echo -n "Verificando MongoDB... "
if docker-compose exec mongodb mongo --eval "db.adminCommand('ping')" > /dev/null 2>&1; then
    echo "✅ OK"
else
    echo "❌ FAIL"
    all_healthy=false
fi

echo -n "Verificando Redis... "
if docker-compose exec redis redis-cli ping > /dev/null 2>&1; then
    echo "✅ OK"
else
    echo "❌ FAIL"
    all_healthy=false
fi

echo -n "Verificando RabbitMQ... "
if docker-compose exec rabbitmq rabbitmq-diagnostics ping > /dev/null 2>&1; then
    echo "✅ OK"
else
    echo "❌ FAIL"
    all_healthy=false
fi

if $all_healthy; then
    echo "🎉 Todos los servicios están funcionando correctamente!"
    exit 0
else
    echo "⚠️  Algunos servicios tienen problemas."
    exit 1
fi
```

## 🚀 Comandos de Despliegue

### Despliegue Completo

```bash
# 1. Clonar proyecto y navegar al directorio
git clone <repository-url>
cd ecommerce-microservices

# 2. Configurar variables de entorno
cp .env.example .env
# Editar .env con valores reales

# 3. Construir e iniciar servicios
docker-compose up -d --build

# 4. Esperar que todos los servicios estén listos
sleep 60

# 5. Inicializar bases de datos
./scripts/init-db.sh

# 6. Cargar datos de prueba
./scripts/seed-data.sh

# 7. Verificar salud de servicios
./scripts/health-check.sh

# 8. Ver logs
docker-compose logs -f

# 9. Acceder a la aplicación
echo "🌐 Aplicación disponible en: http://localhost"
echo "📊 Grafana: http://localhost:3000 (admin/admin)"
echo "🔍 Kibana: http://localhost:5601"
echo "📈 Prometheus: http://localhost:9090"
echo "🐰 RabbitMQ: http://localhost:15672 (admin/password)"
```

### Comandos de Gestión

```bash
# Ver estado de servicios
docker-compose ps

# Escalar servicios específicos
docker-compose up -d --scale users-service=3 --scale products-service=3

# Ver logs de un servicio específico
docker-compose logs -f users-service

# Ejecutar comandos en contenedores
docker-compose exec postgres psql -U postgres
docker-compose exec redis redis-cli
docker-compose exec mongodb mongo

# Backup de bases de datos
docker-compose exec postgres pg_dump -U postgres products_db > backup_products.sql
docker-compose exec mongodb mongodump --db orders_db --out /backup

# Restaurar bases de datos
docker-compose exec postgres psql -U postgres products_db < backup_products.sql
docker-compose exec mongodb mongorestore --db orders_db /backup/orders_db

# Detener servicios
docker-compose down

# Limpiar todo (incluyendo volúmenes)
docker-compose down -v --remove-orphans
docker system prune -f
```

## 🧪 Testing y Validación

### Test de Carga con k6

```javascript
// tests/load-test.js
import http from 'k6/http';
import { check, sleep } from 'k6';

export let options = {
  stages: [
    { duration: '2m', target: 10 }, // Ramp up
    { duration: '5m', target: 10 }, // Stay at 10 users
    { duration: '2m', target: 20 }, // Ramp up to 20 users
    { duration: '5m', target: 20 }, // Stay at 20 users
    { duration: '2m', target: 0 },  // Ramp down
  ],
};

const BASE_URL = 'http://localhost';

export default function () {
  // Test homepage
  let response = http.get(`${BASE_URL}/`);
  check(response, {
    'homepage status is 200': (r) => r.status === 200,
    'homepage loads in < 500ms': (r) => r.timings.duration < 500,
  });

  sleep(1);

  // Test products API
  response = http.get(`${BASE_URL}/api/products/`);
  check(response, {
    'products API status is 200': (r) => r.status === 200,
    'products API responds in < 200ms': (r) => r.timings.duration < 200,
    'products API returns array': (r) => Array.isArray(r.json()),
  });

  sleep(2);

  // Test specific product
  response = http.get(`${BASE_URL}/api/products/1`);
  check(response, {
    'product detail status is 200': (r) => r.status === 200,
    'product has required fields': (r) => {
      const product = r.json();
      return product.id && product.name && product.price;
    },
  });

  sleep(1);
}
```

### Ejecutar Tests

```bash
# Instalar k6
curl https://github.com/grafana/k6/releases/download/v0.45.0/k6-v0.45.0-linux-amd64.tar.gz -L | tar xvz --strip-components 1

# Ejecutar test de carga
./k6 run tests/load-test.js

# Test con diferentes perfiles
./k6 run --vus 50 --duration 5m tests/load-test.js

# Test con métricas en InfluxDB
./k6 run --out influxdb=http://localhost:8086/k6 tests/load-test.js
```

## 🎯 Mejores Prácticas Implementadas

### Seguridad

1. **Contenedores no-root**: Todos los servicios ejecutan como usuarios no privilegiados
2. **Secrets management**: Variables sensibles en `.env` y secrets de Docker
3. **Network isolation**: Redes separadas para frontend, backend y monitoreo
4. **Rate limiting**: Implementado en NGINX para proteger APIs
5. **Health checks**: Todos los servicios tienen health checks configurados

### Performance

1. **Caching**: Redis para cache de datos frecuentemente accedidos
2. **Connection pooling**: Pools de conexiones optimizados para bases de datos
3. **Load balancing**: NGINX con algoritmos de balanceo inteligente
4. **Resource limits**: Límites de CPU y memoria para todos los contenedores
5. **Image optimization**: Imágenes multi-stage para reducir tamaño

### Observabilidad

1. **Logging centralizado**: ELK stack para agregación de logs
2. **Métricas**: Prometheus + Grafana para monitoreo de aplicaciones
3. **Tracing**: Preparado para integrar con Jaeger o Zipkin
4. **Alerting**: Configuración de alertas basadas en métricas
5. **Dashboards**: Dashboards predefinidos en Grafana

### DevOps

1. **Infrastructure as Code**: Todo definido en docker-compose
2. **Automated deployment**: Scripts de despliegue automatizados
3. **Environment management**: Configuración por entornos
4. **Backup automation**: Scripts para backup de datos
5. **Rollback capability**: Capacidad de rollback con versionado de imágenes

## 🎯 Resultados Esperados

Al completar esta práctica habrás implementado:

- [x] Arquitectura de microservicios completa y funcional
- [x] Comunicación entre servicios con API Gateway
- [x] Gestión de datos con múltiples bases de datos
- [x] Sistema de caching distribuido
- [x] Monitoreo y observabilidad completos
- [x] Prácticas de seguridad y performance
- [x] Scripts de automatización y despliegue

## 📚 Recursos Adicionales

- [Docker Compose Best Practices](https://docs.docker.com/compose/production/)
- [Microservices Patterns](https://microservices.io/patterns/)
- [NGINX Configuration Guide](https://nginx.org/en/docs/)
- [Prometheus Monitoring](https://prometheus.io/docs/guides/getting_started/)
- [ELK Stack Guide](https://www.elastic.co/guide/en/elastic-stack/current/index.html)

---

**Próximo tema:** [Práctica: CI/CD para Contenedores](./11-practica-cicd-contenedores.md)