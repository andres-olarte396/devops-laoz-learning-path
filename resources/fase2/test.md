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

> Contenido de referencia: [01-scripts-automatizacion.md](01-scripts-automatizacion.md)

### Pregunta 1

> Ver: [01-scripts-automatizacion.md - Variables especiales en Bash](01-scripts-automatizacion.md)

¿Cuál es la diferencia principal entre `$@` y `$*` en un script de Bash?

a) No hay diferencia, ambos representan todos los argumentos  
b) `$@` trata cada argumento como elemento separado, `$*` como string único  
c) `$*` es para arrays, `$@` para strings  
d) `$@` incluye el nombre del script, `$*` no  

### Pregunta 2
>
### Pregunta 2

> Ver: [01-scripts-automatizacion.md - Manejo de errores en PowerShell](01-scripts-automatizacion.md)

En PowerShell, ¿cuál es la forma correcta de manejar errores en un script?

a) `try { } catch { } finally { }`  
b) `if ($Error) { }`  
c) `ErrorAction = "Stop"`  
d) Todas las anteriores  

### Pregunta 3
>
>  *Ver: [01-scripts-automatizacion.md - Ejecución segura de comandos en Python](01-scripts-automatizacion.md)*

**Verdadero/Falso**: En Python, `subprocess.run()` es más seguro que `os.system()` para ejecutar comandos del sistema.

### Pregunta 4

¿Qué hace este comando de Bash?

```bash
find /var/log -name "*.log" -mtime +7 -exec rm {} \;
```

a) Busca archivos .log modificados en los últimos 7 días  
b) Elimina archivos .log más antiguos que 7 días  
c) Comprime archivos .log de más de 7 días  
d) Mueve archivos .log antiguos a backup  

### Pregunta 5
>
>  *Ver: [01-scripts-automatizacion.md - Mejores prácticas de seguridad](01-scripts-automatizacion.md)*

En un script de automatización, ¿cuál es la mejor práctica para manejar credenciales?

a) Hardcodearlas en el script  
b) Usar variables de entorno  
c) Almacenarlas en un archivo de configuración  
d) Pedirlas por input cada vez  

### Pregunta 6

¿Qué significa el código de salida `0` en un script?

a) Error crítico  
b) Ejecución exitosa  
c) Warning  
d) Proceso cancelado  

### Pregunta 7

**Completa el código**: Para hacer un backup automático con timestamp en Bash:

```bash
DATE=$(______)
cp database.sql "backup_${DATE}.sql"
```

### Pregunta 8

En PowerShell, ¿cómo obtienes el directorio actual del script?

a) `$PWD`  
b) `$PSScriptRoot`  
c) `Get-Location`  
d) `$MyInvocation.MyCommand.Path`  

### Pregunta 9

¿Cuál es el propósito del comando `set -e` en Bash?

a) Activar el modo debug  
b) Salir inmediatamente si un comando falla  
c) Mostrar cada comando antes de ejecutarlo  
d) Ignorar errores  

### Pregunta 10

**Verdadero/Falso**: `cron` es la única manera de programar tareas automáticas en sistemas Linux.

---

##  **Sección 2: Herramientas de Automatización** (8 preguntas)

>  **Contenido de referencia:** [02-herramientas-automatizacion.md](02-herramientas-automatizacion.md)

### Pregunta 11

En un Makefile, ¿qué indica una línea que comienza con TAB?

a) Un comentario  
b) Una variable  
c) Un comando  
d) Una dependencia  

### Pregunta 12
>
>  *Ver: [02-herramientas-automatizacion.md - Estructura de playbooks Ansible](02-herramientas-automatizacion.md)*

¿Cuál es la estructura básica de un playbook de Ansible?

a) tasks, handlers, vars  
b) hosts, tasks, roles  
c) name, hosts, tasks  
d) inventory, playbook, modules  

### Pregunta 13

En Puppet, ¿qué es un "manifest"?

a) Un archivo de configuración de red  
b) Un archivo con definiciones de recursos  
c) Un log de cambios  
d) Un certificado de seguridad  

### Pregunta 14

**Completa el código** de Ansible para instalar nginx:

```yaml
- name: Install nginx
  ______:
    name: nginx
    state: present
```

### Pregunta 15

¿Cuál es la diferencia principal entre Chef y Ansible?

a) Chef usa Ruby, Ansible usa Python  
b) Chef requiere agente, Ansible no  
c) Chef es push, Ansible es pull  
d) Chef es para Windows, Ansible para Linux  

### Pregunta 16

En Make, ¿qué hace la directiva `.PHONY`?

a) Define variables phantom  
b) Indica targets que no son archivos  
c) Ejecuta comandos en paralelo  
d) Oculta la salida de comandos  

### Pregunta 17

**Verdadero/Falso**: Ansible idempotente significa que ejecutar el mismo playbook múltiples veces produce el mismo resultado.

### Pregunta 18

¿Qué archivo usa Puppet para definir la configuración del agente?

a) `puppet.conf`  
b) `site.pp`  
c) `manifest.pp`  
d) `modules.conf`  

---

##  **Sección 3: Jenkins CI/CD** (8 preguntas)

>  **Contenido de referencia:** [03-jenkins-ci.md](03-jenkins-ci.md)

### Pregunta 19
>
>  *Ver: [03-jenkins-ci.md - Tipos de pipelines: Declarativo vs Imperativo](03-jenkins-ci.md)*

¿Cuál es la diferencia entre un pipeline declarativo e imperativo en Jenkins?

a) Declarativo usa script{}, imperativo usa steps{}  
b) Declarativo es más estructurado, imperativo más flexible  
c) No hay diferencia real  
d) Imperativo es solo para pipelines simples  

### Pregunta 20

**Completa el código** de Jenkinsfile para definir un stage:

```groovy
pipeline {
    agent any
    stages {
        stage('Build') {
            ______ {
                sh 'npm install'
            }
        }
    }
}
```

### Pregunta 21

¿Qué hace la directiva `when` en un pipeline de Jenkins?

a) Define cuándo se ejecuta el pipeline  
b) Establece condiciones para ejecutar un stage  
c) Configura el timeout  
d) Define variables de entorno  

### Pregunta 22

En Jenkins, ¿qué es un "agente"?

a) Un usuario con permisos administrativos  
b) Un nodo donde se ejecutan los trabajos  
c) Un plugin de seguridad  
d) Un tipo de credencial  

### Pregunta 23

**Verdadero/Falso**: Blue Ocean es la interfaz moderna recomendada para Jenkins.

### Pregunta 24

¿Cuál es el propósito de `archiveArtifacts` en Jenkins?

a) Comprimir archivos del workspace  
b) Guardar archivos para uso posterior  
c) Subir archivos a repositorio  
d) Limpiar archivos temporales  

### Pregunta 25

¿Qué información contiene el archivo `Jenkinsfile`?

a) Configuración del servidor Jenkins  
b) Definición del pipeline como código  
c) Lista de plugins instalados  
d) Configuración de usuarios  

### Pregunta 26

En Jenkins, ¿cómo se configuran credenciales de forma segura?

a) Variables de entorno en el pipeline  
b) Archivos de configuración  
c) Jenkins Credential Store  
d) Hardcodeadas en el script  

---

##  **Sección 4: Plataformas CI/CD Modernas** (7 preguntas)

>  **Contenido de referencia:** [04-plataformas-ci-cd-modernas.md](04-plataformas-ci-cd-modernas.md)

### Pregunta 27
>
>  *Ver: [04-plataformas-ci-cd-modernas.md - Conceptos básicos de GitHub Actions](04-plataformas-ci-cd-modernas.md)*

En GitHub Actions, ¿qué define un "workflow"?

a) Un repositorio de código  
b) Un conjunto de jobs automatizados  
c) Una rama específica  
d) Un usuario colaborador  

### Pregunta 28

**Completa el YAML** de GitHub Actions:

```yaml
name: CI
on: [push]
jobs:
  test:
    ______: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
```

### Pregunta 29

¿Cuál es la principal ventaja de GitLab CI/CD sobre Jenkins?

a) Mayor velocidad de ejecución  
b) Integración nativa con Git  
c) Mejor interfaz de usuario  
d) Soporte para más lenguajes  

### Pregunta 30

En Azure DevOps, ¿qué son los "artifacts"?

a) Errores en el código  
b) Archivos generados por el build  
c) Métricas de performance  
d) Logs de ejecución  

### Pregunta 31

**Verdadero/Falso**: GitHub Actions permite crear workflows que se ejecuten en múltiples sistemas operativos simultáneamente.

### Pregunta 32

¿Qué significa "matrix build" en CI/CD?

a) Build que usa matrices matemáticas  
b) Build en paralelo con diferentes configuraciones  
c) Build que genera múltiples artefactos  
d) Build con dependencias complejas  

### Pregunta 33

En GitLab CI/CD, ¿cuál es el propósito de la palabra clave `artifacts`?

a) Definir dependencias del proyecto  
b) Especificar archivos a preservar entre jobs  
c) Configurar notificaciones  
d) Establecer variables de entorno  

---

##  **Sección 5: Testing Automation y Quality Gates** (7 preguntas)

>  **Contenido de referencia:** [05-testing-automation-quality-gates.md](05-testing-automation-quality-gates.md)

### Pregunta 34
>
>  *Ver: [05-testing-automation-quality-gates.md - Pirámide de testing](05-testing-automation-quality-gates.md)*

¿Cuál es el orden correcto en la pirámide de testing?

a) E2E → Integration → Unit  
b) Unit → Integration → E2E  
c) Integration → Unit → E2E  
d) Unit → E2E → Integration  

### Pregunta 35

**Completa el código** de Jest para un test unitario:

```javascript
describe('Calculator', () => {
  test('should add two numbers', () => {
    const result = add(2, 3);
    ______(result).toBe(5);
  });
});
```

### Pregunta 36

¿Qué es SAST en security testing?

a) Static Application Security Testing  
b) Software Application Security Tools  
c) Secure Application Source Testing  
d) System Application Security Test  

### Pregunta 37

En SonarQube, ¿qué indica el "Quality Gate"?

a) La puerta de entrada al repositorio  
b) Criterios mínimos de calidad de código  
c) El gateway de la aplicación  
d) La configuración de red  

### Pregunta 38

**Verdadero/Falso**: Los tests de integración deben ejecutarse antes que los tests unitarios en un pipeline CI/CD.

### Pregunta 39

¿Cuál es el propósito de un "smoke test"?

a) Verificar que la aplicación no se incendie  
b) Test básico para verificar funcionalidad crítica  
c) Test de carga máxima  
d) Test de seguridad  

### Pregunta 40

¿Qué herramienta se usa comúnmente para performance testing?

a) Jest  
b) K6  
c) ESLint  
d) SonarQube  

---

##  **Sección 6: Estrategias de Deployment** (5 preguntas)

>  **Contenido de referencia:** [06-estrategias-deployment-avanzadas.md](06-estrategias-deployment-avanzadas.md)

### Pregunta 40
>
>  *Ver: [06-estrategias-deployment-avanzadas.md - Estrategia Blue-Green](06-estrategias-deployment-avanzadas.md)*

¿En qué consiste un "Blue-Green deployment"?

a) Usar dos colores para la interfaz  
b) Mantener dos entornos idénticos para switch instantáneo  
c) Desplegar en horarios específicos  
d) Usar dos equipos de desarrollo  

### Pregunta 42

**Verdadero/Falso**: En un Canary deployment, se dirige un pequeño porcentaje de tráfico a la nueva versión para monitorear su comportamiento.

### Pregunta 43

¿Cuál es la principal ventaja de un Rolling deployment?

a) Es más rápido que otros métodos  
b) No requiere downtime  
c) Usa menos recursos  
d) Es más fácil de configurar  

### Pregunta 44

¿Qué son los "Feature Flags"?

a) Banderas nacionales en el código  
b) Configuraciones para habilitar/deshabilitar funcionalidades  
c) Marcadores de errores  
d) Indicadores de performance  

### Pregunta 45

En GitOps, ¿dónde se almacena la configuración de infraestructura?

a) En la base de datos  
b) En un repositorio Git  
c) En el servidor de producción  
d) En archivos de configuración locales  

---

##  **Respuestas Correctas**

### Sección 1: Scripts y Automatización

1. **b)** `$@` trata cada argumento como elemento separado, `$*` como string único
2. **d)** Todas las anteriores
3. **Verdadero** - `subprocess.run()` es más seguro y controlable
4. **b)** Elimina archivos .log más antiguos que 7 días
5. **b)** Usar variables de entorno
6. **b)** Ejecución exitosa
7. **`date +%Y%m%d_%H%M%S`** o **`date +%Y-%m-%d`**
8. **b)** `$PSScriptRoot`
9. **b)** Salir inmediatamente si un comando falla
10. **Falso** - También están systemd timers, at, etc.

### Sección 2: Herramientas de Automatización

11. **c)** Un comando
12. **c)** name, hosts, tasks
13. **b)** Un archivo con definiciones de recursos
14. **`package`** o **`apt`** o **`yum`**
15. **b)** Chef requiere agente, Ansible no
16. **b)** Indica targets que no son archivos
17. **Verdadero**
18. **a)** `puppet.conf`

### Sección 3: Jenkins CI/CD

19. **b)** Declarativo es más estructurado, imperativo más flexible
20. **`steps`**
21. **b)** Establece condiciones para ejecutar un stage
22. **b)** Un nodo donde se ejecutan los trabajos
23. **Verdadero**
24. **b)** Guardar archivos para uso posterior
25. **b)** Definición del pipeline como código
26. **c)** Jenkins Credential Store

### Sección 4: Plataformas CI/CD Modernas

27. **b)** Un conjunto de jobs automatizados
28. **`runs-on`**
29. **b)** Integración nativa con Git
30. **b)** Archivos generados por el build
31. **Verdadero**
32. **b)** Build en paralelo con diferentes configuraciones
33. **b)** Especificar archivos a preservar entre jobs

### Sección 5: Testing y Quality Gates

34. **b)** Unit → Integration → E2E
35. **`expect`**
36. **a)** Static Application Security Testing
37. **b)** Criterios mínimos de calidad de código
38. **Falso** - Los unit tests van primero
39. **b)** Test básico para verificar funcionalidad crítica
40. **b)** K6

### Sección 6: Estrategias de Deployment

41. **b)** Mantener dos entornos idénticos para switch instantáneo
42. **Verdadero**
43. **b)** No requiere downtime
44. **b)** Configuraciones para habilitar/deshabilitar funcionalidades
45. **b)** En un repositorio Git

---

##  **Evaluación de Resultados**

### Escala de Calificación

- **42-45 puntos (93-100%)**:  **Experto** - Dominio excepcional en CI/CD
- **38-41 puntos (84-91%)**:  **Avanzado** - Sólido conocimiento práctico
- **32-37 puntos (71-82%)**:  **Competente** - Aprobado, refuerza algunos conceptos
- **27-31 puntos (60-69%)**:  **En desarrollo** - Revisa módulos específicos
- **Menos de 27 puntos (<60%)**:  **Necesita estudio** - Repasa toda la Fase 2

### Recomendaciones por Sección

- **Sección 1-2**: Si obtuviste <70%, repasa scripting y automation tools
- **Sección 3**: Si obtuviste <70%, practica más con Jenkins pipelines
- **Sección 4**: Si obtuviste <70%, experimenta con GitHub Actions/GitLab CI
- **Sección 5**: Si obtuviste <70%, implementa testing automation en proyectos
- **Sección 6**: Si obtuviste <70%, practica deployment strategies con containers

---

##  **Próximos Pasos**

### Si Aprobaste (≥70%)

1.  **Continúa a Fase 3**: Contenedores y Orquestación
2.  **Practica**: Implementa un pipeline completo end-to-end
3.  **Aplica**: Usa lo aprendido en proyectos reales

### Si No Aprobaste (<70%)

1.  **Repasa**: Los módulos donde obtuviste menor puntaje
2.  **Practica**: Implementa los laboratorios de cada módulo
3.  **Repite**: El test cuando te sientas preparado

---

**¡Felicitaciones por completar el test de la Fase 2!** Este conocimiento en Automation y CI/CD te posiciona para implementar pipelines de nivel empresarial. 
