<<<<<<< HEAD
# Test de la Fase 1: Fundamentos de DevOps

**Evalúa tu conocimiento de los conceptos fundamentales de DevOps!**

Este test abarca todo el contenido de la Fase 1 completada, incluyendo los 8 temas principales que has estudiado.

> **Instrucciones:**Selecciona la respuesta correcta para cada pregunta. Las respuestas se encuentran al final del documento.

---

## Temario Cubierto en el Test

**Fase 1 Completada - 8 Temas:**

1. **[¿Qué es DevOps?](01-que-es-devops.md)**- Cultura, herramientas y prácticas
2. **[Beneficios de DevOps](02-beneficios-debops.md)**- Ventajas técnicas y de negocio
3. **[DevOps vs Agile vs Waterfall](03-devops-agile-waterfall.md)**- Comparación de metodologías
4. **[Ciclo de Vida DevOps](04-ciclo-de-vida-devops.md)**- 8 fases del proceso
5. **[Herramientas y Tecnologías Clave](05-herramientas-tecnologias-clave.md)**- Stack tecnológico
6. **[Introducción a la Nube](06-introduccion-nube.md)**- Fundamentos cloud
7. **[Conceptos Clave: CI/CD, IaC y Observabilidad](07-conceptos-clave.md)**- Pilares técnicos
8. **[Primeros Pasos con Git](08-primeros-pasos-git.md)**- Control de versiones mastery

---

## 1. ¿Qué es DevOps? (Cultura, herramientas y prácticas)

>  **Contenido de referencia:**[01-que-es-devops.md](01-que-es-devops.md)

1. **¿Cuál es el objetivo principal de DevOps según el modelo CALMS?**

   - [ ] a) Reducir costos de hardware
   - [ ] b) Unificar desarrollo y operaciones para entregar software más rápido y confiable
   - [ ] c) Eliminar la necesidad de pruebas de software
   - [ ] d) Aumentar la duración de los proyectos

2. **¿Cuál de los siguientes NO es un principio fundamental de DevOps?**

   - [ ] a) Automatización
   - [ ] b) Colaboración
   - [ ] c) Silos organizacionales
   - [ ] d) Medición

3. **¿Qué significa "Shift Left" en DevOps?**

   - [ ] a) Mover las pruebas y seguridad más temprano en el ciclo de desarrollo
   - [ ] b) Cambiar la arquitectura del software
   - [ ] c) Reducir el número de desarrolladores
   - [ ] d) Implementar solo en servidores de la izquierda

# Test de Autoevaluación: Fundamentos de DevOps (Fase 0 y 1)

Este test está diseñado para ayudarte a evaluar tu comprensión de los conceptos fundamentales introducidos en las Fases 0 y 1. Cada pregunta incluye un enlace a la sección relevante del material de estudio para que puedas repasar.

---

## Sección 1: Fase 0 - Dominio de la Terminal

<<<<<<< HEAD
>  **Contenido de referencia:**[02-beneficios-debops.md](02-beneficios-debops.md)

4. **¿Cuál de los siguientes es un beneficio clave de DevOps?**

   - [ ] a) Mayor tiempo de entrega de software
   - [ ] b) Reducción de la colaboración entre equipos
   - [ ] c) Entrega más rápida de software con mayor calidad
   - [ ] d) Aumento de errores en producción

5. **¿Cómo mejora DevOps la calidad del software?**

   - [ ] a) Eliminando la necesidad de pruebas
   - [ ] b) Detectando errores tempranamente mediante pruebas automatizadas
   - [ ] c) Reduciendo la comunicación entre equipos
   - [ ] d) Aumentando el tiempo de desarrollo

6. **¿Qué métrica indica el éxito de una implementación DevOps?**

   - [ ] a) Mayor tiempo entre releases
   - [ ] b) Mean Time To Recovery (MTTR) reducido
   - [ ] c) Menos comunicación entre equipos
   - [ ] d) Mayor costo operacional

**Pregunta 1:**¿Cuál es el comando para crear un nuevo directorio llamado `proyecto` y luego mover un archivo `plan.txt` dentro de él?

- [ ] a) `mkdir proyecto` y luego `cp plan.txt proyecto/`
- [ ] b) `newdir proyecto` y luego `move plan.txt proyecto/`
- [ ] c) `mkdir proyecto` y luego `mv plan.txt proyecto/`
- [ ] d) `create proyecto` y luego `cp plan.txt proyecto/`

<details>
  <summary>Ver Fuente</summary>
  <p><a href="./../fase0/01-fundamentos-shell.md#2-navegación-y-manipulación-de-archivos">Fuente: Fase 0 - Fundamentos de la Shell</a></p>
</details>

**Pregunta 2:**Quieres que un script llamado `deploy.sh` sea ejecutable por su propietario, pero solo de lectura para todos los demás. ¿Qué comando `chmod` usarías?

- [ ] a) `chmod 644 deploy.sh`
- [ ] b) `chmod 755 deploy.sh`
- [ ] c) `chmod 744 deploy.sh`
- [ ] d) `chmod 655 deploy.sh`

<details>
  <summary>Ver Fuente</summary>
  <p><a href="./../fase0/01-fundamentos-shell.md#permisos">Fuente: Fase 0 - Permisos de Archivos</a></p>
</details>

**Pregunta 3:**¿Qué cadena de comandos usarías para buscar la palabra "ERROR" (ignorando mayúsculas/minúsculas) en un archivo `app.log` y guardar el resultado en un nuevo archivo llamado `errores.log`?

- [ ] a) `grep -i "ERROR" app.log > errores.log`
- [ ] b) `find "ERROR" in app.log > errores.log`
- [ ] c) `cat app.log | find "ERROR" >> errores.log`
- [ ] d) `grep "ERROR" app.log >> errores.log`

<details>
  <summary>Ver Fuente</summary>
  <p><a href="./../fase0/02-herramientas-cli.md#1-búsqueda-y-filtrado-de-texto">Fuente: Fase 0 - Herramientas de Línea de Comandos</a></p>
</details>

**Pregunta 4:**¿Cuál es la diferencia fundamental entre un bucle `for` y un bucle `while` en Bash scripting?

<details>
  <summary>Ver Fuente</summary>
  <p><a href="./../fase0/03-scripting-basico.md#bucles-for-y-while">Fuente: Fase 0 - Scripting Básico</a></p>
</details>

---

## Sección 2: Fase 1 - Fundamentos de DevOps

<<<<<<< HEAD
>  **Contenido de referencia:**[03-devops-agile-waterfall.md](03-devops-agile-waterfall.md)

7. **¿Cuál de los siguientes modelos es lineal y secuencial?**

   - [ ] a) Agile
   - [ ] b) DevOps
   - [ ] c) Waterfall
   - [ ] d) Scrum

8. **¿Qué metodología se enfoca en iteraciones cortas y entregas incrementales?**

   - [ ] a) Waterfall
   - [ ] b) Agile
   - [ ] c) DevOps (solo)
   - [ ] d) Ninguna de las anteriores

9. **¿Cuál es la principal diferencia entre Agile y DevOps?**

   - [ ] a) Agile solo se enfoca en desarrollo, DevOps incluye operaciones
   - [ ] b) DevOps es más lento que Agile
   - [ ] c) No hay diferencias
   - [ ] d) Agile requiere más automatización

**Pregunta 5:**Según la sección de cultura, ¿qué es un "Post-Mortem Sin Culpa" y por qué es crucial para la seguridad psicológica de un equipo?

<details>
  <summary>Ver Fuente</summary>
  <p><a href="./01-que-es-devops.md#el-mindset-devops-más-allá-de-las-prácticas">Fuente: Fase 1 - Cultura de DevOps</a></p>
</details>

**Pregunta 6:**¿Cuál de las siguientes NO es una de las 4 métricas clave de DORA?

- [ ] a) Deployment Frequency
- [ ] b) Lead Time for Changes
- [ ] c) Number of Lines of Code
- [ ] d) Time to Restore Service

<details>
  <summary>Ver Fuente</summary>
  <p>Aunque el contenido detallado sobre DORA aún no se ha escrito, el <a href="../../roadmap.md">Roadmap</a> las introduce como un concepto clave. La respuesta se puede inferir de los nombres de las métricas, que se centran en el proceso de entrega, no en la cantidad de código.</p>
</details>

**Pregunta 7:**Estás trabajando en una nueva funcionalidad en una rama llamada `feature/login`. Has hecho varios commits pequeños. Antes de fusionarla a `main`, quieres combinar todos esos commits en uno solo para mantener el historial limpio. ¿Qué estrategia de merge usarías?

- [ ] a) Three-Way Merge
- [ ] b) Fast-Forward Merge
- [ ] c) Rebase y Merge
- [ ] d) Squash Merge

<<<<<<< HEAD
## 4. Ciclo de Vida de DevOps (8 Fases)

>  **Contenido de referencia:**[04-ciclo-de-vida-devops.md](04-ciclo-de-vida-devops.md)

10. **¿Cuáles son las 8 fases del ciclo de vida DevOps en orden?**

    - [ ] a) Plan → Code → Build → Test → Release → Deploy → Operate → Monitor
    - [ ] b) Code → Plan → Test → Build → Deploy → Release → Monitor → Operate
    - [ ] c) Plan → Build → Code → Test → Deploy → Release → Operate → Monitor
    - [ ] d) Plan → Code → Test → Build → Release → Deploy → Operate → Monitor

11. **¿En qué fase se implementa Infrastructure as Code (IaC)?**

    - [ ] a) Plan
    - [ ] b) Code
    - [ ] c) Deploy
    - [ ] d) En múltiples fases (Code, Build, Deploy)

12. **¿Qué ocurre en la fase "Operate"?**

    - [ ] a) Se escribe el código de la aplicación
    - [ ] b) Se gestionan los entornos de producción y se escala según demanda
    - [ ] c) Se planifican nuevas funcionalidades
    - [ ] d) Se ejecutan únicamente las pruebas unitarias

---

## 5. Herramientas y Tecnologías Clave

>  **Contenido de referencia:**[05-herramientas-tecnologias-clave.md](05-herramientas-tecnologias-clave.md)

13. **¿Cuál de las siguientes herramientas se utiliza para la Integración Continua?**

    - [ ] a) Docker
    - [ ] b) Jenkins
    - [ ] c) Terraform
    - [ ] d) Prometheus

14. **¿Para qué se utiliza principalmente Docker en DevOps?**

    - [ ] a) Control de versiones
    - [ ] b) Containerización de aplicaciones
    - [ ] c) Monitoreo de aplicaciones
    - [ ] d) Gestión de bases de datos

15. **¿Cuál es el propósito principal de Kubernetes?**

    - [ ] a) Desarrollo de aplicaciones
    - [ ] b) Orquestación de contenedores
    - [ ] c) Control de versiones
    - [ ] d) Testing automatizado

16. **¿Qué herramienta se usa comúnmente para Infrastructure as Code?**

    - [ ] a) Jenkins
    - [ ] b) Docker
    - [ ] c) Terraform
    - [ ] d) Git

---

## 6. Introducción a la Nube

>  **Contenido de referencia:**[06-introduccion-nube.md](06-introduccion-nube.md)

17. **¿Cuáles son los tres principales modelos de servicio en la nube?**

    - [ ] a) SaaS, PaaS, IaaS
    - [ ] b) Public, Private, Hybrid
    - [ ] c) AWS, Azure, GCP
    - [ ] d) Development, Testing, Production

18. **¿Qué significa "escalabilidad horizontal" en la nube?**

    - [ ] a) Aumentar la potencia de una máquina existente
    - [ ] b) Agregar más máquinas al sistema
    - [ ] c) Reducir el número de servidores
    - [ ] d) Cambiar el proveedor de nube

19. **¿Cuál es una ventaja principal del cloud computing para DevOps?**

    - [ ] a) Mayor costo operacional
    - [ ] b) Elasticidad y aprovisionamiento rápido de recursos
    - [ ] c) Menos automatización
    - [ ] d) Mayor tiempo de configuración

---

## 7. Conceptos Clave: CI/CD, IaC y Observabilidad

>  **Contenido de referencia:**[07-conceptos-clave.md](07-conceptos-clave.md)

20. **¿Qué significa CI/CD?**

    - [ ] a) Control de Infraestructura / Control de Datos
    - [ ] b) Continuous Integration / Continuous Deployment
    - [ ] c) Cloud Infrastructure / Cloud Deployment
    - [ ] d) Code Integration / Code Development

21. **¿Cuál es el objetivo principal de la Infraestructura como Código (IaC)?**

    - [ ] a) Escribir código de aplicación más rápido
    - [ ] b) Gestionar la infraestructura mediante archivos de configuración versionados
    - [ ] c) Eliminar la necesidad de servidores
    - [ ] d) Reducir la seguridad del sistema

22. **¿Qué abarca la "Observabilidad" en DevOps?**

    - [ ] a) Solo logs de aplicación
    - [ ] b) Métricas, logs y trazas distribuidas
    - [ ] c) Solo monitoreo de servidores
    - [ ] d) Únicamente alertas de error

23. **¿Cuál es la diferencia entre Continuous Deployment y Continuous Delivery?**

    - [ ] a) No hay diferencia
    - [ ] b) Continuous Deployment automatiza el despliegue a producción, Continuous Delivery requiere aprobación manual
    - [ ] c) Continuous Delivery es más rápido
    - [ ] d) Continuous Deployment solo funciona en desarrollo

---

## 8. Primeros Pasos con Git

>  **Contenido de referencia:**[08-primeros-pasos-git.md](08-primeros-pasos-git.md)

24. **¿Cuál es el comando para clonar un repositorio remoto?**

    - [ ] a) `git pull origin main`
    - [ ] b) `git clone <URL>`
    - [ ] c) `git checkout <URL>`
    - [ ] d) `git fetch <URL>`

25. **¿Qué estrategia de branching es más adecuada para equipos grandes?**

    - [ ] a) Trabajar directamente en main
    - [ ] b) Git Flow
    - [ ] c) Feature branches sin revisión
    - [ ] d) Una sola rama para todo

26. **¿Cuál es el propósito de un Pull Request/Merge Request?**

    - [ ] a) Descargar código del repositorio
    - [ ] b) Solicitar revisión de código antes de integrar cambios
    - [ ] c) Eliminar ramas del repositorio
    - [ ] d) Crear un nuevo repositorio

27. **¿Qué comando se usa para crear y cambiar a una nueva rama?**

    - [ ] a) `git branch nueva-rama`
    - [ ] b) `git checkout nueva-rama`
    - [ ] c) `git checkout -b nueva-rama`
    - [ ] d) `git switch nueva-rama`

28. **¿Cuál es una buena práctica para mensajes de commit?**

    - [ ] a) Usar mensajes genéricos como "fix"
    - [ ] b) Escribir mensajes descriptivos que expliquen el "qué" y "por qué"
    - [ ] c) No incluir mensajes
    - [ ] d) Usar solo emojis

---

## Respuestas Correctas

### 1. ¿Qué es DevOps? (Cultura, herramientas y prácticas)

1. **b)**Unificar desarrollo y operaciones para entregar software más rápido y confiable
2. **c)**Silos organizacionales (esto NO es un principio de DevOps)
3. **a)**Mover las pruebas y seguridad más temprano en el ciclo de desarrollo

### 2. Beneficios de DevOps

4. **c)**Entrega más rápida de software con mayor calidad
5. **b)**Detectando errores tempranamente mediante pruebas automatizadas
6. **b)**Mean Time To Recovery (MTTR) reducido

### 3. Diferencia entre DevOps, Agile y Waterfall

7. **c)**Waterfall
8. **b)**Agile
9. **a)**Agile solo se enfoca en desarrollo, DevOps incluye operaciones

### 4. Ciclo de Vida de DevOps (8 Fases)

10. **a)**Plan → Code → Build → Test → Release → Deploy → Operate → Monitor
11. **d)**En múltiples fases (Code, Build, Deploy)
12. **b)**Se gestionan los entornos de producción y se escala según demanda

### 5. Herramientas y Tecnologías Clave

13. **b)**Jenkins
14. **b)**Containerización de aplicaciones
15. **b)**Orquestación de contenedores
16. **c)**Terraform

### 6. Introducción a la Nube

17. **a)**SaaS, PaaS, IaaS
18. **b)**Agregar más máquinas al sistema
19. **b)**Elasticidad y aprovisionamiento rápido de recursos

### 7. Conceptos Clave: CI/CD, IaC y Observabilidad

20. **b)**Continuous Integration / Continuous Deployment
21. **b)**Gestionar la infraestructura mediante archivos de configuración versionados
22. **b)**Métricas, logs y trazas distribuidas
23. **b)**Continuous Deployment automatiza el despliegue a producción, Continuous Delivery requiere aprobación manual

### 8. Primeros Pasos con Git

24. **b)**`git clone <URL>`
25. **b)**Git Flow
26. **b)**Solicitar revisión de código antes de integrar cambios
27. **c)**`git checkout -b nueva-rama`
28. **b)**Escribir mensajes descriptivos que expliquen el "qué" y "por qué"

### 9. Preguntas Integradoras

29. **a)**Git → Jenkins → Docker → Kubernetes → Prometheus
30. **b)**Planificar → Desarrollar → CI/CD automático → Monitoreo → Feedback

---

## Evaluación de Resultados

### **Rangos de Puntuación**

- **26-30 puntos (87-100%)**: Excelente!  Dominas los fundamentos de DevOps
- **21-25 puntos (70-86%)**: Muy bien!  Tienes un buen entendimiento
- **16-20 puntos (53-69%)**: Bien  Necesitas repasar algunos conceptos
- **11-15 puntos (37-52%)**: Regular  Revisa el material nuevamente
- **0-10 puntos (0-36%)**: Insuficiente  Estudia el contenido completo

### **Recomendaciones por Puntuación**

#### **Puntuación Alta (26-30)**

- Felicitaciones! Estás listo para la Fase 2
- Considera ser mentor de otros estudiantes
- Comparte tu experiencia en la comunidad

#### **Puntuación Buena (21-25)**

- Revisa los temas donde tuviste errores
- Practica más con las herramientas específicas
- Procede a la Fase 2 con confianza

#### **Puntuación Regular (16-20)**

- Repasa los archivos donde tuviste más errores
- Completa nuevamente los laboratorios prácticos
- Busca recursos adicionales para conceptos difíciles

#### **Puntuación Baja (11-15)**

- Vuelve a estudiar el material completo
- Toma más tiempo con cada tema
- Busca ayuda en la comunidad

#### **Puntuación Insuficiente (0-10)**

- Reinicia la Fase 1 desde el principio
- Dedica más tiempo a cada archivo
- Considera un enfoque más lento y gradual

---

## Siguientes Pasos

### **Si Aprobaste (21+ puntos)**

1. **Celebra tu logro** Has completado exitosamente la Fase 1
2. **Documenta tu aprendizaje**en tu portfolio personal
3. **Procede a la [Fase 2: Automatización y CI/CD](../fase2/)**
4. **Comparte tu experiencia**con otros estudiantes

### **Si Necesitas Mejorar (≤20 puntos)**

1. **Identifica tus áreas débiles**basándote en las respuestas incorrectas
2. **Repasa el material específico**que necesitas reforzar
3. **Completa nuevamente los laboratorios**de esos temas
4. **Retoma el test**cuando te sientas más preparado

### **Recursos Adicionales**

- [Roadmap Completo](../../roadmap.md)
- [Notas de Git](../../notes/git-notes.md)
- [Notas de Docker](../../notes/docker-notes.md)
- [Recursos de Libros](../books.md)
- [Herramientas Recomendadas](../tools.md)

---

## Consejos para el Éxito

### **Para Futuros Tests**

1. **Lee atentamente**cada pregunta y todas las opciones
2. **Elimina opciones incorrectas**para aumentar probabilidades
3. **Revisa material**cuando no estés seguro de una respuesta
4. **Practica regularmente**para mantener conocimientos frescos

### **Para Continuar Aprendiendo**

1. **Mantén la práctica**: Los conceptos DevOps necesitan aplicación constante
2. **Únete a comunidades**: Aprende de experiencias de otros profesionales
3. **Experimenta**: Prueba herramientas y conceptos en proyectos personales
4. **Documenta**: Lleva un registro de tu progreso y aprendizajes

---

## Certificado de Completado

**Si obtuviste 21 puntos o más, has completado exitosamente la Fase 1!**

```
 CERTIFICADO DE COMPLETADO

[Tu Nombre]

Ha completado exitosamente la

FASE 1: FUNDAMENTOS DE DEVOPS

Del curso "DevOps Learning Path"

Puntuación obtenida: [Tu puntuación]/30
Fecha de completado: [Fecha actual]

Listo para continuar con la Fase 2!
```

---

_Felicitaciones por completar la evaluación de la Fase 1! Recuerda que el aprendizaje DevOps es un viaje continuo._

**Última actualización:**12 de octubre de 2025

## 9. Preguntas Integradoras

29. **¿Cuál es la relación correcta entre estas herramientas en un pipeline DevOps?**

    - [ ] a) Git → Jenkins → Docker → Kubernetes → Prometheus
    - [ ] b) Docker → Git → Kubernetes → Jenkins → Prometheus
    - [ ] c) Jenkins → Git → Prometheus → Docker → Kubernetes
    - [ ] d) Kubernetes → Docker → Git → Jenkins → Prometheus

30. **En un entorno DevOps maduro, ¿cuál sería el flujo ideal para una nueva funcionalidad?**

    - [ ] a) Desarrollar → Probar manualmente → Desplegar a producción
    - [ ] b) Planificar → Desarrollar → CI/CD automático → Monitoreo → Feedback
    - [ ] c) Escribir código → Subir directamente a producción
    - [ ] d) Desarrollar → Esperar fin de sprint → Desplegar todo junto

---

## Resumen de Preguntas por Tema

1. **¿Qué es DevOps?**Preguntas 1, 2, 3.

   - 1.1. b)
   - 1.2. b)
   - 1.3. b)

2. **Beneficios de DevOps**Preguntas 4, 5, 6.

   - 2.1. c)
   - 2.2. b)
   - 2.3. b)

3. **Diferencia entre DevOps, Agile y Waterfall**Preguntas 7, 8, 9.

   - 3.1. c)
   - 3.2. b)
   - 3.3. c)

4. **Ciclo de vida de DevOps**Preguntas 10, 11, 12, 13, 14, 15.

   - 4.1. b)
   - 4.2. b)
   - 4.3. a)
   - 4.4. b)
   - 4.5. b)
   - 4.6. b)

<details>
  <summary>Ver Fuente</summary>
  <p><a href="./07-primeros-pasos-git.md#5-merges-y-resolución-de-conflictos">Fuente: Fase 1 - Merges y Resolución de Conflictos</a></p>
</details>

**Pregunta 8:**¿Para qué sirve el comando `git stash`?

- [ ] a) Para eliminar commits antiguos.
- [ ] b) Para guardar temporalmente cambios que no están listos para ser commitidos.
- [ ] c) Para fusionar dos ramas.
- [ ] d) Para crear un nuevo repositorio.

<details>
  <summary>Ver Fuente</summary>
  <p><a href="./07-primeros-pasos-git.md#7-flujos-de-trabajo-avanzados">Fuente: Fase 1 - Flujos de Trabajo Avanzados (Stash)</a></p>
</details>

**Pregunta 9:**¿Qué es la "Infraestructura como Código" (IaC) y qué herramienta mencionada en la Fase 1 se utiliza para este propósito?

<details>
  <summary>Ver Fuente</summary>
  <p><a href="./06-conceptos-clave-devops.md#2-infraestructura-como-código-iac">Fuente: Fase 1 - Conceptos Clave (IaC)</a></p>
</details>
