# 05. Testing Automation y Quality Gates

**Automatiza testing y asegura calidad de código!** Este módulo te enseñará a implementar testing automation completo y quality gates efectivos en tus pipelines CI/CD.

##  Objetivos de Aprendizaje

Al completar este módulo serás capaz de:

- **Implementar testing automation** en múltiples niveles
- **Configurar quality gates** efectivos
- **Integrar herramientas** de análisis de código
- **Automatizar security testing** en pipelines
- **Crear test reports** y métricas
- **Implementar performance testing** automático
- **Gestionar test data** y ambientes

##  Contenido del Módulo

### 1. Pirámide de Testing Automation

####  **Unit Testing**

```javascript
// tests/unit/calculator.test.js
import { describe, it, expect, beforeEach, afterEach } from 'vitest'
import { Calculator } from '../../src/calculator.js'

describe('Calculator', () => {
  let calculator

  beforeEach(() => {
    calculator = new Calculator()
  })

  afterEach(() => {
    calculator = null
  })

  describe('add method', () => {
    it('should add two positive numbers correctly', () => {
      // Arrange
      const a = 5
      const b = 3
      const expected = 8

      // Act
      const result = calculator.add(a, b)

      // Assert
      expect(result).toBe(expected)
    })

    it('should handle negative numbers', () => {
      expect(calculator.add(-5, 3)).toBe(-2)
      expect(calculator.add(-5, -3)).toBe(-8)
    })

    it('should handle zero values', () => {
      expect(calculator.add(0, 5)).toBe(5)
      expect(calculator.add(5, 0)).toBe(5)
      expect(calculator.add(0, 0)).toBe(0)
    })

    it('should handle decimal numbers', () => {
      expect(calculator.add(1.5, 2.3)).toBeCloseTo(3.8)
    })

    it('should throw error for invalid inputs', () => {
      expect(() => calculator.add('a', 5)).toThrow('Invalid input')
      expect(() => calculator.add(null, 5)).toThrow('Invalid input')
      expect(() => calculator.add(undefined, 5)).toThrow('Invalid input')
    })
  })

  describe('divide method', () => {
    it('should divide numbers correctly', () => {
      expect(calculator.divide(10, 2)).toBe(5)
      expect(calculator.divide(7, 2)).toBe(3.5)
    })

    it('should throw error when dividing by zero', () => {
      expect(() => calculator.divide(5, 0)).toThrow('Division by zero')
    })
  })
})
```

```python
# tests/unit/test_user_service.py
import pytest
from unittest.mock import Mock, patch
from src.services.user_service import UserService
from src.models.user import User
from src.exceptions.user_exceptions import UserNotFoundError, InvalidUserDataError

class TestUserService:
    @pytest.fixture
    def mock_user_repository(self):
        return Mock()

    @pytest.fixture
    def user_service(self, mock_user_repository):
        return UserService(mock_user_repository)

    @pytest.fixture
    def sample_user_data(self):
        return {
            'id': 1,
            'email': 'test@example.com',
            'name': 'Test User',
            'age': 25
        }

    def test_create_user_success(self, user_service, mock_user_repository, sample_user_data):
        # Arrange
        mock_user_repository.save.return_value = User(**sample_user_data)

        # Act
        result = user_service.create_user(sample_user_data)

        # Assert
        assert result.email == sample_user_data['email']
        assert result.name == sample_user_data['name']
        mock_user_repository.save.assert_called_once()

    def test_create_user_invalid_email(self, user_service, sample_user_data):
        # Arrange
        sample_user_data['email'] = 'invalid-email'

        # Act & Assert
        with pytest.raises(InvalidUserDataError) as exc_info:
            user_service.create_user(sample_user_data)

        assert 'Invalid email format' in str(exc_info.value)

    def test_get_user_by_id_success(self, user_service, mock_user_repository, sample_user_data):
        # Arrange
        user_id = 1
        expected_user = User(**sample_user_data)
        mock_user_repository.find_by_id.return_value = expected_user

        # Act
        result = user_service.get_user_by_id(user_id)

        # Assert
        assert result == expected_user
        mock_user_repository.find_by_id.assert_called_once_with(user_id)

    def test_get_user_by_id_not_found(self, user_service, mock_user_repository):
        # Arrange
        user_id = 999
        mock_user_repository.find_by_id.return_value = None

        # Act & Assert
        with pytest.raises(UserNotFoundError):
            user_service.get_user_by_id(user_id)

    @patch('src.services.user_service.send_welcome_email')
    def test_create_user_sends_welcome_email(self, mock_send_email, user_service, mock_user_repository, sample_user_data):
        # Arrange
        mock_user_repository.save.return_value = User(**sample_user_data)

        # Act
        user_service.create_user(sample_user_data)

        # Assert
        mock_send_email.assert_called_once_with(sample_user_data['email'])

    @pytest.mark.parametrize("age,expected_category", [
        (17, "minor"),
        (18, "adult"),
        (25, "adult"),
        (65, "senior"),
        (70, "senior")
    ])
    def test_get_user_category(self, user_service, age, expected_category):
        # Act
        result = user_service.get_user_category(age)

        # Assert
        assert result == expected_category
```

####  **Integration Testing**

```javascript
// tests/integration/api.test.js
import { describe, it, expect, beforeAll, afterAll, beforeEach } from 'vitest'
import request from 'supertest'
import { app } from '../../src/app.js'
import { database } from '../../src/database.js'
import { seedDatabase, cleanDatabase } from '../helpers/database-helper.js'

describe('User API Integration Tests', () => {
  beforeAll(async () => {
    await database.connect()
  })

  afterAll(async () => {
    await database.disconnect()
  })

  beforeEach(async () => {
    await cleanDatabase()
    await seedDatabase()
  })

  describe('POST /api/users', () => {
    it('should create a new user successfully', async () => {
      // Arrange
      const userData = {
        email: 'newuser@example.com',
        name: 'New User',
        password: 'SecurePassword123!'
      }

      // Act
      const response = await request(app)
        .post('/api/users')
        .send(userData)
        .expect(201)

      // Assert
      expect(response.body).toHaveProperty('id')
      expect(response.body.email).toBe(userData.email)
      expect(response.body.name).toBe(userData.name)
      expect(response.body).not.toHaveProperty('password') // Password should be filtered out
    })

    it('should return 400 for duplicate email', async () => {
      // Arrange
      const userData = {
        email: 'existing@example.com', // This email exists in seed data
        name: 'Duplicate User',
        password: 'SecurePassword123!'
      }

      // Act & Assert
      const response = await request(app)
        .post('/api/users')
        .send(userData)
        .expect(400)

      expect(response.body.error).toContain('Email already exists')
    })

    it('should return 400 for invalid email format', async () => {
      // Arrange
      const userData = {
        email: 'invalid-email',
        name: 'Test User',
        password: 'SecurePassword123!'
      }

      // Act & Assert
      await request(app)
        .post('/api/users')
        .send(userData)
        .expect(400)
    })
  })

  describe('GET /api/users/:id', () => {
    it('should return user by id', async () => {
      // Arrange
      const userId = 1 // From seed data

      // Act
      const response = await request(app)
        .get(`/api/users/${userId}`)
        .expect(200)

      // Assert
      expect(response.body.id).toBe(userId)
      expect(response.body).toHaveProperty('email')
      expect(response.body).toHaveProperty('name')
    })

    it('should return 404 for non-existent user', async () => {
      // Act & Assert
      await request(app)
        .get('/api/users/999')
        .expect(404)
    })
  })

  describe('Authentication Flow', () => {
    it('should authenticate user and return JWT token', async () => {
      // Arrange
      const credentials = {
        email: 'test@example.com',
        password: 'TestPassword123!'
      }

      // Act
      const response = await request(app)
        .post('/api/auth/login')
        .send(credentials)
        .expect(200)

      // Assert
      expect(response.body).toHaveProperty('token')
      expect(response.body).toHaveProperty('user')
      expect(response.body.user.email).toBe(credentials.email)
    })

    it('should access protected route with valid token', async () => {
      // Arrange - Login first
      const loginResponse = await request(app)
        .post('/api/auth/login')
        .send({
          email: 'test@example.com',
          password: 'TestPassword123!'
        })

      const token = loginResponse.body.token

      // Act
      const response = await request(app)
        .get('/api/users/profile')
        .set('Authorization', `Bearer ${token}`)
        .expect(200)

      // Assert
      expect(response.body).toHaveProperty('email')
    })

    it('should reject access to protected route without token', async () => {
      // Act & Assert
      await request(app)
        .get('/api/users/profile')
        .expect(401)
    })
  })
})
```

####  **End-to-End Testing**

```javascript
// tests/e2e/user-registration.spec.js
import { test, expect } from '@playwright/test'

test.describe('User Registration Flow', () => {
  test.beforeEach(async ({ page }) => {
    await page.goto('/register')
  })

  test('should complete user registration successfully', async ({ page }) => {
    // Fill registration form
    await page.fill('[data-testid="email-input"]', 'newuser@example.com')
    await page.fill('[data-testid="name-input"]', 'Test User')
    await page.fill('[data-testid="password-input"]', 'SecurePassword123!')
    await page.fill('[data-testid="confirm-password-input"]', 'SecurePassword123!')

    // Accept terms
    await page.check('[data-testid="terms-checkbox"]')

    // Submit form
    await page.click('[data-testid="register-button"]')

    // Verify success
    await expect(page.locator('[data-testid="success-message"]')).toBeVisible()
    await expect(page).toHaveURL('/dashboard')

    // Verify user is logged in
    await expect(page.locator('[data-testid="user-menu"]')).toContainText('Test User')
  })

  test('should show validation errors for invalid input', async ({ page }) => {
    // Submit empty form
    await page.click('[data-testid="register-button"]')

    // Check validation errors
    await expect(page.locator('[data-testid="email-error"]')).toContainText('Email is required')
    await expect(page.locator('[data-testid="name-error"]')).toContainText('Name is required')
    await expect(page.locator('[data-testid="password-error"]')).toContainText('Password is required')
  })

  test('should handle duplicate email registration', async ({ page }) => {
    // Try to register with existing email
    await page.fill('[data-testid="email-input"]', 'existing@example.com')
    await page.fill('[data-testid="name-input"]', 'Test User')
    await page.fill('[data-testid="password-input"]', 'SecurePassword123!')
    await page.fill('[data-testid="confirm-password-input"]', 'SecurePassword123!')
    await page.check('[data-testid="terms-checkbox"]')
    await page.click('[data-testid="register-button"]')

    // Verify error message
    await expect(page.locator('[data-testid="email-error"]')).toContainText('Email already exists')
  })

  test('should enforce password strength requirements', async ({ page }) => {
    await page.fill('[data-testid="email-input"]', 'test@example.com')
    await page.fill('[data-testid="name-input"]', 'Test User')

    // Test weak password
    await page.fill('[data-testid="password-input"]', '123')
    await page.blur('[data-testid="password-input"]')

    await expect(page.locator('[data-testid="password-strength"]')).toContainText('Weak')
    await expect(page.locator('[data-testid="password-error"]')).toContainText('Password must be at least 8 characters')
  })
})

// tests/e2e/checkout-flow.spec.js
test.describe('E-commerce Checkout Flow', () => {
  test('should complete purchase flow successfully', async ({ page }) => {
    // Login
    await page.goto('/login')
    await page.fill('[data-testid="email"]', 'customer@example.com')
    await page.fill('[data-testid="password"]', 'password123')
    await page.click('[data-testid="login-button"]')

    // Add products to cart
    await page.goto('/products')
    await page.click('[data-testid="product-1"] [data-testid="add-to-cart"]')
    await page.click('[data-testid="product-2"] [data-testid="add-to-cart"]')

    // Go to cart
    await page.click('[data-testid="cart-icon"]')
    await expect(page.locator('[data-testid="cart-item"]')).toHaveCount(2)

    // Proceed to checkout
    await page.click('[data-testid="checkout-button"]')

    // Fill shipping information
    await page.fill('[data-testid="address"]', '123 Test Street')
    await page.fill('[data-testid="city"]', 'Test City')
    await page.fill('[data-testid="zipcode"]', '12345')
    await page.selectOption('[data-testid="country"]', 'US')

    // Fill payment information
    await page.fill('[data-testid="card-number"]', '4242424242424242')
    await page.fill('[data-testid="expiry"]', '12/25')
    await page.fill('[data-testid="cvv"]', '123')
    await page.fill('[data-testid="cardholder-name"]', 'Test Customer')

    // Complete purchase
    await page.click('[data-testid="place-order-button"]')

    // Verify success
    await expect(page.locator('[data-testid="order-confirmation"]')).toBeVisible()
    await expect(page.locator('[data-testid="order-number"]')).toBeVisible()

    // Verify email notification (mock check)
    // This would typically involve checking a test email service
  })
})
```

### 2. Quality Gates Configuration

####  **SonarQube Quality Gates**

```yaml
# sonar-project.properties
sonar.projectKey=my-project
sonar.projectName=My Project
sonar.projectVersion=1.0

# Source settings
sonar.sources=src
sonar.tests=tests
sonar.exclusions=**/node_modules/**,**/dist/**,**/coverage/**

# Coverage settings
sonar.javascript.lcov.reportPaths=coverage/lcov.info
sonar.coverage.exclusions=**/tests/**,**/*.test.js,**/*.spec.js

# Quality Gate conditions
sonar.qualitygate.wait=true
```

```bash
#!/bin/bash
# quality-gate-check.sh

echo " Running SonarQube analysis..."

# Run SonarQube scanner
sonar-scanner \
  -Dsonar.projectKey=$SONAR_PROJECT_KEY \
  -Dsonar.sources=src \
  -Dsonar.host.url=$SONAR_HOST_URL \
  -Dsonar.login=$SONAR_TOKEN

# Wait for quality gate result
echo "⏳ Waiting for Quality Gate result..."

TASK_ID=$(curl -s -u $SONAR_TOKEN: \
  "$SONAR_HOST_URL/api/ce/activity?component=$SONAR_PROJECT_KEY&ps=1" | \
  jq -r '.tasks[0].id')

# Poll for completion
while true; do
  STATUS=$(curl -s -u $SONAR_TOKEN: \
    "$SONAR_HOST_URL/api/ce/task?id=$TASK_ID" | \
    jq -r '.task.status')

  if [ "$STATUS" = "SUCCESS" ]; then
    break
  elif [ "$STATUS" = "FAILED" ]; then
    echo " SonarQube analysis failed"
    exit 1
  else
    echo "⏳ Analysis in progress..."
    sleep 10
  fi
done

# Check quality gate status
QG_STATUS=$(curl -s -u $SONAR_TOKEN: \
  "$SONAR_HOST_URL/api/qualitygates/project_status?projectKey=$SONAR_PROJECT_KEY" | \
  jq -r '.projectStatus.status')

if [ "$QG_STATUS" = "OK" ]; then
  echo " Quality Gate passed!"
  exit 0
else
  echo " Quality Gate failed!"

  # Get detailed metrics
  curl -s -u $SONAR_TOKEN: \
    "$SONAR_HOST_URL/api/measures/component?component=$SONAR_PROJECT_KEY&metricKeys=coverage,bugs,vulnerabilities,code_smells,duplicated_lines_density" | \
    jq '.component.measures[] | {metric: .metric, value: .value}'

  exit 1
fi
```

####  **Custom Quality Gates**

```javascript
// quality-gate-checker.js
import fs from 'fs'
import path from 'path'

class QualityGateChecker {
  constructor(config) {
    this.config = config
    this.results = {}
  }

  async runChecks() {
    console.log(' Running Quality Gate checks...')

    await this.checkCodeCoverage()
    await this.checkCodeQuality()
    await this.checkSecurity()
    await this.checkPerformance()
    await this.checkLicenses()

    return this.generateReport()
  }

  async checkCodeCoverage() {
    console.log(' Checking code coverage...')

    const coverageReport = JSON.parse(
      fs.readFileSync('coverage/coverage-summary.json', 'utf8')
    )

    const totalCoverage = coverageReport.total.lines.pct
    const threshold = this.config.coverage.threshold

    this.results.coverage = {
      value: totalCoverage,
      threshold,
      passed: totalCoverage >= threshold,
      details: coverageReport
    }

    console.log(`Coverage: ${totalCoverage}% (threshold: ${threshold}%)`)
  }

  async checkCodeQuality() {
    console.log(' Checking code quality...')

    // ESLint results
    const eslintReport = JSON.parse(
      fs.readFileSync('reports/eslint-report.json', 'utf8')
    )

    const errorCount = eslintReport.reduce((sum, file) => sum + file.errorCount, 0)
    const warningCount = eslintReport.reduce((sum, file) => sum + file.warningCount, 0)

    this.results.codeQuality = {
      errors: errorCount,
      warnings: warningCount,
      passed: errorCount === 0 && warningCount <= this.config.quality.maxWarnings,
      maxWarnings: this.config.quality.maxWarnings
    }

    console.log(`Errors: ${errorCount}, Warnings: ${warningCount}`)
  }

  async checkSecurity() {
    console.log(' Checking security...')

    // npm audit results
    const auditReport = JSON.parse(
      fs.readFileSync('reports/audit-report.json', 'utf8')
    )

    const highVulns = auditReport.metadata.vulnerabilities.high || 0
    const criticalVulns = auditReport.metadata.vulnerabilities.critical || 0

    this.results.security = {
      highVulnerabilities: highVulns,
      criticalVulnerabilities: criticalVulns,
      passed: criticalVulns === 0 && highVulns <= this.config.security.maxHigh,
      maxHigh: this.config.security.maxHigh
    }

    console.log(`Critical: ${criticalVulns}, High: ${highVulns}`)
  }

  async checkPerformance() {
    console.log(' Checking performance...')

    // Lighthouse CI results
    if (fs.existsSync('reports/lighthouse-ci.json')) {
      const lighthouseReport = JSON.parse(
        fs.readFileSync('reports/lighthouse-ci.json', 'utf8')
      )

      const performanceScore = lighthouseReport.performance * 100
      const threshold = this.config.performance.threshold

      this.results.performance = {
        score: performanceScore,
        threshold,
        passed: performanceScore >= threshold
      }

      console.log(`Performance Score: ${performanceScore} (threshold: ${threshold})`)
    } else {
      this.results.performance = { passed: true, skipped: true }
    }
  }

  async checkLicenses() {
    console.log(' Checking licenses...')

    if (fs.existsSync('reports/license-report.json')) {
      const licenseReport = JSON.parse(
        fs.readFileSync('reports/license-report.json', 'utf8')
      )

      const forbiddenLicenses = Object.keys(licenseReport)
        .filter(pkg => this.config.licenses.forbidden.includes(licenseReport[pkg]))

      this.results.licenses = {
        forbiddenPackages: forbiddenLicenses,
        passed: forbiddenLicenses.length === 0
      }

      console.log(`Forbidden licenses found: ${forbiddenLicenses.length}`)
    } else {
      this.results.licenses = { passed: true, skipped: true }
    }
  }

  generateReport() {
    const allPassed = Object.values(this.results).every(result => result.passed)

    const report = {
      timestamp: new Date().toISOString(),
      passed: allPassed,
      results: this.results,
      summary: this.generateSummary()
    }

    // Save report
    fs.writeFileSync('reports/quality-gate-report.json', JSON.stringify(report, null, 2))

    // Generate HTML report
    this.generateHTMLReport(report)

    return report
  }

  generateSummary() {
    const summary = {
      total: Object.keys(this.results).length,
      passed: Object.values(this.results).filter(r => r.passed).length,
      failed: Object.values(this.results).filter(r => !r.passed).length,
      skipped: Object.values(this.results).filter(r => r.skipped).length
    }

    return summary
  }

  generateHTMLReport(report) {
    const html = `
<!DOCTYPE html>
<html>
<head>
    <title>Quality Gate Report</title>
    <style>
        body { font-family: Arial, sans-serif; margin: 20px; }
        .header { background: ${report.passed ? '#d4edda' : '#f8d7da'}; padding: 20px; border-radius: 5px; }
        .metric { margin: 10px 0; padding: 10px; border-left: 4px solid #ccc; }
        .passed { border-left-color: #28a745; }
        .failed { border-left-color: #dc3545; }
        .skipped { border-left-color: #ffc107; }
        .details { margin-top: 10px; font-size: 0.9em; color: #666; }
    </style>
</head>
<body>
    <div class="header">
        <h1>Quality Gate Report</h1>
        <p><strong>Status:</strong> ${report.passed ? ' PASSED' : ' FAILED'}</p>
        <p><strong>Timestamp:</strong> ${report.timestamp}</p>
    </div>

    ${Object.entries(report.results).map(([key, result]) => `
        <div class="metric ${result.passed ? 'passed' : result.skipped ? 'skipped' : 'failed'}">
            <h3>${key.toUpperCase()}: ${result.passed ? ' PASSED' : result.skipped ? '⏭ SKIPPED' : ' FAILED'}</h3>
            <div class="details">${JSON.stringify(result, null, 2)}</div>
        </div>
    `).join('')}

    <div class="summary">
        <h2>Summary</h2>
        <p>Total checks: ${report.summary.total}</p>
        <p>Passed: ${report.summary.passed}</p>
        <p>Failed: ${report.summary.failed}</p>
        <p>Skipped: ${report.summary.skipped}</p>
    </div>
</body>
</html>
    `

    fs.writeFileSync('reports/quality-gate-report.html', html)
  }
}

// Configuration
const config = {
  coverage: {
    threshold: 80
  },
  quality: {
    maxWarnings: 5
  },
  security: {
    maxHigh: 0
  },
  performance: {
    threshold: 90
  },
  licenses: {
    forbidden: ['GPL-3.0', 'AGPL-3.0']
  }
}

// Run quality gate
const checker = new QualityGateChecker(config)
checker.runChecks().then(report => {
  console.log('\n Quality Gate Report:')
  console.log(`Status: ${report.passed ? ' PASSED' : ' FAILED'}`)
  console.log(`Summary: ${report.summary.passed}/${report.summary.total} checks passed`)

  if (!report.passed) {
    process.exit(1)
  }
}).catch(error => {
  console.error(' Quality Gate check failed:', error)
  process.exit(1)
})
```

### 3. Security Testing Automation

####  **SAST (Static Application Security Testing)**

```yaml
# .github/workflows/security-testing.yml
name: Security Testing

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

jobs:
  sast-analysis:
    name: Static Application Security Testing
    runs-on: ubuntu-latest

    steps:
      - name: Checkout code
        uses: actions/checkout@v4
        with:
          fetch-depth: 0

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '18'

      - name: Install dependencies
        run: npm ci

      - name: Run ESLint Security
        run: npx eslint . --ext .js,.ts --format json --output-file eslint-security.json
        continue-on-error: true

      - name: Run Semgrep SAST
        uses: returntocorp/semgrep-action@v1
        with:
          config: >-
            p/security-audit
            p/secrets
            p/javascript
            p/typescript
        env:
          SEMGREP_APP_TOKEN: ${{ secrets.SEMGREP_APP_TOKEN }}

      - name: Run CodeQL Analysis
        uses: github/codeql-action/analyze@v3
        with:
          languages: javascript

      - name: Run npm audit
        run: |
          npm audit --audit-level moderate --json > npm-audit.json
          npm audit --audit-level moderate
        continue-on-error: true

      - name: Run OWASP Dependency Check
        uses: dependency-check/Dependency-Check_Action@main
        with:
          project: 'my-project'
          path: '.'
          format: 'JSON'
          args: >
            --enableRetired
            --enableExperimental
            --suppression dependency-check-suppressions.xml

      - name: Upload SARIF results
        uses: github/codeql-action/upload-sarif@v3
        if: always()
        with:
          sarif_file: results.sarif

  container-security:
    name: Container Security Scanning
    runs-on: ubuntu-latest

    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Build Docker image
        run: docker build -t security-test:latest .

      - name: Run Trivy vulnerability scanner
        uses: aquasecurity/trivy-action@master
        with:
          image-ref: 'security-test:latest'
          format: 'sarif'
          output: 'trivy-results.sarif'

      - name: Run Grype vulnerability scanner
        run: |
          curl -sSfL https://raw.githubusercontent.com/anchore/grype/main/install.sh | sh -s -- -b /usr/local/bin
          grype security-test:latest -o json > grype-results.json

      - name: Run Snyk Container Test
        uses: snyk/actions/docker@master
        env:
          SNYK_TOKEN: ${{ secrets.SNYK_TOKEN }}
        with:
          image: security-test:latest
          args: --file=Dockerfile --severity-threshold=high

      - name: Upload Trivy scan results
        uses: github/codeql-action/upload-sarif@v3
        if: always()
        with:
          sarif_file: 'trivy-results.sarif'

  dast-testing:
    name: Dynamic Application Security Testing
    runs-on: ubuntu-latest
    needs: [sast-analysis]

    services:
      app:
        image: my-app:latest
        ports:
          - 3000:3000
        env:
          NODE_ENV: test

    steps:
      - name: Wait for app to be ready
        run: |
          for i in {1..30}; do
            if curl -f http://localhost:3000/health; then
              echo "App is ready"
              break
            fi
            sleep 2
          done

      - name: Run OWASP ZAP Baseline Scan
        uses: zaproxy/action-baseline@v0.7.0
        with:
          target: 'http://localhost:3000'
          rules_file_name: '.zap/rules.tsv'
          cmd_options: '-a'

      - name: Run Nuclei Scan
        run: |
          docker run -v ${{ github.workspace }}:/nuclei-templates \
            projectdiscovery/nuclei \
            -target http://localhost:3000 \
            -json-export nuclei-results.json

      - name: Upload ZAP results
        uses: actions/upload-artifact@v4
        if: always()
        with:
          name: zap-scan-results
          path: report_html.html
```

####  **Secret Scanning**

```bash
#!/bin/bash
# secret-scanner.sh

echo " Scanning for secrets and sensitive data..."

# Install tools
echo "Installing secret scanning tools..."
curl -sSfL https://raw.githubusercontent.com/trufflesecurity/trufflehog/main/scripts/install.sh | sh -s -- -b /usr/local/bin
pip install detect-secrets

# TruffleHog scan
echo "Running TruffleHog scan..."
trufflehog git file://. --json > trufflehog-results.json

# Detect-secrets scan
echo "Running detect-secrets scan..."
detect-secrets scan --all-files --force-use-all-plugins > .secrets.baseline

# Git-secrets scan (if available)
if command -v git-secrets &> /dev/null; then
    echo "Running git-secrets scan..."
    git secrets --scan
fi

# GitLeaks scan
if command -v gitleaks &> /dev/null; then
    echo "Running GitLeaks scan..."
    gitleaks detect --source . --report-format json --report-path gitleaks-results.json
fi

# Custom regex patterns
echo "Running custom secret patterns..."
SECRETS_FOUND=0

# API Keys
if grep -r --include="*.js" --include="*.ts" --include="*.json" --include="*.yml" --include="*.yaml" \
   -E "(api[_-]?key|apikey|api_key).*['\"][0-9a-zA-Z]{20,}['\"]" . ; then
    echo " Potential API keys found"
    SECRETS_FOUND=1
fi

# AWS Credentials
if grep -r --include="*.js" --include="*.ts" --include="*.json" --include="*.yml" --include="*.yaml" \
   -E "(aws[_-]?access[_-]?key|aws[_-]?secret)" . ; then
    echo " Potential AWS credentials found"
    SECRETS_FOUND=1
fi

# Database URLs
if grep -r --include="*.js" --include="*.ts" --include="*.json" --include="*.yml" --include="*.yaml" \
   -E "(postgres|mysql|mongodb)://[^@]+:[^@]+@" . ; then
    echo " Potential database credentials found"
    SECRETS_FOUND=1
fi

# Private Keys
if grep -r --include="*.pem" --include="*.key" \
   -E "BEGIN.*PRIVATE.*KEY" . ; then
    echo " Private keys found"
    SECRETS_FOUND=1
fi

# Generate report
cat > secret-scan-report.json << EOF
{
  "timestamp": "$(date -Iseconds)",
  "tools": {
    "trufflehog": "$(if [ -f trufflehog-results.json ]; then echo 'completed'; else echo 'failed'; fi)",
    "detect_secrets": "$(if [ -f .secrets.baseline ]; then echo 'completed'; else echo 'failed'; fi)",
    "gitleaks": "$(if [ -f gitleaks-results.json ]; then echo 'completed'; else echo 'failed'; fi)",
    "custom_patterns": "completed"
  },
  "secrets_found": $SECRETS_FOUND
}
EOF

if [ $SECRETS_FOUND -eq 1 ]; then
    echo " Secrets detected! Review the findings."
    exit 1
else
    echo " No secrets detected."
    exit 0
fi
```

### 4. Performance Testing Automation

####  **Load Testing with K6**

```javascript
// tests/performance/load-test.js
import http from 'k6/http'
import { check, sleep } from 'k6'
import { Rate, Trend } from 'k6/metrics'

// Custom metrics
export const errorRate = new Rate('errors')
export const responseTimeTrend = new Trend('response_time')

// Test configuration
export const options = {
  stages: [
    { duration: '1m', target: 20 },   // Ramp-up
    { duration: '3m', target: 20 },   // Stay at 20 users
    { duration: '1m', target: 50 },   // Ramp-up to 50 users
    { duration: '3m', target: 50 },   // Stay at 50 users
    { duration: '1m', target: 100 },  // Ramp-up to 100 users
    { duration: '3m', target: 100 },  // Stay at 100 users
    { duration: '2m', target: 0 },    // Ramp-down
  ],
  thresholds: {
    http_req_duration: ['p(95)<500', 'p(99)<1000'],
    http_req_failed: ['rate<0.05'], // Error rate should be less than 5%
    errors: ['rate<0.1'],
  },
}

const BASE_URL = __ENV.BASE_URL || 'http://localhost:3000'

export function setup() {
  // Create test user
  const response = http.post(`${BASE_URL}/api/auth/register`, {
    email: `testuser${Date.now()}@example.com`,
    password: 'TestPassword123!',
    name: 'Load Test User'
  })

  check(response, {
    'User created successfully': (r) => r.status === 201,
  })

  return { userId: response.json('id') }
}

export default function(data) {
  // Test scenarios
  testHomePage()
  testAPIEndpoints()
  testUserJourney()

  sleep(1)
}

function testHomePage() {
  const response = http.get(`${BASE_URL}/`)

  check(response, {
    'Homepage status is 200': (r) => r.status === 200,
    'Homepage loads within 500ms': (r) => r.timings.duration < 500,
  })

  errorRate.add(response.status !== 200)
  responseTimeTrend.add(response.timings.duration)
}

function testAPIEndpoints() {
  // Test API health endpoint
  const healthResponse = http.get(`${BASE_URL}/api/health`)

  check(healthResponse, {
    'Health check status is 200': (r) => r.status === 200,
    'Health check response is fast': (r) => r.timings.duration < 100,
  })

  // Test products endpoint
  const productsResponse = http.get(`${BASE_URL}/api/products`)

  check(productsResponse, {
    'Products endpoint status is 200': (r) => r.status === 200,
    'Products endpoint response time < 1s': (r) => r.timings.duration < 1000,
    'Products response has data': (r) => r.json('data').length > 0,
  })

  errorRate.add(productsResponse.status !== 200)
}

function testUserJourney() {
  // Login
  const loginResponse = http.post(`${BASE_URL}/api/auth/login`, {
    email: 'loadtest@example.com',
    password: 'TestPassword123!'
  })

  check(loginResponse, {
    'Login successful': (r) => r.status === 200,
  })

  if (loginResponse.status === 200) {
    const token = loginResponse.json('token')

    // Authenticated requests
    const headers = {
      'Authorization': `Bearer ${token}`,
      'Content-Type': 'application/json',
    }

    // Get user profile
    const profileResponse = http.get(`${BASE_URL}/api/user/profile`, { headers })

    check(profileResponse, {
      'Profile fetch successful': (r) => r.status === 200,
    })

    // Update profile
    const updateResponse = http.put(`${BASE_URL}/api/user/profile`, {
      name: 'Updated Load Test User'
    }, { headers })

    check(updateResponse, {
      'Profile update successful': (r) => r.status === 200,
    })
  }
}

export function teardown(data) {
  // Cleanup test data if needed
  console.log('Load test completed')
}
```

####  **Performance Monitoring**

```javascript
// tests/performance/lighthouse-ci.js
const lighthouse = require('lighthouse')
const chromeLauncher = require('chrome-launcher')
const fs = require('fs')

async function runLighthouse(url) {
  const chrome = await chromeLauncher.launch({ chromeFlags: ['--headless'] })

  const options = {
    logLevel: 'info',
    output: 'json',
    onlyCategories: ['performance', 'accessibility', 'best-practices', 'seo'],
    port: chrome.port,
  }

  const runnerResult = await lighthouse(url, options)

  await chrome.kill()

  return runnerResult
}

async function runPerformanceTests() {
  const urls = [
    'http://localhost:3000',
    'http://localhost:3000/products',
    'http://localhost:3000/about',
  ]

  const results = []

  for (const url of urls) {
    console.log(`Testing ${url}...`)

    const result = await runLighthouse(url)
    const lhr = result.lhr

    const scores = {
      url,
      performance: lhr.categories.performance.score * 100,
      accessibility: lhr.categories.accessibility.score * 100,
      bestPractices: lhr.categories['best-practices'].score * 100,
      seo: lhr.categories.seo.score * 100,
      metrics: {
        firstContentfulPaint: lhr.audits['first-contentful-paint'].numericValue,
        largestContentfulPaint: lhr.audits['largest-contentful-paint'].numericValue,
        cumulativeLayoutShift: lhr.audits['cumulative-layout-shift'].numericValue,
        firstInputDelay: lhr.audits['max-potential-fid'].numericValue,
        totalBlockingTime: lhr.audits['total-blocking-time'].numericValue,
      }
    }

    results.push(scores)

    // Check thresholds
    const thresholds = {
      performance: 90,
      accessibility: 95,
      bestPractices: 90,
      seo: 90
    }

    let passed = true
    for (const [category, threshold] of Object.entries(thresholds)) {
      if (scores[category] < threshold) {
        console.log(` ${category}: ${scores[category]} (threshold: ${threshold})`)
        passed = false
      } else {
        console.log(` ${category}: ${scores[category]}`)
      }
    }

    if (!passed) {
      process.exit(1)
    }
  }

  // Save results
  fs.writeFileSync('lighthouse-results.json', JSON.stringify(results, null, 2))

  console.log(' All performance tests passed!')
}

runPerformanceTests().catch(error => {
  console.error('Performance tests failed:', error)
  process.exit(1)
})
```

##  Laboratorios Prácticos

### Lab 1: Implementar Testing Pyramid

**Objetivo**: Crear una suite completa de tests siguiendo la pirámide de testing.

**Estructura del proyecto**:

```
tests/
 unit/
    services/
    models/
    utils/
 integration/
    api/
    database/
 e2e/
    user-flows/
    critical-paths/
 performance/
     load-tests/
     stress-tests/
```

### Lab 2: Quality Gates Pipeline

**Objetivo**: Implementar quality gates automáticos en CI/CD.

1. **Configurar herramientas**:
   - SonarQube para análisis de código
   - OWASP ZAP para security testing
   - Lighthouse CI para performance
   - Jest para coverage

2. **Crear pipeline**:

```yaml
quality-gate:
  stage: quality
  script:
    - npm run test:coverage
    - npm run security:scan
    - npm run performance:test
    - npm run quality:check
  artifacts:
    reports:
      coverage_report:
        coverage_format: cobertura
        path: coverage/cobertura-coverage.xml
      junit: reports/junit.xml
```

##  Ejercicios Prácticos

### Ejercicio 1: Test Data Management

Implementar un sistema de gestión de datos de test que:

- Genere datos sintéticos
- Mantenga datos consistentes entre tests
- Limpie datos después de cada test

### Ejercicio 2: Flaky Tests Detection

Crear un sistema que:

- Detecte tests inestables automáticamente
- Reporte estadísticas de flakiness
- Sugiera mejoras para tests problemáticos

### Ejercicio 3: Visual Regression Testing

Implementar testing de regresión visual que:

- Compare screenshots automáticamente
- Detecte cambios visuales no intencionados
- Integre con el pipeline CI/CD

##  Troubleshooting

### Problemas Comunes

1. **Tests Lentos**:

```javascript
// Optimizar tests con beforeAll
beforeAll(async () => {
  await setupDatabase()
})

// Usar mocks para dependencias externas
jest.mock('external-api')
```

2. **Tests Flaky**:

```javascript
// Agregar waits explícitos
await page.waitForSelector('[data-testid="element"]')

// Usar retry logic
await retry(async () => {
  const element = await page.$('[data-testid="element"]')
  expect(element).toBeTruthy()
}, 3)
```

3. **Memory Leaks en Tests**:

```javascript
afterEach(() => {
  jest.clearAllMocks()
  jest.clearAllTimers()
})
```

##  Recursos Adicionales

- [Testing Best Practices](https://github.com/goldbergyoni/javascript-testing-best-practices)
- [Jest Documentation](https://jestjs.io/docs/getting-started)
- [Playwright Documentation](https://playwright.dev/docs/intro)
- [K6 Load Testing](https://k6.io/docs/)
- [SonarQube Quality Gates](https://docs.sonarqube.org/latest/user-guide/quality-gates/)

##  Checklist de Completado

- [ ] Implementado unit testing completo
- [ ] Configurado integration testing
- [ ] Creado E2E tests críticos
- [ ] Implementado quality gates automáticos
- [ ] Configurado security testing (SAST/DAST)
- [ ] Implementado performance testing
- [ ] Configurado coverage reporting
- [ ] Creado custom quality metrics
- [ ] Implementado test data management
- [ ] Configurado CI/CD integration

---

**Excelente!** Has dominado testing automation y quality gates. Continúa con el módulo final para aprender sobre estrategias de deployment avanzadas.
