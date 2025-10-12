# Test de Conocimientos: Fase 0 - Fundamentos de la Terminal

Este test evalúa tu comprensión de los conceptos básicos de la línea de comandos, herramientas CLI y scripting, cubiertos en la Fase 0.

**Instrucciones:** Elige la respuesta que consideres correcta para cada pregunta.

---

### Parte 1: Fundamentos de la Shell

**1. ¿Qué comando se utiliza para listar los archivos y directorios en la ubicación actual?**

- a) `dir`
- b) `list`
- c) `ls`
- d) `show`

<details>
  <summary>Ver Fuente</summary>
  <p><a href="./01-fundamentos-shell.md">Fuente: Fundamentos de la Shell</a></p>
</details>

**2. ¿Cuál es el comando para cambiar al directorio padre?**

- a) `cd ..`
- b) `cd /`
- c) `cd ~`
- d) `up`

<details>
  <summary>Ver Fuente</summary>
  <p><a href="./01-fundamentos-shell.md">Fuente: Fundamentos de la Shell</a></p>
</details>

**3. Si quieres crear un nuevo directorio llamado `proyectos`, ¿qué comando usarías?**

- a) `new dir proyectos`
- b) `crdir proyectos`
- c) `mkdir proyectos`
- d) `make directory proyectos`

<details>
  <summary>Ver Fuente</summary>
  <p><a href="./01-fundamentos-shell.md">Fuente: Fundamentos de la Shell</a></p>
</details>

---

### Parte 2: Herramientas CLI Esenciales

**4. ¿Qué herramienta usarías para buscar la palabra `error` dentro de un archivo de log llamado `app.log`?**

- a) `find error in app.log`
- b) `search error app.log`
- c) `grep "error" app.log`
- d) `cat app.log | find "error"`

<details>
  <summary>Ver Fuente</summary>
  <p><a href="./02-herramientas-cli.md">Fuente: Herramientas CLI Esenciales</a></p>
</details>

**5. ¿Qué comando muestra el contenido completo de un archivo de texto en la terminal?**

- a) `display`
- b) `cat`
- c) `type`
- d) `open`

<details>
  <summary>Ver Fuente</summary>
  <p><a href="./02-herramientas-cli.md">Fuente: Herramientas CLI Esenciales</a></p>
</details>

**6. Para encontrar la ruta absoluta de un ejecutable como `node`, ¿qué comando es el más adecuado?**

- a) `whereis node`
- b) `find node`
- c) `which node`
- d) `locate node`

<details>
  <summary>Ver Fuente</summary>
  <p><a href="./02-herramientas-cli.md">Fuente: Herramientas CLI Esenciales</a></p>
</details>

---

### Parte 3: Scripting Básico en Bash

**7. ¿Cuál es la primera línea recomendada para un script de Bash que asegura que se ejecute con el intérprete correcto?**

- a) `#!/bin/bash`
- b) `// use bash`
- c) `<?bash`
- d) `# start bash`

<details>
  <summary>Ver Fuente</summary>
  <p><a href="./03-scripting-basico.md">Fuente: Scripting Básico en Bash</a></p>
</details>

**8. ¿Cuál es la sintaxis correcta para asignar el valor "hola" a una variable llamada `saludo`?**

- a) `let saludo = "hola"`
- b) `saludo = "hola"`
- c) `$saludo = "hola"`
- d) `saludo="hola"`

<details>
  <summary>Ver Fuente</summary>
  <p><a href="./03-scripting-basico.md">Fuente: Scripting Básico en Bash</a></p>
</details>

**9. ¿Qué estructura de control se utiliza para ejecutar un bloque de código solo si una condición es verdadera?**

- a) `for ... in ... do`
- b) `while ... do`
- c) `if [ ... ]; then`
- d) `case ... in`

<details>
  <summary>Ver Fuente</summary>
  <p><a href="./03-scripting-basico.md">Fuente: Scripting Básico en Bash</a></p>
</details>

---

<details>
  <summary>Ver Respuestas</summary>

  1. **c) `ls`**
  2. **a) `cd ..`**
  3. **c) `mkdir proyectos`**
  4. **c) `grep "error" app.log`**
  5. **b) `cat`**
  6. **c) `which node`**(o `whereis` en algunos sistemas, pero `which` es más común para esto).
  7. **a) `#!/bin/bash`**
  8. **d) `saludo="hola"`**(Importante: sin espacios alrededor del `=`).
  9. **c) `if [ ... ]; then`**

</details>
