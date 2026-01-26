# 04. Plataformas CI/CD Modernas

**Domina GitHub Actions, GitLab CI/CD y Azure DevOps!**Este módulo te enseñará a usar las plataformas CI/CD más populares del mercado actual.

## Objetivos de Aprendizaje

Al completar este módulo serás capaz de:

- **Crear workflows**avanzados en GitHub Actions
- **Implementar pipelines**complejos en GitLab CI/CD
- **Configurar Azure DevOps**para equipos empresariales
- **Comparar plataformas**y elegir la mejor opción
- **Migrar entre plataformas**de CI/CD
- **Implementar GitOps**con estas herramientas
- **Optimizar costos**y performance

## Contenido del Módulo

### 1. GitHub Actions

#### **Workflow Básico**

```yaml
# .github/workflows/ci.yml
name: CI/CD Pipeline

on:
  push:
    branches: [ main, develop ]
  pull_request:
    branches: [ main ]
  schedule:
    - cron: '0 2 * * *'  # Ejecutar diariamente a las 2 AM
  workflow_dispatch:      # Permitir ejecución manual
    inputs:
      environment:
        description: 'Environment to deploy'
        required: true
        default: 'staging'
        type: choice
        options:
          - staging
          - production

env:
  NODE_VERSION: '18'
  REGISTRY: ghcr.io
  IMAGE_NAME: ${{ github.repository }}

jobs:
  test:
    name:  Test
    runs-on: ubuntu-latest

    strategy:
      matrix:
        node-version: [16, 18, 20]
        os: [ubuntu-latest, windows-latest, macos-latest]
        include:
          - node-version: 18
            os: ubuntu-latest
            coverage: true
        exclude:
          - node-version: 16
            os: macos-latest

    steps:
      - name:  Checkout code
        uses: actions/checkout@v4
        with:
          fetch-depth: 0  # Para SonarCloud

      - name:  Setup Node.js ${{ matrix.node-version }}
        uses: actions/setup-node@v4
        with:
          node-version: ${{ matrix.node-version }}
          cache: 'npm'

      - name:  Install dependencies
        run: npm ci

      - name:  Lint code
        run: npm run lint

      - name:  Run tests
        run: npm test
        env:
          CI: true

      - name:  Generate coverage report
        if: matrix.coverage
        run: npm run test:coverage

      - name:  Upload coverage to Codecov
        if: matrix.coverage
        uses: codecov/codecov-action@v3
        with:
          token: ${{ secrets.CODECOV_TOKEN }}
          files: ./coverage/lcov.info
          flags: unittests
          name: codecov-umbrella

      - name:  Run security audit
        run: npm audit --audit-level moderate

      - name:  SonarCloud Scan
        if: matrix.coverage
        uses: SonarSource/sonarcloud-github-action@master
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
          SONAR_TOKEN: ${{ secrets.SONAR_TOKEN }}

  build:
    name:  Build
    runs-on: ubuntu-latest
    needs: test
    outputs:
      image-digest: ${{ steps.build.outputs.digest }}
      image-tags: ${{ steps.meta.outputs.tags }}

    steps:
      - name:  Checkout code
        uses: actions/checkout@v4

      - name:  Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: ${{ env.NODE_VERSION }}
          cache: 'npm'

      - name:  Install dependencies
        run: npm ci

      - name:  Build application
        run: npm run build

      - name:  Upload build artifacts
        uses: actions/upload-artifact@v4
        with:
          name: build-files
          path: dist/
          retention-days: 7

      - name:  Set up Docker Buildx
        uses: docker/setup-buildx-action@v3

      - name:  Login to Container Registry
        uses: docker/login-action@v3
        with:
          registry: ${{ env.REGISTRY }}
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}

      - name:  Extract metadata
        id: meta
        uses: docker/metadata-action@v5
        with:
          images: ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}
          tags: |
            type=ref,event=branch
            type=ref,event=pr
            type=sha,prefix={{branch}}-
            type=raw,value=latest,enable={{is_default_branch}}

      - name:  Build and push Docker image
        id: build
        uses: docker/build-push-action@v5
        with:
          context: .
          platforms: linux/amd64,linux/arm64
          push: true
          tags: ${{ steps.meta.outputs.tags }}
          labels: ${{ steps.meta.outputs.labels }}
          cache-from: type=gha
          cache-to: type=gha,mode=max
          provenance: true
          sbom: true

  security:
    name:  Security Scan
    runs-on: ubuntu-latest
    needs: build
    if: github.event_name != 'pull_request'

    permissions:
      security-events: write

    steps:
      - name:  Checkout code
        uses: actions/checkout@v4

      - name:  Run Trivy vulnerability scanner
        uses: aquasecurity/trivy-action@master
        with:
          image-ref: ${{ needs.build.outputs.image-tags }}
          format: 'sarif'
          output: 'trivy-results.sarif'

      - name:  Upload Trivy scan results
        uses: github/codeql-action/upload-sarif@v3
        with:
          sarif_file: 'trivy-results.sarif'

      - name:  CodeQL Analysis
        uses: github/codeql-action/analyze@v3
        with:
          languages: javascript

  deploy-staging:
    name:  Deploy to Staging
    runs-on: ubuntu-latest
    needs: [build, security]
    if: github.ref == 'refs/heads/develop'
    environment:
      name: staging
      url: https://staging.myapp.com

    steps:
      - name:  Checkout code
        uses: actions/checkout@v4

      - name:  Configure AWS credentials
        uses: aws-actions/configure-aws-credentials@v4
        with:
          aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          aws-region: us-east-1

      - name:  Deploy to ECS
        run: |
          aws ecs update-service \
            --cluster staging-cluster \
            --service myapp-staging \
            --force-new-deployment

      - name: ⏳ Wait for deployment
        run: |
          aws ecs wait services-stable \
            --cluster staging-cluster \
            --services myapp-staging

      - name:  Run smoke tests
        run: |
          curl -f https://staging.myapp.com/health || exit 1
          npm run test:smoke -- --baseUrl=https://staging.myapp.com

  deploy-production:
    name:  Deploy to Production
    runs-on: ubuntu-latest
    needs: [build, security]
    if: github.ref == 'refs/heads/main'
    environment:
      name: production
      url: https://myapp.com

    steps:
      - name:  Checkout code
        uses: actions/checkout@v4

      - name:  Configure AWS credentials
        uses: aws-actions/configure-aws-credentials@v4
        with:
          aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          aws-region: us-east-1

      - name:  Blue-Green Deployment
        run: |
          # Obtener configuración actual
          CURRENT_SERVICE=$(aws elbv2 describe-target-groups \
            --names prod-blue prod-green \
            --query 'TargetGroups[?contains(LoadBalancerArns, `prod-alb`)].TargetGroupName' \
            --output text)

          # Determinar servicio de destino
          if [ "$CURRENT_SERVICE" = "prod-blue" ]; then
            TARGET_SERVICE="prod-green"
            TARGET_TG="prod-green"
          else
            TARGET_SERVICE="prod-blue"
            TARGET_TG="prod-blue"
          fi

          echo "Deploying to $TARGET_SERVICE"

          # Actualizar servicio inactivo
          aws ecs update-service \
            --cluster production-cluster \
            --service $TARGET_SERVICE \
            --force-new-deployment

          # Esperar a que esté estable
          aws ecs wait services-stable \
            --cluster production-cluster \
            --services $TARGET_SERVICE

          # Cambiar tráfico del ALB
          aws elbv2 modify-listener \
            --listener-arn ${{ secrets.PROD_LISTENER_ARN }} \
            --default-actions Type=forward,TargetGroupArn=${{ secrets[TARGET_TG + '_ARN'] }}

          echo "Traffic switched to $TARGET_SERVICE"

      - name:  Post-deployment tests
        run: |
          sleep 30  # Esperar propagación
          curl -f https://myapp.com/health || exit 1
          npm run test:e2e -- --baseUrl=https://myapp.com

      - name:  Notify deployment
        if: always()
        uses: 8398a7/action-slack@v3
        with:
          status: ${{ job.status }}
          channel: '#deployments'
          webhook_url: ${{ secrets.SLACK_WEBHOOK }}
          fields: repo,message,commit,author,action,eventName,ref,workflow

  cleanup:
    name:  Cleanup
    runs-on: ubuntu-latest
    needs: [deploy-staging, deploy-production]
    if: always()

    steps:
      - name:  Delete old artifacts
        uses: actions/github-script@v7
        with:
          script: |
            const artifacts = await github.rest.actions.listWorkflowRunArtifacts({
              owner: context.repo.owner,
              repo: context.repo.repo,
              run_id: context.runId,
            });

            const oldArtifacts = artifacts.data.artifacts.filter(artifact => {
              const ageInDays = (Date.now() - new Date(artifact.created_at)) / (1000 * 60 * 60 * 24);
              return ageInDays > 7;
            });

            for (const artifact of oldArtifacts) {
              await github.rest.actions.deleteArtifact({
                owner: context.repo.owner,
                repo: context.repo.repo,
                artifact_id: artifact.id,
              });
            }
```

#### **Reusable Workflows**

```yaml
# .github/workflows/reusable-build.yml
name: Reusable Build Workflow

on:
  workflow_call:
    inputs:
      node-version:
        required: false
        type: string
        default: '18'
      build-command:
        required: false
        type: string
        default: 'npm run build'
      artifact-name:
        required: false
        type: string
        default: 'build-artifacts'
    outputs:
      artifact-id:
        description: "ID of the uploaded artifact"
        value: ${{ jobs.build.outputs.artifact-id }}
    secrets:
      SONAR_TOKEN:
        required: false

jobs:
  build:
    runs-on: ubuntu-latest
    outputs:
      artifact-id: ${{ steps.upload.outputs.artifact-id }}

    steps:
      - uses: actions/checkout@v4

      - uses: actions/setup-node@v4
        with:
          node-version: ${{ inputs.node-version }}
          cache: 'npm'

      - run: npm ci

      - run: ${{ inputs.build-command }}

      - name: Upload artifacts
        id: upload
        uses: actions/upload-artifact@v4
        with:
          name: ${{ inputs.artifact-name }}
          path: dist/
```

#### **Custom Actions**

```yaml
# .github/actions/deploy-to-k8s/action.yml
name: 'Deploy to Kubernetes'
description: 'Deploy application to Kubernetes cluster'

inputs:
  cluster-name:
    description: 'Kubernetes cluster name'
    required: true
  namespace:
    description: 'Kubernetes namespace'
    required: true
    default: 'default'
  image-tag:
    description: 'Docker image tag'
    required: true
  kubeconfig:
    description: 'Kubeconfig content'
    required: true

outputs:
  deployment-url:
    description: 'URL of the deployed application'
    value: ${{ steps.deploy.outputs.url }}

runs:
  using: 'composite'
  steps:
    - name: Setup kubectl
      uses: azure/setup-kubectl@v3
      with:
        version: 'v1.28.0'

    - name: Configure kubeconfig
      shell: bash
      run: |
        echo "${{ inputs.kubeconfig }}" | base64 -d > kubeconfig
        export KUBECONFIG=kubeconfig

    - name: Deploy to Kubernetes
      id: deploy
      shell: bash
      run: |
        kubectl set image deployment/myapp \
          myapp=${{ inputs.image-tag }} \
          -n ${{ inputs.namespace }}

        kubectl rollout status deployment/myapp \
          -n ${{ inputs.namespace }} \
          --timeout=300s

        URL=$(kubectl get ingress myapp \
          -n ${{ inputs.namespace }} \
          -o jsonpath='{.spec.rules[0].host}')

        echo "url=https://$URL" >> $GITHUB_OUTPUT
```

### 2. GitLab CI/CD

#### **Pipeline Completo**

```yaml
# .gitlab-ci.yml
stages:
  - validate
  - build
  - test
  - security
  - package
  - deploy
  - monitor

variables:
  DOCKER_DRIVER: overlay2
  DOCKER_TLS_CERTDIR: "/certs"
  IMAGE_TAG: $CI_REGISTRY_IMAGE:$CI_COMMIT_SHORT_SHA
  KUBECONFIG: /etc/deploy/config

# Templates para reutilización
.docker_template: &docker_template
  image: docker:latest
  services:
    - docker:dind
  before_script:
    - echo $CI_REGISTRY_PASSWORD | docker login -u $CI_REGISTRY_USER --password-stdin $CI_REGISTRY

.kubectl_template: &kubectl_template
  image: bitnami/kubectl:latest
  before_script:
    - mkdir -p /etc/deploy
    - echo $KUBE_CONFIG | base64 -d > /etc/deploy/config

.node_template: &node_template
  image: node:18-alpine
  cache:
    key: ${CI_COMMIT_REF_SLUG}
    paths:
      - node_modules/
      - .npm/
  before_script:
    - npm ci --cache .npm --prefer-offline

# Validation Stage
lint:
  stage: validate
  <<: *node_template
  script:
    - npm run lint
    - npm run type-check
  rules:
    - if: $CI_PIPELINE_SOURCE == "merge_request_event"
    - if: $CI_COMMIT_BRANCH == $CI_DEFAULT_BRANCH

dependency-check:
  stage: validate
  <<: *node_template
  script:
    - npm audit --audit-level moderate
    - npx license-checker --summary
  artifacts:
    reports:
      dependency_scanning: gl-dependency-scanning-report.json
  rules:
    - if: $CI_PIPELINE_SOURCE == "merge_request_event"
    - if: $CI_COMMIT_BRANCH == $CI_DEFAULT_BRANCH

# Build Stage
build:
  stage: build
  <<: *node_template
  script:
    - npm run build
  artifacts:
    name: "build-$CI_COMMIT_SHORT_SHA"
    paths:
      - dist/
    expire_in: 1 week
  rules:
    - if: $CI_PIPELINE_SOURCE == "merge_request_event"
    - if: $CI_COMMIT_BRANCH == $CI_DEFAULT_BRANCH

# Test Stage
unit-tests:
  stage: test
  <<: *node_template
  script:
    - npm run test:unit -- --coverage
  coverage: '/All files[^|]*\|[^|]*\s+([\d\.]+)/'
  artifacts:
    when: always
    reports:
      junit: reports/junit.xml
      coverage_report:
        coverage_format: cobertura
        path: coverage/cobertura-coverage.xml
    paths:
      - coverage/
    expire_in: 1 week
  rules:
    - if: $CI_PIPELINE_SOURCE == "merge_request_event"
    - if: $CI_COMMIT_BRANCH == $CI_DEFAULT_BRANCH

integration-tests:
  stage: test
  <<: *node_template
  services:
    - postgres:13
    - redis:6-alpine
  variables:
    POSTGRES_DB: testdb
    POSTGRES_USER: testuser
    POSTGRES_PASSWORD: testpass
    DATABASE_URL: postgres://testuser:testpass@postgres:5432/testdb
    REDIS_URL: redis://redis:6379
  script:
    - npm run test:integration
  artifacts:
    when: always
    reports:
      junit: reports/integration-junit.xml
  rules:
    - if: $CI_COMMIT_BRANCH == $CI_DEFAULT_BRANCH
    - if: $CI_PIPELINE_SOURCE == "merge_request_event"
      when: manual

e2e-tests:
  stage: test
  image: mcr.microsoft.com/playwright:latest
  dependencies:
    - build
  variables:
    BASE_URL: http://localhost:3000
  script:
    - npm ci
    - npm start &
    - sleep 30
    - npx playwright test
  artifacts:
    when: always
    paths:
      - playwright-report/
      - test-results/
    expire_in: 1 week
  rules:
    - if: $CI_COMMIT_BRANCH == $CI_DEFAULT_BRANCH
    - if: $CI_PIPELINE_SOURCE == "merge_request_event"
      when: manual

# Security Stage
sast:
  stage: security
  include:
    - template: Security/SAST.gitlab-ci.yml
  rules:
    - if: $CI_COMMIT_BRANCH == $CI_DEFAULT_BRANCH
    - if: $CI_PIPELINE_SOURCE == "merge_request_event"

secret-detection:
  stage: security
  include:
    - template: Security/Secret-Detection.gitlab-ci.yml
  rules:
    - if: $CI_COMMIT_BRANCH == $CI_DEFAULT_BRANCH
    - if: $CI_PIPELINE_SOURCE == "merge_request_event"

# Package Stage
docker-build:
  stage: package
  <<: *docker_template
  script:
    - docker build -t $IMAGE_TAG .
    - docker push $IMAGE_TAG
    - |
      if [ "$CI_COMMIT_BRANCH" == "$CI_DEFAULT_BRANCH" ]; then
        docker tag $IMAGE_TAG $CI_REGISTRY_IMAGE:latest
        docker push $CI_REGISTRY_IMAGE:latest
      fi
  rules:
    - if: $CI_COMMIT_BRANCH == $CI_DEFAULT_BRANCH
    - if: $CI_PIPELINE_SOURCE == "merge_request_event"

container-scanning:
  stage: package
  include:
    - template: Security/Container-Scanning.gitlab-ci.yml
  variables:
    CS_IMAGE: $IMAGE_TAG
  dependencies:
    - docker-build
  rules:
    - if: $CI_COMMIT_BRANCH == $CI_DEFAULT_BRANCH
    - if: $CI_PIPELINE_SOURCE == "merge_request_event"

# Deploy Stages
.deploy_template: &deploy_template
  <<: *kubectl_template
  script:
    - |
      # Crear namespace si no existe
      kubectl create namespace $NAMESPACE --dry-run=client -o yaml | kubectl apply -f -

      # Aplicar configuraciones
      envsubst < k8s/deployment.yaml | kubectl apply -f -
      envsubst < k8s/service.yaml | kubectl apply -f -
      envsubst < k8s/ingress.yaml | kubectl apply -f -

      # Esperar deployment
      kubectl rollout status deployment/$APP_NAME -n $NAMESPACE --timeout=300s

      # Verificar health
      kubectl wait --for=condition=ready pod -l app=$APP_NAME -n $NAMESPACE --timeout=300s

      # Obtener URL
      URL=$(kubectl get ingress $APP_NAME -n $NAMESPACE -o jsonpath='{.spec.rules[0].host}')
      echo "Application deployed at: https://$URL"

deploy-staging:
  stage: deploy
  <<: *deploy_template
  environment:
    name: staging
    url: https://staging.myapp.com
    on_stop: stop-staging
  variables:
    NAMESPACE: staging
    APP_NAME: myapp-staging
    REPLICAS: 2
  rules:
    - if: $CI_COMMIT_BRANCH == "develop"

deploy-production:
  stage: deploy
  <<: *deploy_template
  environment:
    name: production
    url: https://myapp.com
  variables:
    NAMESPACE: production
    APP_NAME: myapp-production
    REPLICAS: 5
  when: manual
  rules:
    - if: $CI_COMMIT_BRANCH == $CI_DEFAULT_BRANCH

stop-staging:
  stage: deploy
  <<: *kubectl_template
  script:
    - kubectl delete namespace staging
  environment:
    name: staging
    action: stop
  when: manual
  rules:
    - if: $CI_COMMIT_BRANCH == "develop"

# Canary Deployment
deploy-canary:
  stage: deploy
  <<: *kubectl_template
  environment:
    name: production-canary
    url: https://canary.myapp.com
  variables:
    NAMESPACE: production
    APP_NAME: myapp-canary
    REPLICAS: 1
  script:
    - |
      # Deploy canary version
      envsubst < k8s/canary-deployment.yaml | kubectl apply -f -
      kubectl rollout status deployment/$APP_NAME -n $NAMESPACE --timeout=300s

      # Update Istio VirtualService for traffic splitting
      envsubst < k8s/canary-virtual-service.yaml | kubectl apply -f -

      echo "Canary deployment ready with 10% traffic"
  when: manual
  rules:
    - if: $CI_COMMIT_BRANCH == $CI_DEFAULT_BRANCH

promote-canary:
  stage: deploy
  <<: *kubectl_template
  script:
    - |
      # Switch 100% traffic to canary
      kubectl patch deployment myapp-production -n production \
        -p '{"spec":{"template":{"spec":{"containers":[{"name":"myapp","image":"'$IMAGE_TAG'"}]}}}}'

      # Wait for rollout
      kubectl rollout status deployment/myapp-production -n production --timeout=300s

      # Remove canary deployment
      kubectl delete deployment myapp-canary -n production

      echo "Canary promoted to production"
  environment:
    name: production
    url: https://myapp.com
  when: manual
  rules:
    - if: $CI_COMMIT_BRANCH == $CI_DEFAULT_BRANCH

# Monitor Stage
performance-test:
  stage: monitor
  image: loadimpact/k6:latest
  script:
    - k6 run --out json=performance-results.json tests/performance/load-test.js
  artifacts:
    reports:
      performance: performance-results.json
    paths:
      - performance-results.json
    expire_in: 1 week
  rules:
    - if: $CI_COMMIT_BRANCH == $CI_DEFAULT_BRANCH
      when: manual

notify-success:
  stage: monitor
  image: alpine:latest
  before_script:
    - apk add --no-cache curl
  script:
    - |
      curl -X POST $SLACK_WEBHOOK \
        -H 'Content-type: application/json' \
        --data "{
          \"text\": \" Deployment successful: $CI_PROJECT_NAME\",
          \"attachments\": [{
            \"color\": \"good\",
            \"fields\": [
              {\"title\": \"Project\", \"value\": \"$CI_PROJECT_NAME\", \"short\": true},
              {\"title\": \"Branch\", \"value\": \"$CI_COMMIT_BRANCH\", \"short\": true},
              {\"title\": \"Commit\", \"value\": \"$CI_COMMIT_SHORT_SHA\", \"short\": true},
              {\"title\": \"Pipeline\", \"value\": \"$CI_PIPELINE_URL\", \"short\": true}
            ]
          }]
        }"
  rules:
    - if: $CI_COMMIT_BRANCH == $CI_DEFAULT_BRANCH
      when: on_success

notify-failure:
  stage: monitor
  image: alpine:latest
  before_script:
    - apk add --no-cache curl
  script:
    - |
      curl -X POST $SLACK_WEBHOOK \
        -H 'Content-type: application/json' \
        --data "{
          \"text\": \" Deployment failed: $CI_PROJECT_NAME\",
          \"attachments\": [{
            \"color\": \"danger\",
            \"fields\": [
              {\"title\": \"Project\", \"value\": \"$CI_PROJECT_NAME\", \"short\": true},
              {\"title\": \"Branch\", \"value\": \"$CI_COMMIT_BRANCH\", \"short\": true},
              {\"title\": \"Commit\", \"value\": \"$CI_COMMIT_SHORT_SHA\", \"short\": true},
              {\"title\": \"Pipeline\", \"value\": \"$CI_PIPELINE_URL\", \"short\": true}
            ]
          }]
        }"
  rules:
    - if: $CI_COMMIT_BRANCH == $CI_DEFAULT_BRANCH
      when: on_failure
```

### 3. Azure DevOps

#### **Pipeline YAML**

```yaml
# azure-pipelines.yml
trigger:
  branches:
    include:
      - main
      - develop
  paths:
    include:
      - src/*
      - tests/*
    exclude:
      - docs/*
      - README.md

pr:
  branches:
    include:
      - main
      - develop
  drafts: false

schedules:
  - cron: "0 2 * * *"
    displayName: Nightly build
    branches:
      include:
        - main
    always: true

variables:
  - group: common-variables
  - name: buildConfiguration
    value: 'Release'
  - name: vmImageName
    value: 'ubuntu-latest'
  - name: dockerRegistryServiceConnection
    value: 'acr-connection'
  - name: imageRepository
    value: 'myapp'
  - name: containerRegistry
    value: 'myregistry.azurecr.io'
  - name: dockerfilePath
    value: '$(Build.SourcesDirectory)/Dockerfile'
  - name: tag
    value: '$(Build.BuildId)'

resources:
  repositories:
    - repository: templates
      type: git
      name: DevOps/pipeline-templates
      ref: refs/heads/main

stages:
  - stage: Validate
    displayName: 'Validate Code'
    jobs:
      - job: Lint
        displayName: 'Code Linting'
        pool:
          vmImage: $(vmImageName)
        steps:
          - task: NodeTool@0
            inputs:
              versionSpec: '18.x'
            displayName: 'Install Node.js'

          - task: Cache@2
            inputs:
              key: 'npm | "$(Agent.OS)" | package-lock.json'
              restoreKeys: |
                npm | "$(Agent.OS)"
              path: $(npm_config_cache)
            displayName: 'Cache npm'

          - script: |
              npm ci
              npm run lint
              npm run type-check
            displayName: 'npm install and lint'

          - task: PublishTestResults@2
            condition: succeededOrFailed()
            inputs:
              testResultsFormat: 'JUnit'
              testResultsFiles: 'lint-results.xml'
              testRunTitle: 'Lint Results'

  - stage: Build
    displayName: 'Build Application'
    dependsOn: Validate
    jobs:
      - job: Build
        displayName: 'Build'
        pool:
          vmImage: $(vmImageName)
        steps:
          - task: NodeTool@0
            inputs:
              versionSpec: '18.x'
            displayName: 'Install Node.js'

          - task: Cache@2
            inputs:
              key: 'npm | "$(Agent.OS)" | package-lock.json'
              restoreKeys: |
                npm | "$(Agent.OS)"
              path: $(npm_config_cache)
            displayName: 'Cache npm'

          - script: |
              npm ci
              npm run build
            displayName: 'npm install and build'

          - task: PublishBuildArtifacts@1
            inputs:
              PathtoPublish: 'dist'
              ArtifactName: 'build-artifact'
              publishLocation: 'Container'

  - stage: Test
    displayName: 'Test Application'
    dependsOn: Build
    jobs:
      - job: UnitTests
        displayName: 'Unit Tests'
        pool:
          vmImage: $(vmImageName)
        steps:
          - template: templates/node-test.yml
            parameters:
              testCommand: 'npm run test:unit'
              testResultsFiles: 'test-results.xml'
              codeCoverageFiles: 'coverage/cobertura-coverage.xml'

      - job: IntegrationTests
        displayName: 'Integration Tests'
        pool:
          vmImage: $(vmImageName)
        services:
          postgres: postgres:13
          redis: redis:6
        variables:
          DATABASE_URL: 'postgres://postgres:postgres@localhost:5432/testdb'
          REDIS_URL: 'redis://localhost:6379'
        steps:
          - template: templates/node-test.yml
            parameters:
              testCommand: 'npm run test:integration'
              testResultsFiles: 'integration-results.xml'

      - job: SecurityScan
        displayName: 'Security Scanning'
        pool:
          vmImage: $(vmImageName)
        steps:
          - task: NodeTool@0
            inputs:
              versionSpec: '18.x'

          - script: |
              npm ci
              npm audit --audit-level moderate
            displayName: 'Security audit'

          - task: SonarCloudPrepare@1
            inputs:
              SonarCloud: 'sonarcloud-connection'
              organization: 'myorganization'
              scannerMode: 'CLI'
              configMode: 'manual'
              cliProjectKey: 'myproject'
              cliProjectName: 'My Project'
              cliSources: 'src'

          - task: SonarCloudAnalyze@1

          - task: SonarCloudPublish@1
            inputs:
              pollingTimeoutSec: '300'

  - stage: Package
    displayName: 'Package Application'
    dependsOn: Test
    condition: and(succeeded(), ne(variables['Build.Reason'], 'PullRequest'))
    jobs:
      - job: Docker
        displayName: 'Build Docker Image'
        pool:
          vmImage: $(vmImageName)
        steps:
          - task: Docker@2
            displayName: 'Build and push image'
            inputs:
              command: 'buildAndPush'
              repository: $(imageRepository)
              dockerfile: $(dockerfilePath)
              containerRegistry: $(dockerRegistryServiceConnection)
              tags: |
                $(tag)
                latest

          - task: AzureContainerRegistry@2
            displayName: 'Scan image for vulnerabilities'
            inputs:
              azureSubscription: 'azure-subscription'
              azureContainerRegistry: $(containerRegistry)
              repository: $(imageRepository)
              tag: $(tag)
              scanType: 'vulnerability'

  - stage: DeployStaging
    displayName: 'Deploy to Staging'
    dependsOn: Package
    condition: and(succeeded(), eq(variables['Build.SourceBranch'], 'refs/heads/develop'))
    variables:
      - group: staging-variables
    jobs:
      - deployment: DeployToStaging
        displayName: 'Deploy to Staging Environment'
        pool:
          vmImage: $(vmImageName)
        environment: 'staging'
        strategy:
          runOnce:
            deploy:
              steps:
                - template: templates/kubernetes-deploy.yml
                  parameters:
                    environment: 'staging'
                    namespace: 'staging'
                    imageTag: $(tag)
                    replicas: 2
                    kubernetesServiceConnection: 'k8s-staging'

                - task: AzureCLI@2
                  displayName: 'Health Check'
                  inputs:
                    azureSubscription: 'azure-subscription'
                    scriptType: 'bash'
                    scriptLocation: 'inlineScript'
                    inlineScript: |
                      echo "Waiting for application to be ready..."
                      for i in {1..30}; do
                        if curl -f https://staging.myapp.com/health; then
                          echo "Application is healthy"
                          exit 0
                        fi
                        sleep 10
                      done
                      echo "Health check failed"
                      exit 1

  - stage: DeployProduction
    displayName: 'Deploy to Production'
    dependsOn: Package
    condition: and(succeeded(), eq(variables['Build.SourceBranch'], 'refs/heads/main'))
    variables:
      - group: production-variables
    jobs:
      - deployment: DeployToProduction
        displayName: 'Deploy to Production Environment'
        pool:
          vmImage: $(vmImageName)
        environment: 'production'
        strategy:
          canary:
            increments: [10, 25, 50, 100]
            deploy:
              steps:
                - template: templates/kubernetes-deploy.yml
                  parameters:
                    environment: 'production'
                    namespace: 'production'
                    imageTag: $(tag)
                    replicas: 5
                    kubernetesServiceConnection: 'k8s-production'
                    deploymentStrategy: 'canary'

            on:
              success:
                steps:
                  - task: AzureCLI@2
                    displayName: 'Post-deployment Tests'
                    inputs:
                      azureSubscription: 'azure-subscription'
                      scriptType: 'bash'
                      scriptLocation: 'inlineScript'
                      inlineScript: |
                        # Run smoke tests
                        npm run test:smoke -- --baseUrl=https://myapp.com

                        # Performance test
                        curl -f https://myapp.com/metrics | grep response_time

              failure:
                steps:
                  - task: AzureCLI@2
                    displayName: 'Rollback Deployment'
                    inputs:
                      azureSubscription: 'azure-subscription'
                      scriptType: 'bash'
                      scriptLocation: 'inlineScript'
                      inlineScript: |
                        kubectl rollout undo deployment/myapp -n production
                        echo "Deployment rolled back due to failure"

  - stage: Monitor
    displayName: 'Post-Deployment Monitoring'
    dependsOn:
      - DeployStaging
      - DeployProduction
    condition: or(succeeded('DeployStaging'), succeeded('DeployProduction'))
    jobs:
      - job: MonitorApplication
        displayName: 'Monitor Application Health'
        pool:
          vmImage: $(vmImageName)
        steps:
          - task: AzureCLI@2
            displayName: 'Application Insights Check'
            inputs:
              azureSubscription: 'azure-subscription'
              scriptType: 'bash'
              scriptLocation: 'inlineScript'
              inlineScript: |
                # Query Application Insights for errors
                az monitor app-insights query \
                  --app myapp-insights \
                  --analytics-query "exceptions | where timestamp > ago(10m) | count"

          - task: InvokeRESTAPI@1
            displayName: 'Notify Slack'
            inputs:
              connectionType: 'connectedServiceName'
              serviceConnection: 'slack-webhook'
              method: 'POST'
              body: |
                {
                  "text": " Deployment completed successfully",
                  "channel": "#deployments",
                  "attachments": [{
                    "color": "good",
                    "fields": [
                      {"title": "Project", "value": "$(Build.Repository.Name)", "short": true},
                      {"title": "Build", "value": "$(Build.BuildNumber)", "short": true},
                      {"title": "Branch", "value": "$(Build.SourceBranchName)", "short": true},
                      {"title": "Commit", "value": "$(Build.SourceVersion)", "short": true}
                    ]
                  }]
                }
```

## Comparación de Plataformas

| Característica | GitHub Actions | GitLab CI/CD | Azure DevOps |
|----------------|----------------|--------------|--------------|
| **Hosting**| SaaS/Self-hosted | SaaS/Self-hosted | SaaS/Self-hosted |
| **Precio**| Gratis (2000min/mes) | Gratis (400min/mes) | Gratis (1800min/mes) |
| **Runners**| GitHub/Self-hosted | GitLab/Self-hosted | Microsoft/Self-hosted |
| **Workflows**| YAML | YAML | YAML/Visual |
| **Marketplace**|  Extenso |  Moderado |  Extenso |
| **Multi-repo**|  |  |  |
| **Artifacts**|  |  |  |
| **Environments**|  |  |  |
| **Security**|  Excellent |  Excellent |  Excellent |
| **Enterprise**|  |  |  |

## Laboratorios Prácticos

### Lab 1: Migración entre Plataformas

**Objetivo**: Migrar un pipeline de Jenkins a GitHub Actions.

**Jenkins Pipeline Original**:

```groovy
pipeline {
    agent any
    stages {
        stage('Build') {
            steps {
                sh 'npm install'
                sh 'npm run build'
            }
        }
        stage('Test') {
            steps {
                sh 'npm test'
            }
        }
        stage('Deploy') {
            steps {
                sh 'npm run deploy'
            }
        }
    }
}
```

**GitHub Actions Equivalente**:

```yaml
name: CI/CD Pipeline
on: [push]
jobs:
  build-test-deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: '18'
      - run: npm install
      - run: npm run build
      - run: npm test
      - run: npm run deploy
```

### Lab 2: GitOps con ArgoCD

**Objetivo**: Implementar GitOps usando GitHub Actions y ArgoCD.

```yaml
# .github/workflows/gitops.yml
name: GitOps Deployment
on:
  push:
    branches: [main]

jobs:
  update-manifest:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Update Kubernetes manifests
        run: |
          # Build image
          docker build -t myapp:${{ github.sha }} .

          # Update manifest
          sed -i 's|image: myapp:.*|image: myapp:${{ github.sha }}|' k8s/deployment.yaml

          # Commit changes to GitOps repo
          git config user.name "GitHub Actions"
          git config user.email "actions@github.com"
          git add k8s/deployment.yaml
          git commit -m "Update image to ${{ github.sha }}"
          git push
```

## Ejercicios Prácticos

### Ejercicio 1: Pipeline Multi-Cloud

Crear un pipeline que despliegue simultáneamente a:

- AWS ECS
- Azure Container Instances
- Google Cloud Run

### Ejercicio 2: Feature Flags Pipeline

Implementar un pipeline que:

- Detecte feature flags en el código
- Despliegue diferentes versiones según flags
- Permita rollback granular por feature

### Ejercicio 3: Cost Optimization

Desarrollar estrategias para:

- Optimizar uso de runners
- Reducir tiempo de pipeline
- Implementar caching inteligente
- Usar spot instances para builds

## Troubleshooting

### Problemas Comunes

1. **Rate Limiting**:

```bash
# GitHub Actions
# Usar GITHUB_TOKEN para requests API
curl -H "Authorization: token ${{ secrets.GITHUB_TOKEN }}" \
     https://api.github.com/repos/owner/repo
```

2. **Runner Capacity**:

```yaml
# Usar matrix builds para paralelización
strategy:
  matrix:
    runner: [ubuntu-latest, windows-latest, macos-latest]
```

3. **Secret Management**:

```bash
# Rotar secrets regularmente
gh secret set MY_SECRET --body "new-value"
```

## Recursos Adicionales

- [GitHub Actions Documentation](https://docs.github.com/actions)
- [GitLab CI/CD Documentation](https://docs.gitlab.com/ee/ci/)
- [Azure DevOps Documentation](https://docs.microsoft.com/azure/devops/)
- [CI/CD Best Practices](https://docs.microsoft.com/devops/deliver/what-is-continuous-integration)

## Checklist de Completado

- [ ] Configurado GitHub Actions workflow
- [ ] Implementado GitLab CI/CD pipeline
- [ ] Creado Azure DevOps pipeline
- [ ] Configurado environments y approvals
- [ ] Implementado security scanning
- [ ] Configurado notifications
- [ ] Optimizado performance y costos
- [ ] Implementado GitOps workflow
- [ ] Configurado monitoring y alerts
- [ ] Documentado proceso de migración

---

**Excelente!**Ahora dominas las principales plataformas CI/CD modernas. Continúa con el siguiente módulo para aprender sobre testing automation y quality gates.
