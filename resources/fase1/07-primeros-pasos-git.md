# Primeros pasos con Git

Git es el sistema de control de versiones distribuido más popular del mundo y una herramienta fundamental en DevOps. En este módulo aprenderás los conceptos básicos de Git y cómo colaborar efectivamente usando GitHub y GitLab.

---

## **¿Por qué es importante Git en DevOps?**

Git es mucho más que una herramienta de versionado; es el fundamento que permite:

- **Colaboración eficiente** entre equipos de desarrollo
- **Trazabilidad completa** de todos los cambios en el código
- **Integración** con pipelines de CI/CD
- **Gestión de ramas** para diferentes entornos (desarrollo, testing, producción)
- **Reversión rápida** de cambios problemáticos

En DevOps, Git actúa como el punto de entrada para la mayoría de automatizaciones: cada commit puede disparar builds, tests, deployments y monitoreo.

### **Recurso complementario**

- **[Video: Fundamentos de DevOps](../../assets/Fundamentos_de_DevOps.mp4)**

    Este video profundiza en los fundamentos conceptuales de DevOps y cómo Git se integra como pieza fundamental en el ecosistema DevOps completo.

---

## **1. Fundamentos del Control de Versiones**

### **¿Qué es el control de versiones?**

El control de versiones es un sistema que registra los cambios realizados en archivos a lo largo del tiempo, permitiendo:

- **Historial completo**: Ver qué cambió, cuándo y quién lo hizo
- **Recuperación**: Volver a versiones anteriores cuando sea necesario
- **Comparación**: Analizar diferencias entre versiones
- **Ramificación**: Trabajar en múltiples líneas de desarrollo simultáneamente

### **Git vs otros sistemas**

| Característica | Git (Distribuido) | SVN/CVS (Centralizado) |
|---|---|---|
| **Arquitectura** | Cada desarrollador tiene el historial completo | Servidor central con historial |
| **Trabajo offline** | Completamente funcional | Limitado |
| **Velocidad** | Operaciones locales muy rápidas | Depende de la red |
| **Branching** | Muy eficiente y rápido | Lento y complejo |
| **Integridad** | Checksums SHA-1 para todo | Números de revisión |

### **Conceptos básicos de Git**

```bash
# Los tres estados de Git
Working Directory → Staging Area → Repository
     (modificado)     (preparado)    (confirmado)
```

- **Working Directory**: Archivos en tu directorio de trabajo
- **Staging Area**: Archivos preparados para el próximo commit
- **Repository**: Historial permanente de commits

---

## **2. Comandos Esenciales de Git**

### **Configuración inicial**

```bash
# Configurar identidad (requerido)
git config --global user.name "Tu Nombre"
git config --global user.email "tu.email@example.com"

# Configurar editor predeterminado
git config --global core.editor "code --wait"  # VS Code
git config --global core.editor "vim"          # Vim

# Ver configuración
git config --list
```

### **Operaciones básicas**

```bash
# Inicializar repositorio
git init
git clone <url-repositorio>

# Estado y diferencias
git status                    # Estado actual
git diff                     # Cambios no preparados
git diff --staged            # Cambios preparados
git log --oneline           # Historial resumido

# Preparar y confirmar cambios
git add <archivo>           # Preparar archivo específico
git add .                   # Preparar todos los cambios
git commit -m "Mensaje"     # Confirmar cambios
git commit -am "Mensaje"    # Preparar y confirmar archivos modificados

# Sincronización
git push origin <rama>      # Enviar cambios
git pull origin <rama>      # Obtener cambios
git fetch                   # Descargar sin fusionar
```

### **Gestión de archivos**

```bash
# Rastrear y descartar cambios
git add <archivo>           # Comenzar a rastrear
git rm <archivo>            # Eliminar del repositorio
git mv <viejo> <nuevo>      # Renombrar/mover archivo

# Deshacer cambios
git checkout -- <archivo>   # Descartar cambios locales
git reset HEAD <archivo>    # Quitar del staging
git reset --hard HEAD      # Descartar TODOS los cambios locales
```

---

## **3. GitHub y GitLab: Plataformas de Colaboración**

### **GitHub vs GitLab: Comparación**

| Aspecto | GitHub | GitLab |
|---|---|---|
| **Alojamiento** | Principalmente en la nube | Cloud y on-premise |
| **CI/CD** | GitHub Actions (desde 2019) | GitLab CI/CD integrado desde el inicio |
| **DevOps** | Ecosystem de terceros | Suite DevOps completa integrada |
| **Precio** | Gratuito para repositorios públicos | Más funciones gratuitas |
| **Comunidad** | Más grande, especialmente open source | Enfoque enterprise |

### **Configuración en GitHub**

```bash
# Agregar repositorio remoto
git remote add origin https://github.com/usuario/repo.git
git remote add origin git@github.com:usuario/repo.git  # SSH

# Verificar remotos
git remote -v

# Cambiar URL del remoto
git remote set-url origin <nueva-url>
```

### **Flujo básico de trabajo**

```bash
# 1. Clonar repositorio
git clone https://github.com/usuario/proyecto.git
cd proyecto

# 2. Crear rama para nueva funcionalidad
git checkout -b feature/nueva-funcionalidad

# 3. Realizar cambios y commits
git add .
git commit -m "Implementar nueva funcionalidad"

# 4. Subir rama al repositorio remoto
git push origin feature/nueva-funcionalidad

# 5. Crear Pull Request desde la interfaz web
# 6. Después de aprobación, fusionar con main/master
```

### **Configuración SSH para GitHub/GitLab**

```bash
# 1. Generar clave SSH
ssh-keygen -t ed25519 -C "tu.email@example.com"

# 2. Agregar clave al agente SSH
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/id_ed25519

# 3. Copiar clave pública y agregar en GitHub/GitLab
cat ~/.ssh/id_ed25519.pub

# 4. Probar conexión
ssh -T git@github.com
ssh -T git@gitlab.com
```

---

## **4. Trabajo con Ramas (Branching)**

### **Conceptos de branching**

Las ramas permiten desarrollar funcionalidades de forma aislada sin afectar el código principal.

```bash
# Listar ramas
git branch                  # Ramas locales
git branch -r              # Ramas remotas
git branch -a              # Todas las ramas

# Crear y cambiar ramas
git branch <nombre-rama>    # Crear rama
git checkout <nombre-rama>  # Cambiar a rama
git checkout -b <nombre>    # Crear y cambiar

# Eliminar ramas
git branch -d <rama>        # Eliminar rama local
git push origin --delete <rama>  # Eliminar rama remota
```

### **Estrategias de branching**

#### **Git Flow**
```
main/master    ─●─────●─────●─────●─ (Producción)
              /       \     \
develop   ─●─●─────●───●─────●───●─ (Desarrollo)
         /   \         \   /
feature/x    ●─●─●─●───●─/       (Funcionalidades)
```

#### **GitHub Flow (Simplificado)**
```
main    ─●─────●─────●─────●─ (Siempre desplegable)
        /       \     \
feature  ●─●─●─●─/     ●─●─/  (Ramas cortas)
```

#### **Comandos para branching**

```bash
# Git Flow básico
git checkout -b feature/login-usuario
# ... desarrollar funcionalidad ...
git checkout develop
git merge feature/login-usuario
git branch -d feature/login-usuario

# GitHub Flow
git checkout -b fix/corregir-bug-sesion
# ... corregir bug ...
git checkout main
git merge fix/corregir-bug-sesion
git branch -d fix/corregir-bug-sesion
```

---

## **5. Merges y Resolución de Conflictos**

### **Tipos de merge**

#### **Fast-Forward Merge**
```bash
git checkout main
git merge feature/nueva-funcionalidad
# Si no hay commits en main desde que se creó la rama
```

#### **Three-Way Merge**
```bash
git checkout main
git merge feature/nueva-funcionalidad
# Crea un commit de merge cuando hay cambios en ambas ramas
```

#### **Squash Merge**
```bash
git checkout main
git merge --squash feature/nueva-funcionalidad
git commit -m "Agregar nueva funcionalidad completa"
# Combina todos los commits de la rama en uno solo
```

### **Resolución de conflictos**

Los conflictos ocurren cuando Git no puede fusionar automáticamente los cambios.

#### **Identificar conflictos**
```bash
git merge feature/otra-funcionalidad
# Auto-merging index.html
# CONFLICT (content): Merge conflict in index.html
# Automatic merge failed; fix conflicts and then commit the result.

git status
# On branch main
# You have unmerged paths.
# Unmerged paths:
#   both modified:   index.html
```

#### **Resolver conflictos manualmente**
```html
<!-- Archivo con conflicto -->
<!DOCTYPE html>
<html>
<head>
<<<<<<< HEAD
    <title>Mi Aplicación - Versión Principal</title>
=======
    <title>Mi Aplicación - Nueva Funcionalidad</title>
>>>>>>> feature/otra-funcionalidad
</head>
```

**Pasos para resolución:**
```bash
# 1. Editar archivos para resolver conflictos
# 2. Eliminar marcadores de conflicto (<<<<, ====, >>>>)
# 3. Preparar archivos resueltos
git add index.html

# 4. Completar el merge
git commit  # Se abrirá editor con mensaje por defecto
```

#### **Herramientas para resolución**
```bash
# Configurar herramienta de merge visual
git config --global merge.tool vscode
git config --global mergetool.vscode.cmd 'code --wait $MERGED'

# Usar herramienta visual
git mergetool

# Ver estado del merge
git status
git log --oneline --graph -10
```

### **Mejores prácticas para evitar conflictos**

1. **Mantener ramas actualizadas**:
   ```bash
   git checkout feature/mi-rama
   git pull origin main  # O git rebase origin/main
   ```

2. **Commits pequeños y frecuentes**
3. **Comunicación en equipo** sobre archivos compartidos
4. **Usar .gitignore** apropiadamente

---

## **6. Pull Requests (GitHub) / Merge Requests (GitLab)**

### **¿Qué son los Pull Requests?**

Los Pull Requests (PR) son una funcionalidad que permite:
- **Revisar código** antes de fusionar cambios
- **Discutir** implementaciones y mejoras
- **Ejecutar pruebas automáticas** antes del merge
- **Mantener historial** de decisiones técnicas

### **Proceso de Pull Request**

#### **1. Preparar la rama**
```bash
# Asegurar que la rama esté actualizada
git checkout feature/nueva-api
git rebase main  # O git merge main
git push origin feature/nueva-api
```

#### **2. Crear PR desde la interfaz web**

**Elementos esenciales de un buen PR:**

```markdown
## Descripción
Implementa endpoint para autenticación de usuarios usando JWT.

## Cambios realizados
- ✅ Crear middleware de autenticación
- ✅ Implementar generación de tokens JWT
- ✅ Agregar validación de tokens
- ✅ Documentar nuevos endpoints

## Testing
- [x] Pruebas unitarias para AuthService
- [x] Pruebas de integración para endpoints
- [x] Validar tokens expirados
- [x] Verificar middleware en rutas protegidas

## Screenshots/Evidencia
![API Response](./docs/api-response.png)

## Checklist
- [x] Código sigue estándares del proyecto
- [x] Todas las pruebas pasan
- [x] Documentación actualizada
- [x] No hay conflictos con main
```

#### **3. Proceso de revisión**

```bash
# Revisor puede hacer checkout del PR localmente
git fetch origin pull/123/head:pr-123
git checkout pr-123

# Después de revisión, el autor puede hacer cambios
git checkout feature/nueva-api
# ... hacer cambios según feedback ...
git add .
git commit -m "Corregir validación según review"
git push origin feature/nueva-api  # Actualiza automáticamente el PR
```

### **GitHub Actions en Pull Requests**

```yaml
# .github/workflows/pr-checks.yml
name: PR Checks

on:
  pull_request:
    branches: [ main, develop ]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v2
    - name: Setup Node.js
      uses: actions/setup-node@v2
      with:
        node-version: '16'
    
    - name: Install dependencies
      run: npm ci
    
    - name: Run tests
      run: npm test
    
    - name: Run linting
      run: npm run lint
    
    - name: Check code coverage
      run: npm run coverage
```

---

## **7. Flujos de Trabajo Avanzados**

### **Git Rebase vs Merge**

#### **Git Merge (preserva historial)**
```bash
git checkout main
git merge feature/nueva-funcionalidad
# Crea commit de merge, mantiene historial de ramas
```

#### **Git Rebase (historial lineal)**
```bash
git checkout feature/nueva-funcionalidad
git rebase main
git checkout main
git merge feature/nueva-funcionalidad  # Fast-forward merge
```

**Cuándo usar cada uno:**
- **Merge**: Para ramas importantes, cuando quieres preservar el contexto de desarrollo
- **Rebase**: Para limpiar historial, en ramas de funcionalidades pequeñas

### **Cherry Pick**
```bash
# Aplicar un commit específico de otra rama
git cherry-pick <hash-del-commit>

# Cherry pick múltiples commits
git cherry-pick <commit1> <commit2> <commit3>
```

### **Stash para trabajo temporal**
```bash
# Guardar trabajo temporal
git stash push -m "Trabajo en progreso en formulario"
git stash list

# Aplicar stash
git stash pop           # Aplica y elimina el último stash
git stash apply stash@{1}  # Aplica stash específico sin eliminarlo

# Crear rama desde stash
git stash branch nueva-funcionalidad-desde-stash stash@{0}
```

---

## **8. Mejores Prácticas y Convenciones**

### **Convenciones para commits**

#### **Conventional Commits**
```bash
# Formato: <tipo>[ámbito opcional]: <descripción>
feat: agregar autenticación JWT
fix: corregir validación de email en formulario
docs: actualizar README con instrucciones de instalación
style: formatear código según estándares ESLint
refactor: reorganizar estructura de carpetas de componentes
test: agregar pruebas para AuthService
chore: actualizar dependencias de desarrollo

# Con ámbito
feat(api): implementar endpoint de usuarios
fix(ui): corregir alineación de botones en mobile
```

#### **Commits descriptivos**
```bash
# ❌ Mal
git commit -m "fix"
git commit -m "cambios"
git commit -m "update"

# ✅ Bien
git commit -m "fix: corregir validación de edad en formulario registro"
git commit -m "feat: implementar cache Redis para sesiones usuario"
git commit -m "docs: agregar ejemplos de uso de API en README"
```

### **Organización de ramas**

```bash
# Convenciones de nombres
feature/JIRA-123-implementar-autenticacion
bugfix/corregir-memory-leak-dashboard
hotfix/seguridad-validacion-inputs
release/v2.1.0
experimental/nuevas-metricas-performance

# Ramas por ambiente
main/master     # Producción
develop         # Integración
staging         # Pre-producción
qa              # Testing
```

### **Archivos .gitignore importantes**

```gitignore
# Node.js
node_modules/
npm-debug.log*
.env
.env.local
dist/
build/

# Python
__pycache__/
*.pyc
*.pyo
venv/
.env

# Java
target/
*.jar
*.war
*.class

# IDE
.vscode/settings.json
.idea/
*.swp

# OS
.DS_Store
Thumbs.db

# Logs y temporales
*.log
tmp/
temp/

# Secretos y configuración
config/secrets.yml
.env.production
certificates/
```

---

## **9. Integración con DevOps**

### **Recurso complementario**

- **[Audio: DevOps Desvelado - Cultura, El Ciclo Infinito y las Herramientas](../../assets/DevOps_Desvelado__Cultura,_El_Ciclo_Infinito_y_las_Herramientas - 1758768483444.m4a)**

    Este audio profundiza en los aspectos culturales de DevOps, el concepto del ciclo infinito de mejora continua, y cómo Git se integra como herramienta fundamental en este ecosistema.

### **Git en pipelines CI/CD**

Git actúa como disparador para la mayoría de pipelines DevOps:

```yaml
# Ejemplo: GitHub Actions
on:
  push:
    branches: [ main, develop ]
  pull_request:
    branches: [ main ]

jobs:
  build-and-deploy:
    if: github.ref == 'refs/heads/main'
    steps:
      - name: Checkout code
        uses: actions/checkout@v2
        with:
          fetch-depth: 0  # Obtener historial completo
      
      - name: Get version from Git tag
        run: |
          VERSION=$(git describe --tags --abbrev=0 2>/dev/null || echo "v0.1.0")
          echo "VERSION=$VERSION" >> $GITHUB_ENV
```

### **Git hooks para automatización**

```bash
# pre-commit hook (ejecuta antes de cada commit)
#!/bin/sh
# .git/hooks/pre-commit

echo "Ejecutando pruebas antes del commit..."
npm test

if [ $? -ne 0 ]; then
  echo "❌ Las pruebas fallaron. Commit cancelado."
  exit 1
fi

echo "Ejecutando linting..."
npm run lint

if [ $? -ne 0 ]; then
  echo "❌ Linting falló. Commit cancelado."
  exit 1
fi

echo "✅ Pre-commit checks pasaron."
```

### **Versionado semántico con Git tags**

```bash
# Crear tags para versiones
git tag -a v1.2.0 -m "Versión 1.2.0: Nueva funcionalidad de autenticación"
git push origin v1.2.0

# Listar tags
git tag -l
git tag -l "v1.*"  # Tags que coincidan con patrón

# Ver información de un tag
git show v1.2.0

# Checkout a una versión específica
git checkout v1.2.0
```

---

## **10. Herramientas y Recursos Adicionales**

### **Herramientas GUI para Git**

| Herramienta | Plataforma | Características |
|---|---|---|
| **GitKraken** | Windows, Mac, Linux | Interfaz visual intuitiva, integración con servicios |
| **SourceTree** | Windows, Mac | Gratuita, de Atlassian |
| **GitHub Desktop** | Windows, Mac | Simple, ideal para principiantes |
| **VS Code Git** | Integrado | Extensiones Git Lens, Git History |

### **Comandos útiles para debugging**

```bash
# Ver quién modificó cada línea
git blame <archivo>

# Buscar en historial
git log --grep="autenticación"
git log --author="Juan Pérez"
git log --since="2024-01-01"

# Ver cambios introducidos por un commit
git show <hash-commit>

# Comparar entre branches
git diff main..feature/nueva-funcionalidad
git diff --name-only main..develop

# Encontrar cuándo se introdujo un bug
git bisect start
git bisect bad                    # Commit actual es malo
git bisect good v1.0.0           # Esta versión funcionaba
# Git irá dividiendo el historial para encontrar el commit problemático
```

### **Configuraciones útiles**

```bash
# Aliases para comandos frecuentes
git config --global alias.st status
git config --global alias.co checkout
git config --global alias.br branch
git config --global alias.ci commit
git config --global alias.unstage 'reset HEAD --'
git config --global alias.visual '!gitk'
git config --global alias.tree 'log --oneline --graph --decorate --all'

# Configurar colores
git config --global color.ui auto

# Configurar push por defecto
git config --global push.default current

# Configurar rebase automático en pull
git config --global pull.rebase true
```

---

## **Casos de Estudio Prácticos**

### **Caso 1: Colaboración en equipo**

**Escenario**: Un equipo de 4 desarrolladores trabajando en una aplicación web.

```bash
# Desarrollador 1 (Team Lead)
git checkout -b feature/user-authentication
# ... implementa autenticación básica ...
git add .
git commit -m "feat: implementar autenticación básica con JWT"
git push origin feature/user-authentication

# Desarrollador 2 (desde la misma rama)
git fetch origin
git checkout feature/user-authentication
git pull origin feature/user-authentication
# ... agrega validaciones adicionales ...
git add .
git commit -m "feat: agregar validaciones de seguridad en auth"
git push origin feature/user-authentication

# Desarrollador 3 (nueva funcionalidad relacionada)
git checkout feature/user-authentication
git checkout -b feature/user-profile
# ... implementa perfil de usuario ...
git add .
git commit -m "feat: implementar gestión de perfil usuario"
git push origin feature/user-profile
```

### **Caso 2: Hotfix de emergencia**

**Escenario**: Bug crítico en producción que necesita arreglo inmediato.

```bash
# 1. Crear hotfix desde main
git checkout main
git pull origin main
git checkout -b hotfix/critical-security-fix

# 2. Implementar fix
# ... corregir vulnerabilidad ...
git add .
git commit -m "fix: resolver vulnerabilidad de inyección SQL en login"

# 3. Probar localmente
npm test
npm run security-scan

# 4. Desplegar a producción
git checkout main
git merge hotfix/critical-security-fix
git push origin main
git tag -a v2.1.1 -m "Hotfix: Security vulnerability"
git push origin v2.1.1

# 5. Integrar en develop
git checkout develop
git merge hotfix/critical-security-fix
git push origin develop

# 6. Limpiar rama
git branch -d hotfix/critical-security-fix
git push origin --delete hotfix/critical-security-fix
```

### **Caso 3: Resolución de conflicto complejo**

**Escenario**: Dos desarrolladores modificaron el mismo archivo de configuración.

```bash
# Desarrollador A: Modificó configuración de base de datos
git checkout -b feature/database-optimization
# Cambió: database.host = "prod-db-server"
git add config/database.yml
git commit -m "feat: optimizar configuración BD para producción"

# Desarrollador B: Modificó configuración de cache
git checkout main
git checkout -b feature/redis-cache
# Cambió: cache.enabled = true (misma sección del archivo)
git add config/database.yml
git commit -m "feat: habilitar cache Redis"

# Al hacer merge: CONFLICTO
git checkout main
git merge feature/database-optimization  # OK
git merge feature/redis-cache            # CONFLICTO

# Resolución:
git status
# both modified: config/database.yml

# Editar archivo:
# production:
#   database:
# <<<<<<< HEAD
#     host: "prod-db-server"
# =======
#     host: "localhost"
#   cache:
#     enabled: true
# >>>>>>> feature/redis-cache

# Resolución final:
# production:
#   database:
#     host: "prod-db-server"  # De feature/database-optimization
#   cache:
#     enabled: true           # De feature/redis-cache

git add config/database.yml
git commit -m "resolve: merge database optimization and Redis cache"
```

---

## **Recursos y Enlaces de Referencia**

### **Documentación oficial**
- [Documentación oficial de Git](https://git-scm.com/doc)
- [Pro Git Book (gratuito)](https://git-scm.com/book)
- [GitHub Docs](https://docs.github.com/)
- [GitLab Documentation](https://docs.gitlab.com/)

### **Repositorios de práctica y proyectos**

- **[GitHub Projects Learning](https://github.com/andres-olarte396/github-projects-learning)** - Repositorio con proyectos prácticos y ejercicios para profundizar en Git y GitHub. Incluye ejemplos de flujos de trabajo, resolución de conflictos, y colaboración en equipos.

### **Recursos visuales**

- **[Mapa Mental: Fundamentos de Git](../../assets/mind-map-fundamentals.png)**

    ![Mapa Mental de Fundamentos](../../assets/mind-map-fundamentals.png)

    Mapa mental visual que sintetiza todos los conceptos fundamentales de Git cubiertos en este módulo, ideal para revisión rápida y estudio.

### **Herramientas de aprendizaje interactivo**

- [Learn Git Branching](https://learngitbranching.js.org/) - Simulador visual interactivo
- [GitHub Skills](https://skills.github.com/) - Cursos prácticos de GitHub
- [Atlassian Git Tutorials](https://www.atlassian.com/git/tutorials)

### **Cheat Sheets**

- [Git Cheat Sheet oficial](https://git-scm.com/docs)
- [GitHub Git Cheat Sheet](https://github.github.com/training-kit/downloads/github-git-cheat-sheet.pdf)

### **Blogs y recursos de la comunidad**

- [Oh Shit, Git!?!](https://ohshitgit.com/) - Soluciones para situaciones complicadas
- [Git Tips](https://github.com/git-tips/tips) - Repositorio con tips avanzados

---

## **Próximos Pasos**

Una vez que domines estos conceptos de Git, estarás listo para avanzar a la **Fase 2: Automatización y CI/CD**, donde aprenderás cómo Git se integra con:

- **Pipelines de CI/CD** automatizados
- **Jenkins, GitHub Actions, GitLab CI**
- **Estrategias de deployment** basadas en ramas
- **Testing automatizado** en cada commit

Git es el primer eslabón en la cadena de herramientas DevOps, y dominar estos conceptos te proporcionará una base sólida para todo lo que viene después.

---

## **Ejercicios Prácticos Recomendados**

### **Ejercicios básicos**

1. **Configurar un repositorio completo** con ramas de develop, staging y main
2. **Simular conflictos** y practicar resolución manual
3. **Crear Pull Requests** con revisiones de código
4. **Implementar Git hooks** para automatizar validaciones
5. **Configurar diferentes flujos de trabajo** (Git Flow vs GitHub Flow)

### **Proyectos prácticos avanzados**

Para profundizar en Git y GitHub con ejercicios prácticos y proyectos reales, explora el **[repositorio GitHub Projects Learning](https://github.com/andres-olarte396/github-projects-learning)**. Este repositorio incluye:

- **Proyectos colaborativos** para practicar flujos de trabajo en equipo
- **Ejercicios de resolución de conflictos** con escenarios realistas
- **Ejemplos de Git hooks** y automatización
- **Templates de Pull Requests** y mejores prácticas
- **Casos de estudio** de flujos de trabajo DevOps

¡Felicidades! Has completado el módulo de Git. Ahora tienes las herramientas fundamentales para colaborar efectivamente en proyectos DevOps y estás listo para avanzar a la automatización de procesos.