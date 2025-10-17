# Módulo 02: Modelo de Seguridad Zero Trust

## 🎯 Objetivos

- Comprender el principio fundamental de Zero Trust: "Nunca confiar, siempre verificar".
- Identificar los pilares de una arquitectura Zero Trust.
- Contrastar el modelo Zero Trust con el enfoque tradicional de seguridad perimetral.

## 📜 Contenido

### 1. ¿Qué es Zero Trust?

- **Filosofía Central**: No existe un "perímetro de red seguro". Cada solicitud de acceso, sin importar de dónde provenga (dentro o fuera de la red corporativa), debe ser tratada como si proviniera de una red no controlada.
- **Principios Clave**:
  1. **Verificar explícitamente**: Siempre autenticar y autorizar basado en todos los puntos de datos disponibles (identidad, ubicación, salud del dispositivo, etc.).
  2. **Usar acceso de mínimo privilegio**: Limitar el acceso de los usuarios a los recursos estrictamente necesarios para su función.
  3. **Asumir la brecha (Assume Breach)**: Minimizar el radio de explosión y segmentar el acceso. Si un componente se ve comprometido, el daño debe ser contenido.

### 2. Pilares de una Arquitectura Zero Trust

Una implementación completa de Zero Trust se basa en varios pilares interconectados:

- **Identidad (Identities)**:
  - **Quién solicita el acceso**.
  - **Controles**: Autenticación multifactor (MFA), Single Sign-On (SSO), políticas de acceso condicional.
- **Dispositivos (Endpoints)**:
  - **Desde dónde se solicita el acceso**.
  - **Controles**: Gestión de dispositivos (MDM), comprobación de la salud del dispositivo (ej. antivirus actualizado, parches de seguridad).
- **Aplicaciones (Applications)**:
  - **A qué se está intentando acceder**.
  - **Controles**: API Gateways, microsegmentación, control de acceso a nivel de aplicación.
- **Datos (Data)**:
  - **Qué se está protegiendo**.
  - **Controles**: Clasificación de datos, cifrado en reposo y en tránsito, prevención de pérdida de datos (DLP).
- **Red (Network)**:
  - **Cómo se entregan los datos**.
  - **Controles**: Microsegmentación de red, firewalls de nueva generación (NGFW), protección contra DDoS.
- **Visibilidad y Análisis (Visibility and Analytics)**:
  - **Monitoreo continuo**.
  - **Controles**: SIEM, análisis de comportamiento de usuarios y entidades (UEBA) para detectar anomalías.

### 3. Zero Trust vs. Seguridad Perimetral Tradicional

| Característica | Modelo Perimetral (Castillo y Foso)                    | Modelo Zero Trust                                                             |
| :------------- | :----------------------------------------------------- | :---------------------------------------------------------------------------- |
| **Confianza**  | Confianza implícita dentro de la red.                  | Desconfianza por defecto. La confianza es dinámica y se evalúa continuamente. |
| **Perímetro**  | Definido y estático. El objetivo es proteger el borde. | El perímetro es fluido y se define por identidades y recursos.                |
| **Acceso**     | Acceso amplio una vez dentro de la red.                | Acceso de mínimo privilegio, segmentado y "just-in-time".                     |
| **Enfoque**    | Prevenir la intrusión.                                 | Asumir que la brecha ya ocurrió o es inminente.                               |

## 🏢 Ejemplo Práctico: Acceso a un Servicio Interno

- **Escenario Tradicional**: Un desarrollador se conecta a la VPN de la empresa. Una vez dentro, tiene acceso a múltiples servicios internos (Jenkins, SonarQube, base de datos de desarrollo) porque su IP está en un rango "de confianza".
- **Escenario Zero Trust**:
  1. El desarrollador intenta acceder a Jenkins.
  2. Un **proveedor de identidad** (ej. Azure AD, Okta) intercepta la solicitud.
  3. Se verifica la **identidad** del desarrollador (MFA obligatorio).
  4. Se comprueba la **salud del dispositivo** (¿está gestionado por la empresa? ¿tiene el antivirus activo?).
  5. Una **política de acceso condicional** evalúa el contexto (¿es una ubicación conocida? ¿es un horario laboral?).
  6. Si todas las comprobaciones son exitosas, se concede un token de acceso de corta duración **solo para Jenkins**. El acceso a la base de datos requeriría una nueva evaluación.

## ✍️ Ejercicio

Describe cómo aplicarías el principio de "verificar explícitamente" para asegurar el acceso a un clúster de Kubernetes. ¿Qué elementos verificarías antes de permitir que un usuario o servicio ejecute `kubectl apply`?
