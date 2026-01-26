# Módulo 05: Análisis Dinámico de Seguridad de Aplicaciones (DAST)

## 🎯 Objetivos

- Comprender qué es DAST y en qué se diferencia de SAST.
- Aprender a ejecutar escaneos DAST contra una aplicación en ejecución utilizando OWASP ZAP.
- Analizar los resultados de un escaneo DAST para identificar vulnerabilidades explotables.

## 📜 Contenido

### 1. ¿Qué es DAST?

- **Definición**: **Dynamic Application Security Testing** (DAST) es una técnica de "caja negra" que analiza una aplicación **mientras está en ejecución**. Simula ataques externos contra la interfaz de la aplicación (HTTP, APIs) para encontrar vulnerabilidades que un atacante podría explotar.
- **Ventajas**:
  - **Bajos Falsos Positivos**: Como encuentra vulnerabilidades explotándolas, los hallazgos suelen ser problemas reales.
  - **Independiente del Lenguaje**: No necesita acceso al código fuente, por lo que funciona con cualquier tecnología.
  - **Detecta Problemas de Entorno**: Puede encontrar vulnerabilidades relacionadas con la configuración del servidor, cabeceras HTTP, etc., que SAST no puede ver.
- **Desventajas**:
  - **Detección Tardía**: Requiere una aplicación en ejecución, por lo que se usa más tarde en el ciclo de vida (entornos de Staging, QA).
  - **Menor Cobertura**: Puede no cubrir todas las rutas de código si el "spider" o explorador no las descubre.
  - **No señala la línea de código**: Indica la URL y el parámetro vulnerable, pero no la línea de código exacta que lo causa.

### 2. SAST vs. DAST: Un Enfoque Combinado

SAST y DAST no son excluyentes, sino complementarios.

| Característica       | SAST (Static)                           | DAST (Dynamic)                                                  |
| :------------------- | :-------------------------------------- | :-------------------------------------------------------------- |
| **Perspectiva**      | "Desde adentro" (Caja Blanca)           | "Desde afuera" (Caja Negra)                                     |
| **Cuándo se usa**    | Temprano (IDE, CI)                      | Tarde (QA, Staging)                                             |
| **Requisito**        | Código fuente                           | Aplicación en ejecución                                         |
| **Encuentra**        | SQL Injection, contraseñas hardcodeadas | Cross-Site Scripting (XSS), fallos de configuración de servidor |
| **Falsos Positivos** | Más altos                               | Más bajos                                                       |

La mejor estrategia es usar **SAST en cada commit** para una retroalimentación rápida y **DAST en cada despliegue a un entorno de pruebas** para una validación más profunda.

### 3. Herramientas Populares de DAST

- **OWASP ZAP (Zed Attack Proxy)**: La herramienta de código abierto más popular. Puede funcionar como un proxy para interceptar tráfico o como un escáner automatizado.
- **Burp Suite**: La herramienta preferida por los profesionales de la seguridad. Tiene una versión gratuita y una comercial muy potente.
- **Arachni**: Un framework de escaneo de vulnerabilidades web de código abierto.

### 4. Fases de un Escaneo DAST con OWASP ZAP

Un escaneo automatizado con ZAP generalmente sigue estos pasos:

1. **Spidering (Exploración)**: ZAP navega la aplicación a partir de una URL base para descubrir todas las páginas y funcionalidades disponibles.
2. **AJAX Spider**: Un explorador especializado para aplicaciones modernas que usan mucho JavaScript (React, Angular, Vue).
3. **Active Scanning (Escaneo Activo)**: ZAP lanza miles de ataques conocidos contra cada página, parámetro y endpoint descubierto. Busca vulnerabilidades como XSS, SQL Injection, CSRF, etc.
4. **Reporting (Generación de Reporte)**: ZAP genera un informe detallado (HTML, XML, JSON) con todas las vulnerabilidades encontradas, su severidad y recomendaciones para solucionarlas.

## 🏢 Ejemplo Práctico: Escaneo DAST en un Pipeline

DAST se integra típicamente en la fase de CD (Continuous Deployment) después de un despliegue exitoso a un entorno de QA.

```yaml
name: Deploy to QA and DAST Scan
on:
  push:
    branches: [develop]
jobs:
  deploy-to-qa:
    runs-on: ubuntu-latest
    steps:
      - name: Deploy to QA Environment
        # ... aquí irían los pasos para desplegar la aplicación ...
        run: echo "Deploying to http://qa-environment.com"

  dast-scan:
    needs: deploy-to-qa
    runs-on: ubuntu-latest
    steps:
      - name: OWASP ZAP Baseline Scan
        uses: zaproxy/action-baseline@v0.7.0
        with:
          target: "http://qa-environment.com"
          rules_file_name: ".zap/rules.tsv"
          cmd_options: "-a"
```

## ✍️ Ejercicio

1. Descarga e instala [OWASP ZAP](https://www.zaproxy.org/download/).
2. Usa la "Automated Scan" (la forma más sencilla) para escanear un sitio web de pruebas público como `http://testphp.vulnweb.com/`.
3. Explora el informe generado por ZAP.
4. Identifica una vulnerabilidad de severidad "High" o "Medium".
5. Describe la vulnerabilidad y explica por qué representa un riesgo.
