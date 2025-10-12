# Test de Evaluación - Fase 2: Automatización y CI/CD

**Evalúa tu dominio en Automation y CI/CD!** Este test comprensivo cubre todos los módulos de la Fase 2, desde scripts básicos hasta estrategias de deployment empresariales.

## Instrucciones del Test

- **Tiempo recomendado**: 90 minutos
- **Total de preguntas**: 45 preguntas
- **Modalidad**: Múltiple opción, verdadero/falso, y respuestas cortas
- **Puntaje mínimo para aprobar**: 70% (32/45 preguntas correctas)
- **Recursos permitidos**: Ninguno (test cerrado)

---

## Sección 1: Scripts y Automatización Básica (10 preguntas)

**1. ¿Cuál es la diferencia principal entre `$@` y `$*` en un script de Bash?**

- [ ] a) No hay diferencia, ambos representan todos los argumentos
- [ ] b) `$@` trata cada argumento como elemento separado, `$*` como string único
- [ ] c) `$*` es para arrays, `$@` para strings
- [ ] d) `$@` incluye el nombre del script, `$*` no

<details>
  <summary>Ver Fuente</summary>
  <p><a href="./01-scripts-automatizacion.md">Fuente: Scripts y Automatización Básica</a></p>
</details>

**2. En PowerShell, ¿cuál es la forma correcta de manejar errores en un script?**

- [ ] a) `try { } catch { } finally { }`
- [ ] b) `if ($Error) { }`
- [ ] c) `ErrorAction = "Stop"`
- [ ] d) Todas las anteriores

<details>
  <summary>Ver Fuente</summary>
  <p><a href="./01-scripts-automatizacion.md">Fuente: Scripts y Automatización Básica</a></p>
</details>

**3. Verdadero/Falso: En Python, `subprocess.run()` es más seguro que `os.system()` para ejecutar comandos del sistema.**

<details>
  <summary>Ver Fuente</summary>
  <p><a href="./01-scripts-automatizacion.md">Fuente: Scripts y Automatización Básica</a></p>
</details>

**4. ¿Qué hace este comando de Bash?**

```bash
find /var/log -name "*.log" -mtime +7 -exec rm {} \;
```

- [ ] a) Busca archivos .log modificados en los últimos 7 días
- [ ] b) Elimina archivos .log más antiguos que 7 días
- [ ] c) Comprime archivos .log de más de 7 días
- [ ] d) Mueve archivos .log antiguos a backup

<details>
  <summary>Ver Fuente</summary>
  <p><a href="./01-scripts-automatizacion.md">Fuente: Scripts y Automatización Básica</a></p>
</details>

**5. En un script de automatización, ¿cuál es la mejor práctica para manejar credenciales?**

- [ ] a) Hardcodearlas en el script
- [ ] b) Usar variables de entorno
- [ ] c) Almacenarlas en un archivo de configuración
- [ ] d) Pedirlas por input cada vez

<details>
  <summary>Ver Fuente</summary>
  <p><a href="./01-scripts-automatizacion.md">Fuente: Scripts y Automatización Básica</a></p>
</details>

**6. ¿Qué significa el código de salida `0` en un script?**

- [ ] a) Error crítico
- [ ] b) Ejecución exitosa
- [ ] c) Warning
- [ ] d) Proceso cancelado

<details>
  <summary>Ver Fuente</summary>
  <p><a href="./01-scripts-automatizacion.md">Fuente: Scripts y Automatización Básica</a></p>
</details>

**7. Completa el código: Para hacer un backup automático con timestamp en Bash: **

```bash
DATE=$(______)
cp database.sql "backup_${DATE}.sql"
```

<details>
  <summary>Ver Fuente</summary>
  <p><a href="./01-scripts-automatizacion.md">Fuente: Scripts y Automatización Básica</a></p>
</details>

**8. En PowerShell, ¿cómo obtienes el directorio actual del script?**

- [ ] a) `$PWD`
- [ ] b) `$PSScriptRoot`
- [ ] c) `Get-Location`
- [ ] d) `$MyInvocation.MyCommand.Path`

<details>
  <summary>Ver Fuente</summary>
  <p><a href="./01-scripts-automatizacion.md">Fuente: Scripts y Automatización Básica</a></p>
</details>

**9. ¿Cuál es el propósito del comando `set -e` en Bash?**

- [ ] a) Activar el modo debug
- [ ] b) Salir inmediatamente si un comando falla
- [ ] c) Mostrar cada comando antes de ejecutarlo
- [ ] d) Ignorar errores

<details>
  <summary>Ver Fuente</summary>
  <p><a href="./01-scripts-automatizacion.md">Fuente: Scripts y Automatización Básica</a></p>
</details>

**10. Verdadero/Falso: `cron` es la única manera de programar tareas automáticas en sistemas Linux.**

<details>
  <summary>Ver Fuente</summary>
  <p><a href="./01-scripts-automatizacion.md">Fuente: Scripts y Automatización Básica</a></p>
</details>

---

## Sección 2: Herramientas de Automatización (8 preguntas)

**11. En un Makefile, ¿qué indica una línea que comienza con TAB?**

- [ ] a) Un comentario
- [ ] b) Una variable
- [ ] c) Un comando
- [ ] d) Una dependencia

<details>
  <summary>Ver Fuente</summary>
  <p><a href="./02-herramientas-automatizacion.md">Fuente: Herramientas de Automatización</a></p>
</details>

**12. ¿Cuál es la estructura básica de un playbook de Ansible?**

- [ ] a) tasks, handlers, vars
- [ ] b) hosts, tasks, roles
- [ ] c) name, hosts, tasks
- [ ] d) inventory, playbook, modules

<details>
  <summary>Ver Fuente</summary>
  <p><a href="./02-herramientas-automatizacion.md">Fuente: Herramientas de Automatización</a></p>
</details>

**13. En Puppet, ¿qué es un "manifest"?**

- [ ] a) Un archivo de configuración de red
- [ ] b) Un archivo con definiciones de recursos
- [ ] c) Un log de cambios
- [ ] d) Un certificado de seguridad

<details>
  <summary>Ver Fuente</summary>
  <p><a href="./02-herramientas-automatizacion.md">Fuente: Herramientas de Automatización</a></p>
</details>

**14. Completa el código de Ansible para instalar nginx: **

```yaml
- name: Install nginx
  ______:
    name: nginx
    state: present
```

<details>
  <summary>Ver Fuente</summary>
  <p><a href="./02-herramientas-automatizacion.md">Fuente: Herramientas de Automatización</a></p>
</details>

**15. ¿Cuál es la principal diferencia entre Chef y Ansible?**

- [ ] a) Chef es para Windows, Ansible para Linux
- [ ] b) Chef requiere agente, Ansible no
- [ ] c) No hay diferencias significativas
- [ ] d) Chef es más rápido

<details>
  <summary>Ver Fuente</summary>
  <p><a href="./02-herramientas-automatizacion.md">Fuente: Herramientas de Automatización</a></p>
</details>

**16. En un Makefile, ¿qué indica `.PHONY:`?**

- [ ] a) Una variable especial
- [ ] b) Indica targets que no son archivos
- [ ] c) Un comentario
- [ ] d) Una dependencia circular

<details>
  <summary>Ver Fuente</summary>
  <p><a href="./02-herramientas-automatizacion.md">Fuente: Herramientas de Automatización</a></p>
</details>

**17. Verdadero/Falso: Terraform puede gestionar tanto infraestructura en la nube como on-premises.**

<details>
  <summary>Ver Fuente</summary>
  <p><a href="./02-herramientas-automatizacion.md">Fuente: Herramientas de Automatización</a></p>
</details>

**18. ¿Cuál es el archivo principal de configuración de Puppet?**

- [ ] a) `puppet.conf`
- [ ] b) `manifest.pp`
- [ ] c) `site.pp`
- [ ] d) `puppet.yaml`

<details>
  <summary>Ver Fuente</summary>
  <p><a href="./02-herramientas-automatizacion.md">Fuente: Herramientas de Automatización</a></p>
</details>

---

## Sección 3: Jenkins y Pipelines (8 preguntas)

**19. ¿Cuál es la diferencia entre pipeline declarativo e imperativo en Jenkins?**

- [ ] a) No hay diferencia
- [ ] b) Declarativo es más estructurado, imperativo más flexible
- [ ] c) Imperativo es más nuevo
- [ ] d) Declarativo solo funciona en Windows

<details>
  <summary>Ver Fuente</summary>
  <p><a href="./03-jenkins-ci.md">Fuente: Jenkins y Pipelines</a></p>
</details>

**20. Completa el bloque de pipeline Jenkins: **

```groovy
pipeline {
    agent any
    stages {
        stage('Build') {
            ______ {
                sh 'make build'
            }
        }
    }
}
```

<details>
  <summary>Ver Fuente</summary>
  <p><a href="./03-jenkins-ci.md">Fuente: Jenkins y Pipelines</a></p>
</details>

**21. ¿Para qué sirve la directiva `when` en un pipeline Jenkins?**

- [ ] a) Para definir variables
- [ ] b) Establece condiciones para ejecutar un stage
- [ ] c) Para manejar errores
- [ ] d) Para configurar notificaciones

<details>
  <summary>Ver Fuente</summary>
  <p><a href="./03-jenkins-ci.md">Fuente: Jenkins y Pipelines</a></p>
</details>

**22. En Jenkins, ¿qué es un "node"?**

- [ ] a) Una dependencia del proyecto
- [ ] b) Un nodo donde se ejecutan los trabajos
- [ ] c) Un archivo de configuración
- [ ] d) Un plugin específico

<details>
  <summary>Ver Fuente</summary>
  <p><a href="./03-jenkins-ci.md">Fuente: Jenkins y Pipelines</a></p>
</details>

**23. Verdadero/Falso: Jenkins puede integrar con sistemas de control de versiones como Git.**

<details>
  <summary>Ver Fuente</summary>
  <p><a href="./03-jenkins-ci.md">Fuente: Jenkins y Pipelines</a></p>
</details>

**24. ¿Qué es un "artifact" en Jenkins?**

- [ ] a) Un error en el pipeline
- [ ] b) Archivos generados por el build
- [ ] c) Una configuración del sistema
- [ ] d) Un plugin de terceros

<details>
  <summary>Ver Fuente</summary>
  <p><a href="./03-jenkins-ci.md">Fuente: Jenkins y Pipelines</a></p>
</details>

**25. ¿Qué significa "Pipeline as Code"?**

- [ ] a) Escribir código dentro del pipeline
- [ ] b) Definición del pipeline como código
- [ ] c) Ejecutar solo código compilado
- [ ] d) Usar solo lenguajes de programación

<details>
  <summary>Ver Fuente</summary>
  <p><a href="./03-jenkins-ci.md">Fuente: Jenkins y Pipelines</a></p>
</details>

**26. ¿Dónde se almacenan de forma segura las credenciales en Jenkins?**

- [ ] a) En el código del pipeline
- [ ] b) En variables de entorno del sistema
- [ ] c) Jenkins Credential Store
- [ ] d) En archivos de configuración

<details>
  <summary>Ver Fuente</summary>
  <p><a href="./03-jenkins-ci.md">Fuente: Jenkins y Pipelines</a></p>
</details>

---

## Sección 4: Plataformas CI/CD Modernas (7 preguntas)

**27. ¿Qué es un "workflow" en GitHub Actions?**

- [ ] a) Un script de bash
- [ ] b) Un conjunto de jobs automatizados
- [ ] c) Una configuración de red
- [ ] d) Un tipo de repositorio

<details>
  <summary>Ver Fuente</summary>
  <p><a href="./04-plataformas-ci-cd-modernas.md">Fuente: Plataformas CI/CD Modernas</a></p>
</details>

**28. Completa la configuración de GitHub Actions: **

```yaml
jobs:
  build:
    ______: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
```

<details>
  <summary>Ver Fuente</summary>
  <p><a href="./04-plataformas-ci-cd-modernas.md">Fuente: Plataformas CI/CD Modernas</a></p>
</details>

**29. ¿Cuál es una ventaja principal de GitLab CI/CD?**

- [ ] a) Solo funciona con GitLab
- [ ] b) Integración nativa con Git
- [ ] c) Es más lento que otras opciones
- [ ] d) Requiere instalación separada

<details>
  <summary>Ver Fuente</summary>
  <p><a href="./04-plataformas-ci-cd-modernas.md">Fuente: Plataformas CI/CD Modernas</a></p>
</details>

**30. En CircleCI, ¿qué son los "artifacts"?**

- [ ] a) Errores de configuración
- [ ] b) Archivos generados por el build
- [ ] c) Variables de entorno
- [ ] d) Scripts de deployment

<details>
  <summary>Ver Fuente</summary>
  <p><a href="./04-plataformas-ci-cd-modernas.md">Fuente: Plataformas CI/CD Modernas</a></p>
</details>

**31. Verdadero/Falso: Azure DevOps puede integrar con repositorios de GitHub.**

<details>
  <summary>Ver Fuente</summary>
  <p><a href="./04-plataformas-ci-cd-modernas.md">Fuente: Plataformas CI/CD Modernas</a></p>
</details>

**32. ¿Para qué sirve una "matrix strategy" en GitHub Actions?**

- [ ] a) Para definir variables
- [ ] b) Build en paralelo con diferentes configuraciones
- [ ] c) Para manejar errores
- [ ] d) Para configurar notificaciones

<details>
  <summary>Ver Fuente</summary>
  <p><a href="./04-plataformas-ci-cd-modernas.md">Fuente: Plataformas CI/CD Modernas</a></p>
</details>

**33. ¿Qué hace la directiva `artifacts:` en GitLab CI?**

- [ ] a) Define variables del proyecto
- [ ] b) Especifica archivos a preservar entre jobs
- [ ] c) Configura notificaciones
- [ ] d) Establece dependencias

<details>
  <summary>Ver Fuente</summary>
  <p><a href="./04-plataformas-ci-cd-modernas.md">Fuente: Plataformas CI/CD Modernas</a></p>
</details>

---

## Sección 5: Testing y Quality Gates (7 preguntas)

**34. ¿Cuál es el orden correcto de ejecución de tests en un pipeline?**

- [ ] a) E2E → Integration → Unit
- [ ] b) Unit → Integration → E2E
- [ ] c) Integration → Unit → E2E
- [ ] d) No importa el orden

<details>
  <summary>Ver Fuente</summary>
  <p><a href="./05-testing-automation-quality-gates.md">Fuente: Testing y Quality Gates</a></p>
</details>

**35. Completa el comando para test de carga con Artillery: **

```bash
artillery quick --count 100 --num 10 http://localhost:3000
```

¿Qué herramienta similar pero más moderna podrías usar?

<details>
  <summary>Ver Fuente</summary>
  <p><a href="./05-testing-automation-quality-gates.md">Fuente: Testing y Quality Gates</a></p>
</details>

**36. ¿Qué significa SAST en el contexto de testing?**

- [ ] a) Static Application Security Testing
- [ ] b) Scalable Application System Testing
- [ ] c) Simple Automated Software Testing
- [ ] d) Secure Application Standard Testing

<details>
  <summary>Ver Fuente</summary>
  <p><a href="./05-testing-automation-quality-gates.md">Fuente: Testing y Quality Gates</a></p>
</details>

**37. ¿Qué son los "quality gates"?**

- [ ] a) Herramientas de testing específicas
- [ ] b) Criterios mínimos de calidad de código
- [ ] c) Tipos de tests automatizados
- [ ] d) Configuraciones de CI/CD

<details>
  <summary>Ver Fuente</summary>
  <p><a href="./05-testing-automation-quality-gates.md">Fuente: Testing y Quality Gates</a></p>
</details>

**38. Verdadero/Falso: Los tests de integración deben ejecutarse antes que los unit tests.**

<details>
  <summary>Ver Fuente</summary>
  <p><a href="./05-testing-automation-quality-gates.md">Fuente: Testing y Quality Gates</a></p>
</details>

**39. ¿Qué es un "smoke test"?**

- [ ] a) Test de rendimiento
- [ ] b) Test básico para verificar funcionalidad crítica
- [ ] c) Test de seguridad
- [ ] d) Test de interfaz

<details>
  <summary>Ver Fuente</summary>
  <p><a href="./05-testing-automation-quality-gates.md">Fuente: Testing y Quality Gates</a></p>
</details>

**40. ¿Cuál de estas es una herramienta moderna de testing de carga?**

- [ ] a) JMeter
- [ ] b) K6
- [ ] c) Artillery
- [ ] d) Todas las anteriores

<details>
  <summary>Ver Fuente</summary>
  <p><a href="./05-testing-automation-quality-gates.md">Fuente: Testing y Quality Gates</a></p>
</details>

---

## Sección 6: Estrategias de Deployment (5 preguntas)

**41. ¿Qué es un deployment "Blue-Green"?**

- [ ] a) Usar dos colores en la interfaz
- [ ] b) Mantener dos entornos idénticos para switch instantáneo
- [ ] c) Desplegar solo en horarios específicos
- [ ] d) Una estrategia de backup

<details>
  <summary>Ver Fuente</summary>
  <p><a href="./06-estrategias-deployment-avanzadas.md">Fuente: Estrategias de Deployment</a></p>
</details>

**42. Verdadero/Falso: El deployment "Rolling" permite actualizaciones graduales sin downtime.**

<details>
  <summary>Ver Fuente</summary>
  <p><a href="./06-estrategias-deployment-avanzadas.md">Fuente: Estrategias de Deployment</a></p>
</details>

**43. ¿Cuál es la principal ventaja del deployment "Canary"?**

- [ ] a) Es más rápido
- [ ] b) No requiere downtime
- [ ] c) Permite probar con un subconjunto de usuarios
- [ ] d) Usa menos recursos

<details>
  <summary>Ver Fuente</summary>
  <p><a href="./06-estrategias-deployment-avanzadas.md">Fuente: Estrategias de Deployment</a></p>
</details>

**44. ¿Qué son los "feature flags"?**

- [ ] a) Errores en el código
- [ ] b) Configuraciones para habilitar/deshabilitar funcionalidades
- [ ] c) Tipos de tests
- [ ] d) Herramientas de monitoreo

<details>
  <summary>Ver Fuente</summary>
  <p><a href="./06-estrategias-deployment-avanzadas.md">Fuente: Estrategias de Deployment</a></p>
</details>

**45. ¿Dónde es recomendable almacenar las configuraciones de deployment?**

- [ ] a) En la base de datos
- [ ] b) En un repositorio Git
- [ ] c) En archivos locales
- [ ] d) En variables de sistema

<details>
  <summary>Ver Fuente</summary>
  <p><a href="./06-estrategias-deployment-avanzadas.md">Fuente: Estrategias de Deployment</a></p>
</details>

---

## Respuestas Correctas

<details>
  <summary>Ver Respuestas</summary>

  **1. b) `$@` trata cada argumento como elemento separado, `$*` como string único**
  **2. d) Todas las anteriores**
  **3. Verdadero**
  **4. b) Elimina archivos .log más antiguos que 7 días**
  **5. b) Usar variables de entorno**
  **6. b) Ejecución exitosa**
  **7. `date +%Y%m%d_%H%M%S`**
  **8. b) `$PSScriptRoot`**
  **9. b) Salir inmediatamente si un comando falla**
  **10. Falso**
  **11. c) Un comando**
  **12. c) name, hosts, tasks**
  **13. b) Un archivo con definiciones de recursos**
  **14. `package` o `apt` o `yum`**
  **15. b) Chef requiere agente, Ansible no**
  **16. b) Indica targets que no son archivos**
  **17. Verdadero**
  **18. a) `puppet.conf`**
  **19. b) Declarativo es más estructurado, imperativo más flexible**
  **20. `steps`**
  **21. b) Establece condiciones para ejecutar un stage**
  **22. b) Un nodo donde se ejecutan los trabajos**
  **23. Verdadero**
  **24. b) Archivos generados por el build**
  **25. b) Definición del pipeline como código**
  **26. c) Jenkins Credential Store**
  **27. b) Un conjunto de jobs automatizados**
  **28. `runs-on`**
  **29. b) Integración nativa con Git**
  **30. b) Archivos generados por el build**
  **31. Verdadero**
  **32. b) Build en paralelo con diferentes configuraciones**
  **33. b) Especifica archivos a preservar entre jobs**
  **34. b) Unit → Integration → E2E**
  **35. `K6`**
  **36. a) Static Application Security Testing**
  **37. b) Criterios mínimos de calidad de código**
  **38. Falso**
  **39. b) Test básico para verificar funcionalidad crítica**
  **40. d) Todas las anteriores**
  **41. b) Mantener dos entornos idénticos para switch instantáneo**
  **42. Verdadero**
  **43. c) Permite probar con un subconjunto de usuarios**
  **44. b) Configuraciones para habilitar/deshabilitar funcionalidades**
  **45. b) En un repositorio Git**

</details>

---

## Evaluación de Resultados

### Escala de Calificación

- **42-45 puntos (93-100%)**: **Experto** - Dominio excepcional en automatización y CI/CD
- **38-41 puntos (84-91%)**: **Avanzado** - Sólido conocimiento de pipelines y deployment
- **32-37 puntos (71-82%)**: **Competente** - Aprobado, conocimiento suficiente para continuar
- **27-31 puntos (60-69%)**: **En desarrollo** - Revisa conceptos específicos antes de continuar
- **Menos de 27 puntos (<60%)**: **Necesita estudio** - Repasa toda la Fase 2

### Recomendaciones por Sección

#### **Scripts y Automatización Básica (Preguntas 1-10)**

Si obtuviste <70%:

- Practica scripting en Bash, PowerShell y Python
- Refuerza conceptos de manejo de errores y variables de entorno
- Implementa scripts de automatización simples
- Revisa [01-scripts-automatizacion.md](01-scripts-automatizacion.md)

#### **Herramientas de Automatización (Preguntas 11-20)**

Si obtuviste <70%:

- Familiarízate con Ansible, Terraform y Vagrant
- Practica configuración Infrastructure as Code
- Comprende diferencias entre herramientas de orquestación
- Revisa [02-herramientas-automatizacion.md](02-herramientas-automatizacion.md)

#### **Jenkins y CI/CD (Preguntas 21-28)**

Si obtuviste <70%:

- Configura un Jenkins local y crea pipelines básicos
- Aprende sintaxis de Jenkinsfile y Groovy DSL
- Practica con plugins esenciales (Git, Docker, Pipeline)
- Revisa [03-jenkins-cicd.md](03-jenkins-cicd.md)

#### **Integración con Git y Plataformas (Preguntas 29-35)**

Si obtuviste <70%:

- Implementa GitHub Actions workflows
- Configura GitLab CI/CD pipelines
- Practica webhooks y triggers automáticos
- Revisa [04-integracion-git-plataformas.md](04-integracion-git-plataformas.md)

#### **Testing y Quality Gates (Preguntas 36-40)**

Si obtuviste <70%:

- Implementa testing automatizado en pipelines
- Configura quality gates y code coverage
- Aprende herramientas SAST/DAST
- Revisa [05-testing-quality-gates.md](05-testing-quality-gates.md)

#### **Deployment Strategies (Preguntas 41-45)**

Si obtuviste <70%:

- Practica Blue-Green y Canary deployments
- Implementa feature flags y rolling updates
- Comprende estrategias de rollback
- Revisa [06-deployment-strategies.md](06-deployment-strategies.md)

### Plan de Mejora por Puntuación

#### **Puntuación Alta (42-45 puntos)**

🎉 **¡Excelente!** Eres un experto en CI/CD

- Procede con confianza a [Fase 3: Contenedores y Orquestación](../fase3/)
- Considera liderar proyectos de implementación CI/CD
- Comparte tu conocimiento mentoreando a otros
- Explora herramientas avanzadas como ArgoCD y Flux

#### **Puntuación Buena (38-41 puntos)**

✅ **¡Muy bien!** Tienes sólidos conocimientos prácticos

- Revisa las preguntas que fallaste para refinar detalles
- Implementa un pipeline completo end-to-end como práctica
- Procede a la Fase 3 después del repaso específico

#### **Puntuación Competente (32-37 puntos)**

📚 **Aprobado** pero necesitas más práctica

- Dedica tiempo extra a las secciones con errores
- Implementa los laboratorios prácticos de cada módulo
- Crea un proyecto personal con pipeline completo
- Considera retomar el test después de la práctica

#### **Puntuación En Desarrollo (27-31 puntos)**

⚠️ **Necesitas más preparación**

- Identifica tus 3 secciones más débiles
- Estudia intensivamente esos módulos específicos
- Practica con herramientas reales (Jenkins, GitHub Actions)
- Busca proyectos open source para contribuir
- Retoma el test cuando tengas más experiencia práctica

#### **Puntuación Insuficiente (<27 puntos)**

🔄 **Reinicia el proceso de aprendizaje**

- Vuelve a estudiar toda la Fase 2 desde el principio
- Toma más tiempo con cada módulo y laboratorio
- Busca mentoría en comunidades DevOps
- Considera un enfoque más gradual con proyectos pequeños
- Practica scripting básico antes de CI/CD avanzado

### Recursos Adicionales para Mejorar

#### **Documentación y Guías**

- [Roadmap Completo DevOps](../../roadmap.md)
- [Notas de Docker](../../notes/docker-notes.md)
- [Notas de Kubernetes](../../notes/kubernetes-notes.md)
- [Recursos de Libros](../books.md)
- [Herramientas Recomendadas](../tools.md)

#### **Práctica Hands-on**

- Configura Jenkins local con Docker
- Crea repositorios con GitHub Actions/GitLab CI
- Implementa testing automatizado en proyectos
- Practica con Terraform y Ansible
- Únete a comunidades CI/CD y DevOps

#### **Certificaciones Preparatorias**

- Jenkins Certified Engineer
- GitHub Actions certification
- AWS DevOps Engineer Professional
- Azure DevOps Engineer Expert
- GitLab Certified CI/CD Specialist

#### **Proyectos Prácticos Sugeridos**

1. **Pipeline Básico**: Aplicación web con build, test y deploy
2. **Multi-stage Pipeline**: Con quality gates y approval processes
3. **Infrastructure Pipeline**: Terraform + Ansible automation
4. **Microservices CI/CD**: Pipeline para múltiples servicios
5. **Security-first Pipeline**: Con SAST, DAST y dependency scanning

---

## Certificado de Completado

**Si obtuviste 32 puntos o más, has completado exitosamente la Fase 2!**

```plaintext
╔══════════════════════════════════════════════════════╗
║              CERTIFICADO DE COMPLETADO              ║
║                                                      ║
║                    [Tu Nombre]                      ║
║                                                      ║
║           Ha completado exitosamente la             ║
║                                                      ║
║         FASE 2: AUTOMATIZACIÓN Y CI/CD              ║
║                                                      ║
║            Del curso "DevOps Learning Path"         ║
║                                                      ║
║    Puntuación obtenida: [Tu puntuación]/45         ║
║    Fecha de completado: 12 de octubre de 2025      ║
║                                                      ║
║      ¡Listo para continuar con la Fase 3!          ║
║                                                      ║
╚══════════════════════════════════════════════════════╝
```

---

## Próximos Pasos

### **Si Aprobaste (≥32 puntos)**

1. **🎊 Celebra tu logro** - Dominas automatización y CI/CD
2. **📝 Actualiza tu perfil** - Agrega estas habilidades a tu CV y LinkedIn
3. **➡️ Procede a la [Fase 3: Contenedores y Orquestación](../fase3/)**
4. **🛠️ Implementa pipelines** - Aplica lo aprendido en proyectos reales
5. **🤝 Contribuye** - Ayuda a otros en comunidades DevOps

### **Si Necesitas Mejorar (<32 puntos)**

1. **🔍 Análisis detallado** - Identifica secciones específicas problemáticas
2. **📖 Estudio dirigido** - Enfócate en módulos con menor puntuación
3. **🔬 Práctica intensiva** - Implementa laboratorios y proyectos
4. **🏗️ Construye experiencia** - Crea pipelines desde cero
5. **🔄 Reevaluación** - Retoma el test con más preparación

### **Recursos de Práctica Recomendados**

#### **Para Scripting y Automatización**
- Automatiza tareas cotidianas con scripts
- Crea herramientas CLI personalizadas
- Implementa monitoring y alerting básico

#### **Para CI/CD Pipelines**
- Configura Jenkins en Docker
- Experimenta con GitHub Actions
- Implementa pipelines multi-rama
- Practica deployment strategies

#### **Para Testing Automation**
- Integra unit tests en pipelines
- Configura code coverage reports
- Implementa integration testing
- Practica con quality gates

---

**¡Felicitaciones por completar la evaluación de la Fase 2!** 
*Este conocimiento en automatización y CI/CD te posiciona para implementar pipelines de nivel empresarial y liderar transformaciones DevOps.*
