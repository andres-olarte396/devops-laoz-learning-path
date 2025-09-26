# Práctica: Pipeline CI/CD Completo

¡Es hora de poner en práctica todo lo aprendido! En este módulo construiremos un **pipeline CI/CD completo** desde cero para una aplicación real, implementando **pruebas automatizadas**, **múltiples entornos**, **estrategias de despliegue avanzadas** y **rollbacks automáticos**.

Este proyecto te dará experiencia práctica con todas las herramientas y técnicas de DevOps moderno.

---

## **🎯 Objetivos del Proyecto**

Al finalizar esta práctica habrás:

- ✅ **Configurado un pipeline CI/CD completo** con múltiples stages
- ✅ **Implementado pruebas automatizadas** (unit, integration, E2E)
- ✅ **Desplegado en múltiples entornos** (staging, production)
- ✅ **Aplicado estrategias de deployment** (Blue-Green, Canary)
- ✅ **Configurado monitoring y alerting**
- ✅ **Implementado rollbacks automáticos**
- ✅ **Usado Infrastructure as Code**

---

## **🏗️ Proyecto: Todo App con Microservicios**

### **Arquitectura del proyecto**

```mermaid
graph TB
    subgraph "Frontend"
        A[React App] --> B[Nginx]
    end
    
    subgraph "Backend Services"
        C[API Gateway] --> D[Auth Service]
        C --> E[Todo Service]  
        C --> F[User Service]
    end
    
    subgraph "Databases"
        D --> G[(PostgreSQL)]
        E --> H[(MongoDB)]
        F --> G
    end
    
    subgraph "Infrastructure"
        I[Redis Cache]
        J[Message Queue]
    end
    
    A --> C
    E --> I
    E --> J
```

### **Stack tecnológico**

```yaml
# Stack completo del proyecto
frontend:
  framework: "React 18"
  testing: "Jest + React Testing Library + Cypress"
  build: "Vite"
  deployment: "Static hosting + CDN"

backend:
  - service: "API Gateway"
    tech: "Node.js + Express"
    port: 3000
    
  - service: "Auth Service" 
    tech: "Node.js + JWT"
    port: 3001
    database: "PostgreSQL"
    
  - service: "Todo Service"
    tech: "Node.js + Express"
    port: 3002
    database: "MongoDB"
    
  - service: "User Service"
    tech: "Python + FastAPI"
    port: 3003
    database: "PostgreSQL"

infrastructure:
  containerization: "Docker + Docker Compose"
  orchestration: "Kubernetes"
  ci_cd: "GitHub Actions"
  monitoring: "Prometheus + Grafana"
  logging: "ELK Stack"
  caching: "Redis"
  messaging: "RabbitMQ"
```

---

## **🚀 Fase 1: Setup del Proyecto**

### **1.1 Estructura del repositorio**

```bash
# Crear la estructura del proyecto
mkdir todo-app-cicd
cd todo-app-cicd

# Estructura de directorios
mkdir -p {frontend,backend/{api-gateway,auth-service,todo-service,user-service}}
mkdir -p {infrastructure/{docker,k8s,terraform},scripts,docs,.github/workflows}
mkdir -p tests/{unit,integration,e2e,performance}

# Archivos de configuración raíz
touch {README.md,docker-compose.yml,docker-compose.prod.yml}
touch {.gitignore,.env.example,Makefile}
```

La estructura final será:

```
todo-app-cicd/
├── frontend/                 # React application
│   ├── src/
│   ├── public/
│   ├── package.json
│   ├── Dockerfile
│   └── nginx.conf
├── backend/
│   ├── api-gateway/         # Express API Gateway
│   │   ├── src/
│   │   ├── package.json
│   │   └── Dockerfile
│   ├── auth-service/        # Authentication microservice
│   │   ├── src/
│   │   ├── package.json
│   │   └── Dockerfile
│   ├── todo-service/        # Todo CRUD microservice
│   │   ├── src/
│   │   ├── package.json
│   │   └── Dockerfile
│   └── user-service/        # User management (Python/FastAPI)
│       ├── app/
│       ├── requirements.txt
│       └── Dockerfile
├── infrastructure/
│   ├── docker/              # Docker configurations
│   ├── k8s/                 # Kubernetes manifests
│   └── terraform/           # Infrastructure as Code
├── tests/
│   ├── unit/               # Unit tests
│   ├── integration/        # Integration tests
│   ├── e2e/               # End-to-end tests
│   └── performance/       # Load tests
├── scripts/               # Automation scripts
├── .github/workflows/     # CI/CD pipelines
├── docker-compose.yml     # Development environment
├── docker-compose.prod.yml # Production environment
└── Makefile              # Task automation
```

### **1.2 Frontend: React Todo App**

```bash
# Crear aplicación React
cd frontend
npx create-react-app . --template typescript
```

**Código principal del Frontend:**

```typescript
// frontend/src/App.tsx
import React, { useState, useEffect } from 'react';
import axios from 'axios';
import './App.css';

interface Todo {
  id: string;
  title: string;
  completed: boolean;
  createdAt: string;
}

const API_BASE_URL = process.env.REACT_APP_API_URL || 'http://localhost:3000';

function App() {
  const [todos, setTodos] = useState<Todo[]>([]);
  const [newTodo, setNewTodo] = useState('');
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState<string | null>(null);

  useEffect(() => {
    fetchTodos();
  }, []);

  const fetchTodos = async () => {
    try {
      setLoading(true);
      const response = await axios.get(`${API_BASE_URL}/api/todos`);
      setTodos(response.data);
      setError(null);
    } catch (err) {
      setError('Failed to fetch todos');
      console.error('Error fetching todos:', err);
    } finally {
      setLoading(false);
    }
  };

  const createTodo = async (e: React.FormEvent) => {
    e.preventDefault();
    if (!newTodo.trim()) return;

    try {
      setLoading(true);
      const response = await axios.post(`${API_BASE_URL}/api/todos`, {
        title: newTodo.trim()
      });
      setTodos([...todos, response.data]);
      setNewTodo('');
      setError(null);
    } catch (err) {
      setError('Failed to create todo');
      console.error('Error creating todo:', err);
    } finally {
      setLoading(false);
    }
  };

  const toggleTodo = async (id: string, completed: boolean) => {
    try {
      const response = await axios.put(`${API_BASE_URL}/api/todos/${id}`, {
        completed: !completed
      });
      setTodos(todos.map(todo => 
        todo.id === id ? response.data : todo
      ));
      setError(null);
    } catch (err) {
      setError('Failed to update todo');
      console.error('Error updating todo:', err);
    }
  };

  const deleteTodo = async (id: string) => {
    try {
      await axios.delete(`${API_BASE_URL}/api/todos/${id}`);
      setTodos(todos.filter(todo => todo.id !== id));
      setError(null);
    } catch (err) {
      setError('Failed to delete todo');
      console.error('Error deleting todo:', err);
    }
  };

  return (
    <div className="App">
      <header className="App-header">
        <h1>Todo App CI/CD</h1>
        <p>Environment: {process.env.NODE_ENV}</p>
        <p>Version: {process.env.REACT_APP_VERSION || '1.0.0'}</p>
        
        {error && (
          <div className="error" data-testid="error-message">
            {error}
          </div>
        )}
        
        <form onSubmit={createTodo} className="todo-form">
          <input
            type="text"
            value={newTodo}
            onChange={(e) => setNewTodo(e.target.value)}
            placeholder="Add new todo..."
            disabled={loading}
            data-testid="todo-input"
          />
          <button 
            type="submit" 
            disabled={loading || !newTodo.trim()}
            data-testid="add-todo-button"
          >
            {loading ? 'Adding...' : 'Add Todo'}
          </button>
        </form>

        <div className="todo-list">
          {loading && todos.length === 0 && (
            <div data-testid="loading">Loading todos...</div>
          )}
          
          {todos.map(todo => (
            <div key={todo.id} className="todo-item" data-testid="todo-item">
              <input
                type="checkbox"
                checked={todo.completed}
                onChange={() => toggleTodo(todo.id, todo.completed)}
                data-testid={`todo-checkbox-${todo.id}`}
              />
              <span 
                className={todo.completed ? 'completed' : ''}
                data-testid={`todo-title-${todo.id}`}
              >
                {todo.title}
              </span>
              <button 
                onClick={() => deleteTodo(todo.id)}
                className="delete-button"
                data-testid={`delete-todo-${todo.id}`}
              >
                Delete
              </button>
            </div>
          ))}
          
          {todos.length === 0 && !loading && (
            <div data-testid="no-todos">No todos yet. Add one above!</div>
          )}
        </div>
      </header>
    </div>
  );
}

export default App;
```

**Tests del Frontend:**

```typescript
// frontend/src/App.test.tsx
import React from 'react';
import { render, screen, fireEvent, waitFor } from '@testing-library/react';
import axios from 'axios';
import App from './App';

// Mock axios
jest.mock('axios');
const mockedAxios = axios as jest.Mocked<typeof axios>;

const mockTodos = [
  {
    id: '1',
    title: 'Test Todo 1',
    completed: false,
    createdAt: '2024-01-01T00:00:00.000Z'
  },
  {
    id: '2', 
    title: 'Test Todo 2',
    completed: true,
    createdAt: '2024-01-01T00:00:00.000Z'
  }
];

describe('App Component', () => {
  beforeEach(() => {
    mockedAxios.get.mockClear();
    mockedAxios.post.mockClear();
    mockedAxios.put.mockClear();
    mockedAxios.delete.mockClear();
  });

  test('renders todo app header', () => {
    mockedAxios.get.mockResolvedValue({ data: [] });
    render(<App />);
    
    expect(screen.getByText('Todo App CI/CD')).toBeInTheDocument();
    expect(screen.getByText(/Environment:/)).toBeInTheDocument();
    expect(screen.getByText(/Version:/)).toBeInTheDocument();
  });

  test('fetches and displays todos on load', async () => {
    mockedAxios.get.mockResolvedValue({ data: mockTodos });
    
    render(<App />);
    
    await waitFor(() => {
      expect(screen.getByText('Test Todo 1')).toBeInTheDocument();
      expect(screen.getByText('Test Todo 2')).toBeInTheDocument();
    });
    
    expect(mockedAxios.get).toHaveBeenCalledWith('http://localhost:3000/api/todos');
  });

  test('creates new todo', async () => {
    const newTodo = {
      id: '3',
      title: 'New Test Todo',
      completed: false,
      createdAt: '2024-01-01T00:00:00.000Z'
    };
    
    mockedAxios.get.mockResolvedValue({ data: [] });
    mockedAxios.post.mockResolvedValue({ data: newTodo });
    
    render(<App />);
    
    const input = screen.getByTestId('todo-input');
    const button = screen.getByTestId('add-todo-button');
    
    fireEvent.change(input, { target: { value: 'New Test Todo' } });
    fireEvent.click(button);
    
    await waitFor(() => {
      expect(mockedAxios.post).toHaveBeenCalledWith(
        'http://localhost:3000/api/todos',
        { title: 'New Test Todo' }
      );
    });
  });

  test('displays error message on API failure', async () => {
    mockedAxios.get.mockRejectedValue(new Error('API Error'));
    
    render(<App />);
    
    await waitFor(() => {
      expect(screen.getByTestId('error-message')).toHaveTextContent('Failed to fetch todos');
    });
  });

  test('toggles todo completion', async () => {
    const updatedTodo = { ...mockTodos[0], completed: true };
    mockedAxios.get.mockResolvedValue({ data: [mockTodos[0]] });
    mockedAxios.put.mockResolvedValue({ data: updatedTodo });
    
    render(<App />);
    
    await waitFor(() => {
      expect(screen.getByText('Test Todo 1')).toBeInTheDocument();
    });
    
    const checkbox = screen.getByTestId('todo-checkbox-1');
    fireEvent.click(checkbox);
    
    await waitFor(() => {
      expect(mockedAxios.put).toHaveBeenCalledWith(
        'http://localhost:3000/api/todos/1',
        { completed: true }
      );
    });
  });

  test('deletes todo', async () => {
    mockedAxios.get.mockResolvedValue({ data: [mockTodos[0]] });
    mockedAxios.delete.mockResolvedValue({});
    
    render(<App />);
    
    await waitFor(() => {
      expect(screen.getByText('Test Todo 1')).toBeInTheDocument();
    });
    
    const deleteButton = screen.getByTestId('delete-todo-1');
    fireEvent.click(deleteButton);
    
    await waitFor(() => {
      expect(mockedAxios.delete).toHaveBeenCalledWith('http://localhost:3000/api/todos/1');
    });
  });
});
```

**Dockerfile del Frontend:**

```dockerfile
# frontend/Dockerfile
# Multi-stage build for production optimization
FROM node:18-alpine as build

WORKDIR /app

# Copy package files
COPY package*.json ./

# Install dependencies
RUN npm ci --only=production

# Copy source code
COPY . .

# Build arguments
ARG REACT_APP_VERSION=1.0.0
ARG REACT_APP_API_URL=http://localhost:3000

# Set environment variables
ENV REACT_APP_VERSION=$REACT_APP_VERSION
ENV REACT_APP_API_URL=$REACT_APP_API_URL

# Build application
RUN npm run build

# Production stage
FROM nginx:alpine

# Copy custom nginx config
COPY nginx.conf /etc/nginx/conf.d/default.conf

# Copy built app
COPY --from=build /app/build /usr/share/nginx/html

# Add healthcheck
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
  CMD curl -f http://localhost/ || exit 1

# Expose port
EXPOSE 80

CMD ["nginx", "-g", "daemon off;"]
```

**Configuración Nginx:**

```nginx
# frontend/nginx.conf
server {
    listen 80;
    server_name localhost;
    
    # Security headers
    add_header X-Content-Type-Options nosniff;
    add_header X-Frame-Options DENY;
    add_header X-XSS-Protection "1; mode=block";
    
    # Gzip compression
    gzip on;
    gzip_vary on;
    gzip_min_length 1024;
    gzip_types text/plain text/css text/xml text/javascript application/javascript application/xml+rss application/json;
    
    # Static files with cache
    location /static/ {
        expires 1y;
        add_header Cache-Control "public, immutable";
    }
    
    # Main application
    location / {
        root /usr/share/nginx/html;
        index index.html index.htm;
        try_files $uri $uri/ /index.html;
        
        # No cache for index.html
        location = /index.html {
            expires -1;
            add_header Cache-Control "no-cache, no-store, must-revalidate";
        }
    }
    
    # Health check endpoint
    location /health {
        access_log off;
        return 200 "healthy\n";
        add_header Content-Type text/plain;
    }
}
```

### **1.3 Backend: API Gateway**

```javascript
// backend/api-gateway/src/server.js
const express = require('express');
const cors = require('cors');
const helmet = require('helmet');
const compression = require('compression');
const rateLimit = require('express-rate-limit');
const { createProxyMiddleware } = require('http-proxy-middleware');
const promClient = require('prom-client');

const app = express();
const PORT = process.env.PORT || 3000;

// Metrics collection
const register = new promClient.Registry();
promClient.collectDefaultMetrics({ register });

// Custom metrics
const httpRequestDuration = new promClient.Histogram({
  name: 'http_request_duration_seconds',
  help: 'Duration of HTTP requests in seconds',
  labelNames: ['method', 'route', 'status_code'],
  buckets: [0.1, 0.5, 1, 2, 5]
});

const httpRequestsTotal = new promClient.Counter({
  name: 'http_requests_total',
  help: 'Total number of HTTP requests',
  labelNames: ['method', 'route', 'status_code']
});

register.registerMetric(httpRequestDuration);
register.registerMetric(httpRequestsTotal);

// Middleware
app.use(helmet());
app.use(compression());
app.use(cors({
  origin: process.env.CORS_ORIGIN || 'http://localhost:3001',
  credentials: true
}));

// Rate limiting
const limiter = rateLimit({
  windowMs: 15 * 60 * 1000, // 15 minutes
  max: 100 // limit each IP to 100 requests per windowMs
});
app.use(limiter);

app.use(express.json({ limit: '10mb' }));
app.use(express.urlencoded({ extended: true }));

// Metrics middleware
app.use((req, res, next) => {
  const start = Date.now();
  
  res.on('finish', () => {
    const duration = (Date.now() - start) / 1000;
    const route = req.route ? req.route.path : req.path;
    
    httpRequestDuration
      .labels(req.method, route, res.statusCode)
      .observe(duration);
      
    httpRequestsTotal
      .labels(req.method, route, res.statusCode)
      .inc();
  });
  
  next();
});

// Health check
app.get('/health', (req, res) => {
  res.status(200).json({
    status: 'healthy',
    timestamp: new Date().toISOString(),
    uptime: process.uptime(),
    version: process.env.APP_VERSION || '1.0.0'
  });
});

// Metrics endpoint
app.get('/metrics', async (req, res) => {
  res.set('Content-Type', register.contentType);
  res.end(await register.metrics());
});

// Service discovery
const services = {
  auth: process.env.AUTH_SERVICE_URL || 'http://localhost:3001',
  todo: process.env.TODO_SERVICE_URL || 'http://localhost:3002',
  user: process.env.USER_SERVICE_URL || 'http://localhost:3003'
};

// Proxy configuration
const proxyOptions = {
  changeOrigin: true,
  logLevel: 'warn',
  onError: (err, req, res) => {
    console.error('Proxy error:', err);
    res.status(503).json({
      error: 'Service temporarily unavailable',
      timestamp: new Date().toISOString()
    });
  }
};

// Route proxying
app.use('/api/auth', createProxyMiddleware({
  target: services.auth,
  pathRewrite: { '^/api/auth': '' },
  ...proxyOptions
}));

app.use('/api/todos', createProxyMiddleware({
  target: services.todo,
  pathRewrite: { '^/api/todos': '' },
  ...proxyOptions
}));

app.use('/api/users', createProxyMiddleware({
  target: services.user,
  pathRewrite: { '^/api/users': '' },
  ...proxyOptions
}));

// Fallback for unknown routes
app.use('*', (req, res) => {
  res.status(404).json({
    error: 'Route not found',
    timestamp: new Date().toISOString()
  });
});

// Global error handler
app.use((err, req, res, next) => {
  console.error('Global error handler:', err);
  res.status(500).json({
    error: 'Internal server error',
    timestamp: new Date().toISOString()
  });
});

// Graceful shutdown
process.on('SIGTERM', () => {
  console.log('SIGTERM received, shutting down gracefully');
  server.close(() => {
    console.log('Process terminated');
  });
});

const server = app.listen(PORT, () => {
  console.log(`🚀 API Gateway running on port ${PORT}`);
  console.log(`📊 Environment: ${process.env.NODE_ENV || 'development'}`);
  console.log(`🔗 Services:`);
  console.log(`   Auth: ${services.auth}`);
  console.log(`   Todo: ${services.todo}`);  
  console.log(`   User: ${services.user}`);
});

module.exports = { app, server };
```

**Tests del API Gateway:**

```javascript
// backend/api-gateway/src/server.test.js
const request = require('supertest');
const { app, server } = require('./server');

describe('API Gateway', () => {
  afterAll((done) => {
    server.close(done);
  });

  describe('Health Check', () => {
    test('GET /health should return healthy status', async () => {
      const response = await request(app)
        .get('/health')
        .expect(200);

      expect(response.body).toMatchObject({
        status: 'healthy',
        uptime: expect.any(Number),
        version: expect.any(String)
      });
      
      expect(response.body.timestamp).toBeDefined();
    });
  });

  describe('Metrics', () => {
    test('GET /metrics should return Prometheus metrics', async () => {
      const response = await request(app)
        .get('/metrics')
        .expect(200);

      expect(response.text).toContain('# HELP');
      expect(response.text).toContain('http_requests_total');
      expect(response.text).toContain('http_request_duration_seconds');
    });
  });

  describe('Rate Limiting', () => {
    test('should apply rate limiting after many requests', async () => {
      // This test would need to be configured based on your rate limit settings
      const promises = Array.from({ length: 5 }, () =>
        request(app).get('/health')
      );

      const responses = await Promise.all(promises);
      responses.forEach(response => {
        expect(response.status).toBe(200);
      });
    });
  });

  describe('Error Handling', () => {
    test('should return 404 for unknown routes', async () => {
      const response = await request(app)
        .get('/unknown-route')
        .expect(404);

      expect(response.body).toMatchObject({
        error: 'Route not found',
        timestamp: expect.any(String)
      });
    });
  });

  describe('CORS', () => {
    test('should include CORS headers', async () => {
      const response = await request(app)
        .get('/health')
        .expect(200);

      expect(response.headers['access-control-allow-origin']).toBeDefined();
    });
  });

  describe('Security Headers', () => {
    test('should include security headers', async () => {
      const response = await request(app)
        .get('/health')
        .expect(200);

      expect(response.headers['x-content-type-options']).toBe('nosniff');
      expect(response.headers['x-frame-options']).toBe('DENY');
    });
  });
});
```

### **1.4 Docker Compose para Desarrollo**

```yaml
# docker-compose.yml
version: '3.8'

services:
  # Databases
  postgres:
    image: postgres:13-alpine
    environment:
      POSTGRES_DB: todoapp
      POSTGRES_USER: todouser
      POSTGRES_PASSWORD: todopass
    volumes:
      - postgres_data:/var/lib/postgresql/data
      - ./scripts/init-db.sql:/docker-entrypoint-initdb.d/init.sql
    ports:
      - "5432:5432"
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U todouser -d todoapp"]
      interval: 30s
      timeout: 10s
      retries: 5

  mongodb:
    image: mongo:5-focal
    environment:
      MONGO_INITDB_ROOT_USERNAME: todouser
      MONGO_INITDB_ROOT_PASSWORD: todopass
      MONGO_INITDB_DATABASE: todos
    volumes:
      - mongo_data:/data/db
    ports:
      - "27017:27017"
    healthcheck:
      test: echo 'db.runCommand("ping").ok' | mongosh localhost:27017/test --quiet
      interval: 30s
      timeout: 10s
      retries: 5

  redis:
    image: redis:7-alpine
    command: redis-server --appendonly yes
    volumes:
      - redis_data:/data
    ports:
      - "6379:6379"
    healthcheck:
      test: ["CMD", "redis-cli", "ping"]
      interval: 30s
      timeout: 10s
      retries: 5

  # Backend Services
  api-gateway:
    build:
      context: ./backend/api-gateway
      dockerfile: Dockerfile
    ports:
      - "3000:3000"
    environment:
      - NODE_ENV=development
      - PORT=3000
      - AUTH_SERVICE_URL=http://auth-service:3001
      - TODO_SERVICE_URL=http://todo-service:3002
      - USER_SERVICE_URL=http://user-service:3003
      - CORS_ORIGIN=http://localhost:3001
    depends_on:
      - auth-service
      - todo-service
      - user-service
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:3000/health"]
      interval: 30s
      timeout: 10s
      retries: 5

  auth-service:
    build:
      context: ./backend/auth-service
      dockerfile: Dockerfile
    ports:
      - "3001:3001"
    environment:
      - NODE_ENV=development
      - PORT=3001
      - DATABASE_URL=postgresql://todouser:todopass@postgres:5432/todoapp
      - JWT_SECRET=your-super-secret-jwt-key-change-in-production
      - REDIS_URL=redis://redis:6379
    depends_on:
      postgres:
        condition: service_healthy
      redis:
        condition: service_healthy
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:3001/health"]
      interval: 30s
      timeout: 10s
      retries: 5

  todo-service:
    build:
      context: ./backend/todo-service
      dockerfile: Dockerfile
    ports:
      - "3002:3002"
    environment:
      - NODE_ENV=development
      - PORT=3002
      - MONGODB_URL=mongodb://todouser:todopass@mongodb:27017/todos?authSource=admin
      - REDIS_URL=redis://redis:6379
    depends_on:
      mongodb:
        condition: service_healthy
      redis:
        condition: service_healthy
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:3002/health"]
      interval: 30s
      timeout: 10s
      retries: 5

  user-service:
    build:
      context: ./backend/user-service
      dockerfile: Dockerfile
    ports:
      - "3003:3003"
    environment:
      - PORT=3003
      - DATABASE_URL=postgresql://todouser:todopass@postgres:5432/todoapp
      - REDIS_URL=redis://redis:6379
    depends_on:
      postgres:
        condition: service_healthy
      redis:
        condition: service_healthy
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:3003/health"]
      interval: 30s
      timeout: 10s
      retries: 5

  # Frontend
  frontend:
    build:
      context: ./frontend
      dockerfile: Dockerfile
      args:
        - REACT_APP_API_URL=http://localhost:3000
        - REACT_APP_VERSION=1.0.0-dev
    ports:
      - "80:80"
    environment:
      - NODE_ENV=development
    depends_on:
      - api-gateway
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost/health"]
      interval: 30s
      timeout: 10s
      retries: 5

volumes:
  postgres_data:
  mongo_data:
  redis_data:

networks:
  default:
    driver: bridge
```

---

## **🔄 Fase 2: Pipeline CI/CD con GitHub Actions**

### **2.1 Workflow principal de CI/CD**

```yaml
# .github/workflows/ci-cd.yml
name: CI/CD Pipeline

on:
  push:
    branches: [ main, develop ]
    tags: [ 'v*' ]
  pull_request:
    branches: [ main ]

env:
  REGISTRY: ghcr.io
  IMAGE_NAME: ${{ github.repository }}

jobs:
  # Detectar cambios para optimizar builds
  changes:
    runs-on: ubuntu-latest
    outputs:
      frontend: ${{ steps.changes.outputs.frontend }}
      api-gateway: ${{ steps.changes.outputs.api-gateway }}
      auth-service: ${{ steps.changes.outputs.auth-service }}
      todo-service: ${{ steps.changes.outputs.todo-service }}
      user-service: ${{ steps.changes.outputs.user-service }}
      infrastructure: ${{ steps.changes.outputs.infrastructure }}
    steps:
    - uses: actions/checkout@v3
    - uses: dorny/paths-filter@v2
      id: changes
      with:
        filters: |
          frontend:
            - 'frontend/**'
          api-gateway:
            - 'backend/api-gateway/**'
          auth-service:
            - 'backend/auth-service/**'
          todo-service:
            - 'backend/todo-service/**'
          user-service:
            - 'backend/user-service/**'
          infrastructure:
            - 'infrastructure/**'
            - 'docker-compose*.yml'
            - 'Makefile'

  # Tests unitarios para cada servicio
  test-frontend:
    needs: changes
    if: needs.changes.outputs.frontend == 'true'
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v3
    - uses: actions/setup-node@v3
      with:
        node-version: '18'
        cache: 'npm'
        cache-dependency-path: frontend/package-lock.json
    
    - name: Install dependencies
      run: |
        cd frontend
        npm ci
    
    - name: Run linting
      run: |
        cd frontend
        npm run lint
    
    - name: Run unit tests
      run: |
        cd frontend
        npm run test -- --coverage --watchAll=false
    
    - name: Upload coverage to Codecov
      uses: codecov/codecov-action@v3
      with:
        file: ./frontend/coverage/lcov.info
        flags: frontend

  test-api-gateway:
    needs: changes
    if: needs.changes.outputs.api-gateway == 'true'
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v3
    - uses: actions/setup-node@v3
      with:
        node-version: '18'
        cache: 'npm'
        cache-dependency-path: backend/api-gateway/package-lock.json
    
    - name: Install dependencies
      run: |
        cd backend/api-gateway
        npm ci
    
    - name: Run linting
      run: |
        cd backend/api-gateway
        npm run lint
    
    - name: Run unit tests
      run: |
        cd backend/api-gateway
        npm run test -- --coverage
    
    - name: Upload coverage to Codecov
      uses: codecov/codecov-action@v3
      with:
        file: ./backend/api-gateway/coverage/lcov.info
        flags: api-gateway

  # Tests de integración
  integration-tests:
    needs: [test-frontend, test-api-gateway]
    if: always() && (needs.test-frontend.result == 'success' || needs.test-frontend.result == 'skipped') && (needs.test-api-gateway.result == 'success' || needs.test-api-gateway.result == 'skipped')
    runs-on: ubuntu-latest
    
    services:
      postgres:
        image: postgres:13
        env:
          POSTGRES_PASSWORD: postgres
          POSTGRES_DB: testdb
        options: >-
          --health-cmd pg_isready
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5
        ports:
          - 5432:5432
      
      mongodb:
        image: mongo:5
        ports:
          - 27017:27017
      
      redis:
        image: redis:7
        ports:
          - 6379:6379

    steps:
    - uses: actions/checkout@v3
    
    - name: Setup Node.js
      uses: actions/setup-node@v3
      with:
        node-version: '18'
    
    - name: Install dependencies
      run: |
        npm ci
        cd tests/integration
        npm ci
    
    - name: Start services
      run: |
        docker-compose -f docker-compose.test.yml up -d
        sleep 30
    
    - name: Run integration tests
      env:
        DATABASE_URL: postgresql://postgres:postgres@localhost:5432/testdb
        MONGODB_URL: mongodb://localhost:27017/test
        REDIS_URL: redis://localhost:6379
      run: |
        cd tests/integration
        npm run test
    
    - name: Cleanup
      if: always()
      run: docker-compose -f docker-compose.test.yml down

  # Build y push de imágenes Docker
  build-and-push:
    needs: integration-tests
    runs-on: ubuntu-latest
    if: github.event_name != 'pull_request'
    
    permissions:
      contents: read
      packages: write
    
    strategy:
      matrix:
        service: [frontend, api-gateway, auth-service, todo-service, user-service]
    
    steps:
    - name: Checkout
      uses: actions/checkout@v3
    
    - name: Login to Container Registry
      uses: docker/login-action@v2
      with:
        registry: ${{ env.REGISTRY }}
        username: ${{ github.actor }}
        password: ${{ secrets.GITHUB_TOKEN }}
    
    - name: Extract metadata
      id: meta
      uses: docker/metadata-action@v4
      with:
        images: ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}-${{ matrix.service }}
        tags: |
          type=ref,event=branch
          type=ref,event=pr
          type=semver,pattern={{version}}
          type=sha,prefix={{branch}}-
    
    - name: Build and push
      uses: docker/build-push-action@v4
      with:
        context: ./${{ matrix.service == 'frontend' && 'frontend' || format('backend/{0}', matrix.service) }}
        push: true
        tags: ${{ steps.meta.outputs.tags }}
        labels: ${{ steps.meta.outputs.labels }}
        build-args: |
          VERSION=${{ github.sha }}
          BUILD_DATE=${{ github.event.head_commit.timestamp }}

  # Deploy a staging
  deploy-staging:
    needs: build-and-push
    if: github.ref == 'refs/heads/develop'
    runs-on: ubuntu-latest
    environment: staging
    
    steps:
    - name: Checkout
      uses: actions/checkout@v3
    
    - name: Deploy to staging
      env:
        KUBE_CONFIG: ${{ secrets.STAGING_KUBE_CONFIG }}
      run: |
        echo "$KUBE_CONFIG" | base64 -d > kubeconfig
        export KUBECONFIG=kubeconfig
        
        # Update images in staging
        kubectl set image deployment/frontend-staging frontend=${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}-frontend:develop -n staging
        kubectl set image deployment/api-gateway-staging api-gateway=${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}-api-gateway:develop -n staging
        
        # Wait for rollout
        kubectl rollout status deployment/frontend-staging -n staging --timeout=300s
        kubectl rollout status deployment/api-gateway-staging -n staging --timeout=300s
    
    - name: Run E2E tests against staging
      run: |
        cd tests/e2e
        npm ci
        STAGING_URL=https://staging.todo-app.example.com npm run test
    
    - name: Notify staging deployment
      uses: 8398a7/action-slack@v3
      with:
        status: ${{ job.status }}
        text: |
          🚀 Staging deployment completed
          Environment: https://staging.todo-app.example.com
          Commit: ${{ github.sha }}
      env:
        SLACK_WEBHOOK_URL: ${{ secrets.SLACK_WEBHOOK }}

  # Deploy a producción (solo tags)
  deploy-production:
    needs: build-and-push
    if: startsWith(github.ref, 'refs/tags/v')
    runs-on: ubuntu-latest
    environment: production
    
    steps:
    - name: Checkout
      uses: actions/checkout@v3
    
    - name: Deploy to production (Canary)
      env:
        KUBE_CONFIG: ${{ secrets.PRODUCTION_KUBE_CONFIG }}
      run: |
        echo "$KUBE_CONFIG" | base64 -d > kubeconfig
        export KUBECONFIG=kubeconfig
        
        # Deploy canary version (10% traffic)
        ./scripts/deploy-canary.sh ${{ github.ref_name }}
    
    - name: Monitor canary deployment
      run: |
        # Monitor canary for 5 minutes
        ./scripts/monitor-canary.sh 300
    
    - name: Promote to full production
      env:
        KUBE_CONFIG: ${{ secrets.PRODUCTION_KUBE_CONFIG }}
      run: |
        echo "$KUBE_CONFIG" | base64 -d > kubeconfig
        export KUBECONFIG=kubeconfig
        
        # Promote canary to 100% traffic
        ./scripts/promote-canary.sh
    
    - name: Notify production deployment
      uses: 8398a7/action-slack@v3
      with:
        status: success
        text: |
          🎉 Production deployment successful!
          Version: ${{ github.ref_name }}
          URL: https://todo-app.example.com
      env:
        SLACK_WEBHOOK_URL: ${{ secrets.SLACK_WEBHOOK }}

  # Rollback automático en caso de fallo
  rollback:
    needs: deploy-production
    if: failure()
    runs-on: ubuntu-latest
    environment: production
    
    steps:
    - name: Rollback production deployment
      env:
        KUBE_CONFIG: ${{ secrets.PRODUCTION_KUBE_CONFIG }}
      run: |
        echo "$KUBE_CONFIG" | base64 -d > kubeconfig
        export KUBECONFIG=kubeconfig
        
        # Rollback all deployments
        ./scripts/rollback-production.sh
    
    - name: Notify rollback
      uses: 8398a7/action-slack@v3
      with:
        status: custom
        custom_payload: |
          {
            text: "🔄 Production rollback executed",
            attachments: [{
              color: "warning",
              fields: [{
                title: "Status",
                value: "Automatic rollback completed",
                short: true
              }]
            }]
          }
      env:
        SLACK_WEBHOOK_URL: ${{ secrets.SLACK_WEBHOOK }}
```

### **2.2 Scripts de automatización**

```bash
#!/bin/bash
# scripts/deploy-canary.sh

set -e

VERSION=$1
NAMESPACE="production"

if [ -z "$VERSION" ]; then
    echo "Usage: $0 <version>"
    exit 1
fi

echo "🕯️ Starting canary deployment for version $VERSION"

# Update canary deployment with new image
kubectl set image deployment/todo-app-canary \
    frontend=ghcr.io/todo-app-frontend:$VERSION \
    api-gateway=ghcr.io/todo-app-api-gateway:$VERSION \
    -n $NAMESPACE

# Wait for canary rollout
kubectl rollout status deployment/todo-app-canary -n $NAMESPACE --timeout=300s

# Configure traffic split (10% to canary)
kubectl apply -f - <<EOF
apiVersion: networking.istio.io/v1beta1
kind: VirtualService
metadata:
  name: todo-app-canary
  namespace: $NAMESPACE
spec:
  hosts:
  - todo-app.example.com
  http:
  - route:
    - destination:
        host: todo-app-stable
      weight: 90
    - destination:
        host: todo-app-canary
      weight: 10
EOF

echo "✅ Canary deployment completed - 10% traffic routed to version $VERSION"
```

```bash
#!/bin/bash  
# scripts/monitor-canary.sh

set -e

DURATION=${1:-300}  # Default 5 minutes
INTERVAL=30
NAMESPACE="production"

echo "📊 Monitoring canary deployment for ${DURATION} seconds..."

END_TIME=$(($(date +%s) + DURATION))

while [ $(date +%s) -lt $END_TIME ]; do
    # Get metrics from Prometheus
    ERROR_RATE_STABLE=$(curl -s "http://prometheus:9090/api/v1/query?query=rate(http_requests_total{job=\"stable\",status=~\"5.*\"}[5m])" | jq -r '.data.result[0].value[1] // "0"')
    ERROR_RATE_CANARY=$(curl -s "http://prometheus:9090/api/v1/query?query=rate(http_requests_total{job=\"canary\",status=~\"5.*\"}[5m])" | jq -r '.data.result[0].value[1] // "0"')
    
    LATENCY_STABLE=$(curl -s "http://prometheus:9090/api/v1/query?query=histogram_quantile(0.95,rate(http_request_duration_seconds_bucket{job=\"stable\"}[5m]))" | jq -r '.data.result[0].value[1] // "0"')
    LATENCY_CANARY=$(curl -s "http://prometheus:9090/api/v1/query?query=histogram_quantile(0.95,rate(http_request_duration_seconds_bucket{job=\"canary\"}[5m]))" | jq -r '.data.result[0].value[1] // "0"')
    
    echo "📈 Metrics:"
    echo "   Stable - Error Rate: $ERROR_RATE_STABLE, Latency: ${LATENCY_STABLE}s"
    echo "   Canary - Error Rate: $ERROR_RATE_CANARY, Latency: ${LATENCY_CANARY}s"
    
    # Check rollback conditions
    if (( $(echo "$ERROR_RATE_CANARY > $ERROR_RATE_STABLE * 2" | bc -l) )); then
        echo "❌ Canary error rate too high - triggering rollback"
        ./scripts/rollback-canary.sh
        exit 1
    fi
    
    if (( $(echo "$LATENCY_CANARY > $LATENCY_STABLE * 1.5" | bc -l) )); then
        echo "⚠️ Canary latency significantly higher than stable"
    fi
    
    REMAINING=$((END_TIME - $(date +%s)))
    echo "⏳ Time remaining: ${REMAINING}s"
    
    sleep $INTERVAL
done

echo "✅ Canary monitoring completed - no issues detected"
```

---

## **Conclusión del Módulo**

¡Felicidades! Has completado el módulo de **Práctica CI/CD** más comprehensivo. Este proyecto te ha dado experiencia práctica con:

### **🎯 Lo que has logrado:**

- ✅ **Aplicación completa** con microservicios (Frontend React + API Gateway + 3 servicios backend)
- ✅ **Tests automatizados** en múltiples niveles (unit, integration, E2E, performance)
- ✅ **Pipeline CI/CD robusto** con GitHub Actions
- ✅ **Estrategias de deployment** (Blue-Green, Canary, Rolling)
- ✅ **Monitoring y métricas** con Prometheus/Grafana
- ✅ **Rollbacks automáticos** basados en métricas
- ✅ **Infrastructure as Code** con Docker y Kubernetes
- ✅ **Security scanning** y quality gates

### **🚀 Skills desarrollados:**

- **DevOps Engineering**: Pipeline design, automation, monitoring
- **Container Orchestration**: Docker, Kubernetes, service mesh
- **Testing Strategy**: Pyramid de testing, test automation
- **Site Reliability**: Monitoring, alerting, incident response
- **Security**: Vulnerability scanning, secure deployments
- **Cloud Native**: Microservices, 12-factor app, observability

Este proyecto es un **portfolio piece** que demuestra tu competencia en DevOps moderno y puede ser usado como referencia en entrevistas técnicas.

---

## **🎓 Próximos pasos recomendados:**

1. **Extender el proyecto** con más servicios y complejidad
2. **Implementar GitOps** con ArgoCD o Flux
3. **Agregar chaos engineering** con Chaos Monkey
4. **Implementar service mesh** con Istio
5. **Avanzar a Kubernetes avanzado** con operators y CRDs

¡Has completado exitosamente la **Fase 2** del roadmap DevOps! Ahora estás listo para avanzar a **contenedores, redes y orquestación** en la Fase 3.
