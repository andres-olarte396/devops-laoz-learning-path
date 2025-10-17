# Módulo 01: Principios de Seguridad de la Información (CIA Triad)

## 🎯 Objetivos

- Comprender los tres pilares de la seguridad de la información: Confidencialidad, Integridad y Disponibilidad.
- Identificar cómo se aplica la tríada CIA en arquitecturas de software y sistemas DevOps.
- Analizar amenazas comunes que afectan a cada pilar.

## 📜 Contenido

### 1. Confidencialidad (Confidentiality)

- **Definición**: Asegurar que la información solo sea accesible para personal autorizado. Prevenir la divulgación no autorizada de datos sensibles.
- **Mecanismos de Control**:
  - **Cifrado (Encryption)**: Cifrado en tránsito (TLS/SSL) y en reposo (AES-256).
  - **Control de Acceso**: Autenticación (quién eres), autorización (qué puedes hacer) y auditoría (qué hiciste). RBAC, ABAC.
  - **Clasificación de Datos**: Etiquetar datos según su sensibilidad (público, interno, confidencial, restringido).
- **Amenazas**: Interceptación de datos (Man-in-the-Middle), robo de credenciales, ingeniería social.

### 2. Integridad (Integrity)

- **Definición**: Proteger la información contra modificaciones no autorizadas o no intencionadas. Garantizar que los datos sean precisos y consistentes.
- **Mecanismos de Control**:
  - **Hashing**: Generar hashes (SHA-256, MD5) para verificar la integridad de archivos y mensajes.
  - **Firmas Digitales**: Usar criptografía asimétrica para verificar la autenticidad y la integridad del remitente.
  - **Control de Versiones (Git)**: Mantener un historial inmutable de cambios en el código y la configuración.
  - **Permissions**: Limitar permisos de escritura a usuarios y servicios autorizados.
- **Amenazas**: Alteración de datos en tránsito, corrupción de archivos, acceso no autorizado con privilegios de escritura.

### 3. Disponibilidad (Availability)

- **Definición**: Asegurar que los sistemas y los datos estén operativos y accesibles para los usuarios autorizados cuando se necesiten.
- **Mecanismos de Control**:
  - **Redundancia**: Componentes redundantes (servidores, bases de datos, redes) para eliminar puntos únicos de fallo.
  - **Alta Disponibilidad (HA)**: Clusters, balanceo de carga y failover automático.
  - **Recuperación ante Desastres (DR)**: Backups regulares y planes de recuperación probados (RTO/RPO).
  - **Protección contra DDoS**: Servicios de mitigación de ataques de denegación de servicio.
- **Amenazas**: Ataques DDoS, fallos de hardware, errores de software, desastres naturales.

## 🏢 Ejemplo Práctico en un Pipeline de CI/CD

- **Confidencialidad**:
  - Los secretos (tokens, claves API) se almacenan en HashiCorp Vault, no en el código fuente.
  - El acceso al pipeline está restringido por roles en Jenkins/GitHub Actions.
- **Integridad**:
  - Cada commit de Git tiene un hash único.
  - Las imágenes de Docker se firman con Docker Content Trust.
  - Los artefactos de compilación se verifican con checksums (SHA-256).
- **Disponibilidad**:
  - El pipeline se ejecuta en agentes distribuidos para evitar un punto único de fallo.
  - Los despliegues se realizan con estrategias como Blue/Green para minimizar el tiempo de inactividad.

## ✍️ Ejercicio

Analiza una aplicación que uses a diario (ej. tu email, una red social) y describe un mecanismo de control para cada pilar de la tríada CIA que observes.

- **Confidencialidad**: ¿Cómo protege tus datos privados?
- **Integridad**: ¿Cómo asegura que los mensajes que envías no sean alterados?
- **Disponibilidad**: ¿Qué sucede si un servidor falla? ¿Cómo se mantiene el servicio activo?
