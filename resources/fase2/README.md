# Fase 2: Automatización y CI/CD

**Automatiza tu flujo de trabajo DevOps!** Esta fase te enseñará a implementar automatización completa en tu ciclo de desarrollo, desde scripts básicos hasta pipelines de CI/CD empresariales.

##  Estado de la Fase

###  FASE EN DESARROLLO

- **6 archivos planificados** con contenido técnico detallado
- **Automatización completa** del workflow DevOps
- **Pipelines profesionales** de CI/CD
- **Estrategias de despliegue** modernas

##  Objetivos de Aprendizaje

Al completar esta fase serás capaz de:

- **Automatizar tareas** repetitivas con scripts y herramientas
- **Configurar pipelines** de Integración Continua (CI)
- **Implementar Entrega Continua** (CD) con múltiples estrategias
- **Gestionar entornos** de desarrollo, pruebas y producción
- **Configurar pruebas automatizadas** en pipelines
- **Implementar rollbacks** automáticos y estrategias de despliegue
- **Utilizar múltiples plataformas** CI/CD (Jenkins, GitHub Actions, GitLab)
- **Optimizar tiempos** de build y despliegue

##  Contenido de la Fase

###  Semana 1: Automatización de Tareas

#### [01. Scripts de Automatización](01-scripts-automatizacion.md)

- Scripts de shell/bash para tareas repetitivas
- PowerShell para entornos Windows
- Python para automatización avanzada
- **Duración:** 6-8 horas
- **Tipo:** Hands-on + Scripting

#### [02. Herramientas de Automatización](02-herramientas-automatizacion.md)

- Make para builds automatizados
- Ansible para configuración de servidores
- Puppet y Chef (introducción)
- **Duración:** 8-10 horas
- **Tipo:** Configuración + Laboratorios

###  Semana 2: Integración Continua (CI)

#### [03. Configuración de Jenkins](03-jenkins-ci.md)

- Instalación y configuración de Jenkins
- Creación de pipelines básicos
- Integración con Git y GitHub
- **Duración:** 8-10 horas
- **Tipo:** Instalación + Configuración

#### [04. Plataformas CI/CD Modernas](04-plataformas-cicd.md)

- GitHub Actions workflows
- GitLab CI/CD pipelines
- CircleCI configuración
- **Duración:** 10-12 horas
- **Tipo:** Comparativo + Hands-on

###  Semana 3: Pruebas Automatizadas

#### [05. Testing en Pipelines](05-testing-pipelines.md)

- Pruebas unitarias automatizadas
- Pruebas de integración
- Pruebas de regresión y E2E
- **Duración:** 8-10 horas
- **Tipo:** Testing + Automation

###  Semana 4: Entrega/Despliegue Continuo (CD)

#### [06. Estrategias de Despliegue](06-estrategias-despliegue.md)

- Blue-Green Deployment
- Canary Releases
- Rolling Updates
- Rollbacks automáticos
- **Duración:** 10-12 horas
- **Tipo:** Estratégico + Implementación

##  Plan de Estudio Recomendado

###  **Cronograma Sugerido (4 semanas)**

| Semana | Archivos | Enfoque | Tiempo |
|--------|----------|---------|--------|
| 1 | 01-02 | Automatización básica | 14-18 horas |
| 2 | 03-04 | Integración continua | 18-22 horas |
| 3 | 05 | Testing automatizado | 8-10 horas |
| 4 | 06 | Despliegue continuo | 10-12 horas |

**Total estimado:** 50-62 horas de estudio

###  **Modalidades de Estudio**

####  **Intensivo (3 semanas)**

- 18-22 horas por semana
- Para personas con experiencia en scripts
- Enfoque en implementación práctica

####  **Relajado (6-8 semanas)**

- 8-10 horas por semana
- Ideal para principiantes en automatización
- Más tiempo para dominar herramientas

####  **Flexible**

- A tu propio ritmo
- Puedes pausar en cada herramienta
- Practica hasta dominar cada concepto

##  Prerrequisitos

###  **Conocimientos Necesarios**

-  **Fase 1 completada** (Fundamentos DevOps)
- Git intermedio (branching, merging, PRs)
- Línea de comandos avanzada
- Conceptos básicos de testing
- Experiencia con al menos un lenguaje de programación

###  **Herramientas Requeridas**

#### **Locales:**

- Docker Desktop instalado
- Node.js/npm (para ejemplos)
- Python 3.x
- Git configurado
- VS Code con extensiones DevOps

#### **Cuentas Cloud:**

- GitHub (con GitHub Actions)
- GitLab (para GitLab CI/CD)
- Docker Hub
- Una cuenta cloud (AWS/Azure/GCP)

#### **Servidores (Opcionales):**

- VM para Jenkins (puede ser local)
- Entorno de staging para pruebas

##  Evaluación y Progreso

###  **Criterios de Completado**

Para considerar la Fase 2 completada, deberías ser capaz de:

1. **Crear scripts** de automatización para tareas comunes
2. **Configurar un pipeline CI/CD** completo desde cero
3. **Implementar estrategias** de despliegue Blue-Green
4. **Configurar pruebas automatizadas** en el pipeline
5. **Gestionar múltiples entornos** (dev, staging, prod)
6. **Resolver problemas** comunes de CI/CD
7. **Optimizar tiempos** de build y despliegue

###  **Checklist de Progreso**

- [ ] Automatizado tareas repetitivas con scripts
- [ ] Configurado Jenkins para CI básico
- [ ] Creado workflows en GitHub Actions
- [ ] Implementado pipeline en GitLab CI/CD
- [ ] Configurado pruebas automatizadas completas
- [ ] Desplegado con estrategia Blue-Green
- [ ] Implementado Canary Release
- [ ] Configurado rollbacks automáticos
- [ ] Optimizado tiempos de pipeline
- [ ] Gestionado secretos y variables de entorno
- [ ] Configurado notificaciones de build
- [ ] Implementado quality gates

###  **Proyecto Final de Fase**

**Pipeline CI/CD Completo:** Crear un pipeline profesional que incluya:

- Aplicación web con tests
- Pipeline multi-stage (build, test, deploy)
- Despliegue automático a staging
- Despliegue manual a producción con aprobación
- Rollback automático en caso de fallos
- Notificaciones por Slack/email
- Métricas de tiempo y calidad

##  Navegación

###  **Anterior**

- [Fase 1: Fundamentos DevOps](../fase1/)
- [Roadmap Principal](../../roadmap.md)

###  **Siguiente**

- [Fase 3: Contenedores y Orquestación](../fase3/)

###  **Enlaces Útiles**

- [Notas de Git](../../notes/git-notes.md)
- [Notas de Docker](../../notes/docker-notes.md)
- [Scripts de Automatización](../../scripts/)
- [Proyectos CI/CD](../../projects/ci-cd-pipeline/)

##  Herramientas y Tecnologías

###  **Stack Tecnológico de la Fase**

#### **Automatización:**

- **Bash/Shell** - Scripts Unix/Linux
- **PowerShell** - Scripts Windows
- **Python** - Automatización avanzada
- **Make** - Build automation
- **Ansible** - Configuration management

#### **CI/CD Platforms:**

- **Jenkins** - Self-hosted CI/CD
- **GitHub Actions** - Git-integrated workflows
- **GitLab CI/CD** - DevOps platform completo
- **CircleCI** - Cloud-native CI/CD
- **Azure DevOps** - Microsoft ecosystem

#### **Testing:**

- **Jest** - JavaScript testing
- **PyTest** - Python testing
- **NUnit** - .NET testing
- **Selenium** - Web UI testing
- **Postman/Newman** - API testing

#### **Deployment:**

- **Docker** - Containerización
- **Helm** - Kubernetes packages
- **Terraform** - Infrastructure provisioning
- **AWS CLI/Azure CLI** - Cloud deployments

##  Consejos para el Éxito

###  **Estrategias de Aprendizaje**

1. **Comienza simple**: Automatiza una tarea pequeña primero
2. **Iteración gradual**: Mejora el pipeline paso a paso
3. **Falla rápido**: Prueba, falla, aprende, mejora
4. **Documentación**: Documenta cada pipeline creado
5. **Comunidad**: Comparte pipelines y aprende de otros

###  **Errores Comunes a Evitar**

- **Pipelines complejos de inicio**: Comienza simple y evoluciona
- **No probar localmente**: Siempre prueba antes de push
- **Ignorar seguridad**: Nunca hagas commit de secretos
- **Pipelines lentos**: Optimiza desde el principio
- **Sin rollback plan**: Siempre ten plan de reversa

###  **Mejores Prácticas de Seguridad**

- **Secretos seguros**: Usa variables de entorno cifradas
- **Acceso mínimo**: Principio de menor privilegio
- **Auditoría**: Log todas las acciones importantes
- **Scanning**: Escanea dependencias y contenedores
- **Aislamiento**: Entornos separados por branches

###  **Optimización de Performance**

- **Paralelización**: Ejecuta tests en paralelo
- **Caché**: Cachea dependencias y builds
- **Stages selectivos**: Solo ejecuta lo necesario
- **Artifacts**: Reutiliza builds entre stages
- **Monitoreo**: Mide y optimiza tiempos

##  Soporte y Comunidad

###  **Cómo Obtener Ayuda**

1. **GitHub Issues**: Para reportar problemas con pipelines
2. **Discusiones**: Para compartir configuraciones exitosas
3. **Pull Requests**: Para contribuir con templates
4. **Redes sociales**: Comparte tus automatizaciones

###  **Contribuye al Curso**

- Comparte templates de pipelines exitosos
- Documenta casos de uso específicos
- Reporta herramientas o prácticas desactualizadas
- Ayuda a resolver problemas de otros estudiantes

##  Certificaciones Relacionadas

Al completar esta fase estarás preparado para:

- **GitHub Actions Certification**
- **Jenkins Certified Engineer**
- **GitLab Certified Associate**
- **AWS DevOps Engineer Associate**
- **Azure DevOps Engineer Expert**

##  Próximos Pasos

###  **Al Completar la Fase 2**

1. **Portfolio**: Documenta tus pipelines en GitHub
2. **Blog post**: Escribe sobre tu experiencia con CI/CD
3. **Mentoreo**: Ayuda a otros con automatización
4. **Fase 3**: Avanza a contenedores y Kubernetes

###  **Mantenimiento Continuo**

- Mantén tus pipelines actualizados
- Experimenta con nuevas herramientas
- Contribuye a proyectos open source
- Participa en comunidades DevOps

---

##  Licencia

Este contenido está bajo la licencia [MIT](../../LICENSE).

**Automatiza tu camino hacia el éxito DevOps!**

---

*Última actualización: 12 de octubre de 2025*
