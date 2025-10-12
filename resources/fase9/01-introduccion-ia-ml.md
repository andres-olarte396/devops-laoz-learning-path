# Introducción a la Inteligencia Artificial y Machine Learning

## Objetivos
Al completar este módulo serás capaz de:
- Comprender los conceptos fundamentales de IA y ML
- Identificar casos de uso apropiados para IA/ML en DevOps
- Implementar modelos básicos con ML.NET
- Integrar servicios de IA en pipelines de CI/CD

## ¿Qué es la Inteligencia Artificial?

La **Inteligencia Artificial (IA)** es un campo de la informática que busca crear sistemas capaces de realizar tareas que normalmente requieren inteligencia humana.

### Conceptos Clave

**Machine Learning (ML)**: Subcampo de la IA que permite a las máquinas aprender sin ser explícitamente programadas.

**Deep Learning**: Subcampo del ML que usa redes neuronales artificiales con múltiples capas.

**Natural Language Processing (NLP)**: Capacidad de las máquinas para entender y procesar el lenguaje humano.

## ML.NET: Machine Learning para .NET

ML.NET es un framework de machine learning multiplataforma y de código abierto para desarrolladores .NET.

### Instalación

```bash
# Instalar ML.NET CLI
dotnet tool install -g mlnet

# Instalar paquete NuGet
dotnet add package Microsoft.ML
```

### Primer Modelo: Predicción de Precios

```csharp
using Microsoft.ML;
using Microsoft.ML.Data;

// Definir los datos de entrada
public class HouseData
{
    [LoadColumn(0)]
    public float Size { get; set; }

    [LoadColumn(1)]
    public float Price { get; set; }
}

// Definir la predicción
public class Prediction
{
    [ColumnName("Score")]
    public float Price { get; set; }
}

class Program
{
    static void Main(string[] args)
    {
        // Crear contexto ML
        var mlContext = new MLContext(seed: 0);

        // Cargar datos
        var data = mlContext.Data.LoadFromTextFile<HouseData>(
            path: "housing.csv",
            hasHeader: true,
            separatorChar: ','
        );

        // Definir pipeline de entrenamiento
        var pipeline = mlContext.Transforms.Concatenate("Features", new[] { "Size" })
            .Append(mlContext.Regression.Trainers.Sdca(labelColumnName: "Price", maximumNumberOfIterations: 100));

        // Entrenar modelo
        var model = pipeline.Fit(data);

        // Crear predictor
        var predictionFunction = mlContext.Model.CreatePredictionEngine<HouseData, Prediction>(model);

        // Realizar predicción
        var prediction = predictionFunction.Predict(new HouseData { Size = 1200f });
        Console.WriteLine($"Precio predicho: ${prediction.Price:C}");

        // Guardar modelo
        mlContext.Model.Save(model, data.Schema, "model.zip");
    }
}
```

## IA en DevOps: AIOps

AIOps (Artificial Intelligence for IT Operations) usa IA para mejorar las operaciones de TI.

### Casos de Uso en DevOps

**1. Detección de Anomalías**
```csharp
public class AnomalyDetectionService
{
    private readonly MLContext _mlContext;
    private ITransformer _model;

    public AnomalyDetectionService()
    {
        _mlContext = new MLContext();
        TrainAnomalyDetectionModel();
    }

    private void TrainAnomalyDetectionModel()
    {
        // Datos de métricas del sistema
        var data = _mlContext.Data.LoadFromEnumerable(GetSystemMetrics());

        // Pipeline para detección de anomalías
        var pipeline = _mlContext.Transforms.DetectAnomaliesBySrCnn(
            outputColumnName: "Anomaly",
            inputColumnName: "Value",
            threshold: 0.25,
            batchSize: 512,
            sensitivity: 99.0,
            detectMode: SrCnnDetectMode.AnomalyAndMargin
        );

        _model = pipeline.Fit(data);
    }

    public bool IsAnomaly(float metricValue)
    {
        var input = new MetricData { Value = metricValue };
        var prediction = _mlContext.Model.CreatePredictionEngine<MetricData, AnomalyPrediction>(_model);
        var result = prediction.Predict(input);

        return result.Anomaly[0] == 1;
    }
}

public class MetricData
{
    public float Value { get; set; }
}

public class AnomalyPrediction
{
    [VectorType(3)]
    public double[] Anomaly { get; set; }
}
```

**2. Predicción de Fallos**
```csharp
public class FailurePredictionService
{
    private readonly MLContext _mlContext;
    private ITransformer _model;

    public void TrainFailurePredictionModel()
    {
        _mlContext = new MLContext();

        // Cargar datos históricos de fallos
        var data = _mlContext.Data.LoadFromTextFile<SystemHealth>(
            "system_health.csv", hasHeader: true, separatorChar: ','
        );

        // División en entrenamiento y prueba
        var split = _mlContext.Data.TrainTestSplit(data, testFraction: 0.2);

        // Pipeline de entrenamiento
        var pipeline = _mlContext.Transforms.Concatenate("Features",
                "CpuUsage", "MemoryUsage", "DiskUsage", "NetworkLatency")
            .Append(_mlContext.BinaryClassification.Trainers.SdcaLogisticRegression());

        // Entrenar modelo
        _model = pipeline.Fit(split.TrainSet);

        // Evaluar modelo
        var predictions = _model.Transform(split.TestSet);
        var metrics = _mlContext.BinaryClassification.Evaluate(predictions);

        Console.WriteLine($"Accuracy: {metrics.Accuracy:P2}");
        Console.WriteLine($"F1 Score: {metrics.F1Score:P2}");
    }

    public float PredictFailureProbability(SystemHealth health)
    {
        var predictor = _mlContext.Model.CreatePredictionEngine<SystemHealth, FailurePrediction>(_model);
        var prediction = predictor.Predict(health);
        return prediction.Probability;
    }
}

public class SystemHealth
{
    [LoadColumn(0)]
    public float CpuUsage { get; set; }

    [LoadColumn(1)]
    public float MemoryUsage { get; set; }

    [LoadColumn(2)]
    public float DiskUsage { get; set; }

    [LoadColumn(3)]
    public float NetworkLatency { get; set; }

    [LoadColumn(4)]
    public bool WillFail { get; set; }
}

public class FailurePrediction
{
    [ColumnName("PredictedLabel")]
    public bool PredictedFailure { get; set; }

    [ColumnName("Probability")]
    public float Probability { get; set; }
}
```

## Integración con Azure Cognitive Services

### Análisis de Texto para Logs

```csharp
using Azure.AI.TextAnalytics;

public class LogAnalysisService
{
    private readonly TextAnalyticsClient _client;

    public LogAnalysisService(string endpoint, string apiKey)
    {
        _client = new TextAnalyticsClient(new Uri(endpoint), new AzureKeyCredential(apiKey));
    }

    public async Task<LogAnalysisResult> AnalyzeLogAsync(string logMessage)
    {
        // Análisis de sentimiento
        var sentimentResponse = await _client.AnalyzeSentimentAsync(logMessage);

        // Extracción de entidades
        var entitiesResponse = await _client.RecognizeEntitiesAsync(logMessage);

        // Extracción de frases clave
        var keyPhrasesResponse = await _client.ExtractKeyPhrasesAsync(logMessage);

        return new LogAnalysisResult
        {
            Sentiment = sentimentResponse.Value.Sentiment.ToString(),
            ConfidenceScore = sentimentResponse.Value.ConfidenceScores.Positive,
            Entities = entitiesResponse.Value.Select(e => e.Text).ToList(),
            KeyPhrases = keyPhrasesResponse.Value.ToList(),
            SeverityLevel = DetermineSeverity(sentimentResponse.Value.Sentiment)
        };
    }

    private string DetermineSeverity(TextSentiment sentiment)
    {
        return sentiment switch
        {
            TextSentiment.Negative => "ERROR",
            TextSentiment.Neutral => "WARNING",
            TextSentiment.Positive => "INFO",
            _ => "UNKNOWN"
        };
    }
}

public class LogAnalysisResult
{
    public string Sentiment { get; set; }
    public double ConfidenceScore { get; set; }
    public List<string> Entities { get; set; }
    public List<string> KeyPhrases { get; set; }
    public string SeverityLevel { get; set; }
}
```

## Automatización Inteligente de Pipelines

### Pipeline que Aprende de Fallos

```yaml
# Azure DevOps Pipeline con IA
trigger:
- main

variables:
  buildConfiguration: 'Release'

stages:
- stage: Analyze
  jobs:
  - job: FailurePrediction
    pool:
      vmImage: 'ubuntu-latest'
    steps:
    - task: DotNetCoreCLI@2
      displayName: 'Predicir probabilidad de fallo'
      inputs:
        command: 'run'
        projects: 'tools/FailurePrediction/FailurePrediction.csproj'
        arguments: '--build-metrics $(Build.Repository.Name) $(Build.SourceBranch)'

    - task: PowerShell@2
      displayName: 'Decidir estrategia de build'
      inputs:
        targetType: 'inline'
        script: |
          $failureProbability = Get-Content 'failure_probability.txt'
          if ([float]$failureProbability -gt 0.7) {
            Write-Host "##vso[task.setvariable variable=useIntensiveTesting]true"
            Write-Host "Alta probabilidad de fallo detectada. Activando testing intensivo."
          } else {
            Write-Host "##vso[task.setvariable variable=useIntensiveTesting]false"
          }

- stage: Build
  dependsOn: Analyze
  jobs:
  - job: StandardBuild
    condition: ne(variables['useIntensiveTesting'], 'true')
    steps:
    - script: echo "Build estándar"

  - job: IntensiveBuild
    condition: eq(variables['useIntensiveTesting'], 'true')
    steps:
    - script: echo "Build con testing extensivo"
    - task: DotNetCoreCLI@2
      inputs:
        command: 'test'
        projects: '**/*Tests.csproj'
        arguments: '--configuration $(buildConfiguration) --collect:"XPlat Code Coverage"'
```

## Chatbots para DevOps

### Bot de Soporte con OpenAI

```csharp
using OpenAI.GPT3;
using OpenAI.GPT3.Managers;
using OpenAI.GPT3.ObjectModels.RequestModels;

public class DevOpsChatBot
{
    private readonly OpenAIService _openAIService;

    public DevOpsChatBot(string apiKey)
    {
        _openAIService = new OpenAIService(new OpenAiOptions()
        {
            ApiKey = apiKey
        });
    }

    public async Task<string> GetDevOpsHelpAsync(string question, string context)
    {
        var completionResult = await _openAIService.ChatCompletion.CreateCompletion(
            new ChatCompletionCreateRequest
            {
                Messages = new List<ChatMessage>
                {
                    new ChatMessage("system", GetSystemPrompt()),
                    new ChatMessage("user", $"Contexto: {context}\n\nPregunta: {question}")
                },
                Model = OpenAI.GPT3.ObjectModels.Models.Gpt_3_5_Turbo,
                MaxTokens = 500
            });

        if (completionResult.Successful)
        {
            return completionResult.Choices.First().Message.Content;
        }

        return "Lo siento, no pude procesar tu consulta en este momento.";
    }

    private string GetSystemPrompt()
    {
        return @"Eres un asistente especializado en DevOps y desarrollo de software.
                Ayudas a desarrolladores y equipos de operaciones con:
                - Troubleshooting de pipelines CI/CD
                - Configuración de Docker y Kubernetes
                - Mejores prácticas de seguridad
                - Optimización de procesos DevOps
                - Análisis de logs y métricas

                Proporciona respuestas precisas, con ejemplos de código cuando sea apropiado,
                y siempre menciona las mejores prácticas de seguridad.";
    }
}

// Integración con Microsoft Teams
public class TeamsDevOpsBot : ActivityHandler
{
    private readonly DevOpsChatBot _chatBot;

    public TeamsDevOpsBot(DevOpsChatBot chatBot)
    {
        _chatBot = chatBot;
    }

    protected override async Task OnMessageActivityAsync(ITurnContext<IMessageActivity> turnContext, CancellationToken cancellationToken)
    {
        var userMessage = turnContext.Activity.Text;
        var context = await GetUserContextAsync(turnContext);

        var response = await _chatBot.GetDevOpsHelpAsync(userMessage, context);

        await turnContext.SendActivityAsync(MessageFactory.Text(response), cancellationToken);
    }

    private async Task<string> GetUserContextAsync(ITurnContext turnContext)
    {
        // Obtener contexto del usuario (proyecto actual, rol, etc.)
        return "Usuario trabajando en proyecto .NET con Azure DevOps";
    }
}
```

## Métricas y Monitoreo de IA

### Dashboard de IA para DevOps

```csharp
public class AIMetricsService
{
    private readonly IMetricsCollector _metrics;

    public void RecordModelPrediction(string modelName, float confidence, bool accurate)
    {
        _metrics.RecordValue("ml_prediction_confidence", confidence, new { model = modelName });
        _metrics.Increment("ml_predictions_total", new { model = modelName, accurate = accurate.ToString() });
    }

    public void RecordAnomalyDetection(string system, bool anomalyDetected, bool confirmed)
    {
        _metrics.Increment("anomaly_detections_total", new {
            system = system,
            detected = anomalyDetected.ToString(),
            confirmed = confirmed.ToString()
        });
    }

    public void RecordChatBotInteraction(string intent, bool resolved)
    {
        _metrics.Increment("chatbot_interactions_total", new {
            intent = intent,
            resolved = resolved.ToString()
        });
    }
}
```

## Ejercicios Prácticos

### Ejercicio 1: Modelo de Predicción
Crea un modelo ML.NET que prediga el tiempo de build basado en:
- Número de archivos cambiados
- Tamaño del proyecto
- Complejidad de las pruebas

### Ejercicio 2: Detección de Anomalías
Implementa un sistema que detecte anomalías en:
- Tiempo de respuesta de API
- Uso de memoria
- Número de errores por minuto

### Ejercicio 3: ChatBot DevOps
Desarrolla un chatbot que pueda:
- Analizar logs de error
- Sugerir soluciones para fallos comunes
- Proporcionar comandos de troubleshooting

## Consideraciones Éticas y de Seguridad

### Privacidad de Datos
```csharp
public class PrivacyPreservingML
{
    public void SanitizeData(List<LogEntry> logs)
    {
        foreach (var log in logs)
        {
            // Remover información personal identificable
            log.Message = RemovePII(log.Message);
            log.UserId = HashUserId(log.UserId);
            log.IPAddress = AnonymizeIP(log.IPAddress);
        }
    }

    private string RemovePII(string message)
    {
        // Usar regex para remover emails, números de teléfono, etc.
        var emailPattern = @"\b[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\.[A-Z|a-z]{2,}\b";
        return Regex.Replace(message, emailPattern, "[EMAIL_REMOVED]");
    }
}
```

### Explicabilidad del Modelo
```csharp
public class ModelExplainability
{
    public ModelInsights ExplainPrediction(ITransformer model, MLContext mlContext, object input)
    {
        // Usar LIME (Local Interpretable Model-agnostic Explanations)
        var explainer = mlContext.Model.Explainability.PermutationFeatureImportance(model);

        return new ModelInsights
        {
            FeatureImportance = explainer.Select(f => new FeatureImportance
            {
                Feature = f.Key,
                Importance = f.Value.Mean
            }).ToList()
        };
    }
}
```

## Checklist de IA en DevOps

- [ ] Identificar casos de uso apropiados para IA/ML
- [ ] Implementar modelos con datos limpios y representativos
- [ ] Establecer métricas de calidad del modelo
- [ ] Implementar monitoreo de deriva del modelo
- [ ] Considerar implicaciones éticas y de privacidad
- [ ] Documentar decisiones del modelo
- [ ] Establecer procesos de re-entrenamiento
- [ ] Integrar IA en pipelines existentes

## Recursos Adicionales
- [ML.NET Documentation](https://docs.microsoft.com/en-us/dotnet/machine-learning/)
- [Azure Cognitive Services](https://azure.microsoft.com/en-us/services/cognitive-services/)
- [OpenAI API Documentation](https://platform.openai.com/docs)

## Siguiente Paso
[Natural Language Processing (NLP)](./02-nlp-procesamiento-lenguaje-natural.md)