# Módulo 13: Laboratorio Final - Arquitectura de Referencia Multi-Cloud

## Objetivo

Diseñar y desplegar una aplicación de e-commerce de 3 capas en un proveedor cloud, con un plan de migración a otro proveedor.

## Arquitectura

- **Frontend:** Aplicación web en PaaS (Azure App Service / AWS Elastic Beanstalk)
- **Backend:** API de microservicios en un cluster de Kubernetes gestionado (AKS / EKS)
- **Datos:** Base de datos SQL gestionada (Azure SQL / AWS RDS) y almacenamiento de objetos (Blob Storage / S3)
- **Mensajería:** Cola para procesar pedidos (Service Bus / SQS)
- **Seguridad:** Secretos en Key Vault / Secrets Manager
- **Observabilidad:** Métricas y logs en Azure Monitor / CloudWatch

## Tareas

1. **Diseño:** Dibuja la arquitectura y define los recursos a usar.
2. **IaC:** Escribe el código Terraform/Bicep para desplegar la infraestructura.
3. **CI/CD:** Crea un pipeline en GitHub Actions para desplegar la aplicación.
4. **Observabilidad:** Configura dashboards y alertas para monitorear la salud.
5. **Plan de Migración:** Documenta los pasos para migrar la aplicación a otro proveedor cloud, mapeando los servicios equivalentes.

## Evaluación

- Calidad del diseño de la arquitectura
- Correcta implementación con IaC
- Pipeline de CI/CD funcional
- Dashboards de observabilidad útiles
- Claridad y viabilidad del plan de migración
