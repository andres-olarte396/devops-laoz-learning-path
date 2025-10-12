# Notas de Inteligencia Artificial para Desarrolladores 

## Fundamentos de Machine Learning

### Tipos de Aprendizaje

#### Aprendizaje Supervisado
- **Clasificación**: Predecir categorías (spam/no spam, fraude/legítimo)
- **Regresión**: Predecir valores continuos (precios, temperaturas)

```csharp
// Ejemplo con ML.NET - Clasificación binaria
var pipeline = mlContext.Transforms.Text
    .FeaturizeText("Features", "Text")
    .Append(mlContext.BinaryClassification.Trainers.SdcaLogisticRegression());

var model = pipeline.Fit(trainData);
```

#### Aprendizaje No Supervisado
- **Clustering**: Agrupar datos similares (segmentación de clientes)
- **Reducción de dimensionalidad**: Simplificar datos complejos

#### Aprendizaje por Refuerzo
- **Q-Learning**: Aprender a través de recompensas y castigos
- **Aplicaciones**: Juegos, robótica, sistemas de recomendación

### Algoritmos Fundamentales

#### Regresión Lineal
```python
# Ejemplo básico en Python
from sklearn.linear_model import LinearRegression
import numpy as np

# Datos de ejemplo
X = np.array([[1], [2], [3], [4], [5]])
y = np.array([2, 4, 6, 8, 10])

# Entrenamiento
model = LinearRegression()
model.fit(X, y)

# Predicción
prediction = model.predict([[6]])  # Resultado: 12
```

#### Clasificación con Decision Trees
```csharp
// ML.NET - Árbol de decisión
var pipeline = mlContext.Transforms
    .Categorical.OneHotEncoding("CategoryEncoded", "Category")
    .Append(mlContext.Transforms.Concatenate("Features", "CategoryEncoded", "NumericFeature"))
    .Append(mlContext.MulticlassClassification.Trainers.FastTree());
```

## ML.NET en Aplicaciones .NET

### Configuración Básica
```csharp
// Program.cs
using Microsoft.ML;

var mlContext = new MLContext(seed: 0);

// Definir modelo de datos
public class HouseData
{
    public float Size { get; set; }
    public float Price { get; set; }
}

public class Prediction
{
    [ColumnName("Score")]
    public float Price { get; set; }
}
```

### Entrenamiento de Modelo
```csharp
// Cargar datos
IDataView dataView = mlContext.Data.LoadFromTextFile<HouseData>("data.csv", hasHeader: true, separatorChar: ',');

// Definir pipeline
var pipeline = mlContext.Transforms
    .CopyColumns(outputColumnName: "Label", inputColumnName: "Price")
    .Append(mlContext.Transforms.Categorical.OneHotEncoding(outputColumnName: "SizeEncoded", inputColumnName: "Size"))
    .Append(mlContext.Transforms.Concatenate("Features", "SizeEncoded"))
    .Append(mlContext.Regression.Trainers.Sdca());

// Entrenar
var model = pipeline.Fit(dataView);

// Guardar modelo
mlContext.Model.Save(model, dataView.Schema, "model.zip");
```

### Consumir Modelo en API
```csharp
[ApiController]
[Route("api/[controller]")]
public class PredictionController : ControllerBase
{
    private readonly PredictionEngine<HouseData, Prediction> _predictionEngine;

    public PredictionController(PredictionEngine<HouseData, Prediction> predictionEngine)
    {
        _predictionEngine = predictionEngine;
    }

    [HttpPost("predict")]
    public ActionResult<Prediction> Predict([FromBody] HouseData input)
    {
        var prediction = _predictionEngine.Predict(input);
        return Ok(prediction);
    }
}
```

## Procesamiento de Lenguaje Natural

### Análisis de Sentimiento con Azure Cognitive Services
```csharp
using Azure.AI.TextAnalytics;

public class SentimentAnalysisService
{
    private readonly TextAnalyticsClient _client;

    public SentimentAnalysisService(string endpoint, string apiKey)
    {
        _client = new TextAnalyticsClient(new Uri(endpoint), new AzureKeyCredential(apiKey));
    }

    public async Task<DocumentSentiment> AnalyzeSentimentAsync(string text)
    {
        var response = await _client.AnalyzeSentimentAsync(text);
        return response.Value;
    }
}
```

### Chatbot con OpenAI API
```csharp
using OpenAI.GPT3;
using OpenAI.GPT3.Managers;
using OpenAI.GPT3.ObjectModels.RequestModels;

public class ChatbotService
{
    private readonly OpenAIService _openAIService;

    public ChatbotService(string apiKey)
    {
        _openAIService = new OpenAIService(new OpenAiOptions()
        {
            ApiKey = apiKey
        });
    }

    public async Task<string> GetResponseAsync(string userMessage)
    {
        var completionResult = await _openAIService.ChatCompletion.CreateCompletion(
            new ChatCompletionCreateRequest
            {
                Messages = new List<ChatMessage>
                {
                    ChatMessage.FromSystem("Eres un asistente útil para desarrolladores."),
                    ChatMessage.FromUser(userMessage)
                },
                Model = Models.Gpt_3_5_Turbo,
                MaxTokens = 150
            });

        return completionResult.Choices.First().Message.Content;
    }
}
```

### Text Processing Básico
```csharp
using System.Text.RegularExpressions;

public class TextProcessor
{
    public string CleanText(string input)
    {
        // Remover caracteres especiales
        string cleaned = Regex.Replace(input, @"[^\w\s]", "");
        
        // Convertir a minúsculas
        cleaned = cleaned.ToLower();
        
        // Remover espacios extra
        cleaned = Regex.Replace(cleaned, @"\s+", " ").Trim();
        
        return cleaned;
    }

    public string[] Tokenize(string text)
    {
        return text.Split(' ', StringSplitOptions.RemoveEmptyEntries);
    }

    public Dictionary<string, int> GetWordFrequency(string text)
    {
        var words = Tokenize(CleanText(text));
        return words.GroupBy(w => w)
                   .ToDictionary(g => g.Key, g => g.Count());
    }
}
```

## Integración Python con .NET

### Usando Python.NET
```csharp
using Python.Runtime;

public class PythonMLService
{
    public void InitializePython()
    {
        PythonEngine.Initialize();
    }

    public double[] PredictWithSklearn(double[][] features)
    {
        using (Py.GIL())
        {
            dynamic sklearn = Py.Import("sklearn.linear_model");
            dynamic np = Py.Import("numpy");
            
            var model = sklearn.LinearRegression();
            var X = np.array(features);
            
            // Entrenar modelo (ejemplo simplificado)
            var y = np.array(new double[] { 1, 2, 3, 4 });
            model.fit(X, y);
            
            // Predicción
            var predictions = model.predict(X);
            return predictions.As<double[]>();
        }
    }
}
```

### Microservicio Python para ML
```python
# ml_service.py
from flask import Flask, request, jsonify
import pandas as pd
from sklearn.ensemble import RandomForestClassifier
import joblib

app = Flask(__name__)

# Cargar modelo pre-entrenado
model = joblib.load('trained_model.pkl')

@app.route('/predict', methods=['POST'])
def predict():
    try:
        data = request.json
        features = pd.DataFrame(data['features'])
        predictions = model.predict(features)
        
        return jsonify({
            'predictions': predictions.tolist(),
            'status': 'success'
        })
    except Exception as e:
        return jsonify({
            'error': str(e),
            'status': 'error'
        }), 400

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=5000)
```

## IA en DevOps

### Automatización Inteligente de Testing
```csharp
public class IntelligentTestingService
{
    public async Task<TestRecommendation[]> RecommendTestsAsync(string codeChanges)
    {
        // Usar IA para analizar cambios de código y recomendar tests
        var analysisResult = await AnalyzeCodeChanges(codeChanges);
        
        return analysisResult.RiskyAreas
            .Select(area => new TestRecommendation
            {
                TestType = DetermineTestType(area.Complexity),
                Priority = CalculatePriority(area.RiskScore),
                SuggestedTests = GenerateTestSuggestions(area)
            })
            .ToArray();
    }
}
```

### Predicción de Fallos en Pipeline
```csharp
public class PipelineFailurePrediction
{
    private readonly MLContext _mlContext;
    private readonly ITransformer _model;

    public async Task<PipelineRisk> PredictPipelineRiskAsync(PipelineMetrics metrics)
    {
        var prediction = _predictionEngine.Predict(new PipelineData
        {
            TestCoverage = metrics.TestCoverage,
            CodeComplexity = metrics.CyclomaticComplexity,
            ChangeFrequency = metrics.CommitsLastWeek,
            TeamExperience = metrics.AvgTeamExperience
        });

        return new PipelineRisk
        {
            FailureProbability = prediction.FailureProbability,
            RecommendedActions = GenerateRecommendations(prediction)
        };
    }
}
```

## Ética en IA

### Detección de Sesgos
```csharp
public class BiasDetector
{
    public BiasAnalysis AnalyzeModelBias(IDataView testData, ITransformer model)
    {
        var predictions = model.Transform(testData);
        
        // Analizar predicciones por grupos demográficos
        var results = _mlContext.Data.CreateEnumerable<ModelOutput>(predictions, false)
            .GroupBy(p => p.DemographicGroup)
            .Select(g => new GroupAnalysis
            {
                Group = g.Key,
                SuccessRate = g.Average(p => p.Prediction),
                Count = g.Count()
            })
            .ToList();

        return new BiasAnalysis
        {
            HasSignificantBias = DetectStatisticalBias(results),
            GroupAnalyses = results,
            Recommendations = GenerateFairnessRecommendations(results)
        };
    }
}
```

### Privacy-Preserving ML
```csharp
public class PrivacyPreservingMLService
{
    public void TrainModelWithDifferentialPrivacy(IDataView data, double epsilon)
    {
        // Implementar differential privacy
        var noisyData = AddLaplaceNoise(data, epsilon);
        
        var pipeline = _mlContext.Transforms
            .DropColumns("PersonalIdentifier", "SensitiveAttribute")
            .Append(_mlContext.Transforms.NormalizeMinMax("Features"))
            .Append(_mlContext.BinaryClassification.Trainers.SdcaLogisticRegression());

        var model = pipeline.Fit(noisyData);
    }

    private IDataView AddLaplaceNoise(IDataView data, double epsilon)
    {
        // Implementación de ruido de Laplace para differential privacy
        // Simplificado para el ejemplo
        return data;
    }
}
```

## Recursos y Herramientas

### Frameworks Recomendados
- **ML.NET**: Framework nativo de Microsoft para .NET
- **TensorFlow.NET**: Binding de TensorFlow para C#
- **Accord.NET**: Framework completo para ML en .NET
- **scikit-learn**: Biblioteca estándar para Python ML

### Servicios Cloud de IA
- **Azure Cognitive Services**: APIs pre-entrenadas
- **Azure Machine Learning**: Plataforma completa de MLOps
- **AWS SageMaker**: Plataforma de ML de Amazon
- **Google AI Platform**: Servicios de IA de Google

### Best Practices
1. **Comenzar simple**: Usar modelos simples antes que complejos
2. **Datos de calidad**: La calidad de datos es crítica
3. **Validación cruzada**: Evitar overfitting
4. **Monitoreo continuo**: Seguir el rendimiento en producción
5. **Ética y transparencia**: Considerar implicaciones éticas

### Pipeline de MLOps
```yaml
# Azure DevOps Pipeline para ML
trigger:
  branches:
    include:
    - main
  paths:
    include:
    - src/ml/*

stages:
- stage: DataValidation
  jobs:
  - job: ValidateData
    steps:
    - task: PythonScript@0
      inputs:
        scriptPath: 'scripts/validate_data.py'

- stage: ModelTraining
  jobs:
  - job: TrainModel
    steps:
    - task: PythonScript@0
      inputs:
        scriptPath: 'scripts/train_model.py'
    
    - task: PublishTestResults@2
      inputs:
        testResultsFiles: '**/model_metrics.xml'

- stage: ModelDeployment
  condition: and(succeeded(), eq(variables['Build.SourceBranch'], 'refs/heads/main'))
  jobs:
  - deployment: DeployModel
    environment: 'production'
    strategy:
      runOnce:
        deploy:
          steps:
          - task: AzureCLI@2
            inputs:
              azureSubscription: 'ServiceConnection'
              scriptType: 'bash'
              scriptLocation: 'inlineScript'
              inlineScript: |
                az ml model deploy --name mymodel --version latest
```