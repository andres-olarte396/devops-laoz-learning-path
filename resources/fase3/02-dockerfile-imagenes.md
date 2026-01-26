# Dockerfile: Crear y Gestionar Imágenes

## 🎯 Objetivos de Aprendizaje

Al finalizar este tema serás capaz de:
- Escribir Dockerfiles eficientes y optimizados
- Entender las instrucciones fundamentales de Dockerfile
- Construir imágenes personalizadas paso a paso
- Aplicar mejores prácticas en la construcción de imágenes
- Gestionar el cache de capas para builds rápidos

## 📄 ¿Qué es un Dockerfile?

Un **Dockerfile** es un archivo de texto que contiene una serie de instrucciones para construir una imagen de Docker de forma automatizada y reproducible.

### 🏗️ Estructura Básica

```dockerfile
# Comentario
INSTRUCCION argumentos
```

### 📋 Ejemplo Simple

```dockerfile
# Usar imagen base
FROM ubuntu:20.04

# Instalar dependencias
RUN apt-get update && apt-get install -y python3

# Copiar código
COPY app.py /app/

# Establecer directorio de trabajo
WORKDIR /app

# Exponer puerto
EXPOSE 8000

# Comando por defecto
CMD ["python3", "app.py"]
```

## 📚 Instrucciones Fundamentales

### FROM - Imagen Base

```dockerfile
# Imagen específica con tag
FROM node:16-alpine

# Imagen oficial de Python
FROM python:3.9-slim

# Imagen multipropósito
FROM ubuntu:22.04

# Construcción multi-stage
FROM node:16 AS builder
```

### RUN - Ejecutar Comandos

```dockerfile
# Comando simple
RUN apt-get update

# Múltiples comandos en una capa
RUN apt-get update && \
    apt-get install -y \
        curl \
        git \
        vim && \
    apt-get clean

# Usando JSON array (recomendado para Windows)
RUN ["apt-get", "update"]
```

### COPY vs ADD

```dockerfile
# COPY - Copia archivos/directorios simples
COPY src/ /app/src/
COPY package.json /app/

# ADD - Funcionalidades adicionales (extraer tar, URLs)
ADD https://github.com/user/repo/archive/main.tar.gz /tmp/
ADD archivo.tar.gz /app/

# Mejores prácticas: usar COPY cuando sea posible
COPY requirements.txt /app/
```

### WORKDIR - Directorio de Trabajo

```dockerfile
# Establecer directorio de trabajo
WORKDIR /app

# Rutas relativas funcionan a partir de aquí
COPY . .
RUN npm install

# Equivale a:
# RUN cd /app && npm install
```

### ENV - Variables de Entorno

```dockerfile
# Definir variables
ENV NODE_ENV=production
ENV PORT=3000
ENV DATABASE_URL=postgresql://localhost/myapp

# Múltiples variables
ENV NODE_ENV=production \
    PORT=3000 \
    DEBUG=false

# Usar variables en instrucciones
RUN echo "Environment: $NODE_ENV"
```

### ARG - Argumentos de Construcción

```dockerfile
# Definir argumentos
ARG VERSION=latest
ARG BUILD_DATE

# Usar en FROM
FROM node:${VERSION}

# Usar en instrucciones
RUN echo "Build date: $BUILD_DATE"

# Con valores por defecto
ARG PORT=8080
EXPOSE $PORT
```

```bash
# Pasar argumentos en build
docker build --build-arg VERSION=16 --build-arg BUILD_DATE=$(date) .
```

### EXPOSE - Documentar Puertos

```dockerfile
# Documentar puerto que usa la aplicación
EXPOSE 8080

# Múltiples puertos
EXPOSE 8080 8443

# Con protocolo específico
EXPOSE 53/udp
EXPOSE 80/tcp
```

### VOLUME - Puntos de Montaje

```dockerfile
# Definir volúmenes
VOLUME ["/data"]
VOLUME ["/var/log", "/etc/config"]

# Para directorios que cambian frecuentemente
VOLUME /app/uploads
```

### USER - Cambiar Usuario

```dockerfile
# Crear usuario no-root
RUN addgroup -g 1001 -S nodejs && \
    adduser -S nextjs -u 1001

# Cambiar a usuario no-root
USER nextjs

# O usar UID directamente
USER 1001
```

### CMD vs ENTRYPOINT

```dockerfile
# CMD - Comando por defecto (puede ser sobrescrito)
CMD ["npm", "start"]
CMD ["python", "app.py"]

# ENTRYPOINT - Punto de entrada fijo
ENTRYPOINT ["python", "app.py"]

# Combinando ENTRYPOINT + CMD
ENTRYPOINT ["python"]
CMD ["app.py"]

# Uso flexible
ENTRYPOINT ["./entrypoint.sh"]
CMD ["--help"]
```

## 🔧 Ejemplos Prácticos

### Aplicación Node.js

```dockerfile
# Usar imagen base oficial
FROM node:16-alpine

# Crear directorio de aplicación
WORKDIR /usr/src/app

# Copiar archivos de dependencias
COPY package*.json ./

# Instalar dependencias
RUN npm ci --only=production

# Copiar código fuente
COPY . .

# Crear usuario no-root
RUN addgroup -g 1001 -S nodejs && \
    adduser -S nextjs -u 1001

# Cambiar ownership de archivos
RUN chown -R nextjs:nodejs /usr/src/app
USER nextjs

# Exponer puerto
EXPOSE 3000

# Comando de inicio
CMD ["npm", "start"]
```

### Aplicación Python Flask

```dockerfile
FROM python:3.9-slim

# Instalar dependencias del sistema
RUN apt-get update && apt-get install -y \
    gcc \
    && rm -rf /var/lib/apt/lists/*

# Establecer directorio de trabajo
WORKDIR /app

# Copiar requirements primero (mejor cache)
COPY requirements.txt .

# Instalar dependencias Python
RUN pip install --no-cache-dir -r requirements.txt

# Copiar código de aplicación
COPY . .

# Crear usuario no-root
RUN useradd --create-home --shell /bin/bash app
USER app

# Variables de entorno
ENV FLASK_APP=app.py
ENV FLASK_ENV=production

# Exponer puerto
EXPOSE 5000

# Comando de inicio
CMD ["gunicorn", "--bind", "0.0.0.0:5000", "app:app"]
```

### Aplicación Java Spring Boot

```dockerfile
# Multi-stage build
FROM maven:3.8-openjdk-17 AS builder

WORKDIR /app

# Copiar pom.xml primero para mejor cache
COPY pom.xml .
RUN mvn dependency:download

# Copiar código y compilar
COPY src ./src
RUN mvn clean package -DskipTests

# Imagen de runtime
FROM openjdk:17-jre-slim

WORKDIR /app

# Copiar JAR desde builder stage
COPY --from=builder /app/target/*.jar app.jar

# Crear usuario no-root
RUN useradd --create-home --shell /bin/bash spring
USER spring

# Exponer puerto
EXPOSE 8080

# Configurar JVM
ENV JAVA_OPTS="-Xmx512m -Xms256m"

# Comando de inicio
ENTRYPOINT ["sh", "-c", "java $JAVA_OPTS -jar app.jar"]
```

## 🏗️ Construcción Multi-Stage

Las construcciones multi-stage permiten optimizar el tamaño final de la imagen.

### Ejemplo Avanzado

```dockerfile
# Stage 1: Construcción
FROM node:16 AS builder

WORKDIR /app
COPY package*.json ./
RUN npm ci

COPY . .
RUN npm run build

# Stage 2: Producción
FROM nginx:alpine AS production

# Copiar archivos construidos desde builder
COPY --from=builder /app/dist /usr/share/nginx/html

# Copiar configuración de nginx
COPY nginx.conf /etc/nginx/nginx.conf

EXPOSE 80

CMD ["nginx", "-g", "daemon off;"]

# Stage 3: Desarrollo (opcional)
FROM node:16 AS development

WORKDIR /app
COPY package*.json ./
RUN npm install

COPY . .
EXPOSE 3000

CMD ["npm", "run", "dev"]
```

```bash
# Construir stage específico
docker build --target development -t mi-app:dev .
docker build --target production -t mi-app:prod .
```

## 📊 Optimización de Imágenes

### 1. Usar Imágenes Base Pequeñas

```dockerfile
# ❌ Imagen grande
FROM ubuntu:20.04

# ✅ Imagen optimizada
FROM alpine:3.16

# ✅ Imagen oficial optimizada
FROM node:16-alpine
FROM python:3.9-slim
```

### 2. Minimizar Capas

```dockerfile
# ❌ Múltiples capas RUN
RUN apt-get update
RUN apt-get install -y curl
RUN apt-get install -y git
RUN apt-get clean

# ✅ Una sola capa RUN
RUN apt-get update && \
    apt-get install -y \
        curl \
        git && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*
```

### 3. Usar .dockerignore

```dockerignore
# .dockerignore
node_modules
npm-debug.log
.git
.gitignore
README.md
.env
.nyc_output
coverage
.DS_Store
*.log
```

### 4. Aprovechar Cache de Capas

```dockerfile
# ✅ Copiar dependencies primero
COPY package.json package-lock.json ./
RUN npm ci

# Copiar código después (cambia más frecuentemente)
COPY . .
```

### 5. Multi-stage para Herramientas de Build

```dockerfile
# Builder stage con herramientas pesadas
FROM node:16 AS builder
COPY . .
RUN npm run build

# Runtime stage minimalista
FROM nginx:alpine
COPY --from=builder /app/dist /usr/share/nginx/html
```

## 🔍 Debugging de Dockerfiles

### Construcción Paso a Paso

```bash
# Ver cada paso de la construcción
docker build --progress=plain .

# Sin usar cache
docker build --no-cache .

# Hasta cierto stage
docker build --target builder .
```

### Inspeccionar Capas

```bash
# Ver historial de capas
docker history mi-imagen:tag

# Inspeccionar imagen
docker inspect mi-imagen:tag

# Tamaño de imagen
docker images mi-imagen:tag
```

### Debug Interactivo

```dockerfile
# Añadir punto de debug
FROM ubuntu:20.04
RUN apt-get update
# Debug point: docker run -it <image-id> bash
RUN apt-get install -y python3
```

## 🛠️ Ejercicio Práctico

### Crear una Aplicación Web Completa

```dockerfile
# Dockerfile para aplicación Flask
FROM python:3.9-slim

# Metadatos
LABEL maintainer="tu-email@ejemplo.com" \
      version="1.0" \
      description="Aplicación Flask de ejemplo"

# Variables de build
ARG APP_ENV=production

# Variables de runtime
ENV FLASK_APP=app.py \
    FLASK_ENV=$APP_ENV \
    PYTHONPATH=/app

# Instalar dependencias del sistema
RUN apt-get update && apt-get install -y \
    gcc \
    curl \
    && rm -rf /var/lib/apt/lists/*

# Crear directorio de aplicación
WORKDIR /app

# Copiar y instalar dependencias Python
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# Crear usuario no-root
RUN useradd --create-home --shell /bin/bash flask && \
    chown -R flask:flask /app

# Copiar código de aplicación
COPY --chown=flask:flask . .

# Cambiar a usuario no-root
USER flask

# Crear directorio para datos
VOLUME ["/app/data"]

# Exponer puerto
EXPOSE 5000

# Health check
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
    CMD curl -f http://localhost:5000/health || exit 1

# Comando de inicio
CMD ["gunicorn", "--bind", "0.0.0.0:5000", "--workers", "2", "app:app"]
```

### Comandos de Construcción

```bash
# Construir imagen
docker build -t mi-flask-app:v1.0 .

# Con argumentos personalizados
docker build \
    --build-arg APP_ENV=development \
    -t mi-flask-app:dev .

# Ejecutar contenedor
docker run -d \
    --name flask-container \
    -p 8080:5000 \
    -v $(pwd)/data:/app/data \
    mi-flask-app:v1.0
```

## 📝 Mejores Prácticas

1. **Usa imágenes base oficiales** y específicas
2. **Minimiza el número de capas** combinando comandos RUN
3. **Usa .dockerignore** para excluir archivos innecesarios
4. **Copia dependencias antes que código** para mejor cache
5. **Ejecuta como usuario no-root** por seguridad
6. **Usa multi-stage builds** para imágenes optimizadas
7. **Añade labels** para documentación
8. **Implementa health checks** para monitoreo
9. **Limpia archivos temporales** en la misma capa RUN
10. **Especifica versiones exactas** de dependencias

## 🎯 Resumen

Has aprendido:
- [x] Sintaxis y estructura de Dockerfiles
- [x] Instrucciones fundamentales y su uso
- [x] Técnicas de optimización de imágenes
- [x] Construcciones multi-stage
- [x] Mejores prácticas de seguridad
- [x] Debugging y troubleshooting

## 📚 Recursos Adicionales

- [Dockerfile Reference](https://docs.docker.com/engine/reference/builder/)
- [Best Practices for Writing Dockerfiles](https://docs.docker.com/develop/dev-best-practices/)
- [Multi-stage Builds](https://docs.docker.com/develop/advanced/multistage-build/)
- [Dockerfile Linter](https://github.com/hadolint/hadolint)

---

**Próximo tema:** [Docker Compose para múltiples contenedores](./03-docker-compose.md)