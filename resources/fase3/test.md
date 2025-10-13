# Test de la Fase 3: Contenedores, Redes y Orquestación

**¡Evalúa tu conocimiento de contenedores, Docker, Kubernetes y orquestación!**

Este test abarca todo el contenido de la Fase 3 completada, incluyendo los 11 temas principales que has estudiado.

> **Instrucciones:** Selecciona la respuesta correcta para cada pregunta. Las respuestas se encuentran al final del documento.

---

## Temario Cubierto en el Test

**Fase 3 Completada - 11 Temas:**

1. **[Conceptos Básicos de Docker](01-docker-conceptos-basicos.md)** - Contenedores vs VMs, arquitectura
2. **[Instalación y Configuración de Docker](02-instalacion-configuracion-docker.md)** - Setup y primeros pasos
3. **[Comandos Básicos de Docker](03-comandos-basicos-docker.md)** - CLI y gestión de contenedores
4. **[Creación de Imágenes Docker](04-creacion-imagenes-docker.md)** - Dockerfile y mejores prácticas
5. **[Docker Compose](05-docker-compose.md)** - Multi-contenedor y orquestación básica
6. **[Redes en Docker](06-redes-docker.md)** - Networking y comunicación entre contenedores
7. **[Introducción a Kubernetes](07-introduccion-kubernetes.md)** - Conceptos y arquitectura
8. **[Escalado y Balanceo de Carga](08-escalado-balanceo.md)** - HPA, VPA y estrategias
9. **[Helm Charts](09-helm-charts.md)** - Gestión de paquetes Kubernetes
10. **[Práctica Multi-contenedor](10-practica-multicontenedor.md)** - Aplicación completa
11. **[Práctica CI/CD para Contenedores](11-practica-cicd-contenedores.md)** - Pipeline completo

---

## 1. Conceptos Básicos de Docker

> **Contenido de referencia:** [01-docker-conceptos-basicos.md](01-docker-conceptos-basicos.md)

**1. ¿Cuál es la principal diferencia entre contenedores y máquinas virtuales?**

- [ ] a) Los contenedores incluyen un sistema operativo completo
- [ ] b) Los contenedores comparten el kernel del host, las VMs tienen su propio OS
- [ ] c) Las VMs son más ligeras que los contenedores
- [ ] d) No hay diferencias significativas

<details>
  <summary>Ver Fuente</summary>
  <p><a href="./01-docker-conceptos-basicos.md">Fuente: Conceptos Básicos de Docker</a></p>
</details>

**2. ¿Qué es una imagen Docker?**

- [ ] a) Un contenedor en ejecución
- [ ] b) Una plantilla de solo lectura que contiene el sistema de archivos y configuración
- [ ] c) Un archivo de configuración YAML
- [ ] d) Una máquina virtual ligera

<details>
  <summary>Ver Fuente</summary>
  <p><a href="./01-docker-conceptos-basicos.md">Fuente: Conceptos Básicos de Docker</a></p>
</details>

**3. ¿Cuál de los siguientes NO es un beneficio de la contenedorización?**

- [ ] a) Portabilidad entre entornos
- [ ] b) Aislamiento de aplicaciones
- [ ] c) Mayor consumo de recursos que las VMs
- [ ] d) Escalabilidad rápida

<details>
  <summary>Ver Fuente</summary>
  <p><a href="./01-docker-conceptos-basicos.md">Fuente: Conceptos Básicos de Docker</a></p>
</details>

---

## 2. Instalación y Configuración de Docker

> **Contenido de referencia:** [02-instalacion-configuracion-docker.md](02-instalacion-configuracion-docker.md)

**4. ¿Cuál es el comando para verificar que Docker está correctamente instalado?**

- [ ] a) `docker --help`
- [ ] b) `docker version`
- [ ] c) `docker status`
- [ ] d) `docker check`

<details>
  <summary>Ver Fuente</summary>
  <p><a href="./02-instalacion-configuracion-docker.md">Fuente: Instalación y Configuración de Docker</a></p>
</details>

**5. ¿Qué es Docker Desktop?**

- [ ] a) Un editor de código para Docker
- [ ] b) Una interfaz gráfica que incluye Docker Engine, CLI y herramientas adicionales
- [ ] c) Una versión de Docker solo para servidores
- [ ] d) Un registro de imágenes Docker

<details>
  <summary>Ver Fuente</summary>
  <p><a href="./02-instalacion-configuracion-docker.md">Fuente: Instalación y Configuración de Docker</a></p>
</details>

**6. ¿Por qué es importante configurar Docker para ejecutar sin sudo en Linux?**

- [ ] a) Para mayor seguridad
- [ ] b) Para evitar problemas de permisos en desarrollo y CI/CD
- [ ] c) Docker no funciona con sudo
- [ ] d) No es necesario configurarlo

<details>
  <summary>Ver Fuente</summary>
  <p><a href="./02-instalacion-configuracion-docker.md">Fuente: Instalación y Configuración de Docker</a></p>
</details>

---

## 3. Comandos Básicos de Docker

> **Contenido de referencia:** [03-comandos-basicos-docker.md](03-comandos-basicos-docker.md)

**7. ¿Cuál es el comando para ejecutar un contenedor en modo interactivo?**

- [ ] a) `docker run -d nginx`
- [ ] b) `docker run -it ubuntu bash`
- [ ] c) `docker start ubuntu`
- [ ] d) `docker exec ubuntu`

<details>
  <summary>Ver Fuente</summary>
  <p><a href="./03-comandos-basicos-docker.md">Fuente: Comandos Básicos de Docker</a></p>
</details>

**8. ¿Qué hace el flag `-d` en el comando `docker run`?**

- [ ] a) Ejecuta el contenedor en modo debug
- [ ] b) Ejecuta el contenedor en segundo plano (detached)
- [ ] c) Elimina el contenedor automáticamente
- [ ] d) Establece el directorio de trabajo

<details>
  <summary>Ver Fuente</summary>
  <p><a href="./03-comandos-basicos-docker.md">Fuente: Comandos Básicos de Docker</a></p>
</details>

**9. ¿Cuál es el comando para mapear un puerto del host al contenedor?**

- [ ] a) `docker run -p 8080:80 nginx`
- [ ] b) `docker run --port 8080:80 nginx`
- [ ] c) `docker run -map 8080:80 nginx`
- [ ] d) `docker run --connect 8080:80 nginx`

<details>
  <summary>Ver Fuente</summary>
  <p><a href="./03-comandos-basicos-docker.md">Fuente: Comandos Básicos de Docker</a></p>
</details>

**10. ¿Qué comando se usa para ver los contenedores en ejecución?**

- [ ] a) `docker list`
- [ ] b) `docker containers`
- [ ] c) `docker ps`
- [ ] d) `docker show`

<details>
  <summary>Ver Fuente</summary>
  <p><a href="./03-comandos-basicos-docker.md">Fuente: Comandos Básicos de Docker</a></p>
</details>

---

## 4. Creación de Imágenes Docker

> **Contenido de referencia:** [04-creacion-imagenes-docker.md](04-creacion-imagenes-docker.md)

**11. ¿Cuál es la instrucción correcta para especificar la imagen base en un Dockerfile?**

- [ ] a) `BASE ubuntu:20.04`
- [ ] b) `FROM ubuntu:20.04`
- [ ] c) `IMAGE ubuntu:20.04`
- [ ] d) `USE ubuntu:20.04`

<details>
  <summary>Ver Fuente</summary>
  <p><a href="./04-creacion-imagenes-docker.md">Fuente: Creación de Imágenes Docker</a></p>
</details>

**12. ¿Qué hace la instrucción `WORKDIR` en un Dockerfile?**

- [ ] a) Crea un nuevo directorio
- [ ] b) Establece el directorio de trabajo para las siguientes instrucciones
- [ ] c) Copia archivos al directorio especificado
- [ ] d) Elimina un directorio

<details>
  <summary>Ver Fuente</summary>
  <p><a href="./04-creacion-imagenes-docker.md">Fuente: Creación de Imágenes Docker</a></p>
</details>

**13. ¿Cuál es una buena práctica para optimizar las capas de Docker?**

- [ ] a) Usar una instrucción RUN por cada comando
- [ ] b) Combinar comandos relacionados en una sola instrucción RUN
- [ ] c) No usar cache de Docker
- [ ] d) Instalar todos los paquetes al final

<details>
  <summary>Ver Fuente</summary>
  <p><a href="./04-creacion-imagenes-docker.md">Fuente: Creación de Imágenes Docker</a></p>
</details>

**14. ¿Para qué sirve el archivo `.dockerignore`?**

- [ ] a) Para ignorar errores durante el build
- [ ] b) Para excluir archivos del contexto de build
- [ ] c) Para configurar Docker
- [ ] d) Para documentar la imagen

<details>
  <summary>Ver Fuente</summary>
  <p><a href="./04-creacion-imagenes-docker.md">Fuente: Creación de Imágenes Docker</a></p>
</details>

---

## 5. Docker Compose

> **Contenido de referencia:** [05-docker-compose.md](05-docker-compose.md)

**15. ¿Qué formato utiliza Docker Compose para definir servicios?**

- [ ] a) JSON
- [ ] b) XML
- [ ] c) YAML
- [ ] d) TOML

<details>
  <summary>Ver Fuente</summary>
  <p><a href="./05-docker-compose.md">Fuente: Docker Compose</a></p>
</details>

**16. ¿Cuál es el comando para iniciar todos los servicios definidos en docker-compose.yml?**

- [ ] a) `docker-compose start`
- [ ] b) `docker-compose run`
- [ ] c) `docker-compose up`
- [ ] d) `docker-compose deploy`

<details>
  <summary>Ver Fuente</summary>
  <p><a href="./05-docker-compose.md">Fuente: Docker Compose</a></p>
</details>

**17. ¿Qué ventaja ofrece Docker Compose sobre ejecutar contenedores individuales?**

- [ ] a) Mayor rendimiento
- [ ] b) Gestión simplificada de aplicaciones multi-contenedor
- [ ] c) Menor uso de memoria
- [ ] d) Mayor seguridad automática

<details>
  <summary>Ver Fuente</summary>
  <p><a href="./05-docker-compose.md">Fuente: Docker Compose</a></p>
</details>

**18. ¿Cómo se define una dependencia entre servicios en Docker Compose?**

- [ ] a) Con la clave `requires`
- [ ] b) Con la clave `depends_on`
- [ ] c) Con la clave `needs`
- [ ] d) Con la clave `after`

<details>
  <summary>Ver Fuente</summary>
  <p><a href="./05-docker-compose.md">Fuente: Docker Compose</a></p>
</details>

---

## 6. Redes en Docker

> **Contenido de referencia:** [06-redes-docker.md](06-redes-docker.md)

**19. ¿Cuál es el driver de red por defecto en Docker?**

- [ ] a) host
- [ ] b) bridge
- [ ] c) overlay
- [ ] d) none

<details>
  <summary>Ver Fuente</summary>
  <p><a href="./06-redes-docker.md">Fuente: Redes en Docker</a></p>
</details>

**20. ¿Qué tipo de red permite que los contenedores se comuniquen entre diferentes hosts Docker?**

- [ ] a) bridge
- [ ] b) host
- [ ] c) overlay
- [ ] d) macvlan

<details>
  <summary>Ver Fuente</summary>
  <p><a href="./06-redes-docker.md">Fuente: Redes en Docker</a></p>
</details>

**21. ¿Cómo pueden comunicarse dos contenedores en la misma red personalizada?**

- [ ] a) Solo por dirección IP
- [ ] b) Por nombre del contenedor como hostname
- [ ] c) Solo por mapeo de puertos
- [ ] d) No pueden comunicarse

<details>
  <summary>Ver Fuente</summary>
  <p><a href="./06-redes-docker.md">Fuente: Redes en Docker</a></p>
</details>

---

## 7. Introducción a Kubernetes

> **Contenido de referencia:** [07-introduccion-kubernetes.md](07-introduccion-kubernetes.md)

**22. ¿Cuál es la unidad más pequeña de despliegue en Kubernetes?**

- [ ] a) Container
- [ ] b) Pod
- [ ] c) Service
- [ ] d) Deployment

<details>
  <summary>Ver Fuente</summary>
  <p><a href="./07-introduccion-kubernetes.md">Fuente: Introducción a Kubernetes</a></p>
</details>

**23. ¿Qué componente de Kubernetes es responsable de la programación de pods?**

- [ ] a) kubelet
- [ ] b) kube-proxy
- [ ] c) kube-scheduler
- [ ] d) etcd

<details>
  <summary>Ver Fuente</summary>
  <p><a href="./07-introduccion-kubernetes.md">Fuente: Introducción a Kubernetes</a></p>
</details>

**24. ¿Qué es un Service en Kubernetes?**

- [ ] a) Un contenedor en ejecución
- [ ] b) Una abstracción que expone un conjunto de pods como un servicio de red
- [ ] c) Un archivo de configuración
- [ ] d) Un tipo de almacenamiento

<details>
  <summary>Ver Fuente</summary>
  <p><a href="./07-introduccion-kubernetes.md">Fuente: Introducción a Kubernetes</a></p>
</details>

**25. ¿Cuál es el comando principal para interactuar con un cluster de Kubernetes?**

- [ ] a) `kube`
- [ ] b) `kubectl`
- [ ] c) `kubernetes`
- [ ] d) `k8s`

<details>
  <summary>Ver Fuente</summary>
  <p><a href="./07-introduccion-kubernetes.md">Fuente: Introducción a Kubernetes</a></p>
</details>

---

## 8. Escalado y Balanceo de Carga

> **Contenido de referencia:** [08-escalado-balanceo.md](08-escalado-balanceo.md)

**26. ¿Qué es el Horizontal Pod Autoscaler (HPA)?**

- [ ] a) Un componente que escala verticalmente los pods
- [ ] b) Un componente que escala horizontalmente el número de pods
- [ ] c) Un balanceador de carga
- [ ] d) Un monitor de recursos

<details>
  <summary>Ver Fuente</summary>
  <p><a href="./08-escalado-balanceo.md">Fuente: Escalado y Balanceo de Carga</a></p>
</details>

**27. ¿Cuál es la diferencia entre escalado horizontal y vertical?**

- [ ] a) No hay diferencia
- [ ] b) Horizontal añade más instancias, vertical aumenta recursos de instancias existentes
- [ ] c) Vertical añade más instancias, horizontal aumenta recursos
- [ ] d) Solo se puede usar uno a la vez

<details>
  <summary>Ver Fuente</summary>
  <p><a href="./08-escalado-balanceo.md">Fuente: Escalado y Balanceo de Carga</a></p>
</details>

**28. ¿Qué métrica es comúnmente usada para el escalado automático?**

- [ ] a) Uso de CPU
- [ ] b) Uso de memoria
- [ ] c) Número de peticiones HTTP
- [ ] d) Todas las anteriores

<details>
  <summary>Ver Fuente</summary>
  <p><a href="./08-escalado-balanceo.md">Fuente: Escalado y Balanceo de Carga</a></p>
</details>

---

## 9. Helm Charts

> **Contenido de referencia:** [09-helm-charts.md](09-helm-charts.md)

**29. ¿Qué es Helm en el contexto de Kubernetes?**

- [ ] a) Un contenedor especializado
- [ ] b) Un gestor de paquetes para Kubernetes
- [ ] c) Una alternativa a Docker
- [ ] d) Un tipo de Service

<details>
  <summary>Ver Fuente</summary>
  <p><a href="./09-helm-charts.md">Fuente: Helm Charts</a></p>
</details>

**30. ¿Cómo se llama un paquete de Helm?**

- [ ] a) Package
- [ ] b) Chart
- [ ] c) Template
- [ ] d) Bundle

<details>
  <summary>Ver Fuente</summary>
  <p><a href="./09-helm-charts.md">Fuente: Helm Charts</a></p>
</details>

**31. ¿Cuál es el comando para instalar un chart de Helm?**

- [ ] a) `helm deploy`
- [ ] b) `helm install`
- [ ] c) `helm create`
- [ ] d) `helm apply`

<details>
  <summary>Ver Fuente</summary>
  <p><a href="./09-helm-charts.md">Fuente: Helm Charts</a></p>
</details>

**32. ¿Qué permite el archivo `values.yaml` en Helm?**

- [ ] a) Definir la estructura del chart
- [ ] b) Configurar valores por defecto y personalizables
- [ ] c) Listar dependencias
- [ ] d) Documentar el chart

<details>
  <summary>Ver Fuente</summary>
  <p><a href="./09-helm-charts.md">Fuente: Helm Charts</a></p>
</details>

---

## 10. Práctica Multi-contenedor

> **Contenido de referencia:** [10-practica-multicontenedor.md](10-practica-multicontenedor.md)

**33. ¿Cuál es una ventaja de usar una arquitectura multi-contenedor?**

- [ ] a) Mayor complejidad
- [ ] b) Separación de responsabilidades y escalabilidad independiente
- [ ] c) Menor rendimiento
- [ ] d) Más difícil de mantener

<details>
  <summary>Ver Fuente</summary>
  <p><a href="./10-practica-multicontenedor.md">Fuente: Práctica Multi-contenedor</a></p>
</details>

**34. ¿Cómo se comunican típicamente los contenedores en una aplicación multi-contenedor?**

- [ ] a) Solo por archivos compartidos
- [ ] b) A través de redes Docker y APIs REST/bases de datos
- [ ] c) No pueden comunicarse
- [ ] d) Solo por variables de entorno

<details>
  <summary>Ver Fuente</summary>
  <p><a href="./10-practica-multicontenedor.md">Fuente: Práctica Multi-contenedor</a></p>
</details>

**35. ¿Qué patrón es recomendado para manejar datos persistentes en aplicaciones multi-contenedor?**

- [ ] a) Almacenar datos en el contenedor de aplicación
- [ ] b) Usar volúmenes Docker y contenedores especializados para bases de datos
- [ ] c) No usar persistencia
- [ ] d) Guardar todo en variables de entorno

<details>
  <summary>Ver Fuente</summary>
  <p><a href="./10-practica-multicontenedor.md">Fuente: Práctica Multi-contenedor</a></p>
</details>

---

## 11. Práctica CI/CD para Contenedores

> **Contenido de referencia:** [11-practica-cicd-contenedores.md](11-practica-cicd-contenedores.md)

**36. ¿Cuáles son las fases típicas de un pipeline CI/CD para contenedores?**

- [ ] a) Build → Test → Deploy
- [ ] b) Build → Test → Security Scan → Push → Deploy
- [ ] c) Code → Build → Deploy
- [ ] d) Test → Build → Push

<details>
  <summary>Ver Fuente</summary>
  <p><a href="./11-practica-cicd-contenedores.md">Fuente: Práctica CI/CD para Contenedores</a></p>
</details>

**37. ¿Qué es un registro de contenedores (Container Registry)?**

- [ ] a) Un archivo de configuración
- [ ] b) Un servicio para almacenar y distribuir imágenes Docker
- [ ] c) Un tipo de contenedor
- [ ] d) Una herramienta de monitoring

<details>
  <summary>Ver Fuente</summary>
  <p><a href="./11-practica-cicd-contenedores.md">Fuente: Práctica CI/CD para Contenedores</a></p>
</details>

**38. ¿Qué estrategia de despliegue minimiza el downtime?**

- [ ] a) Recreate
- [ ] b) Blue-Green deployment
- [ ] c) Rolling update
- [ ] d) Todas las anteriores excepto Recreate

<details>
  <summary>Ver Fuente</summary>
  <p><a href="./11-practica-cicd-contenedores.md">Fuente: Práctica CI/CD para Contenedores</a></p>
</details>

**39. ¿Por qué es importante el escaneo de seguridad en pipelines de contenedores?**

- [ ] a) Para mejorar el rendimiento
- [ ] b) Para detectar vulnerabilidades en las imágenes antes del despliegue
- [ ] c) Para reducir el tamaño de las imágenes
- [ ] d) No es importante

<details>
  <summary>Ver Fuente</summary>
  <p><a href="./11-practica-cicd-contenedores.md">Fuente: Práctica CI/CD para Contenedores</a></p>
</details>

**40. ¿Qué herramienta es comúnmente usada para Continuous Deployment en Kubernetes?**

- [ ] a) Docker
- [ ] b) Jenkins
- [ ] c) ArgoCD
- [ ] d) Git

<details>
  <summary>Ver Fuente</summary>
  <p><a href="./11-practica-cicd-contenedores.md">Fuente: Práctica CI/CD para Contenedores</a></p>
</details>

---

## Evaluación de Resultados

### Escala de Calificación

- **37-40 puntos (93-100%)**: **Experto** - Dominio excepcional de contenedores y orquestación
- **33-36 puntos (83-90%)**: **Avanzado** - Sólido conocimiento de Docker y Kubernetes
- **28-32 puntos (70-80%)**: **Competente** - Aprobado, conocimiento suficiente para continuar
- **23-27 puntos (58-68%)**: **En desarrollo** - Revisa conceptos específicos antes de continuar
- **Menos de 23 puntos (<58%)**: **Necesita estudio** - Repasa toda la Fase 3

### Recomendaciones por Sección

#### **Docker Básico (Preguntas 1-10)**

Si obtuviste <70%:

- Repasa conceptos fundamentales de contenedores vs VMs
- Practica comandos básicos de Docker CLI
- Refuerza conceptos de imágenes, contenedores y networking
- Revisa: [01-docker-conceptos-basicos.md](01-docker-conceptos-basicos.md), [02-instalacion-configuracion-docker.md](02-instalacion-configuracion-docker.md), [03-comandos-basicos-docker.md](03-comandos-basicos-docker.md)

#### **Creación de Imágenes (Preguntas 11-14)**

Si obtuviste <70%:

- Estudia la sintaxis y mejores prácticas de Dockerfile
- Practica creación de imágenes multi-stage
- Comprende la optimización de capas y uso de cache
- Revisa: [04-creacion-imagenes-docker.md](04-creacion-imagenes-docker.md)

#### **Docker Compose y Redes (Preguntas 15-21)**

Si obtuviste <70%:

- Domina la sintaxis YAML de Docker Compose
- Comprende los diferentes tipos de redes en Docker
- Practica aplicaciones multi-contenedor
- Revisa: [05-docker-compose.md](05-docker-compose.md), [06-redes-docker.md](06-redes-docker.md)

#### **Kubernetes Básico (Preguntas 22-25)**

Si obtuviste <70%:

- Estudia arquitectura y componentes de Kubernetes
- Comprende conceptos de Pods, Services, Deployments
- Practica comandos básicos de kubectl
- Revisa: [07-introduccion-kubernetes.md](07-introduccion-kubernetes.md)

#### **Escalado y Helm (Preguntas 26-32)**

Si obtuviste <70%:

- Comprende diferencias entre escalado horizontal y vertical
- Estudia HPA, VPA y métricas de escalado
- Domina conceptos básicos de Helm Charts
- Revisa: [08-escalado-balanceo.md](08-escalado-balanceo.md), [09-helm-charts.md](09-helm-charts.md)

#### **Aplicaciones Multi-contenedor y CI/CD (Preguntas 33-40)**

Si obtuviste <70%:

- Practica diseño de arquitecturas multi-contenedor
- Comprende patrones de comunicación entre contenedores
- Estudia pipelines CI/CD específicos para contenedores
- Revisa: [10-practica-multicontenedor.md](10-practica-multicontenedor.md), [11-practica-cicd-contenedores.md](11-practica-cicd-contenedores.md)

### Plan de Mejora por Puntuación

#### **Puntuación Alta (37-40 puntos)**

🎉 **¡Excelente!** Dominio completo de contenedores y orquestación

- Procede con confianza a la Fase 4: Infraestructura como Código
- Considera obtener certificaciones oficiales (CKA, CKAD)
- Comparte tu conocimiento mentoreando a otros
- Practica con proyectos complejos de producción

#### **Puntuación Buena (33-36 puntos)**

✅ **¡Muy bien!** Sólido conocimiento con pequeños gaps

- Revisa las preguntas que fallaste
- Profundiza en las áreas específicas identificadas
- Practica más con laboratorios hands-on
- Continúa con confianza a la siguiente fase

#### **Puntuación Competente (28-32 puntos)**

📚 **Aprobado** pero necesitas reforzar conocimientos

- Identifica tus 2-3 áreas más débiles
- Dedica tiempo adicional a práctica hands-on
- Completa laboratorios específicos en áreas problemáticas
- Considera retomar partes del test después del refuerzo

#### **Puntuación En Desarrollo (23-27 puntos)**

⚠️ **Necesitas preparación adicional**

- Enfócate en fundamentos de Docker antes de Kubernetes
- Practica intensivamente con laboratorios prácticos
- Busca recursos adicionales y tutoriales
- Considera tomar cursos especializados
- Retoma el test cuando te sientas más preparado

#### **Puntuación Insuficiente (<23 puntos)**

🔄 **Reinicia el proceso de aprendizaje**

- Vuelve a estudiar toda la Fase 3 desde el principio
- Toma un enfoque más gradual y práctico
- Instala Docker y practica todos los comandos
- Busca mentores o cursos con instructor
- No avances hasta dominar los fundamentos

### Recursos Adicionales para Mejorar

#### **Laboratorios Prácticos Recomendados**

1. **Docker Básico**
   - Crear y ejecutar contenedores básicos
   - Construir imágenes personalizadas
   - Trabajar con volúmenes y redes

2. **Docker Compose**
   - Aplicación web con base de datos
   - Configuración de múltiples servicios
   - Manejo de variables de entorno

3. **Kubernetes**
   - Instalación de minikube o kind
   - Despliegue de aplicaciones básicas
   - Configuración de Services y Ingress

4. **CI/CD**
   - Pipeline básico con GitHub Actions
   - Build y push de imágenes
   - Despliegue automatizado

#### **Herramientas de Práctica**

- **Docker Playground**: play-with-docker.com
- **Kubernetes Playground**: play-with-k8s.com
- **Katacoda**: Escenarios interactivos
- **Docker Training**: docker.com/resources/what-container

#### **Certificaciones Preparatorias**

- **Docker Certified Associate (DCA)**
- **Certified Kubernetes Administrator (CKA)**
- **Certified Kubernetes Application Developer (CKAD)**
- **Certified Kubernetes Security Specialist (CKS)**

#### **Documentación Oficial**

- [Docker Documentation](https://docs.docker.com/)
- [Kubernetes Documentation](https://kubernetes.io/docs/)
- [Helm Documentation](https://helm.sh/docs/)

---

## Certificado de Completado

**Si obtuviste 28 puntos o más, has completado exitosamente la Fase 3!**

```
╔══════════════════════════════════════════════════════╗
║              CERTIFICADO DE COMPLETADO               ║
║                                                      ║
║                    [Tu Nombre]                       ║
║                                                      ║
║           Ha completado exitosamente la              ║
║                                                      ║
║      FASE 3: CONTENEDORES, REDES Y ORQUESTACIÓN     ║
║                                                      ║
║            Del curso "DevOps Learning Path"          ║
║                                                      ║
║    Puntuación obtenida: [Tu puntuación]/40           ║
║    Fecha de completado: 13 de octubre de 2025        ║
║                                                      ║
║         ¡Listo para continuar con la Fase 4!         ║
║                                                      ║
╚══════════════════════════════════════════════════════╝
```

---

## Próximos Pasos

### **Si Aprobaste (≥28 puntos)**

1. **🎊 Celebra tu logro** - Has dominado contenedores y orquestación
2. **💼 Actualiza tu perfil profesional** - Agrega habilidades de Docker/Kubernetes
3. **🚀 Procede a la Fase 4** - Infraestructura como Código (IaC)
4. **🏗️ Practica con proyectos reales** - Implementa aplicaciones completas
5. **📜 Considera certificaciones** - CKA, CKAD, Docker Certified Associate

### **Si Necesitas Mejorar (<28 puntos)**

1. **🔍 Identifica áreas débiles** - Basándote en las secciones con más errores
2. **🛠️ Practica hands-on** - Instala Docker y Kubernetes localmente
3. **📚 Estudia material específico** - Enfócate en temas problemáticos
4. **👥 Busca ayuda** - Únete a comunidades Docker/Kubernetes
5. **🔄 Retoma el test** - Cuando domines los conceptos fundamentales

### **Plan de Práctica Intensiva**

#### **Semana 1-2: Docker Fundamentals**
```bash
# Práctica diaria recomendada
docker run hello-world
docker run -it ubuntu bash
docker build -t myapp .
docker-compose up -d
```

#### **Semana 3-4: Kubernetes Basics**
```bash
# Comandos esenciales para practicar
kubectl create deployment nginx --image=nginx
kubectl expose deployment nginx --port=80 --type=LoadBalancer
kubectl scale deployment nginx --replicas=3
kubectl get pods,services,deployments
```

#### **Semana 5-6: Proyectos Integradores**
- Aplicación web con base de datos
- Pipeline CI/CD completo
- Monitoreo y logging
- Implementación en cloud

---

## Respuestas Correctas

<details>
  <summary>Ver Respuestas</summary>

  **1. b) Los contenedores comparten el kernel del host, las VMs tienen su propio OS**
  **2. b) Una plantilla de solo lectura que contiene el sistema de archivos y configuración**
  **3. c) Mayor consumo de recursos que las VMs**
  **4. b) `docker version`**
  **5. b) Una interfaz gráfica que incluye Docker Engine, CLI y herramientas adicionales**
  **6. b) Para evitar problemas de permisos en desarrollo y CI/CD**
  **7. b) `docker run -it ubuntu bash`**
  **8. b) Ejecuta el contenedor en segundo plano (detached)**
  **9. a) `docker run -p 8080:80 nginx`**
  **10. c) `docker ps`**
  **11. b) `FROM ubuntu:20.04`**
  **12. b) Establece el directorio de trabajo para las siguientes instrucciones**
  **13. b) Combinar comandos relacionados en una sola instrucción RUN**
  **14. b) Para excluir archivos del contexto de build**
  **15. c) YAML**
  **16. c) `docker-compose up`**
  **17. b) Gestión simplificada de aplicaciones multi-contenedor**
  **18. b) Con la clave `depends_on`**
  **19. b) bridge**
  **20. c) overlay**
  **21. b) Por nombre del contenedor como hostname**
  **22. b) Pod**
  **23. c) kube-scheduler**
  **24. b) Una abstracción que expone un conjunto de pods como un servicio de red**
  **25. b) `kubectl`**
  **26. b) Un componente que escala horizontalmente el número de pods**
  **27. b) Horizontal añade más instancias, vertical aumenta recursos de instancias existentes**
  **28. d) Todas las anteriores**
  **29. b) Un gestor de paquetes para Kubernetes**
  **30. b) Chart**
  **31. b) `helm install`**
  **32. b) Configurar valores por defecto y personalizables**
  **33. b) Separación de responsabilidades y escalabilidad independiente**
  **34. b) A través de redes Docker y APIs REST/bases de datos**
  **35. b) Usar volúmenes Docker y contenedores especializados para bases de datos**
  **36. b) Build → Test → Security Scan → Push → Deploy**
  **37. b) Un servicio para almacenar y distribuir imágenes Docker**
  **38. d) Todas las anteriores excepto Recreate**
  **39. b) Para detectar vulnerabilidades en las imágenes antes del despliegue**
  **40. c) ArgoCD**

</details>

---

**¡Felicitaciones por completar la evaluación de la Fase 3!**

*El dominio de contenedores y orquestación es fundamental para DevOps moderno. Has dado un gran paso hacia la maestría en estas tecnologías clave.*

**Próxima fase:** [Fase 4: Infraestructura como Código](../fase4/README.md)