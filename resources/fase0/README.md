# Fase 0: Dominio de la Terminal

Bienvenido a la **Fase 0** del roadmap DevOps. Esta fase fundamental te preparará con las habilidades esenciales de línea de comandos que necesitarás a lo largo de todo tu viaje en DevOps.

## Objetivos de Aprendizaje

Al completar esta fase, serás capaz de:

- [x] Navegar eficientemente por el sistema de archivos usando la terminal
- [x] Manipular archivos y directorios con comandos básicos
- [x] Entender y gestionar permisos de archivos
- [x] Utilizar redirección y pipes para procesar datos
- [x] Buscar y filtrar información usando herramientas de texto
- [x] Crear scripts básicos para automatizar tareas
- [x] Trabajar cómodamente tanto en sistemas Unix/Linux como Windows

## Contenido de la Fase

### 1. Fundamentos de la Shell

**Archivo:** [01-fundamentos-shell.md](./01-fundamentos-shell.md)

**Temas cubiertos:**

- Comandos básicos de navegación (`ls`, `cd`, `pwd`)
- Manipulación de archivos (`cp`, `mv`, `rm`, `mkdir`)
- Gestión de permisos (`chmod`, `chown`)
- Redirección de entrada/salida (`>`, `>>`, `<`, `|`)

**Tiempo estimado:** 2-3 días

### 2. Herramientas de Línea de Comandos

**Archivo:** [02-herramientas-cli.md](./02-herramientas-cli.md)

**Temas cubiertos:**

- Búsqueda y filtrado (`grep`, `find`, `awk`, `sed`)
- Archivado y compresión (`tar`, `gzip`, `zip`)
- Editores de terminal (`vim`, `nano`)
- Procesos y monitoreo del sistema

**Tiempo estimado:** 3-4 días

### 3. Scripting Básico

**Archivo:** [03-scripting-basico.md](./03-scripting-basico.md)

**Temas cubiertos:**

- Variables y tipos de datos
- Estructuras de control (bucles, condicionales)
- Funciones y parámetros
- Bash scripting para Unix/Linux
- PowerShell para Windows

**Tiempo estimado:** 4-5 días

## Plan de Estudio Recomendado

### Semana 1: Fundamentos

- **Días 1-2:** Introducción y comandos básicos
- **Días 3-4:** Permisos y redirección
- **Días 5-7:** Práctica intensiva y ejercicios

### Semana 2: Herramientas Avanzadas

- **Días 1-3:** Herramientas de búsqueda y filtrado
- **Días 4-5:** Editores de terminal
- **Días 6-7:** Proyectos prácticos

### Semana 3: Automatización

- **Días 1-3:** Conceptos de scripting
- **Días 4-5:** Scripts avanzados
- **Días 6-7:** Proyecto final de automatización

## Prerrequisitos

- **Mínimos:** Conocimientos básicos de computación
- **Recomendados:** Experiencia previa con cualquier sistema operativo
- **Opcionales:** Conocimientos básicos de programación

## Herramientas Necesarias

### Para Windows

- **PowerShell** (incluido en Windows)
- **Git Bash** (se instala con Git)
- **Windows Subsystem for Linux (WSL)** (recomendado)

### Para macOS

- **Terminal** (incluido en macOS)
- **Homebrew** (gestor de paquetes)

### Para Linux

- **Bash/Zsh** (incluido por defecto)
- Distribución de tu elección (Ubuntu, CentOS, etc.)

## Recursos de Práctica

### Entornos Online Gratuitos

- [Replit](https://replit.com) - Terminal online
- [Katacoda](https://katacoda.com) - Escenarios interactivos
- [Play with Docker](https://labs.play-with-docker.com) - Incluye terminal Linux

### Comandos de Referencia Rápida

```bash
# Navegación básica
pwd                 # Mostrar directorio actual
ls -la             # Listar archivos con detalles
cd /path/to/dir    # Cambiar directorio
cd ~               # Ir al directorio home
cd ..              # Subir un nivel

# Manipulación de archivos
touch file.txt     # Crear archivo vacío
mkdir directory    # Crear directorio
cp source dest     # Copiar archivo
mv source dest     # Mover/renombrar archivo
rm file.txt        # Eliminar archivo
rm -rf directory   # Eliminar directorio recursivamente

# Búsqueda y filtrado
grep "pattern" file        # Buscar texto en archivo
find . -name "*.txt"      # Buscar archivos por nombre
ps aux | grep process     # Buscar procesos
history | tail -10        # Últimos 10 comandos

# Redirección
command > file.txt        # Redirigir salida a archivo
command >> file.txt       # Añadir salida a archivo
command < input.txt       # Usar archivo como entrada
command1 | command2       # Pipe: salida de cmd1 a entrada de cmd2
```

## Evaluación y Progreso

### Autoevaluación

- [ ] Puedo navegar por el sistema de archivos sin GUI
- [ ] Entiendo cómo funcionan los permisos de archivos
- [ ] Puedo usar pipes y redirección efectivamente
- [ ] Soy capaz de buscar archivos y contenido eficientemente
- [ ] Puedo crear scripts básicos para automatizar tareas
- [ ] Me siento cómodo usando editores de terminal

### Test Final

**Archivo:** [test.md](./test.md)

Completa el test al final de la fase para evaluar tu comprensión de los conceptos fundamentales de terminal y línea de comandos.

## Proyectos Prácticos Sugeridos

### Proyecto 1: Organizador de Archivos

Crear un script que organice archivos en carpetas según su extensión.

### Proyecto 2: Monitor del Sistema

Script que reporte el uso de CPU, memoria y disco.

### Proyecto 3: Backup Automático

Sistema de respaldo que comprima y archive directorios importantes.

### Proyecto 4: Log Analyzer

Herramienta que analice logs de aplicaciones y extraiga información útil.

## Comandos Esenciales por Categoría

### Gestión de Archivos

```bash
ls, cd, pwd, mkdir, rmdir, rm, cp, mv, touch, find, locate
```

### Visualización de Contenido

```bash
cat, less, more, head, tail, grep, wc, sort, uniq
```

### Permisos y Propiedades

```bash
chmod, chown, chgrp, umask, stat, file
```

### Procesos y Sistema

```bash
ps, top, htop, kill, killall, jobs, bg, fg, nohup
```

### Red y Conectividad

```bash
ping, wget, curl, ssh, scp, netstat
```

### Archivado y Compresión

```bash
tar, gzip, gunzip, zip, unzip
```

## Recursos Adicionales

### Documentación

- [GNU Bash Manual](https://www.gnu.org/software/bash/manual/)
- [Linux Command Line Basics](https://ubuntu.com/tutorials/command-line-for-beginners)
- [PowerShell Documentation](https://docs.microsoft.com/en-us/powershell/)

### Libros Recomendados

- "The Linux Command Line" - William Shotts
- "Learning the Bash Shell" - Cameron Newham
- "PowerShell in Depth" - Don Jones, Richard Siddaway

### Cursos Online

- Linux Academy / A Cloud Guru
- Udemy: "Linux Command Line Basics"
- Codecademy: "Learn the Command Line"

## Troubleshooting Común

### Problemas Frecuentes

**Error: Permission denied**

```bash
# Solución: Verificar permisos o usar sudo
ls -la file.txt
sudo chmod +x script.sh
```

**Error: Command not found**

```bash
# Verificar si el comando existe
which command_name
# Verificar PATH
echo $PATH
```

**Error: No such file or directory**

```bash
# Verificar ruta actual y archivo
pwd
ls -la
```

## Siguiente Fase

Una vez completada la Fase 0, estarás preparado para avanzar a la **[Fase 1: Fundamentos de DevOps](../fase1/README.md)**, donde aplicarás estas habilidades de terminal en el contexto específico de DevOps.

## Contacto y Soporte

Si tienes preguntas o necesitas ayuda durante esta fase:

1. Revisa la documentación de cada tema
2. Practica con los ejercicios incluidos
3. Completa los proyectos sugeridos
4. Utiliza los recursos adicionales

¡Buena suerte en tu viaje hacia el dominio de la terminal y DevOps!
