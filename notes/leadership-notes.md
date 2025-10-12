# Notas de Liderazgo Técnico y Habilidades Blandas �

## Fundamentos del Liderazgo Técnico

### Roles y Responsabilidades

#### Technical Lead vs Engineering Manager
- **Technical Lead**: Se enfoca en decisiones técnicas, arquitectura, mentorización técnica
- **Engineering Manager**: Se enfoca en gestión de personas, procesos, objetivos de negocio
- **Tech Lead Manager**: Híbrido que combina ambos roles

#### Competencias Clave del Líder Técnico
1. **Competencia técnica profunda**
2. **Habilidades de comunicación**
3. **Capacidad de toma de decisiones**
4. **Visión arquitectónica**
5. **Mentoría y desarrollo de equipo**

### Comunicación Efectiva

#### Técnicas de Feedback Constructivo
```
Modelo SBI (Situation-Behavior-Impact):
- Situación: "En la reunión de arquitectura de ayer..."
- Comportamiento: "...interrumpiste varias veces a los miembros del equipo..."
- Impacto: "...lo que causó que algunas ideas no se expresaran completamente"
```

#### Storytelling Técnico
1. **Context**: Establecer el contexto del problema
2. **Action**: Describir las acciones tomadas
3. **Result**: Mostrar los resultados y lecciones aprendidas
4. **Learning**: Extraer insights para el futuro

#### Comunicación con Stakeholders No Técnicos
- Usar analogías y metáforas
- Cuantificar impacto en términos de negocio
- Evitar jerga técnica innecesaria
- Preparar diferentes niveles de detalle

### Toma de Decisiones Técnicas

#### Framework para Decisiones Arquitectónicas
```
ADR (Architecture Decision Record):

# ADR-001: Elección de Base de Datos

## Status
Accepted

## Context
Necesitamos una base de datos para el nuevo microservicio de usuarios...

## Decision
Utilizaremos PostgreSQL como base de datos principal...

## Consequences
Positivas:
- ACID compliance
- Rico ecosistema de herramientas

Negativas:
- Mayor complejidad de setup
- Costo adicional de licenciamiento enterprise
```

#### Matriz de Decisión
| Criterio | Peso | Opción A | Opción B | Opción C |
|----------|------|----------|----------|----------|
| Rendimiento | 40% | 8/10 | 6/10 | 9/10 |
| Mantenibilidad | 30% | 7/10 | 9/10 | 6/10 |
| Costo | 20% | 5/10 | 8/10 | 7/10 |
| Time to Market | 10% | 9/10 | 7/10 | 5/10 |

#### Gestión de Technical Debt
1. **Categorizar deuda técnica**:
   - Prudent & Deliberate: Decisiones conscientes
   - Reckless & Deliberate: Atajos conscientes
   - Prudent & Inadvertent: Aprendizaje posterior
   - Reckless & Inadvertent: Falta de conocimiento

2. **Priorización**:
   - Impacto en velocidad de desarrollo
   - Riesgo de producción
   - Costo de mantenimiento

### Mentoría y Desarrollo de Equipo

#### Técnicas de Mentoría
1. **Active Listening**: Escuchar para entender, no para responder
2. **Socratic Method**: Hacer preguntas que guíen al aprendizaje
3. **Code Reviews constructivos**: Enfocarse en el aprendizaje
4. **Pair Programming**: Transferencia directa de conocimiento

#### Plan de Desarrollo Individual
```markdown
# Plan de Desarrollo - [Nombre del Desarrollador]

## Objetivo a 6 meses
Convertirse en desarrollador senior con capacidad de mentorizar juniors

## Competencias a Desarrollar
1. **Técnicas**:
   - Patrones de diseño avanzados
   - Testing strategies
   - Performance optimization

2. **Blandas**:
   - Presentaciones técnicas
   - Code review skills
   - Collaborative problem solving

## Hitos y Métricas
- [ ] Liderar diseño de 2 features complejas
- [ ] Mentorizar 1 desarrollador junior
- [ ] Presentar 1 tech talk interno
```

#### Gestión de Conflictos en Equipos Técnicos

**Tipos comunes de conflictos**:
1. **Técnicos**: Desacuerdos sobre tecnologías o enfoques
2. **Proceso**: Diferencias en metodologías de trabajo
3. **Interpersonales**: Choques de personalidad o estilos

**Estrategias de resolución**:
1. **Separar persona de problema**
2. **Enfocarse en datos objetivos**
3. **Buscar intereses comunes**
4. **Prototipar soluciones cuando sea viable**

### Pensamiento Sistémico

#### Análisis de Sistemas Complejos
```
Modelo de Iceberg:
- Eventos (what happened?)
- Patrones/Tendencias (what trends?)
- Estructuras (what influences patterns?)
- Modelos Mentales (what beliefs create structures?)
```

#### Mapeo de Stakeholders
```
Matriz de Influencia vs Interés:
- Alto Interés, Alta Influencia: Gestionar de cerca
- Alto Interés, Baja Influencia: Mantener informado
- Bajo Interés, Alta Influencia: Mantener satisfecho
- Bajo Interés, Baja Influencia: Monitorear
```

#### Gestión del Cambio Organizacional

**Modelo de Kotter (8 pasos)**:
1. Crear sentido de urgencia
2. Formar coalición poderosa
3. Desarrollar visión y estrategia
4. Comunicar la visión
5. Empoderar acción amplia
6. Generar triunfos a corto plazo
7. Consolidar ganancias y producir más cambio
8. Anclar nuevos enfoques en la cultura

### Herramientas y Frameworks

#### Técnicas de Facilitación
1. **Design Thinking Workshops**:
   - Empathize → Define → Ideate → Prototype → Test

2. **Event Storming**:
   - Mapear eventos de dominio
   - Identificar agregados y bounded contexts
   - Descubrir procesos de negocio

3. **Architecture Review Sessions**:
   - ATAM (Architecture Tradeoff Analysis Method)
   - QAW (Quality Attribute Workshop)

#### Métricas de Equipo y Performance
```csharp
// Ejemplo de tracking de métricas de equipo
public class TeamMetrics
{
    public class SprintMetrics
    {
        public double Velocity { get; set; }
        public double BurndownAccuracy { get; set; }
        public int DefectEscapeRate { get; set; }
        public TimeSpan AverageLeadTime { get; set; }
        public double CodeCoverage { get; set; }
    }

    public class TeamHealthMetrics
    {
        public double PsychologicalSafety { get; set; } // Survey score 1-10
        public double TeamSatisfaction { get; set; }
        public double LearningGoalProgress { get; set; }
        public int Knowledge SharingEvents { get; set; }
    }
}
```

### Comunicación Estratégica

#### Presentaciones para Ejecutivos
**Estructura recomendada**:
1. **Executive Summary** (30 segundos)
2. **Problema/Oportunidad** (1 minuto)
3. **Solución Propuesta** (2 minutos)
4. **Implementación y Timeline** (1 minuto)
5. **ROI y Métricas** (1 minuto)
6. **Riesgos y Mitigación** (30 segundos)

#### Negociación Técnica
**Principios clave**:
1. **Separar personas de posiciones**
2. **Enfocarse en intereses, no posiciones**
3. **Generar opciones de mutuo beneficio**
4. **Usar criterios objetivos**

### Desarrollo de Visión Arquitectónica

#### Architecture Vision Canvas
```
[Stakeholders] | [Business Goals]
[Quality Attributes] | [Key Concerns]
[Architecture Principles] | [Implementation Strategy]
[Success Criteria] | [Risks & Trade-offs]
```

#### Technology Radar
```
Adopt:
- Kubernetes
- .NET 8
- Azure DevOps

Trial:
- Blazor Server
- gRPC
- Feature Flags

Assess:
- WebAssembly
- GraphQL
- Event Sourcing

Hold:
- Legacy frameworks
- Monolithic deployments
```

### Best Practices de Liderazgo

#### 1-on-1 Meetings Efectivos
**Agenda sugerida**:
1. **Personal check-in** (5 min)
2. **Progress on goals** (10 min)
3. **Challenges and blockers** (10 min)
4. **Learning and development** (10 min)
5. **Feedback bidireccional** (5 min)

#### Building Psychological Safety
1. **Admitir ignorancia e incertidumbre**
2. **Hacer preguntas activamente**
3. **Modelar curiosidad**
4. **Celebrar intentos y aprendizajes de fallos**

#### Influence Without Authority
**Estrategias**:
1. **Reciprocidad**: Hacer favores primero
2. **Consistencia**: Align con valores declarados
3. **Social Proof**: Mostrar que otros lo hacen
4. **Expertise**: Demonstrar competencia técnica
5. **Liking**: Construir relaciones genuinas

### Medición del Impacto de Liderazgo

#### KPIs de Liderazgo Técnico
1. **Team Velocity**: Puntos de historia por sprint
2. **Quality**: Defects per release, code coverage
3. **Innovation**: Ideas implementadas, tech talks
4. **Retention**: Team member retention rate
5. **Growth**: Promotions, skills development

#### 360-Degree Feedback
```
Áreas de evaluación:
- Technical Excellence
- Communication Skills
- Decision Making
- Team Development
- Strategic Thinking
- Change Management
```

### Recursos Recomendados

#### Libros Fundamentales
- "The Manager's Path" - Camille Fournier
- "Staff Engineer" - Will Larson
- "Team Topologies" - Matthew Skelton
- "Crucial Conversations" - Kerry Patterson
- "The First 90 Days" - Michael Watkins

#### Frameworks y Metodologías
- OKRs (Objectives and Key Results)
- SMART Goals
- DISC Assessment
- Situational Leadership
- Lean Startup Methodology

#### Comunidades y Networking
- Local tech meetups
- Conference speaking
- Open source contributions
- Technical blogs and articles
- Mentorship programs