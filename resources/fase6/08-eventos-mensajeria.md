# Módulo 08: Eventos y Mensajería

## Objetivos

- Diseñar arquitecturas event-driven
- Usar colas para desacoplar servicios
- Implementar patrones de mensajería (pub/sub, fan-out)

## Contenido

- Azure Service Bus/Event Grid, AWS SQS/SNS, Google Pub/Sub
- Patrones: Saga, Choreography, CQRS
- Garantías de entrega (at-least-once, at-most-once)
- Dead-letter queues y reintentos

## Ejercicio

- Implementa un patrón pub/sub donde un servicio publica un evento y dos servicios diferentes lo consumen.
