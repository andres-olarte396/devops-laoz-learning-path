# Fase 0: Fundamentos de la Shell

Dominar la línea de comandos es el primer paso para convertirte en un ingeniero de DevOps eficiente. La terminal (o shell) será tu herramienta principal para interactuar con servidores, automatizar tareas y gestionar infraestructura.

---

## 1. ¿Qué es la Shell y por qué es importante?

La **shell**es un programa que toma comandos del teclado y los entrega al sistema operativo para que los ejecute. Actúa como una interfaz entre el usuario y el kernel del sistema.

En DevOps, usarás la shell para:

- **Automatizar tareas repetitivas**mediante scripts.
- **Gestionar servidores remotos**de forma segura a través de SSH.
- **Interactuar con herramientas de la nube**(AWS CLI, Azure CLI, gcloud).
- **Controlar contenedores**(Docker, Kubernetes).
- **Construir y desplegar**aplicaciones.

---

## 2. Navegación y Manipulación de Archivos

Estos son los comandos más básicos y esenciales que usarás constantemente.

| Comando | Descripción | Ejemplo de Uso |
|---|---|---|
| `pwd` | **Print Working Directory**. Muestra la ruta del directorio actual. | `pwd` |
| `ls` | **List**. Lista el contenido del directorio actual. | `ls -la` (lista todo en formato largo) |
| `cd` | **Change Directory**. Cambia de directorio. | `cd /home/usuario/documentos` |
| `cp` | **Copy**. Copia archivos o directorios. | `cp archivo.txt copia.txt` |
| `mv` | **Move**. Mueve o renombra archivos y directorios. | `mv archivo.txt /nuevo/directorio/` |
| `rm` | **Remove**. Elimina archivos. | `rm archivo_temporal.txt` |
| `mkdir`| **Make Directory**. Crea un nuevo directorio. | `mkdir nuevo_proyecto` |
| `cat` | **Concatenate**. Muestra el contenido de un archivo. | `cat archivo.log` |
| `less`| Muestra el contenido de un archivo de forma paginada. | `less archivo_largo.log` |
| `head`| Muestra las primeras 10 líneas de un archivo. | `head -n 20 archivo.log` |
| `tail`| Muestra las últimas 10 líneas de un archivo. | `tail -f /var/log/syslog` (sigue el archivo) |

### **Atajos útiles**

- `.` se refiere al directorio actual.
- `..` se refiere al directorio padre.
- `~` se refiere al directorio "home" del usuario actual.
- `/` se refiere al directorio raíz del sistema.

---

<a name="permisos"></a>

## 3. Permisos de Archivos

En sistemas tipo UNIX (Linux, macOS), cada archivo tiene permisos para tres tipos de usuarios:

- **Propietario (Owner):** El usuario que creó el archivo.
- **Grupo (Group):** Un grupo de usuarios que comparten permisos.
- **Otros (Others):** Todos los demás usuarios.

Los permisos se dividen en tres acciones:

- **Lectura (Read - `r`):** Permiso para ver el contenido del archivo.
- **Escritura (Write - `w`):** Permiso para modificar el archivo.
- **Ejecución (Execute - `x`):** Permiso para ejecutar el archivo (si es un script o programa).

### **Comando `chmod` (Change Mode)**

Se usa para cambiar los permisos de un archivo.

**Modo Octal (Numérico):**
Es la forma más común. Cada acción tiene un valor numérico:

- `r` = 4
- `w` = 2
- `x` = 1

Se suman los valores para cada tipo de usuario (Propietario, Grupo, Otros).

**Ejemplos:**

- `chmod 755 mi_script.sh`
  - **Propietario:**7 (4+2+1) -> `rwx` (Lectura, Escritura, Ejecución)
  - **Grupo:**5 (4+0+1) -> `r-x` (Lectura, Ejecución)
  - **Otros:**5 (4+0+1) -> `r-x` (Lectura, Ejecución)
  *(Este es un permiso muy común para scripts que deben ser ejecutables por otros).*

- `chmod 644 mi_config.txt`
  - **Propietario:**6 (4+2+0) -> `rw-` (Lectura, Escritura)
  - **Grupo:**4 (4+0+0) -> `r--` (Solo Lectura)
  - **Otros:**4 (4+0+0) -> `r--` (Solo Lectura)
  *(Este es un permiso común para archivos de texto que no deben ser modificados por otros).*

### **Comando `chown` (Change Owner)**

Se usa para cambiar el propietario y/o el grupo de un archivo.

```bash
# Cambiar solo el propietario
chown nuevo_propietario mi_archivo.txt

# Cambiar propietario y grupo a la vez
chown nuevo_propietario:nuevo_grupo mi_archivo.txt
```

---

<a name="redireccion"></a>

## 4. Redirección de Entradas y Salidas

La redirección te permite controlar de dónde vienen los datos de un comando (entrada estándar, `stdin`) y a dónde van sus resultados (salida estándar, `stdout`) y errores (error estándar, `stderr`).

| Operador | Descripción | Ejemplo de Uso |
|---|---|---|
| `>` | **Redirigir `stdout`**. Sobrescribe el contenido del archivo. | `ls -l > lista_archivos.txt` |
| `>>` | **Añadir `stdout`**. Agrega el resultado al final del archivo. | `echo "Nuevo log" >> sistema.log` |
| `<` | **Redirigir `stdin`**. Pasa el contenido de un archivo como entrada a un comando. | `sort < lista_desordenada.txt` |
| `|` | **Pipe (Tubería)**. Envía el `stdout` de un comando al `stdin` de otro. | `cat archivo.log | grep "ERROR"` |
| `2>` | **Redirigir `stderr`**. Envía solo los errores a un archivo. | `comando_erroneo 2> errores.log` |
| `&>` | **Redirigir `stdout` y `stderr`**. Envía ambos a un archivo. | `comando_completo &> todo.log` |

### **Ejemplo práctico de Pipe**

Imagina que quieres contar cuántos archivos `.md` hay en el directorio actual:

```bash
# 1. `ls` lista todos los archivos.
# 2. `grep .md` filtra la salida de `ls` y solo muestra las líneas que contienen ".md".
# 3. `wc -l` (Word Count - Lines) cuenta el número de líneas que recibe de `grep`.
ls | grep ".md" | wc -l
```

Este encadenamiento de comandos usando pipes es una técnica fundamental en la automatización y el trabajo con la shell.
