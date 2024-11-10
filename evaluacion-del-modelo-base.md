---
description: >-
  Métricas de desempeño del modelo base en un conjunto de datos etiquetados
  manualmente
---

# 🧪 Evaluación del modelo base

Antes de proceder con el ajuste fino del modelo base, es fundamental evaluar su desempeño en un subconjunto de datos etiquetados del conjunto original. Esta evaluación inicial nos permitirá analizar cómo se comporta el modelo base en condiciones reales y obtener una referencia sobre su capacidad para clasificar correctamente los datos.

## <mark style="background-color:green;">Modelo base</mark>

El modelo **cardiffnlp/twitter-xlm-roberta-base-sentiment-multilingual** es una versión ajustada del modelo cardiffnlp/twitter-xlm-roberta-base, específicamente entrenada en el conjunto de datos cardiffnlp/tweet\_sentiment\_multilingual mediante la biblioteca tweetnlp. Este modelo ha sido diseñado para realizar análisis de sentimientos en múltiples idiomas, logrando métricas destacadas en su evaluación.

Este trabajo fue presentado por varios autores en la conferencia EMNLP 2022, destacando su contribución al campo del procesamiento de lenguaje natural y el análisis de sentimientos en redes sociales.

## <mark style="background-color:green;">Librerías</mark>

````python
```python
import pandas as pd
import torch
from transformers import AutoTokenizer, AutoModelForSequenceClassification
from sklearn.model_selection import train_test_split
from sklearn.preprocessing import LabelBinarizer
from sklearn.metrics import (
    accuracy_score, precision_recall_fscore_support, confusion_matrix,
)
```
````

## <mark style="background-color:green;">Carga de datos</mark>

{% file src=".gitbook/assets/datos_etiquetados.zip" %}

{% hint style="info" %}
En esta etapa el código es el mismo para cada candidato, como ejemplo tomaremos el código para el caso de la candidata Patricia Bullrich.&#x20;
{% endhint %}

````python
```python
df = pd.read_excel('bullrich_etiquetas.xlsx')
df_filtrado = df[df['tag'].notna()]
```
````

## <mark style="background-color:green;">Dividir los datos en entrenamiento y testeo</mark>

```python
X = df_filtrado[['Texto_corregido']]
y = df_filtrado['tag'].dropna()
X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.2, random_state=94)

```

````python
```python
# DataFrames de entrenamiento y prueba completos
train_df = pd.concat([X_train, y_train], axis=1)
test_df = pd.concat([X_test, y_test], axis=1)
```
````

## <mark style="background-color:green;">Cargar el modelo base y su tokenizador</mark>

````python
```python
model_path = "cardiffnlp/twitter-xlm-roberta-base-sentiment"
tokenizer = AutoTokenizer.from_pretrained(model_path)
model = AutoModelForSequenceClassification.from_pretrained(model_path)
```
````

````python
```python
# Lista para almacenar las predicciones
sentimientos = []

# Realizar el análisis de sentimientos en el conjunto de prueba
for _, row in test_df.iterrows():
    text = str(row['Texto_corregido'])  # Asegurar que es una cadena
    
    # Tokenización y predicción
    inputs = tokenizer(text, return_tensors="pt", padding=True, truncation=True, max_length=700)
    outputs = model(**inputs)
    logits = outputs.logits
    predicted_class = torch.argmax(logits, dim=1)
    sentimientos.append(predicted_class.item())

# Agregar las predicciones al DataFrame de prueba
test_df['sentimiento'] = sentimientos
```
````

## <mark style="background-color:green;">Evaluar predicciones modelo base</mark>

````python
```python
def evaluate_model(y_true, y_pred):
    """
    Calcula varias métricas de evaluación para un modelo de clasificación.

    Parámetros:
    y_true : array-like
        Etiquetas verdaderas (reales) del conjunto de datos.
    y_pred : array-like
        Etiquetas predichas por el modelo.

    Retorna:
    dict : Un diccionario que contiene las métricas calculadas:
        - 'accuracy': Exactitud general del modelo.
        - 'precision_weighted': Precisión ponderada por clase.
        - 'recall_weighted': Sensibilidad ponderada por clase.
        - 'f1_score_weighted': Puntaje F1 ponderado por clase.
        - 'precision_per_class': Precisión por clase (array).
        - 'recall_per_class': Sensibilidad por clase (array).
        - 'f1_per_class': Puntaje F1 por clase (array).
        - 'confusion_matrix': Matriz de confusión (2D array).
        - 'specificity_per_class': Especificidad por clase (array).
        - 'ppv_per_class': Valor predictivo positivo (PPV) por clase (array).
        - 'npv_per_class': Valor predictivo negativo (NPV) por clase (array).
    """

    # Calcular exactitud
    accuracy = accuracy_score(y_true, y_pred)
    
    # Calcular precisión, sensibilidad y puntaje F1 ponderado para todas las clases
    precision, recall, f1, _ = precision_recall_fscore_support(y_true, y_pred, average='weighted')
    
    # Calcular precisión, sensibilidad y puntaje F1 específico para cada clase
    per_class_metrics = precision_recall_fscore_support(y_true, y_pred, average=None)
    precision_per_class, recall_per_class, f1_per_class = per_class_metrics[:3]
    
    # Generar la matriz de confusión
    cm = confusion_matrix(y_true, y_pred)
    
    # Preparar los datos para el cálculo de AUC multiclase (si se desea implementar AUC)
    lb = LabelBinarizer()
    y_true_bin = lb.fit_transform(y_true)
    y_pred_bin = lb.transform(y_pred)
    
    # Calcular especificidad (negativos verdaderos / (negativos verdaderos + falsos positivos)) para cada clase
    specificity_per_class = []
    for i in range(len(cm)):
        tn = cm.sum() - (cm[i, :].sum() + cm[:, i].sum() - cm[i, i])  # Negativos verdaderos
        fp = cm[:, i].sum() - cm[i, i]  # Falsos positivos
        specificity = tn / (tn + fp) if (tn + fp) != 0 else 0
        specificity_per_class.append(specificity)
    
    # El valor predictivo positivo (PPV) para cada clase es igual a la precisión de cada clase
    ppv_per_class = precision_per_class
    
    # Calcular el valor predictivo negativo (NPV) para cada clase
    npv_per_class = []
    for i in range(len(cm)):
        tn = cm.sum() - (cm[i, :].sum() + cm[:, i].sum() - cm[i, i])  # Negativos verdaderos
        fn = cm[i, :].sum() - cm[i, i]  # Falsos negativos
        npv = tn / (tn + fn) if (tn + fn) != 0 else 0
        npv_per_class.append(npv)

    return {
        'accuracy': accuracy,  # Exactitud global del modelo
        'precision_weighted': precision,  # Precisión ponderada (considera la proporción de cada clase)
        'recall_weighted': recall,  # Sensibilidad ponderada (considera la proporción de cada clase)
        'f1_score_weighted': f1,  # Puntaje F1 ponderado (balancea precisión y sensibilidad)
        'precision_per_class': precision_per_class,  # Precisión para cada clase
        'recall_per_class': recall_per_class,  # Sensibilidad para cada clase
        'f1_per_class': f1_per_class,  # Puntaje F1 para cada clase
        'confusion_matrix': cm,  # Matriz de confusión para visualizar los aciertos y errores por clase
        'specificity_per_class': specificity_per_class,  # Especificidad para cada clase (falsos positivos controlados)
        'ppv_per_class': ppv_per_class,  # Valor predictivo positivo para cada clase (igual a precisión por clase)
        'npv_per_class': npv_per_class  # Valor predictivo negativo para cada clase (probabilidad de no tener la clase cuando no se predice)
    }
```
````

````python
```python
y_true = test_df['tag']
y_pred = test_df['sentimiento']
metrics = evaluate_model(y_true, y_pred)
```
````

````python
```python
# Imprimir resultados
for metric, value in metrics.items():
    if metric != 'confusion_matrix':
        print(f"{metric}: {value}")
    else:
        print(f"{metric}:\n{value}")
```
````
