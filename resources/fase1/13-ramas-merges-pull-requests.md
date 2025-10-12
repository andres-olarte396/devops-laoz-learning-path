# Ramas, Merges, Pull Requests y Conflictos en Git

## Introducción

El manejo efectivo de **ramas (branches)**, **merges**, **pull requests** y **resolución de conflictos** es fundamental para el trabajo colaborativo en Git. Estos conceptos permiten que múltiples desarrolladores trabajen simultáneamente en diferentes características sin interferir entre sí.

## Conceptos de Ramas (Branches)

### ¿Qué es una Rama?

Una **rama** en Git es un puntero móvil hacia un commit específico. Permite crear líneas de desarrollo independientes para trabajar en diferentes características, experimentos o correcciones.

#### **Ventajas de usar Ramas**

```bash
# Ventajas del branching
✅ Desarrollo paralelo de características
✅ Aislamiento de trabajo experimental  
✅ Facilita code reviews
✅ Permite releases estables
✅ Rollback rápido a versiones estables
```

### Comandos Básicos de Ramas

#### **Crear y Cambiar Ramas**

```bash
# Ver ramas existentes
git branch

# Ver todas las ramas (locales y remotas)
git branch -a

# Crear nueva rama
git branch feature/nueva-caracteristica

# Cambiar a una rama
git checkout feature/nueva-caracteristica

# Crear y cambiar a una rama en un comando
git checkout -b feature/nueva-caracteristica

# Comando moderno para cambiar ramas
git switch feature/nueva-caracteristica

# Crear y cambiar con switch
git switch -c feature/nueva-caracteristica
```

#### **Gestión de Ramas**

```bash
# Renombrar rama actual
git branch -m nuevo-nombre

# Renombrar rama específica
git branch -m viejo-nombre nuevo-nombre

# Eliminar rama local
git branch -d feature/completada

# Forzar eliminación de rama
git branch -D feature/experimental

# Eliminar rama remota
git push origin --delete feature/completada

# Ver ramas merged
git branch --merged

# Ver ramas no merged
git branch --no-merged
```

## Estrategias de Branching

### 1. **Git Flow**

```bash
# Estructura de Git Flow
main/master     ←── Producción estable
develop         ←── Integración de desarrollo
feature/*       ←── Nuevas características
release/*       ←── Preparación de releases
hotfix/*        ←── Correcciones urgentes
```

#### **Implementación de Git Flow**

```bash
# Instalar git-flow (en sistemas Unix)
sudo apt-get install git-flow

# Inicializar git-flow en repositorio
git flow init

# Crear feature branch
git flow feature start mi-nueva-feature

# Finalizar feature
git flow feature finish mi-nueva-feature

# Crear release branch
git flow release start 1.0.0

# Finalizar release
git flow release finish 1.0.0

# Crear hotfix
git flow hotfix start critical-fix

# Finalizar hotfix
git flow hotfix finish critical-fix
```

### 2. **GitHub Flow**

```bash
# Flujo simplificado de GitHub Flow
main           ←── Siempre deployable
feature-branch ←── Trabajo en progreso

# Pasos del GitHub Flow:
# 1. Crear rama desde main
git checkout -b feature/user-authentication

# 2. Hacer commits
git add .
git commit -m "Add user login functionality"

# 3. Push y crear Pull Request
git push origin feature/user-authentication

# 4. Code review y merge
# 5. Deploy desde main
```

### 3. **GitLab Flow**

```bash
# GitLab Flow con environment branches
main           ←── Desarrollo
pre-production ←── Testing/Staging  
production     ←── Producción

# Flujo con feature branches
feature/xyz → main → pre-production → production
```

## Merges y Tipos de Merge

### 1. **Fast-Forward Merge**

```bash
# Scenario: Linear history
# main: A---B
# feature:    C---D

git checkout main
git merge feature/linear

# Resultado: A---B---C---D (main)
```

### 2. **Three-way Merge**

```bash
# Scenario: Divergent history
# main:    A---B---C
# feature:      D---E

git checkout main
git merge feature/divergent

# Resultado: 
# A---B---C---F (main)
#      \     /
#       D---E (merge commit F)
```

### 3. **Squash Merge**

```bash
# Combinar todos los commits en uno solo
git checkout main
git merge --squash feature/multiple-commits
git commit -m "Add complete user management system"

# Mantiene historia limpia en main
```

### 4. **Rebase y Merge**

```bash
# Rebase antes del merge
git checkout feature/clean-history
git rebase main

# Luego merge fast-forward
git checkout main
git merge feature/clean-history
```

## Pull Requests (GitHub) / Merge Requests (GitLab)

### Creación de Pull Request

#### **Vía Línea de Comandos**

```bash
# 1. Crear y desarrollar en rama
git checkout -b feature/user-profile
# ... hacer cambios ...
git add .
git commit -m "Implement user profile page"

# 2. Push de la rama
git push origin feature/user-profile

# 3. Crear PR vía GitHub CLI (opcional)
gh pr create --title "Add user profile functionality" \
             --body "Implements user profile with edit capabilities"
```

#### **Template de Pull Request**

```markdown
## Descripción
Breve descripción de los cambios realizados.

## Tipo de cambio
- [ ] Bug fix (cambio que corrige un issue)
- [ ] Nueva característica (cambio que añade funcionalidad)
- [ ] Breaking change (fix o feature que causa que funcionalidad existente no funcione)
- [ ] Actualización de documentación

## ¿Cómo se ha probado?
- [ ] Tests unitarios
- [ ] Tests de integración  
- [ ] Tests manuales

## Checklist:
- [ ] Mi código sigue las convenciones del proyecto
- [ ] He realizado self-review de mi código
- [ ] He comentado código complejo
- [ ] He actualizado la documentación
- [ ] Mis cambios no generan warnings
- [ ] He añadido tests que prueban mi fix/feature
- [ ] Tests nuevos y existentes pasan localmente

## Screenshots (si aplica)
![image](url)

## Issues relacionados
Fixes #123
```

### Proceso de Review

#### **Como Autor**

```bash
# Preparar código para review
git checkout feature/my-feature

# Asegurar commits limpios
git log --oneline

# Si necesitas limpiar commits
git rebase -i HEAD~3  # Interactive rebase últimos 3 commits

# Push final
git push origin feature/my-feature --force-with-lease
```

#### **Como Reviewer**

```bash
# Checkout del PR para testing local
git fetch origin
git checkout feature/my-feature

# Ejecutar tests
npm test  # o el comando de test correspondiente

# Revisar cambios
git diff main..feature/my-feature

# Ver archivos modificados
git diff --name-only main..feature/my-feature
```

## Resolución de Conflictos

### Tipos de Conflictos

#### **1. Conflictos de Merge**

```bash
# Scenario: Mismas líneas modificadas
# Archivo: index.js

<<<<<<< HEAD
const message = "Hello from main branch";
=======
const message = "Hello from feature branch";
>>>>>>> feature/greeting-update
```

#### **2. Conflictos de Rebase**

```bash
# Durante rebase interactivo
git rebase main
# CONFLICT (content): Merge conflict in file.js

# Resolver y continuar
git add file.js
git rebase --continue
```

### Herramientas para Resolución

#### **Configurar Merge Tool**

```bash
# Configurar VSCode como merge tool
git config --global merge.tool vscode
git config --global mergetool.vscode.cmd 'code --wait $MERGED'

# Configurar Vim como merge tool
git config --global merge.tool vimdiff

# Usar merge tool
git mergetool
```

#### **Resolución Manual**

```bash
# Ver estado del conflicto
git status

# Archivos en conflicto aparecen como:
# both modified: archivo.js

# Editar archivo manualmente
vim archivo.js

# Buscar marcadores de conflicto
# <<<<<<< HEAD
# =======  
# >>>>>>> branch-name

# Después de resolver
git add archivo.js
git commit
```

### Estrategias de Resolución

#### **1. Estrategia "Ours" vs "Theirs"**

```bash
# Durante merge - preferir nuestra versión
git checkout --ours archivo.js

# Durante merge - preferir su versión  
git checkout --theirs archivo.js

# Durante rebase - preferir versión del commit actual
git checkout --theirs archivo.js  # (confuso: en rebase se invierte)

# Durante rebase - preferir versión de la rama base
git checkout --ours archivo.js
```

#### **2. Resolución Línea por Línea**

```javascript
// Antes del conflicto
function calculateTotal(items) {
    return items.reduce((sum, item) => sum + item.price, 0);
}

// Conflicto
<<<<<<< HEAD
function calculateTotal(items) {
    // Añadir validación
    if (!Array.isArray(items)) return 0;
    return items.reduce((sum, item) => sum + item.price, 0);
}
=======
function calculateTotal(items) {
    // Añadir descuento
    const total = items.reduce((sum, item) => sum + item.price, 0);
    return total * 0.9; // 10% descuento
}
>>>>>>> feature/discount

// Resolución combinada
function calculateTotal(items) {
    // Añadir validación
    if (!Array.isArray(items)) return 0;
    // Añadir descuento
    const total = items.reduce((sum, item) => sum + item.price, 0);
    return total * 0.9; // 10% descuento
}
```

## Workflows Avanzados

### 1. **Feature Branch Workflow**

```bash
# 1. Sincronizar con main
git checkout main
git pull origin main

# 2. Crear feature branch
git checkout -b feature/user-settings

# 3. Desarrollo iterativo
git add .
git commit -m "Add user settings model"
git add .  
git commit -m "Implement settings UI"
git add .
git commit -m "Add settings API endpoints"

# 4. Mantener sincronizado con main
git checkout main
git pull origin main
git checkout feature/user-settings
git rebase main  # o git merge main

# 5. Push y crear PR
git push origin feature/user-settings
```

### 2. **Gitflow Release Workflow**

```bash
# Preparar release
git checkout develop
git pull origin develop
git checkout -b release/1.2.0

# Hacer ajustes finales
git commit -am "Bump version to 1.2.0"
git commit -am "Update CHANGELOG"

# Merge a main y develop
git checkout main
git merge release/1.2.0
git tag 1.2.0

git checkout develop
git merge release/1.2.0

# Push everything
git push origin main develop --tags
git branch -d release/1.2.0
```

### 3. **Hotfix Workflow**

```bash
# Crear hotfix desde main
git checkout main
git pull origin main
git checkout -b hotfix/critical-security-fix

# Hacer fix
git commit -am "Fix critical security vulnerability"

# Merge a main y develop
git checkout main
git merge hotfix/critical-security-fix
git tag 1.2.1

git checkout develop  
git merge hotfix/critical-security-fix

# Deploy inmediato
git push origin main develop --tags
```

## Mejores Prácticas

### 1. **Naming Conventions**

```bash
# Feature branches
feature/user-authentication
feature/payment-integration
feature/dashboard-redesign

# Bug fixes
bugfix/header-alignment
bugfix/login-validation
fix/memory-leak

# Hotfixes
hotfix/security-patch
hotfix/critical-bug

# Releases
release/1.0.0
release/2.1.0

# Experimental
experiment/new-architecture
spike/performance-test
```

### 2. **Commit Messages en Branches**

```bash
# Convenciones de commit
git commit -m "feat: add user profile editing"
git commit -m "fix: resolve login redirect issue"  
git commit -m "docs: update API documentation"
git commit -m "refactor: simplify authentication logic"
git commit -m "test: add unit tests for user service"
```

### 3. **Branch Protection Rules**

```bash
# Configurar protección en GitHub/GitLab
# - Require pull request reviews
# - Require status checks to pass
# - Require branches to be up to date
# - Require signed commits
# - Restrict pushes to matching branches
```

## Herramientas Útiles

### 1. **Git Aliases para Branches**

```bash
# Configurar aliases útiles
git config --global alias.co checkout
git config --global alias.br branch
git config --global alias.sw switch
git config --global alias.lg "log --graph --oneline --all"

# Alias para workflow común
git config --global alias.feature "checkout -b feature/"
git config --global alias.hotfix "checkout -b hotfix/"
```

### 2. **Scripts de Automatización**

```bash
#!/bin/bash
# new-feature.sh - Script para crear nueva feature

FEATURE_NAME=$1
if [ -z "$FEATURE_NAME" ]; then
    echo "Usage: ./new-feature.sh <feature-name>"
    exit 1
fi

# Sincronizar con main
git checkout main
git pull origin main

# Crear nueva feature branch
git checkout -b "feature/$FEATURE_NAME"

echo "Created feature branch: feature/$FEATURE_NAME"
echo "Start developing and commit your changes!"
```

### 3. **Git Hooks para Quality**

```bash
#!/bin/sh
# .git/hooks/pre-commit
# Ejecutar tests antes de commit

echo "Running tests..."
npm test

if [ $? -ne 0 ]; then
    echo "Tests failed. Commit aborted."
    exit 1
fi

echo "Tests passed. Proceeding with commit."
```

## Troubleshooting Común

### 1. **Problemas de Merge**

```bash
# Abortar merge en progreso
git merge --abort

# Abortar rebase en progreso  
git rebase --abort

# Ver estado durante conflicto
git status
git diff

# Continuar después de resolver
git add .
git commit  # para merge
git rebase --continue  # para rebase
```

### 2. **Recuperar Trabajo Perdido**

```bash
# Ver historial de referencias
git reflog

# Recuperar commit "perdido"
git checkout <commit-hash>
git checkout -b recovery-branch

# Recuperar rama eliminada
git checkout -b restored-branch <commit-hash>
```

### 3. **Limpiar Branches Obsoletos**

```bash
# Limpiar branches merged localmente
git branch --merged main | grep -v main | xargs git branch -d

# Limpiar referencias remotas obsoletas
git remote prune origin

# Script de limpieza
#!/bin/bash
echo "Cleaning up merged branches..."
git checkout main
git pull origin main
git branch --merged main | grep -v main | xargs -n 1 git branch -d
git remote prune origin
echo "Cleanup complete!"
```

## Ejercicios Prácticos

### Ejercicio 1: Feature Branch Workflow

```bash
# 1. Crear repositorio de prueba
mkdir git-workflow-practice
cd git-workflow-practice
git init

# 2. Setup inicial
echo "# Project" > README.md
git add README.md
git commit -m "Initial commit"

# 3. Crear feature branch
git checkout -b feature/user-login

# 4. Hacer cambios y commits
echo "Login functionality" > login.js
git add login.js
git commit -m "Add login functionality"

# 5. Simular trabajo en main
git checkout main
echo "Updated README" >> README.md
git commit -am "Update README"

# 6. Merge feature
git merge feature/user-login
```

### Ejercicio 2: Resolución de Conflictos

```bash
# 1. Crear conflicto intencionalmente
git checkout main
echo "Main branch content" > conflict.txt
git add conflict.txt
git commit -m "Add content from main"

git checkout -b feature/conflict-test
echo "Feature branch content" > conflict.txt
git add conflict.txt
git commit -m "Add content from feature"

# 2. Intentar merge
git checkout main
git merge feature/conflict-test
# ¡Conflicto!

# 3. Resolver manualmente
# Editar conflict.txt para resolver
git add conflict.txt
git commit -m "Resolve merge conflict"
```

### Ejercicio 3: Pull Request Simulation

```bash
# Simular PR workflow sin GitHub
# 1. Crear "remote" local
mkdir remote-repo.git
cd remote-repo.git
git init --bare
cd ..

# 2. Clone como si fuera remote
git clone remote-repo.git local-repo
cd local-repo

# 3. Feature development
git checkout -b feature/new-feature
echo "New feature" > feature.txt
git add feature.txt
git commit -m "Add new feature"
git push origin feature/new-feature

# 4. Simular review process
git checkout main
git pull origin main
git diff main..origin/feature/new-feature

# 5. "Merge" el PR
git merge origin/feature/new-feature
git push origin main
```

## Recursos Adicionales

### Documentación y Guías

- [Git Branching - Git Book](https://git-scm.com/book/en/v2/Git-Branching-Branches-in-a-Nutshell)
- [GitHub Flow Guide](https://guides.github.com/introduction/flow/)
- [Atlassian Git Workflows](https://www.atlassian.com/git/tutorials/comparing-workflows)
- [GitLab Flow Documentation](https://docs.gitlab.com/ee/topics/gitlab_flow.html)

### Herramientas Visuales

- **GitKraken**: Cliente Git visual
- **Sourcetree**: Cliente Git gratuito de Atlassian
- **Git Graph (VSCode)**: Extensión para visualizar branches
- **GitHub Desktop**: Cliente oficial de GitHub

### Libros Recomendados

- "Pro Git" - Scott Chacon y Ben Straub
- "Git Workflows" - Yan Pritzker
- "Version Control with Git" - Jon Loeliger

---

## Próximos Pasos

Una vez que domines ramas, merges y pull requests, estarás listo para:

1. **CI/CD Avanzado**: Pipelines automatizados basados en branches
2. **Git Hooks**: Automatización de workflows
3. **Monorepos**: Gestión de múltiples proyectos
4. **Advanced Git**: Reflog, bisect, submodules

¡El manejo efectivo de branches es fundamental para el trabajo colaborativo en DevOps!