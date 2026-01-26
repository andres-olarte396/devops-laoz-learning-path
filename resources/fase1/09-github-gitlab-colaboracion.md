# GitHub/GitLab: Colaboración y Flujos de Trabajo

## Introducción

Una vez que dominas los comandos básicos de Git, el siguiente paso es aprender cómo utilizarlo efectivamente para colaborar en equipo usando plataformas como GitHub y GitLab. Estas plataformas no solo alojan tus repositorios, sino que también proporcionan herramientas poderosas para la colaboración, revisión de código y gestión de proyectos.

## Conceptos Fundamentales

### ¿Qué son GitHub y GitLab?

**GitHub** y **GitLab** son plataformas basadas en web que utilizan Git para el control de versiones y agregan funcionalidades adicionales para la colaboración en equipos de desarrollo.

**Diferencias principales:**

- **GitHub**: Mayor adopción, enfoque en open source, propiedad de Microsoft
- **GitLab**: Plataforma más completa con CI/CD integrado, opciones self-hosted

## Ramas (Branches)

### ¿Qué son las Ramas?

Las ramas permiten desarrollar funcionalidades de forma aislada sin afectar el código principal.

### Estrategias de Branching

#### 1. **Git Flow**

```
main/master  ─────●─────●─────●─────
                  │     │     │
develop      ─────●─────●─────●─────
                  │
feature/login ────●─────●
```

**Ramas principales:**

- `main/master`: Código de producción
- `develop`: Integración de funcionalidades
- `feature/*`: Desarrollo de funcionalidades específicas
- `release/*`: Preparación de releases
- `hotfix/*`: Correcciones urgentes en producción

#### 2. **GitHub Flow (Simplificado)**

```
main    ─────●─────●─────●─────
              │     │     │
feature ──────●─────●─────┘
```

**Proceso:**

1. Crear rama desde `main`
2. Desarrollar funcionalidad
3. Abrir Pull Request
4. Revisar y mergear a `main`

#### 3. **GitLab Flow**

Similar a GitHub Flow pero con ambientes específicos:

```
main         ─────●─────●─────●─────
                   │     │     │
pre-production ────●─────●─────●─────
                         │     │
production   ─────────────●─────●───
```

### Comandos Básicos de Ramas

```bash
# Crear y cambiar a una nueva rama
git checkout -b feature/nueva-funcionalidad

# Cambiar entre ramas
git checkout main
git checkout feature/nueva-funcionalidad

# Listar todas las ramas
git branch -a

# Subir rama al repositorio remoto
git push -u origin feature/nueva-funcionalidad

# Eliminar rama local
git branch -d feature/nueva-funcionalidad

# Eliminar rama remota
git push origin --delete feature/nueva-funcionalidad
```

## Merges y Estrategias de Integración

### Tipos de Merge

#### 1. **Merge Commit**

Crea un commit de merge que preserva la historia:

```bash
git checkout main
git merge feature/nueva-funcionalidad
```

#### 2. **Fast-Forward Merge**

Cuando no hay cambios en la rama de destino:

```bash
git checkout main
git merge --ff-only feature/nueva-funcionalidad
```

#### 3. **Squash Merge**

Combina todos los commits de la rama en uno solo:

```bash
git checkout main
git merge --squash feature/nueva-funcionalidad
git commit -m "Agregar nueva funcionalidad"
```

#### 4. **Rebase**

Reescribe la historia colocando commits encima de la rama de destino:

```bash
git checkout feature/nueva-funcionalidad
git rebase main
```

### Cuándo Usar Cada Estrategia

| Estrategia | Cuándo Usar | Ventajas | Desventajas |
|------------|-------------|----------|-------------|
| **Merge Commit** | Features grandes, preservar contexto | Historia completa | Historia compleja |
| **Squash** | Features pequeñas, historia limpia | Historia lineal | Perdida de commits individuales |
| **Rebase** | Desarrollo personal, historia limpia | Historia lineal | Puede ser confuso |

## Pull Requests / Merge Requests

### ¿Qué son?

Los **Pull Requests** (GitHub) o **Merge Requests** (GitLab) son solicitudes para integrar cambios de una rama a otra, permitiendo revisión de código antes de la integración.

### Proceso Típico

#### 1. **Crear Pull Request**

```bash
# Después de hacer commits en tu feature branch
git push origin feature/nueva-funcionalidad
```

En la interfaz web:

1. Navegar al repositorio
2. Hacer clic en "New Pull Request"
3. Seleccionar branch base y branch con cambios
4. Agregar título y descripción
5. Asignar revisores

#### 2. **Plantilla de Pull Request**

```markdown
## Descripción
Breve descripción de los cambios realizados.

## Tipo de cambio
- [ ] Bug fix
- [ ] Nueva funcionalidad
- [ ] Cambio que rompe compatibilidad
- [ ] Actualización de documentación

## ¿Cómo se ha probado?
Describe las pruebas realizadas.

## Checklist
- [ ] Mi código sigue las guías de estilo del proyecto
- [ ] He realizado una auto-revisión de mi código
- [ ] He comentado mi código en áreas difíciles de entender
- [ ] He realizado pruebas que prueban mi cambio
- [ ] Las pruebas nuevas y existentes pasan localmente
```

### Revisión de Código

#### **Mejores Prácticas para Revisores:**

1. **Ser constructivo:** Proporcionar feedback específico y útil
2. **Enfocarse en el código:** No en la persona
3. **Ser oportuno:** Revisar en tiempo razonable
4. **Usar comentarios in-line:** Para feedback específico

#### **Ejemplo de Comentarios de Revisión:**

```markdown
// ✅ Bueno
"Considera usar un Map en lugar de múltiples if-else para mejorar legibilidad"

// ❌ Malo  
"Este código está mal"

// ✅ Bueno
"¿Podríamos extraer esta lógica a una función separada para mejorar testabilidad?"

// ❌ Malo
"Esto es muy complejo"
```

## Gestión de Conflictos

### ¿Qué son los Conflictos?

Los conflictos ocurren cuando Git no puede fusionar automáticamente los cambios porque dos ramas modificaron las mismas líneas de código.

### Ejemplo de Conflicto

```bash
<<<<<<< HEAD
function calculateTotal(items) {
    return items.reduce((sum, item) => sum + item.price, 0);
}
=======
function calculateTotal(products) {
    return products.reduce((total, product) => total + product.cost, 0);
}
>>>>>>> feature/rename-variables
```

### Resolución de Conflictos

#### 1. **Identificar Conflictos**

```bash
git status
# Muestra archivos con conflictos
```

#### 2. **Resolver Manualmente**

```javascript
// Decisión: mantener ambos cambios combinados
function calculateTotal(items) {
    return items.reduce((sum, item) => sum + item.price, 0);
}
```

#### 3. **Marcar como Resuelto**

```bash
git add archivo-con-conflicto.js
git commit -m "Resolver conflicto en calculateTotal"
```

### Herramientas para Resolución de Conflictos

#### **VS Code**

- Interfaz visual para resolver conflictos
- Botones para aceptar cambios actuales, entrantes o ambos

#### **Herramientas de Merge**

```bash
# Configurar herramienta de merge
git config --global merge.tool vimdiff
git config --global merge.tool meld

# Usar herramienta de merge
git mergetool
```

### Estrategias para Evitar Conflictos

1. **Comunicación del equipo:** Coordinar cambios en archivos compartidos
2. **Ramas de corta duración:** Integrar frecuentemente
3. **Refactoring cuidadoso:** Cambios grandes en ramas separadas
4. **Merge frecuente:** Mantener ramas actualizadas con main

## Flujos de Trabajo en Equipo

### Workflow Gitflow

```bash
# Inicializar gitflow
git flow init

# Crear feature
git flow feature start nueva-funcionalidad

# Finalizar feature
git flow feature finish nueva-funcionalidad

# Crear release
git flow release start 1.0.0

# Finalizar release
git flow release finish 1.0.0
```

### Workflow GitHub

```bash
# 1. Clonar repositorio
git clone <url-repositorio>

# 2. Crear rama para feature
git checkout -b feature/mi-feature

# 3. Realizar cambios y commits
git add .
git commit -m "Implementar mi feature"

# 4. Pushear rama
git push origin feature/mi-feature

# 5. Crear Pull Request en GitHub
# 6. Revisar código
# 7. Mergear PR
# 8. Eliminar rama feature
```

## Issues y Project Management

### GitHub Issues

#### **Crear Issue Efectivo:**

```markdown
**Título:** [BUG] El login no funciona con usuarios especiales

**Descripción:**
### Problema
El sistema de login falla cuando el usuario contiene caracteres especiales.

### Pasos para Reproducir
1. Ir a la página de login
2. Ingresar usuario: `user@domain.com`
3. Ingresar password válido
4. Hacer clic en "Iniciar Sesión"

### Resultado Esperado
El usuario debería loguearse exitosamente.

### Resultado Actual
Se muestra error "Invalid credentials"

### Información Adicional
- Browser: Chrome 95.0
- OS: Windows 10
- Versión de la app: 2.1.0
```

### Labels y Milestones

#### **Etiquetas Comunes:**

- `bug`: Errores en el código
- `enhancement`: Mejoras o nuevas funcionalidades
- `documentation`: Cambios en documentación
- `good first issue`: Ideal para nuevos contribuidores
- `priority:high`: Alta prioridad

### GitHub Projects / GitLab Boards

**Columnas típicas:**

- **Backlog:** Issues sin asignar
- **To Do:** Issues asignados pero no iniciados
- **In Progress:** En desarrollo activo
- **Review:** En revisión de código
- **Done:** Completados

## CI/CD Básico

### GitHub Actions

#### **Ejemplo básico (.github/workflows/ci.yml):**

```yaml
name: CI

on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]

jobs:
  test:
    runs-on: ubuntu-latest
    
    steps:
    - uses: actions/checkout@v3
    
    - name: Setup Node.js
      uses: actions/setup-node@v3
      with:
        node-version: '18'
        
    - name: Install dependencies
      run: npm install
      
    - name: Run tests
      run: npm test
      
    - name: Run linter
      run: npm run lint
```

### GitLab CI/CD

#### **Ejemplo básico (.gitlab-ci.yml):**

```yaml
stages:
  - test
  - build
  - deploy

test:
  stage: test
  script:
    - npm install
    - npm test
  only:
    - merge_requests
    - main

build:
  stage: build
  script:
    - npm run build
  artifacts:
    paths:
      - dist/
  only:
    - main
```

## Mejores Prácticas

### Commits

1. **Mensajes descriptivos:**

   ```bash
   # ✅ Bueno
   git commit -m "feat: agregar validación de email en formulario de registro"
   
   # ❌ Malo
   git commit -m "fix stuff"
   ```

2. **Conventional Commits:**

   ```
   feat: nueva funcionalidad
   fix: corrección de bug
   docs: cambios en documentación
   style: formateo, punto y coma faltante, etc.
   refactor: refactoring de código
   test: agregando pruebas faltantes
   chore: mantenimiento
   ```

### Pull Requests

1. **Tamaño manejable:** Máximo 400 líneas de cambio
2. **Un objetivo:** Una funcionalidad o fix por PR
3. **Tests incluidos:** Siempre agregar tests relevantes
4. **Documentación actualizada:** Si aplica

### Branches

1. **Nombres descriptivos:**

   ```bash
   feature/user-authentication
   bugfix/login-validation
   hotfix/security-patch
   ```

2. **Vida corta:** Integrar frecuentemente
3. **Actualizar regularmente:** Hacer rebase/merge de main

## Herramientas y Recursos

### Git GUI Tools

- **SourceTree:** Cliente visual gratuito
- **GitKraken:** Cliente visual con funcionalidades avanzadas
- **VS Code:** Integración Git nativa

### Extensiones útiles

- **GitLens (VS Code):** Información detallada de Git
- **Git Graph (VS Code):** Visualización de ramas
- **GitHub Pull Requests (VS Code):** Gestión de PRs

## Ejercicios Prácticos

### Ejercicio 1: Flujo Básico

1. Crear repositorio en GitHub
2. Clonar localmente
3. Crear rama `feature/readme-update`
4. Modificar README.md
5. Crear Pull Request
6. Mergear PR

### Ejercicio 2: Resolver Conflictos

1. Crear dos ramas que modifiquen el mismo archivo
2. Intentar mergear
3. Resolver conflictos manualmente
4. Completar merge

### Ejercicio 3: Workflow de Equipo

1. Simular trabajo en equipo con múltiples contribuidores
2. Usar GitHub Issues para tracking
3. Implementar GitHub Actions para CI
4. Practicar revisión de código

## Recursos Adicionales

### Documentación Oficial

- [GitHub Docs](https://docs.github.com/)
- [GitLab Docs](https://docs.gitlab.com/)
- [Git Documentation](https://git-scm.com/doc)

### Tutoriales Interactivos

- [GitHub Learning Lab](https://lab.github.com/)
- [Atlassian Git Tutorials](https://www.atlassian.com/git/tutorials)

### Cheat Sheets

- [GitHub Git Cheat Sheet](https://education.github.com/git-cheat-sheet-education.pdf)
- [GitLab Git Cheat Sheet](https://about.gitlab.com/images/press/git-cheat-sheet.pdf)

---

## Próximos Pasos

Una vez que domines la colaboración con Git, GitHub/GitLab, estarás listo para:

1. **Profundizar en CI/CD:** Pipelines más complejos
2. **DevOps avanzado:** Integración con herramientas de deployment
3. **Gestión de proyectos:** Metodologías ágiles con Git
4. **Seguridad:** Análisis de código y secret management

¡La colaboración efectiva con Git es fundamental para cualquier desarrollador y especialmente crucial en DevOps!
