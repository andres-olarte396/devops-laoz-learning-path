# Fase 7: Test de Evaluación - DevSecOps

Este test evalúa tu comprensión de los conceptos y herramientas clave de DevSecOps cubiertos en esta fase.

---

## 📝 Parte 1: Preguntas Teóricas (40%)

1. **Pregunta de Concepto**: Explica con tus propias palabras la diferencia fundamental entre SAST y DAST. ¿En qué etapa del ciclo de vida de DevOps encaja mejor cada uno y por qué?

2. **Pregunta de Escenario**: Estás diseñando un nuevo servicio de autenticación. ¿Cómo aplicarías los tres pilares de la tríada CIA (Confidencialidad, Integridad, Disponibilidad) en tu diseño? Menciona un control técnico para cada pilar.

3. **Pregunta de Herramientas**: ¿Qué es un SBOM (Software Bill of Materials) y por qué es crucial para la seguridad de las aplicaciones modernas? Nombra una herramienta que pueda generar un SBOM.

4. **Pregunta de Principios**: Describe el principio de "mínimo privilegio" en el contexto de la seguridad en la nube. Da un ejemplo de cómo lo implementarías en una política de IAM en AWS o Azure.

5. **Pregunta de Cumplimiento**: Tu empresa necesita cumplir con la normativa PCI DSS porque va a procesar pagos con tarjeta de crédito. ¿Cómo podrías usar la automatización para demostrar continuamente que los firewalls de tus servidores están configurados correctamente y no exponen puertos no autorizados?

---

## 💻 Parte 2: Ejercicios Prácticos (60%)

### Ejercicio 1: Escaneo de IaC (20%)

Se te proporciona el siguiente fragmento de código de Terraform (`main.tf`) que define un grupo de seguridad en AWS.

```terraform
resource "aws_security_group" "web_sg" {
  name        = "web-server-sg"
  description = "Allow HTTP and SSH traffic"

  ingress {
    from_port   = 80
    to_port     = 80
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  ingress {
    from_port   = 22
    to_port     = 22
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"] # ¡Peligro!
  }
}
```

1. **Identifica la Vulnerabilidad**: ¿Cuál es el riesgo de seguridad en este código?
2. **Escanea el Código**: Usando una herramienta de escaneo de IaC como `Checkov` o `tfsec`, ejecuta un análisis sobre este archivo. Pega el resultado del escaneo que muestra la vulnerabilidad detectada.
3. **Corrige el Código**: Modifica el bloque de `ingress` para el puerto 22 de manera que solo permita el acceso desde una IP específica (puedes usar una IP ficticia como `84.22.11.5/32`) en lugar de desde todo Internet.
4. **Verifica la Corrección**: Vuelve a ejecutar la herramienta de escaneo y demuestra que la vulnerabilidad ha sido solucionada.

---

### Ejercicio 2: Gestión de Secretos (20%)

Has encontrado el siguiente código en un repositorio de la empresa:

```python
import mysql.connector

def get_db_connection():
    connection = mysql.connector.connect(
        host="prod.database.example.com",
        user="admin",
        password="SuperSecretPassword123!" # ¡Peligro!
    )
    return connection
```

1. **Identifica el Problema**: ¿Cuál es la vulnerabilidad principal en este fragmento de código?
2. **Propón una Solución**: Describe cómo solucionarías este problema utilizando un sistema de gestión de secretos como HashiCorp Vault. No necesitas escribir el código completo, pero describe los pasos que la aplicación debería seguir para obtener la contraseña de la base de datos de forma segura.
3. **Beneficios**: Menciona al menos dos beneficios de usar Vault en lugar de la solución actual (contraseña hardcodeada).

---

### Ejercicio 3: Pipeline DevSecOps (20%)

Basado en el laboratorio final, se te pide diseñar (solo en papel/texto) un "Quality Gate" para un pipeline de CI/CD. Define al menos **cuatro** condiciones que un build debe cumplir para poder pasar a la fase de despliegue. Las condiciones deben cubrir diferentes aspectos de la seguridad vistos en la fase.

**Ejemplo de una condición**:

- _Condición_: "El escaneo SAST no debe reportar ninguna vulnerabilidad de severidad `CRITICAL`."
- _Herramienta_: SonarQube / SonarCloud.

Define tus cuatro condiciones, especificando la herramienta que usarías para verificar cada una.
