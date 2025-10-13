# Optimización de Imágenes Docker

## 🎯 Objetivos de Aprendizaje

Al finalizar este tema serás capaz de:
- Comprender cómo funcionan las capas en Docker
- Aplicar técnicas avanzadas de optimización de imágenes
- Reducir significativamente el tamaño de las imágenes
- Implementar estrategias de seguridad en imágenes
- Usar herramientas de análisis y optimización

## 📚 Fundamentos de Optimización

### ¿Por qué Optimizar Imágenes?

1. **Velocidad de despliegue**: Imágenes más pequeñas se transfieren más rápido
2. **Consumo de almacenamiento**: Menor uso de disco y ancho de banda
3. **Superficie de ataque**: Menos componentes = menos vulnerabilidades
4. **Costos**: Menores costos de almacenamiento y transferencia
5. **Experiencia del desarrollador**: Builds y pulls más rápidos

### 🏗️ Cómo Funcionan las Capas

```bash
# Cada instrucción en Dockerfile crea una nueva capa
FROM ubuntu:20.04          # Capa 1: Sistema base
RUN apt-get update         # Capa 2: Actualización de paquetes  
RUN apt-get install -y git # Capa 3: Instalación de git
COPY app.py /app/          # Capa 4: Copia de archivo
CMD ["python", "/app/app.py"] # Capa 5: Comando por defecto
```

```bash
# Ver capas de una imagen
docker history mi-imagen:tag
```

## 🔧 Técnicas de Optimización

### 1. Usar Imágenes Base Minimalistas

```dockerfile
# ❌ Imagen completa (Ubuntu ~72MB base)
FROM ubuntu:20.04
RUN apt-get update && apt-get install -y python3

# ✅ Imagen Alpine (~5MB base)
FROM python:3.9-alpine
# Alpine ya incluye Python

# ✅ Imagen distroless (sin shell, ~2MB base)
FROM gcr.io/distroless/python3
COPY app.py /
CMD ["/app.py"]
```

### Comparación de Tamaños Base

| Imagen Base | Tamaño | Uso Recomendado |
|-------------|---------|-----------------|
| `ubuntu:20.04` | ~72MB | Desarrollo, debugging |
| `python:3.9-slim` | ~45MB | Producción equilibrada |
| `python:3.9-alpine` | ~15MB | Producción ligera |
| `gcr.io/distroless/python3` | ~2MB | Máxima seguridad |

### 2. Minimizar Capas RUN

```dockerfile
# ❌ Múltiples capas RUN
FROM alpine:3.16
RUN apk update
RUN apk add --no-cache curl
RUN apk add --no-cache git
RUN apk add --no-cache python3
RUN rm -rf /var/cache/apk/*

# ✅ Una sola capa RUN optimizada
FROM alpine:3.16
RUN apk update && \
    apk add --no-cache \
        curl \
        git \
        python3 && \
    rm -rf /var/cache/apk/*
```

### 3. Orden Óptimo de Instrucciones

```dockerfile
# ✅ Orden optimizado para cache
FROM node:16-alpine

# 1. Copiar archivos de dependencias primero
COPY package*.json ./

# 2. Instalar dependencias (se cachea si no cambian)
RUN npm ci --only=production

# 3. Copiar código fuente al final (cambia frecuentemente)
COPY . .

# 4. Comando de ejecución
CMD ["npm", "start"]
```

### 4. Usar .dockerignore Efectivo

```dockerignore
# .dockerignore - excluir archivos innecesarios

# Control de versiones
.git
.gitignore
.svn

# Dependencias de desarrollo
node_modules
bower_components
npm-debug.log*

# Archivos de IDE
.vscode
.idea
*.swp
*.swo
*~

# Archivos del sistema
.DS_Store
Thumbs.db

# Documentación
README.md
CHANGELOG.md
docs/

# Archivos de testing
test/
tests/
spec/
coverage/
.nyc_output

# Archivos de configuración local
.env.local
.env.development
config/local.json

# Archivos temporales
tmp/
temp/
*.tmp
*.log

# Archivos de build
dist/
build/
target/

# Imágenes Docker
Dockerfile*
docker-compose*
```

## 🏗️ Construcciones Multi-Stage Avanzadas

### Ejemplo: Aplicación Go

```dockerfile
# Stage 1: Construcción
FROM golang:1.19-alpine AS builder

WORKDIR /app

# Copiar go mod files
COPY go.mod go.sum ./
RUN go mod download

# Copiar código fuente
COPY . .

# Compilar aplicación estática
RUN CGO_ENABLED=0 GOOS=linux go build -a -installsuffix cgo -o main .

# Stage 2: Imagen final minimalista
FROM alpine:3.16

# Instalar certificados SSL y crear usuario
RUN apk --no-cache add ca-certificates && \
    addgroup -g 1001 -S appgroup && \
    adduser -S appuser -u 1001 -G appgroup

WORKDIR /root/

# Copiar binario desde builder
COPY --from=builder /app/main .

# Cambiar a usuario no-root
USER appuser

# Exponer puerto
EXPOSE 8080

# Ejecutar aplicación
CMD ["./main"]
```

### Resultado de Optimización

```bash
# Antes: Imagen con herramientas de build
# golang:1.19 (~900MB) + dependencias (~100MB) = ~1GB

# Después: Solo runtime
# alpine:3.16 (~5MB) + binario (~10MB) = ~15MB

# Reducción: 98.5% del tamaño original
```

### Ejemplo: Aplicación Node.js

```dockerfile
# Stage 1: Instalar dependencias de desarrollo
FROM node:16-alpine AS deps
WORKDIR /app
COPY package*.json ./
RUN npm ci --include=dev

# Stage 2: Construcción de la aplicación
FROM node:16-alpine AS builder
WORKDIR /app
COPY --from=deps /app/node_modules ./node_modules
COPY . .
RUN npm run build

# Stage 3: Instalar solo dependencias de producción
FROM node:16-alpine AS prod-deps
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production && npm cache clean --force

# Stage 4: Imagen final de runtime
FROM node:16-alpine AS runtime

# Crear usuario no-root
RUN addgroup -g 1001 -S nodejs && \
    adduser -S nextjs -u 1001

WORKDIR /app

# Copiar archivos necesarios
COPY --from=prod-deps /app/node_modules ./node_modules
COPY --from=builder /app/dist ./dist
COPY --from=builder /app/public ./public

# Cambiar ownership
RUN chown -R nextjs:nodejs /app
USER nextjs

EXPOSE 3000

CMD ["node", "dist/server.js"]
```

## 🛡️ Imágenes Distroless y Scratch

### Usando Scratch (Solo para binarios estáticos)

```dockerfile
# Para aplicaciones Go compiladas estáticamente
FROM golang:1.19-alpine AS builder
WORKDIR /app
COPY . .
RUN CGO_ENABLED=0 go build -ldflags="-w -s" -o app main.go

# Imagen final vacía
FROM scratch
COPY --from=builder /app/app /app
EXPOSE 8080
ENTRYPOINT ["/app"]
```

### Usando Imágenes Distroless

```dockerfile
# Para aplicaciones Java
FROM maven:3.8-openjdk-17 AS builder
WORKDIR /app
COPY pom.xml .
RUN mvn dependency:download
COPY src ./src
RUN mvn package -DskipTests

# Imagen distroless con JRE
FROM gcr.io/distroless/java17-debian11
COPY --from=builder /app/target/*.jar /app.jar
EXPOSE 8080
ENTRYPOINT ["java", "-jar", "/app.jar"]
```

### Ventajas de Distroless

- **Sin shell**: Imposible ejecutar comandos maliciosos
- **Sin package manager**: No se pueden instalar herramientas adicionales
- **Mínima superficie de ataque**: Solo runtime necesario
- **Pequeño tamaño**: Solo librerías esenciales

## 🔍 Herramientas de Análisis

### Docker Image Analysis

```bash
# Analizar capas de imagen
docker history --no-trunc mi-imagen:tag

# Ver tamaño de cada capa
docker history mi-imagen:tag --format "table {{.CreatedBy}}\t{{.Size}}"

# Inspeccionar configuración
docker inspect mi-imagen:tag
```

### Dive - Análisis Detallado

```bash
# Instalar dive
curl -OL https://github.com/wagoodman/dive/releases/download/v0.10.0/dive_0.10.0_linux_amd64.deb
sudo dpkg -i dive_0.10.0_linux_amd64.deb

# Analizar imagen
dive mi-imagen:tag
```

### Hadolint - Linting de Dockerfiles

```bash
# Instalar hadolint
docker pull hadolint/hadolint

# Analizar Dockerfile
docker run --rm -i hadolint/hadolint < Dockerfile

# Con archivo de configuración
docker run --rm -i \
  -v $(pwd)/.hadolint.yaml:/hadolint.yaml \
  hadolint/hadolint hadolint --config /hadolint.yaml Dockerfile
```

### Trivy - Scanner de Vulnerabilidades

```bash
# Instalar trivy
curl -sfL https://raw.githubusercontent.com/aquasecurity/trivy/main/contrib/install.sh | sh -s -- -b /usr/local/bin

# Escanear imagen
trivy image mi-imagen:tag

# Solo vulnerabilidades críticas y altas
trivy image --severity HIGH,CRITICAL mi-imagen:tag

# Generar reporte JSON
trivy image --format json --output result.json mi-imagen:tag
```

## 📊 Técnicas Específicas por Lenguaje

### Python

```dockerfile
# Optimización para Python
FROM python:3.9-slim AS builder

# Instalar dependencias del sistema
RUN apt-get update && apt-get install -y \
    build-essential \
    gcc \
    && rm -rf /var/lib/apt/lists/*

# Instalar dependencias Python
COPY requirements.txt .
RUN pip install --user --no-cache-dir -r requirements.txt

# Imagen final
FROM python:3.9-slim
RUN useradd --create-home --shell /bin/bash app
WORKDIR /home/app

# Copiar dependencias instaladas
COPY --from=builder /root/.local /home/app/.local

# Copiar código
COPY --chown=app:app . .

USER app

# Agregar .local/bin al PATH
ENV PATH=/home/app/.local/bin:$PATH

CMD ["python", "app.py"]
```

### Node.js

```dockerfile
# Optimización para Node.js
FROM node:16-alpine AS builder

WORKDIR /app

# Usar npm ci para instalación determinística
COPY package*.json ./
RUN npm ci --only=production

# Imagen final
FROM node:16-alpine

# Crear usuario no-root
RUN addgroup -g 1001 -S nodejs && \
    adduser -S nextjs -u 1001

WORKDIR /app

# Copiar node_modules optimizados
COPY --from=builder --chown=nextjs:nodejs /app/node_modules ./node_modules

# Copiar código de aplicación
COPY --chown=nextjs:nodejs . .

USER nextjs

# Usar tini para manejo de señales
RUN apk add --no-cache tini
ENTRYPOINT ["/sbin/tini", "--"]

CMD ["node", "server.js"]
```

### Java

```dockerfile
# Optimización para Java Spring Boot
FROM openjdk:17-jdk-slim AS builder

WORKDIR /app

# Copiar archivos de configuración
COPY pom.xml .
COPY .mvn .mvn
COPY mvnw .

# Descargar dependencias (mejor cache)
RUN ./mvnw dependency:download

# Copiar código fuente
COPY src ./src

# Compilar aplicación
RUN ./mvnw package -DskipTests

# Extraer JAR layers para mejor cache
RUN java -Djarmode=layertools -jar target/*.jar extract

# Imagen final optimizada
FROM openjdk:17-jre-slim

# Crear usuario no-root
RUN useradd --create-home --shell /bin/bash spring

WORKDIR /app

# Copiar layers en orden de frecuencia de cambio
COPY --from=builder --chown=spring:spring /app/dependencies/ ./
COPY --from=builder --chown=spring:spring /app/spring-boot-loader/ ./
COPY --from=builder --chown=spring:spring /app/snapshot-dependencies/ ./
COPY --from=builder --chown=spring:spring /app/application/ ./

USER spring

# Configurar JVM para contenedores
ENV JAVA_OPTS="-XX:+UseContainerSupport -XX:MaxRAMPercentage=75.0"

ENTRYPOINT ["sh", "-c", "java $JAVA_OPTS org.springframework.boot.loader.JarLauncher"]
```

## 🔒 Optimización de Seguridad

### Principios de Seguridad

```dockerfile
# 1. Usuario no-root
FROM alpine:3.16
RUN adduser -D -s /bin/sh appuser
USER appuser

# 2. Filesystem de solo lectura
# docker run --read-only --tmpfs /tmp mi-imagen

# 3. Eliminar herramientas innecesarias
RUN apk del --purge \
    wget \
    curl \
    bash

# 4. Usar COPY en lugar de ADD
COPY app.py /app/

# 5. Especificar puertos explícitamente
EXPOSE 8080/tcp
```

### Escaner de Vulnerabilidades en CI/CD

```yaml
# .github/workflows/security.yml
name: Security Scan

on: [push, pull_request]

jobs:
  security:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Build image
        run: docker build -t myapp:${{ github.sha }} .
      
      - name: Scan with Trivy
        run: |
          docker run --rm -v /var/run/docker.sock:/var/run/docker.sock \
            aquasec/trivy:latest image \
            --exit-code 1 \
            --severity HIGH,CRITICAL \
            myapp:${{ github.sha }}
```

## 📏 Medición de Resultados

### Script de Análisis

```bash
#!/bin/bash
# analyze-image.sh

IMAGE=$1

echo "=== Análisis de Imagen: $IMAGE ==="
echo

echo "📊 Tamaño de imagen:"
docker images $IMAGE --format "table {{.Repository}}\t{{.Tag}}\t{{.Size}}"
echo

echo "🏗️ Capas de la imagen:"
docker history $IMAGE --format "table {{.CreatedBy}}\t{{.Size}}" | head -10
echo

echo "🔍 Vulnerabilidades (requiere trivy):"
if command -v trivy &> /dev/null; then
    trivy image --severity HIGH,CRITICAL --quiet $IMAGE
else
    echo "Trivy no instalado. Instalar con: curl -sfL https://raw.githubusercontent.com/aquasecurity/trivy/main/contrib/install.sh | sh"
fi
echo

echo "📦 Información de la imagen:"
docker inspect $IMAGE --format='{{json .Config}}' | jq '{User: .User, WorkingDir: .WorkingDir, ExposedPorts: .ExposedPorts}'
```

### Métricas de Optimización

```bash
# Comparar tamaños
echo "Imagen original:"
docker images mi-app:original --format "{{.Size}}"

echo "Imagen optimizada:"
docker images mi-app:optimized --format "{{.Size}}"

# Tiempo de pull
time docker pull mi-app:original
time docker pull mi-app:optimized
```

## 🛠️ Ejercicio Práctico: Optimización Completa

### Dockerfile Original (No Optimizado)

```dockerfile
FROM ubuntu:20.04

RUN apt-get update
RUN apt-get install -y python3
RUN apt-get install -y python3-pip
RUN apt-get install -y curl
RUN apt-get install -y git

COPY . /app

WORKDIR /app

RUN pip3 install -r requirements.txt

EXPOSE 5000

CMD ["python3", "app.py"]
```

### Dockerfile Optimizado

```dockerfile
# Multi-stage build optimizado
FROM python:3.9-slim AS builder

# Instalar dependencias del sistema en una sola capa
RUN apt-get update && apt-get install -y \
    build-essential \
    && rm -rf /var/lib/apt/lists/*

# Crear entorno virtual
RUN python -m venv /opt/venv
ENV PATH="/opt/venv/bin:$PATH"

# Instalar dependencias Python
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# Imagen final
FROM python:3.9-slim

# Crear usuario no-root
RUN useradd --create-home --shell /bin/bash app

# Copiar entorno virtual
COPY --from=builder /opt/venv /opt/venv
ENV PATH="/opt/venv/bin:$PATH"

# Establecer directorio de trabajo
WORKDIR /home/app

# Copiar solo archivos necesarios
COPY --chown=app:app app.py .
COPY --chown=app:app templates/ ./templates/
COPY --chown=app:app static/ ./static/

# Cambiar a usuario no-root
USER app

# Exponer puerto
EXPOSE 5000

# Health check
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
    CMD python -c "import requests; requests.get('http://localhost:5000/health')" || exit 1

# Comando optimizado
CMD ["python", "app.py"]
```

### Resultados de Optimización

```bash
# Tamaños comparados:
# Imagen original: 1.2GB
# Imagen optimizada: 145MB
# Reducción: 88%

# Vulnerabilidades:
# Original: 127 vulnerabilidades
# Optimizada: 12 vulnerabilidades
# Reducción: 90%
```

## 📝 Lista de Verificación de Optimización

### ✅ Checklist Básico

- [ ] ¿Usas la imagen base más pequeña posible?
- [ ] ¿Combinas comandos RUN para minimizar capas?
- [ ] ¿Tienes un archivo .dockerignore completo?
- [ ] ¿Copias dependencias antes que el código fuente?
- [ ] ¿Eliminas archivos de cache y temporales?
- [ ] ¿Ejecutas como usuario no-root?
- [ ] ¿Usas multi-stage builds cuando es apropiado?

### ✅ Checklist Avanzado

- [ ] ¿Implementas health checks?
- [ ] ¿Usas herramientas de análisis de seguridad?
- [ ] ¿Especificas versiones exactas de dependencias?
- [ ] ¿Optimizas para el cache de capas?
- [ ] ¿Documentas variables de entorno requeridas?
- [ ] ¿Usas init system para manejo de señales?
- [ ] ¿Implementas logging y monitoreo apropiados?

## 🎯 Resumen

Has aprendido:
- [x] Fundamentos de optimización de imágenes Docker
- [x] Técnicas para reducir tamaño y mejorar seguridad
- [x] Uso de construcciones multi-stage avanzadas
- [x] Herramientas de análisis y optimización
- [x] Estrategias específicas por lenguaje de programación
- [x] Implementación de mejores prácticas de seguridad

## 📚 Recursos Adicionales

- [Docker Best Practices](https://docs.docker.com/develop/best-practices/)
- [Distroless Images](https://github.com/GoogleContainerTools/distroless)
- [Dive Tool](https://github.com/wagoodman/dive)
- [Hadolint](https://github.com/hadolint/hadolint)
- [Trivy Scanner](https://github.com/aquasecurity/trivy)
- [Docker Slim](https://github.com/docker-slim/docker-slim)

---

**Próximo tema:** [Arquitectura de Kubernetes](./05-kubernetes-arquitectura.md)