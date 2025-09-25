# Test de Autoevaluación: Fundamentos de DevOps (Fase 0 y 1)

Este test está diseñado para ayudarte a evaluar tu comprensión de los conceptos fundamentales introducidos en las Fases 0 y 1. Cada pregunta incluye un enlace a la sección relevante del material de estudio para que puedas repasar.

---

## Sección 1: Fase 0 - Dominio de la Terminal

**Pregunta 1:** ¿Cuál es el comando para crear un nuevo directorio llamado `proyecto` y luego mover un archivo `plan.txt` dentro de él?

- [ ] a) `mkdir proyecto` y luego `cp plan.txt proyecto/`
- [ ] b) `newdir proyecto` y luego `move plan.txt proyecto/`
- [ ] c) `mkdir proyecto` y luego `mv plan.txt proyecto/`
- [ ] d) `create proyecto` y luego `cp plan.txt proyecto/`

<details>
  <summary>Ver Fuente</summary>
  <p><a href="./../fase0/01-fundamentos-shell.md#2-navegación-y-manipulación-de-archivos">Fuente: Fase 0 - Fundamentos de la Shell</a></p>
</details>

**Pregunta 2:** Quieres que un script llamado `deploy.sh` sea ejecutable por su propietario, pero solo de lectura para todos los demás. ¿Qué comando `chmod` usarías?

- [ ] a) `chmod 644 deploy.sh`
- [ ] b) `chmod 755 deploy.sh`
- [ ] c) `chmod 744 deploy.sh`
- [ ] d) `chmod 655 deploy.sh`

<details>
  <summary>Ver Fuente</summary>
  <p><a href="./../fase0/01-fundamentos-shell.md#permisos">Fuente: Fase 0 - Permisos de Archivos</a></p>
</details>

**Pregunta 3:** ¿Qué cadena de comandos usarías para buscar la palabra "ERROR" (ignorando mayúsculas/minúsculas) en un archivo `app.log` y guardar el resultado en un nuevo archivo llamado `errores.log`?

- [ ] a) `grep -i "ERROR" app.log > errores.log`
- [ ] b) `find "ERROR" in app.log > errores.log`
- [ ] c) `cat app.log | find "ERROR" >> errores.log`
- [ ] d) `grep "ERROR" app.log >> errores.log`

<details>
  <summary>Ver Fuente</summary>
  <p><a href="./../fase0/02-herramientas-cli.md#1-búsqueda-y-filtrado-de-texto">Fuente: Fase 0 - Herramientas de Línea de Comandos</a></p>
</details>

**Pregunta 4:** ¿Cuál es la diferencia fundamental entre un bucle `for` y un bucle `while` en Bash scripting?

<details>
  <summary>Ver Fuente</summary>
  <p><a href="./../fase0/03-scripting-basico.md#bucles-for-y-while">Fuente: Fase 0 - Scripting Básico</a></p>
</details>

---

## Sección 2: Fase 1 - Fundamentos de DevOps

**Pregunta 5:** Según la sección de cultura, ¿qué es un "Post-Mortem Sin Culpa" y por qué es crucial para la seguridad psicológica de un equipo?

<details>
  <summary>Ver Fuente</summary>
  <p><a href="./01-que-es-devops.md#el-mindset-devops-más-allá-de-las-prácticas">Fuente: Fase 1 - Cultura de DevOps</a></p>
</details>

**Pregunta 6:** ¿Cuál de las siguientes NO es una de las 4 métricas clave de DORA?

- [ ] a) Deployment Frequency
- [ ] b) Lead Time for Changes
- [ ] c) Number of Lines of Code
- [ ] d) Time to Restore Service

<details>
  <summary>Ver Fuente</summary>
  <p>Aunque el contenido detallado sobre DORA aún no se ha escrito, el <a href="../../roadmap.md">Roadmap</a> las introduce como un concepto clave. La respuesta se puede inferir de los nombres de las métricas, que se centran en el proceso de entrega, no en la cantidad de código.</p>
</details>

**Pregunta 7:** Estás trabajando en una nueva funcionalidad en una rama llamada `feature/login`. Has hecho varios commits pequeños. Antes de fusionarla a `main`, quieres combinar todos esos commits en uno solo para mantener el historial limpio. ¿Qué estrategia de merge usarías?

- [ ] a) Three-Way Merge
- [ ] b) Fast-Forward Merge
- [ ] c) Rebase y Merge
- [ ] d) Squash Merge

<details>
  <summary>Ver Fuente</summary>
  <p><a href="./07-primeros-pasos-git.md#5-merges-y-resolución-de-conflictos">Fuente: Fase 1 - Merges y Resolución de Conflictos</a></p>
</details>

**Pregunta 8:** ¿Para qué sirve el comando `git stash`?

- [ ] a) Para eliminar commits antiguos.
- [ ] b) Para guardar temporalmente cambios que no están listos para ser commitidos.
- [ ] c) Para fusionar dos ramas.
- [ ] d) Para crear un nuevo repositorio.

<details>
  <summary>Ver Fuente</summary>
  <p><a href="./07-primeros-pasos-git.md#7-flujos-de-trabajo-avanzados">Fuente: Fase 1 - Flujos de Trabajo Avanzados (Stash)</a></p>
</details>

**Pregunta 9:** ¿Qué es la "Infraestructura como Código" (IaC) y qué herramienta mencionada en la Fase 1 se utiliza para este propósito?

<details>
  <summary>Ver Fuente</summary>
  <p><a href="./06-conceptos-clave-devops.md#2-infraestructura-como-código-iac">Fuente: Fase 1 - Conceptos Clave (IaC)</a></p>
</details>