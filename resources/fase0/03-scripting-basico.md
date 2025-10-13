# Fase 0: Scripting Básico

Saber ejecutar comandos es útil, pero el verdadero poder de la terminal se desata con el **scripting**. Un script es simplemente un archivo de texto que contiene una secuencia de comandos. Al ejecutar el script, la shell procesa esos comandos en orden, permitiéndote automatizar tareas complejas y repetitivas.

---

## 1. Introducción a Bash Scripting

Bash (Bourne Again SHell) es la shell por defecto en la mayoría de sistemas Linux y en macOS. Los scripts de Bash son fundamentales para la automatización en estos entornos.

### **Creando tu Primer Script**

1. Crea un archivo con la extensión `.sh`, por ejemplo, `mi_script.sh`.
2. La primera línea del script debe ser el "shebang", que le dice al sistema qué intérprete usar:

    ```bash
    #!/bin/bash
    ```

3. Añade algunos comandos:

    ```bash
    #!/bin/bash
    echo "Hola, Mundo!"
    echo "La fecha actual es: $(date)"
    ```

4. Dale permisos de ejecución al archivo:

    ```bash
    chmod +x mi_script.sh
    ```

5. Ejecuta el script:

    ```bash
    ./mi_script.sh
    ```

### **Variables**

Las variables almacenan datos. Se definen sin espacios alrededor del `=`.

```bash
#!/bin/bash

# Definir una variable
NOMBRE="Mundo"

# Usar la variable (con $)
echo "Hola, $NOMBRE"

# Guardar la salida de un comando en una variable
USUARIO_ACTUAL=$(whoami)
echo "Ejecutando como: $USUARIO_ACTUAL"
```

### **Condicionales (`if`)**

Los condicionales permiten que tu script tome decisiones.

```bash
#!/bin/bash

# Comprobar si un archivo existe
if [ -f "/etc/hosts" ]; then
    echo "El archivo /etc/hosts existe."
else
    echo "El archivo /etc/hosts no existe."
fi

# Comparar números
NUMERO=10
if [ $NUMERO -gt 5 ]; then
    echo "El número es mayor que 5."
fi
```

**Operadores de comparación comunes:**

- `-f <archivo>`: Verdadero si el archivo existe.
- `-d <directorio>`: Verdadero si el directorio existe.
- `-eq`: Igual (números).
- `-ne`: No igual (números).
- `-gt`: Mayor que (números).
- `-lt`: Menor que (números).
- `==`: Igual (cadenas de texto).
- `!=`: No igual (cadenas de texto).

### **Bucles (`for` y `while`)**

Los bucles se usan para repetir tareas.

**Bucle `for`:**Itera sobre una lista de elementos.

```bash
#!/bin/bash

# Iterar sobre una lista de nombres
for NOMBRE in "ana" "luis" "pedro"
do
    echo "Procesando a $NOMBRE"
done

# Iterar sobre los archivos de un directorio
for ARCHIVO in $(ls *.txt)
do
    echo "Encontrado archivo de texto: $ARCHIVO"
done
```

**Bucle `while`:**Se ejecuta mientras una condición sea verdadera.

```bash
#!/bin/bash

CONTADOR=0
while [ $CONTADOR -lt 5 ]; do
    echo "El contador es $CONTADOR"
    CONTADOR=$((CONTADOR + 1))
done
```

---

<a name="powershell"></a>

## 2. Introducción a PowerShell para Windows

PowerShell es la shell moderna y el lenguaje de scripting de Microsoft. Es el estándar para la automatización en entornos Windows. A diferencia de Bash que trabaja con texto, PowerShell trabaja con **objetos**.

### **Creando tu Primer Script de PowerShell**

1. Crea un archivo con la extensión `.ps1`, por ejemplo, `mi_script.ps1`.
2. Añade comandos (llamados "cmdlets"):

    ```powershell
    Write-Host "Hola, Mundo desde PowerShell!"
    Get-Date
    ```

3. Por defecto, la ejecución de scripts está deshabilitada. Para permitirla en tu sesión actual, abre PowerShell y ejecuta:

    ```powershell
    Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope Process
    ```

4. Ejecuta el script:

    ```powershell
    .\mi_script.ps1
    ```

### **Variables**

Las variables en PowerShell comienzan con el símbolo `$`.

```powershell
# Definir una variable
$nombre = "Mundo"

Write-Host "Hola, $nombre"

# Guardar la salida de un cmdlet en una variable
$servicios = Get-Service
Write-Host "Se encontraron $($servicios.Length) servicios."
```

### **Condicionales (`If`)**

La sintaxis es similar a otros lenguajes de programación.

```powershell
# Comprobar si un servicio está corriendo
$servicio = Get-Service -Name "spooler"

if ($servicio.Status -eq "Running") {
    Write-Host "El servicio de cola de impresión está activo."
} else {
    Write-Host "El servicio de cola de impresión no está activo."
    Start-Service -Name "spooler"
}
```

**Operadores de comparación comunes:**

- `-eq`: Igual.
- `-ne`: No igual.
- `-gt`: Mayor que.
- `-lt`: Menor que.
- `-like`: Comodín (ej: `"*win*"`).

### **Bucles (`ForEach-Object` y `While`)**

**Bucle `ForEach-Object`:**Es la forma más común de iterar, a menudo usando el alias `foreach`.

```powershell
# Iterar sobre una lista de procesos y mostrar su nombre y ID
Get-Process | ForEach-Object {
    Write-Host "Proceso: $($_.Name), ID: $($_.Id)"
}
```

`$_` es una variable especial que representa el objeto actual en la tubería (pipe).

**Bucle `While`:**

```powershell
$contador = 0
while ($contador -lt 5) {
    Write-Host "El contador es $contador"
    $contador++
}
```

### **La Tubería (Pipeline) en PowerShell**

Al igual que en Bash, la tubería (`|`) es fundamental. Sin embargo, en lugar de pasar texto, PowerShell pasa **objetos**. Esto te permite encadenar cmdlets de forma muy potente.

```powershell
# Obtener los 5 procesos que más memoria consumen, ordenarlos y mostrar su nombre y consumo de memoria en MB
Get-Process | Sort-Object -Property WS -Descending | Select-Object -First 5 | Format-Table -Property Name, @{Name="Memoria (MB)"; Expression={$_.WS / 1MB -as [int]}}
```

Este ejemplo muestra la naturaleza orientada a objetos de PowerShell y es algo que sería mucho más complejo de lograr con herramientas de texto tradicionales como Bash.
