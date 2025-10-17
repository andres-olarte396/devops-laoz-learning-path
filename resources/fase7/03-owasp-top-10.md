# Módulo 03: OWASP Top 10

## 🎯 Objetivos

- Identificar y comprender los 10 riesgos de seguridad más críticos para las aplicaciones web según OWASP.
- Aprender a reconocer patrones de código y configuraciones que conducen a estas vulnerabilidades.
- Conocer estrategias de mitigación y defensa para cada uno de los riesgos.

## 📜 Contenido

### ¿Qué es el OWASP Top 10?

El **Open Web Application Security Project (OWASP)** publica periódicamente una lista de los 10 riesgos de seguridad más importantes en aplicaciones web. No es un estándar, sino una guía de concienciación que ayuda a los desarrolladores a priorizar sus esfuerzos de seguridad.

### OWASP Top 10 - 2021 (Resumen)

| ID      | Categoría                                    | Descripción                                                                           | Estrategias de Mitigación                                                                              |
| :------ | :------------------------------------------- | :------------------------------------------------------------------------------------ | :----------------------------------------------------------------------------------------------------- |
| **A01** | **Broken Access Control**                    | Fallos en la aplicación de restricciones sobre lo que los usuarios pueden hacer.      | Implementar RBAC/ABAC, denegar por defecto, validar permisos en cada solicitud.                        |
| **A02** | **Cryptographic Failures**                   | Uso incorrecto de criptografía, exponiendo datos sensibles.                           | Usar algoritmos fuertes (AES, RSA), cifrar en tránsito (TLS) y en reposo, no "inventar" criptografía.  |
| **A03** | **Injection**                                | Inyección de código malicioso (SQL, NoSQL, OS, LDAP) a través de entradas de usuario. | Usar consultas parametrizadas (Prepared Statements), ORMs, y escapar todas las entradas.               |
| **A04** | **Insecure Design**                          | Fallos fundamentales en el diseño y la arquitectura de seguridad.                     | Modelado de amenazas (Threat Modeling), aplicar principios de diseño seguro.                           |
| **A05** | **Security Misconfiguration**                | Configuraciones de seguridad incorrectas o por defecto.                               | Hardening de servidores, deshabilitar funcionalidades no usadas, automatizar la configuración (IaC).   |
| **A06** | **Vulnerable & Outdated Components**         | Uso de dependencias con vulnerabilidades conocidas (CVEs).                            | Análisis de Composición de Software (SCA), mantener un inventario de dependencias, actualizar parches. |
| **A07** | **Identification & Authentication Failures** | Fallos en la gestión de identidad y sesiones.                                         | Implementar MFA, políticas de contraseñas robustas, protección contra ataques de fuerza bruta.         |
| **A08** | **Software & Data Integrity Failures**       | Fallos relacionados con la integridad del código y los datos.                         | Firmar digitalmente los artefactos, validar la integridad de las actualizaciones (checksums).          |
| **A09** | **Security Logging & Monitoring Failures**   | Registro y monitoreo insuficientes para detectar y responder a ataques.               | Registrar eventos de seguridad (logins, fallos de acceso), centralizar logs, configurar alertas.       |
| **A10** | **Server-Side Request Forgery (SSRF)**       | Forzar al servidor a realizar peticiones a un destino no deseado.                     | Validar todas las URLs de entrada, usar listas blancas (allow-lists) de dominios.                      |

## 🏢 Ejemplo Práctico: SQL Injection (A03)

- **Código Vulnerable (PHP)**:

  ```php
  $userId = $_POST['userId'];
  $query = "SELECT * FROM users WHERE id = " . $userId; // El atacante puede inyectar código aquí
  ```

  Si un atacante envía `1; DROP TABLE users; --`, la consulta se convierte en `SELECT * FROM users WHERE id = 1; DROP TABLE users; --`, borrando la tabla de usuarios.

- **Código Seguro (PHP con PDO)**:

  ```php
  $userId = $_POST['userId'];
  $stmt = $pdo->prepare("SELECT * FROM users WHERE id = :userId"); // Consulta parametrizada
  $stmt->execute(['userId' => $userId]);
  ```

  El driver de la base de datos se encarga de escapar la entrada, tratando `$userId` siempre como un dato y nunca como código ejecutable.

## ✍️ Ejercicio

1. Elige una vulnerabilidad del OWASP Top 10 (que no sea Injection).
2. Busca un ejemplo de código simple (en cualquier lenguaje) que sea vulnerable a ella.
3. Describe cómo un atacante podría explotar esa vulnerabilidad.
4. Propón una solución o cambio en el código para mitigar el riesgo.
