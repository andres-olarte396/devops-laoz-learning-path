# Fase 0: Herramientas de Línea de Comandos

Con los fundamentos de la navegación cubiertos, es hora de explorar herramientas más potentes que te permitirán buscar, manipular y editar archivos directamente desde la terminal. Estas utilidades son esenciales para el análisis de logs, la gestión de archivos de configuración y la automatización.

---

## 1. Búsqueda y Filtrado de Texto

Estas herramientas son indispensables para encontrar información en archivos de texto, especialmente en logs o código fuente.

### **`grep` (Global Regular Expression Print)**

`grep` busca patrones de texto en archivos. Es probablemente una de las herramientas que más usarás.

| Opción | Descripción | Ejemplo de Uso |
|---|---|---|
| `-i` | **Ignore Case**. Búsqueda insensible a mayúsculas/minúsculas. | `grep -i "error" /var/log/syslog` |
| `-r` | **Recursive**. Busca recursivamente en un directorio. | `grep -r "api_key" /etc/` |
| `-v` | **Invert Match**. Muestra las líneas que NO coinciden con el patrón. | `cat urls.txt | grep -v "localhost"` |
| `-c` | **Count**. Cuenta el número de líneas que coinciden. | `grep -c "SUCCESS" app.log` |
| `-A` / `-B` | **After/Before**. Muestra N líneas antes (`-B`) o después (`-A`) de la coincidencia. | `grep -A 5 -B 2 "FATAL" error.log` |

**Expresiones Regulares (Regex):**

- `grep` soporta expresiones regulares, lo que lo hace extremadamente potente.
- `^patron`: Líneas que empiezan con "patron".
- `patron$`: Líneas que terminan con "patron".
- `[0-9]+`: Uno o más dígitos numéricos.

```bash
# Buscar direcciones IP en un log
grep -E "[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}" access.log
```

### **`find`**

`find` se utiliza para buscar archivos y directorios basándose en criterios como nombre, tamaño, fecha de modificación, etc.

| Criterio | Descripción | Ejemplo de Uso |
|---|---|---|
| `-name` | Busca por nombre de archivo (sensible a mayúsculas). | `find . -name "*.log"` |
| `-iname`| Busca por nombre (insensible a mayúsculas). | `find /home -iname "config.yml"` |
| `-type` | Busca por tipo de archivo (`f` para archivo, `d` para directorio). | `find . -type d -name "node_modules"` |
| `-size` | Busca por tamaño. | `find /var/log -size +100M` (archivos > 100MB) |
| `-mtime`| Busca por fecha de modificación (en días). | `find . -mtime -7` (modificados en los últimos 7 días) |

`find` se puede combinar con otros comandos usando la opción `-exec`:

```bash
# Encontrar todos los archivos .tmp y eliminarlos
find . -type f -name "*.tmp" -exec rm -f {} \;
```

### **`awk` y `sed`**

Son herramientas de procesamiento de texto más avanzadas.

- **`awk`**: Es un lenguaje de programación completo para procesar texto. Es excelente para extraer columnas de datos.

  ```bash
  # Mostrar la primera y tercera columna de un archivo, separadas por comas
  awk -F, '{print $1, $3}' datos.csv

  # Mostrar el espacio libre en disco de los sistemas de archivos
  df -h | awk '{print $1 " tiene " $4 " libre"}'
  ```

- **`sed` (Stream Editor)**: Se usa para realizar sustituciones de texto en un flujo de datos.

  ```bash
  # Reemplazar todas las ocurrencias de "desarrollo" por "produccion" en un archivo
  sed 's/desarrollo/produccion/g' config.yml > config_prod.yml
  ```

---

<a name="archivado"></a>

## 2. Archivado y Compresión

Es fundamental saber cómo empaquetar y comprimir archivos para transferencias o copias de seguridad.

### **`tar` (Tape Archive)**

`tar` agrupa múltiples archivos en un solo archivo (un "tarball"). No comprime por defecto.

| Opción | Descripción | Ejemplo de Uso |
|---|---|---|
| `-c` | **Create**. Crea un nuevo archivo `.tar`. | `tar -cvf mi_backup.tar /ruta/a/directorio` |
| `-x` | **Extract**. Extrae archivos de un `.tar`. | `tar -xvf mi_backup.tar` |
| `-v` | **Verbose**. Muestra los archivos mientras se procesan. | (ver ejemplos arriba) |
| `-f` | **File**. Especifica el nombre del archivo `.tar`. | (ver ejemplos arriba) |
| `-t` | **List**. Lista el contenido de un `.tar` sin extraerlo. | `tar -tvf mi_backup.tar` |

### **Compresión: `gzip` y `zip`**

- **`gzip`**: Es el compresor estándar en sistemas UNIX. Generalmente se usa en combinación con `tar`.

  ```bash
  # Crear un archivo tarball comprimido (la opción -z invoca a gzip)
  tar -czvf mi_backup.tar.gz /ruta/a/directorio

  # Extraer un archivo tarball comprimido
  tar -xzvf mi_backup.tar.gz
  ```

  Los archivos resultantes suelen tener la extensión `.tar.gz` o `.tgz`.

- **`zip`**: Es más común en Windows, pero también está disponible en Linux. Crea archivos `.zip`.

  ```bash
  # Comprimir un directorio
  zip -r mi_archivo.zip /ruta/a/directorio

  # Descomprimir un archivo
  unzip mi_archivo.zip
  ```

---

<a name="editores"></a>

## 3. Editores de Texto en Terminal

Habrá momentos en los que necesites editar un archivo de configuración directamente en un servidor remoto, sin acceso a una interfaz gráfica.

### **`nano`**

- **Ideal para principiantes.**
- **Fácil de usar:**Muestra los atajos de teclado más importantes en la parte inferior de la pantalla.
- **Uso básico:**
  1. Abrir un archivo: `nano mi_archivo.txt`
  2. Editar el texto.
  3. Guardar: `Ctrl + O` (y luego Enter).
  4. Salir: `Ctrl + X`.

### **`vim` (o `neovim`)**

- **Extremadamente potente y eficiente, pero con una curva de aprendizaje pronunciada.**
- **Modal:**Tiene diferentes modos (Normal, Inserción, Visual, Comando).
- **Uso básico:**
  1. Abrir un archivo: `vim mi_archivo.txt`
  2. Entrar al **modo de Inserción**para escribir: presiona la tecla `i`.
  3. Salir del modo de Inserción al **modo Normal**: presiona la tecla `Esc`.
  4. Desde el modo Normal, para guardar y salir:
     - Escribe `:wq` y presiona Enter (Write & Quit).
     - Escribe `:q!` y presiona Enter (Quit sin guardar).

**Comandos básicos en modo Normal:**

- `dd`: Cortar (eliminar) una línea.
- `yy`: Copiar una línea.
- `p`: Pegar.
- `/texto`: Buscar "texto".

Invertir tiempo en aprender `vim` puede aumentar drásticamente tu productividad en la terminal a largo plazo.
