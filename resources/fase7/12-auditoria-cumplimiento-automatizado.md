# Módulo 12: Auditoría y Cumplimiento Automatizado

## 🎯 Objetivos

- Comprender la diferencia entre auditoría de seguridad y cumplimiento normativo.
- Aprender a automatizar la recolección de evidencia para auditorías.
- Utilizar herramientas para monitorear y hacer cumplir políticas de cumplimiento de forma continua.

## 📜 Contenido

### 1. Auditoría vs. Cumplimiento

- **Auditoría de Seguridad**: Es un proceso técnico para evaluar la eficacia de los controles de seguridad de una organización. Se enfoca en encontrar vulnerabilidades y debilidades.
- **Cumplimiento (Compliance)**: Es el proceso de adherirse a un conjunto de reglas, políticas, estándares o leyes definidas por un tercero. Se enfoca en demostrar que se siguen las reglas.
  - **Ejemplos de Marcos de Cumplimiento**:
    - **PCI DSS**: Para organizaciones que manejan tarjetas de crédito.
    - **HIPAA**: Para organizaciones que manejan información de salud en EE.UU.
    - **GDPR**: Regulación de protección de datos en la Unión Europea.
    - **ISO 27001**: Un estándar internacional para la gestión de la seguridad de la información.

### 2. El Desafío del Cumplimiento en Entornos DevOps y Cloud

- **Entornos Dinámicos**: La infraestructura cambia constantemente, lo que hace que las auditorías manuales y puntuales sean obsoletas rápidamente.
- **Velocidad**: Los ciclos de despliegue rápidos dificultan la verificación manual del cumplimiento de cada cambio.
- **Complejidad**: La combinación de microservicios, contenedores y múltiples servicios en la nube crea una superficie de cumplimiento muy amplia.

La solución es el **Cumplimiento Continuo**: integrar la validación del cumplimiento en el pipeline de DevOps.

### 3. Automatización de la Recolección de Evidencia

Una auditoría requiere **evidencia** de que los controles están en su lugar y funcionando. En un entorno DevOps, esta evidencia se puede generar automáticamente.

| Control de Cumplimiento                                                              | Evidencia Automatizada                                                                            |
| :----------------------------------------------------------------------------------- | :------------------------------------------------------------------------------------------------ |
| "El acceso a servidores de producción está restringido"                              | Logs de auditoría de un servidor bastión o de HashiCorp Vault.                                    |
| "Todas las dependencias de software deben estar libres de vulnerabilidades críticas" | Reportes de una herramienta de SCA (Trivy, Snyk) ejecutada en el pipeline.                        |
| "Los cambios en la infraestructura deben ser aprobados"                              | Historial de Pull Requests en Git, mostrando las aprobaciones requeridas.                         |
| "Los datos sensibles deben estar cifrados en reposo"                                 | Reportes de una herramienta de escaneo de IaC (Checkov) que verifica la configuración de cifrado. |

### 4. Herramientas para el Cumplimiento Continuo

- **InSpec**: Un framework de código abierto de Chef para escribir pruebas de cumplimiento como código. Permite definir el estado deseado de un sistema (ej. "el puerto 22 no debe estar abierto") y verificarlo.
- **Open Policy Agent (OPA)**: Un motor de políticas de propósito general de código abierto. Permite desacoplar la lógica de las políticas del código de la aplicación. Se puede usar para hacer cumplir políticas en Kubernetes, microservicios, etc.
- **Herramientas de CSPM (Cloud Security Posture Management)**:
  - **Definición**: Escanean continuamente los entornos de nube (AWS, Azure, GCP) en busca de configuraciones incorrectas que violen las mejores prácticas de seguridad o los marcos de cumplimiento.
  - **Ejemplos**: AWS Security Hub, Azure Defender for Cloud, Prisma Cloud.

## 🏢 Ejemplo Práctico: Usar InSpec para Auditar un Servidor

InSpec utiliza un lenguaje legible para definir los controles.

**Control de InSpec (`sshd_spec.rb`):**

```ruby
control 'sshd-01' do
  impact 1.0
  title 'sshd: PermitRootLogin should be no'
  desc 'Disallow root login to prevent brute-force attacks.'

  describe sshd_config do
    its('PermitRootLogin') { should cmp 'no' }
  end
end

control 'sshd-02' do
  impact 0.7
  title 'sshd: Protocol should be 2'
  desc 'Use the more secure SSH Protocol 2.'

  describe sshd_config do
    its('Protocol') { should cmp '2' }
  end
end
```

- **Ejecución**: Se puede ejecutar este "perfil" de InSpec contra un servidor local o remoto.

  ```bash
  inspec exec sshd_spec.rb -t ssh://user@hostname
  ```

- **Resultado**: InSpec generará un reporte indicando qué controles pasaron y cuáles fallaron, proporcionando evidencia clara para una auditoría.

## ✍️ Ejercicio

Elige uno de los siguientes requisitos de cumplimiento comunes:

1. "Todos los buckets de S3 deben tener el acceso público bloqueado".
2. "Todas las máquinas virtuales deben pertenecer a una red virtual (VNet/VPC), no tener IP pública".
3. "La autenticación multifactor (MFA) debe estar habilitada para todos los usuarios de IAM con acceso a la consola".

Describe cómo podrías automatizar la verificación de este requisito usando una de las herramientas o enfoques mencionados en este módulo (IaC scanning, CSPM, etc.).
