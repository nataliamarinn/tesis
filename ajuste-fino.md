---
cover: >-
  .gitbook/assets/DALL·E 2024-11-12 22.04.57 - A hand-drawn style illustration
  in green tones symbolizing the process of fine-tuning a language model. The
  image features a computer with surrounding.webp
coverY: 0
layout:
  cover:
    visible: true
    size: hero
  title:
    visible: true
  description:
    visible: true
  tableOfContents:
    visible: true
  outline:
    visible: true
  pagination:
    visible: true
---

# ⚙️ Ajuste fino



## Librerías

````python
```python
import pandas as pd
from tqdm import notebook as notebook_tqdm
from sklearn.model_selection import train_test_split
from transformers import (
    XLMRobertaForSequenceClassification,
    XLMRobertaTokenizer,
    AutoTokenizer,
    AutoModelForSequenceClassification,
    TrainingArguments,
    Trainer
)
import torch
import openpyxl
from nltk.tokenize import word_tokenize
from datasets import Dataset
from sklearn.metrics import (
    accuracy_score, 
    precision_recall_fscore_support, 
    confusion_matrix,
    roc_auc_score,
    cohen_kappa_score,
    matthews_corrcoef
)
import psutil
import numpy as np
from tabulate import tabulate

```
````

## Cargar datos

{% file src=".gitbook/assets/datos_etiquetados.zip" %}

````
```python
df = pd.read_excel('massa_etiquetas.xlsx')
df_filtrado = df[df['tag'].notna()]
df_filtrado = df_filtrado[['Fuente', 'Texto_corregido', 'tag']]
```
````

````python
```python
X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.2, random_state=94)
```
````

````
```python
train_df = pd.concat([X_train, y_train], axis=1)
test_df = pd.concat([X_test, y_test], axis=1)
```
````

## Cargar el modelo base

````python
```python
model_path = "cardiffnlp/twitter-xlm-roberta-base-sentiment"
model = XLMRobertaForSequenceClassification.from_pretrained(model_path)
tokenizer = XLMRobertaTokenizer.from_pretrained(model_path)
```
````

## Preparar el entrenamiento y cálculo de métricas

````python
```python
columnas_seleccionadas = ['Fuente', 'Texto_corregido', 'tag']
test_df= test_df[columnas_seleccionadas]
train_df= train_df[columnas_seleccionadas]
```
````

```python
def tokenize_and_encode_labels(examples):
    tokenized = tokenizer(examples["Texto_corregido"], padding="max_length", truncation=True, max_length=512)
    tokenized["labels"] = [int(label) for label in examples["tag"]]
    return tokenized
```

````python
```python
# Convierte el dataframe en una clase Dataset de Hugging Face
train_dataset = Dataset.from_pandas(train_df)
test_dataset = Dataset.from_pandas(test_df)

# Aplica la función de tokenización y codificación de etiquetas
train_dataset = train_dataset.map(tokenize_and_encode_labels, batched=True)
test_dataset = test_dataset.map(tokenize_and_encode_labels, batched=True)
```
````

````python
```python
split_dataset = {
    "train": train_dataset,  # Cambia 'tokenized_train_dataset' por 'train_dataset'
    "test": test_dataset     # Cambia 'tokenized_test_dataset' por 'test_dataset'
}
```
````

````python
```python
training_args = TrainingArguments(
    output_dir="./results",
    num_train_epochs=2,
    per_device_train_batch_size=2,
    per_device_eval_batch_size=2,
    gradient_accumulation_steps=8,
    warmup_steps=50,
    weight_decay=0.01,
    logging_dir="./logs",
    logging_steps=10,
    evaluation_strategy="epoch",
    save_strategy="epoch",
    load_best_model_at_end=True,
    save_total_limit=1,  # Mantener solo el mejor modelo
    fp16=False,
)

```
````

````python
```python
def compute_metrics(pred):
    labels = pred.label_ids
    preds = pred.predictions.argmax(-1)
    
    # Métricas generales
    precision, recall, f1, _ = precision_recall_fscore_support(labels, preds, average='weighted')
    acc = accuracy_score(labels, preds)
    
    # Métricas por clase
    precision_per_class, recall_per_class, f1_per_class, support_per_class = \
        precision_recall_fscore_support(labels, preds, average=None)
    
    # Matriz de confusión
    cm = confusion_matrix(labels, preds)
    
    return {
        'accuracy': acc,
        'f1': f1,
        'precision': precision,
        'recall': recall,
        'precision_per_class': precision_per_class.tolist(),
        'recall_per_class': recall_per_class.tolist(),
        'f1_per_class': f1_per_class.tolist(),
        'support_per_class': support_per_class.tolist(),
        'confusion_matrix': cm.tolist()
    }
```
````

## Entrenamiento y evaluación

````python
```python
trainer = Trainer(
    model=model,
    args=training_args,
    train_dataset=split_dataset["train"],
    eval_dataset=split_dataset["test"],
    compute_metrics=compute_metrics,
)

```
````

````python
```python
# Entrenar el modelo
trainer.train()
```
````

````
```python
# Especifica el directorio donde quieres guardar el modelo
model_dir = "./sentimientos-massa"

# Guarda el modelo y el tokenizador
model.save_pretrained(model_dir)
tokenizer.save_pretrained(model_dir)
```
````

````python
```python
results = trainer.evaluate()
print(results)
```
````

````python
```python
def visualize_metrics(results):
    # Métricas generales
    general_metrics = [
        ["Accuracy", results['eval_accuracy']],
        ["F1 Score", results['eval_f1']],
        ["Precision", results['eval_precision']],
        ["Recall", results['eval_recall']]
    ]
    
    print("Métricas Generales:")
    print(tabulate(general_metrics, headers=["Métrica", "Valor"], tablefmt="grid"))
    print("\n")

    # Métricas por clase
    class_metrics = []
    for i in range(len(results['eval_precision_per_class'])):
        class_metrics.append([
            f"Clase {i}",
            results['eval_precision_per_class'][i],
            results['eval_recall_per_class'][i],
            results['eval_f1_per_class'][i],
            results['eval_support_per_class'][i]
        ])
    
    print("Métricas por Clase:")
    print(tabulate(class_metrics, headers=["Clase", "Precision", "Recall", "F1", "Support"], tablefmt="grid"))
    print("\n")

    # Matriz de confusión
    cm = np.array(results['eval_confusion_matrix'])
    print("Matriz de Confusión:")
    print(tabulate(cm, headers=range(cm.shape[1]), showindex="always", tablefmt="grid"))
```
````

````python
```python
visualize_metrics(results)
```
````
