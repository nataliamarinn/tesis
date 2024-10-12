---
description: >-
  Métricas de desempeño del modelo base en un conjunto de datos etiquetados
  manualmente
---

# 🧪 Evaluación del modelo base

Antes de proceder con el ajuste fino del modelo base, es fundamental evaluar su desempeño en un subconjunto de datos etiquetados del conjunto original. Esta evaluación inicial nos permitirá analizar cómo se comporta el modelo base en condiciones reales y obtener una referencia sobre su capacidad para clasificar correctamente los datos.

## <mark style="background-color:green;">Modelo base</mark>

El modelo **cardiffnlp/twitter-xlm-roberta-base-sentiment-multilingual** es una versión ajustada del modelo cardiffnlp/twitter-xlm-roberta-base, específicamente entrenada en el conjunto de datos cardiffnlp/tweet\_sentiment\_multilingual mediante la biblioteca tweetnlp. Este modelo ha sido diseñado para realizar análisis de sentimientos en múltiples idiomas, logrando métricas destacadas en su evaluación. Este trabajo fue presentado por varios autores en la conferencia EMNLP 2022, destacando su contribución al campo del procesamiento de lenguaje natural y el análisis de sentimientos en redes sociales.

## <mark style="background-color:green;">Librerías</mark>

````python
import pandas as pd
from nltk.tokenize import word_tokenize
import nltk
import datasets
from transformers import AutoModelForSequenceClassification, AutoTokenizer, TrainingArguments, Trainer
from datasets import Dataset
from sklearn.metrics import accuracy_score, precision_recall_fscore_support
import psutil
import pandas as pd
from sklearn.model_selection import train_test_split
from transformers import XLMRobertaForSequenceClassification, XLMRobertaTokenizer,AutoTokenizer,AutoModelForSequenceClassification
import torch

```
````

## <mark style="background-color:green;">Carga de datos</mark>

{% file src=".gitbook/assets/Datos_etiquetados.zip" %}

{% hint style="info" %}
En esta etapa el código es el mismo para cada candidato, como ejemplo tomaremos el código para el caso de la candidata Patricia Bullrich.&#x20;
{% endhint %}

```python
# Bullrich
file_path1 = 'Bullrich_30.xlsx'
df1 = pd.read_excel(file_path1)
df_filtrado1= df1.dropna(subset=['tag_hf1'])
columnas_seleccionadas = ['Fuente', 'Texto_corregido', 'tag_hf1']
df11= df_filtrado1[columnas_seleccionadas]
```

## <mark style="background-color:green;">Dividir los datos en entrenamiento y testeo</mark>

```python
# Dividir el DataFrame en conjunto de entrenamiento y pruebas
df1_training, df1_testing = train_test_split(df11, test_size=0.2, random_state=42)

# Imprimir la forma de los nuevos DataFrames
print("Shape de df1_training:", df1_training.shape)
print("Shape de df1_testing:", df1_testing.shape)
```

## <mark style="background-color:green;">Cargar el modelo base y su tokenizador</mark>

```python
model_path = "cardiffnlp/twitter-xlm-roberta-base-sentiment"
model = XLMRobertaForSequenceClassification.from_pretrained(model_path)
tokenizer = XLMRobertaTokenizer.from_pretrained(model_path)
```

## Clase MyDataset

La clase `MyDataset` se crea para facilitar el trabajo con datos etiquetados en PyTorch, una biblioteca popular para el aprendizaje automático. Al heredar de `Dataset`, que es una clase base de PyTorch, podemos personalizar cómo se manejan nuestros datos.&#x20;

En este caso, estamos convirtiendo un DataFrame que contiene textos y sus etiquetas en un formato que el modelo puede entender. Esto implica tokenizar el texto y crear una "máscara de atención" para indicar qué partes del texto son relevantes. Además, las etiquetas se transforman en un formato llamado "one-hot encoding", que convierte las categorías (como negativo, neutro y positivo) en vectores binarios.&#x20;

Este proceso es esencial porque permite que el modelo aprenda de manera efectiva a partir de nuestros datos etiquetados, optimizando su rendimiento en tareas como la clasificación de sentimientos.

```python
class MyDataset(Dataset):
    def __init__(self, dataframe, tokenizer, max_length=512, num_labels=3):
        """
        Inicializa el dataset con los datos etiquetados.

        Args:
            dataframe (DataFrame): DataFrame que contiene los datos etiquetados.
            tokenizer (Tokenizer): para convertir texto a tokens.
            max_length (int): Longitud máxima de secuencia permitida por el tokenizer (máximo de tokens)
            num_labels (int): Número de etiquetas en el conjunto de datos. (en este caso tenemos 3 etiquetas:negativo-neutro-positivo)
        """
        self.tokenizer = tokenizer
        self.data = [] #dataset vacío
        for idx, row in dataframe.iterrows():
            text = str(row['Texto_corregido'])
            label = int(row['tag_hf1'])  # Convertir la etiqueta a entero
            # Codificar el texto utilizando el tokenizador
            encoding = tokenizer(text, truncation=True, padding='max_length', max_length=max_length) #padding(relleno) se realiza para homogenizar longitudes y simplificar el procesamiento y la eficiencia computacional
            # Convertir la etiqueta a one-hot encoding: es una forma de respresentar las categorías como vectores binarios. 
            one_hot_label = torch.zeros(num_labels)
            one_hot_label[label] = 1
            # Agregar los datos codificados al dataset
            self.data.append({'input_ids': encoding['input_ids'], #input_ids se refiere a los IDs de los tokens en la secuencia de texto (estos IDs son los que entiende el modelo)
                              'attention_mask': encoding['attention_mask'], #es una máscara que indica qué partes de la secuencia son importantes para el modelo y cuáles corresponden al padding
                              'labels': one_hot_label}) #representa las etiquetas de clasificación codificadas en el formato one-hot
            
    def __len__(self):
        return len(self.data)
    
    def __getitem__(self, idx):
        return {key: val for key, val in self.data[idx].items()}
```

Creamos una instancia de la clase MyDataset para los datos del entrenamiento

```
train_dataset = MyDataset(df1_training, tokenizer)

```
