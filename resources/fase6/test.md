# Evaluación Final - Fase 6: Nube y Arquitectura Moderna

## Parte 1: Preguntas Teóricas (40 puntos)

1. Describe la diferencia entre IaaS, PaaS y SaaS y da un ejemplo de cada uno. (5 pts)
2. ¿Qué es el modelo de responsabilidad compartida en la nube? (5 pts)
3. Explica la diferencia entre alta disponibilidad (multi-AZ) y recuperación ante desastres (multi-región). (5 pts)
4. ¿Qué es una identidad administrada y por qué es preferible a usar claves de acceso? (5 pts)
5. Describe un patrón de arquitectura event-driven y menciona un servicio de mensajería que lo facilite. (5 pts)
6. ¿Qué es FinOps y por qué es importante en la nube? (5 pts)
7. ¿Qué es el modelo Zero Trust? (5 pts)
8. ¿Cuál es la diferencia entre RTO y RPO? (5 pts)

## Parte 2: Ejercicios Prácticos (60 puntos)

1. **Diseño de Arquitectura (20 pts):**
   - Diseña una arquitectura para una aplicación web que debe ser altamente disponible y escalable. Debe incluir un balanceador de carga, un grupo de servidores web, una base de datos y una red virtual con subredes pública y privada. Dibuja el diagrama.

2. **Infraestructura como Código (20 pts):**
   - Escribe un fragmento de código Terraform o Bicep para crear una red virtual con dos subredes (una pública y una privada).

3. **CI/CD (10 pts):**
   - Describe los pasos principales de un pipeline de CI/CD para desplegar una aplicación en un servicio PaaS (ej. Azure App Service).

4. **Observabilidad (10 pts):**
   - Escribe una consulta (usando KQL o similar) para encontrar todos los logs de error (nivel "Error" o "Critical") de una aplicación en las últimas 24 horas.
