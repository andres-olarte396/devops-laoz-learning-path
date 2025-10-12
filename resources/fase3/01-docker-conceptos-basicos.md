# Docker: Conceptos Básicos

## Objetivos
Al completar este módulo serás capaz de:
- Comprender qué son los contenedores y cómo funcionan
- Distinguir entre imágenes y contenedores
- Trabajar con volúmenes y redes en Docker
- Ejecutar contenedores básicos

## ¿Qué es Docker?

Docker es una plataforma de contenedorización que permite empaquetar aplicaciones y sus dependencias en contenedores ligeros y portables.

### Conceptos Fundamentales

#### Contenedores vs Máquinas Virtuales
```
┌─────────────────────┐  ┌─────────────────────┐
│     Aplicación      │  │     Aplicación      │
├─────────────────────┤  ├─────────────────────┤
│      Runtime        │  │      Runtime        │
├─────────────────────┤  ├─────────────────────┤
│    Guest OS         │  │                     │
├─────────────────────┤  │   Docker Engine     │
│    Hypervisor       │  │                     │
├─────────────────────┤  ├─────────────────────┤
│     Host OS         │  │     Host OS         │
└─────────────────────┘  └─────────────────────┘
   Máquina Virtual            Contenedor
```

### Imágenes

Las imágenes son plantillas de solo lectura que contienen:
- Sistema operativo base
- Aplicación y dependencias
- Configuración y metadatos

```bash
# Listar imágenes locales
docker images

# Descargar imagen del registry
docker pull nginx:latest

# Buscar imágenes en Docker Hub
docker search python
```

### Contenedores

Los contenedores son instancias ejecutables de imágenes:

```bash
# Ejecutar contenedor simple
docker run hello-world

# Ejecutar contenedor interactivo
docker run -it ubuntu:20.04 bash

# Ejecutar contenedor en background
docker run -d --name mi-nginx nginx:latest

# Listar contenedores en ejecución
docker ps

# Listar todos los contenedores
docker ps -a

# Detener contenedor
docker stop mi-nginx

# Eliminar contenedor
docker rm mi-nginx
```

### Volúmenes

Los volúmenes permiten persistir datos fuera del contenedor:

#### Tipos de volúmenes:

1. **Bind mounts**: Montan directorio del host
```bash
docker run -v /host/path:/container/path nginx
```

2. **Named volumes**: Volúmenes gestionados por Docker
```bash
# Crear volumen
docker volume create mi-volumen

# Usar volumen
docker run -v mi-volumen:/data nginx
```

3. **tmpfs mounts**: Almacenamiento temporal en memoria
```bash
docker run --tmpfs /tmp nginx
```

### Redes

Docker maneja varios tipos de redes:

#### Tipos de red:
- **bridge**: Red por defecto para contenedores
- **host**: Usa directamente la red del host
- **none**: Sin acceso a red
- **overlay**: Para Docker Swarm

```bash
# Listar redes
docker network ls

# Crear red personalizada
docker network create mi-red

# Ejecutar contenedor en red específica
docker run --network mi-red nginx

# Inspeccionar red
docker network inspect mi-red
```

### Comandos Esenciales

```bash
# Información del sistema Docker
docker info

# Versión de Docker
docker version

# Logs de contenedor
docker logs nombre-contenedor

# Ejecutar comando en contenedor
docker exec -it nombre-contenedor bash

# Copiar archivos
docker cp archivo.txt contenedor:/ruta/

# Estadísticas de uso
docker stats

# Limpiar recursos
docker system prune
```

## Ejemplo Práctico

### Ejecutar servidor web simple:

```bash
# 1. Descargar imagen de nginx
docker pull nginx:latest

# 2. Ejecutar contenedor con puerto expuesto
docker run -d \
  --name mi-servidor \
  -p 8080:80 \
  -v $(pwd)/html:/usr/share/nginx/html \
  nginx:latest

# 3. Verificar que funciona
curl http://localhost:8080

# 4. Ver logs
docker logs mi-servidor

# 5. Limpiar
docker stop mi-servidor
docker rm mi-servidor
```

## Ejercicios

### Ejercicio 1: Contenedor básico
1. Ejecuta un contenedor de Ubuntu interactivo
2. Instala `curl` dentro del contenedor
3. Sal del contenedor y observa que los cambios se pierden

### Ejercicio 2: Volúmenes
1. Crea un volumen llamado `mi-datos`
2. Ejecuta un contenedor que monte este volumen en `/data`
3. Crea un archivo dentro de `/data`
4. Ejecuta otro contenedor con el mismo volumen y verifica que el archivo existe

### Ejercicio 3: Redes
1. Crea una red llamada `mi-app-red`
2. Ejecuta dos contenedores en esta red
3. Verifica que pueden comunicarse entre ellos

## Recursos Adicionales
- [Docker Documentation](https://docs.docker.com/)
- [Docker Hub](https://hub.docker.com/)
- [Play with Docker](https://labs.play-with-docker.com/)

## Siguiente Paso
[Crear y gestionar imágenes con Dockerfile](./02-dockerfile-imagenes.md)