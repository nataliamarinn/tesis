---
description: Nuevos Modelos de Hugging Face
---

# 💻 Modelos Finales

Luego de la evaluación de los modelos, los mismos fueron publicados en la plataforma Hugging Face:

* Modelo Myriam Bregman: [natmarinn/sentimientos bregman](https://huggingface.co/natmarinn/sentimientos-bregman)
* Modelo Patricia Bullrich: [natmarinn / sentimientos-bullrich](https://huggingface.co/natmarinn/sentimientos-bullrich)
* Modelo Sergio Massa: [natmarinn / sentimientos-massa](https://huggingface.co/natmarinn/sentimientos-massa)
* Modelo Javier Milei: [natmarinn/sentimientos-milei](https://huggingface.co/natmarinn/sentimientos-milei)
* Modelo Juan Schiaretti: [natmarinn/sentimientos-schiaretti](https://huggingface.co/natmarinn/sentimientos-schiaretti)

## <mark style="background-color:green;">Ejemplo de uso</mark>

{% hint style="info" %}
Al ser similar la aplicación se toma como ejemplo el uso del modelo sentimientos-milei para clasificar comentarios referidos al candidato Javier Milei.
{% endhint %}

```python
from transformers import XLMRobertaForSequenceClassification, XLMRobertaTokenizer
import torch
```

```python
# Cargar el modelo y el tokenizador
model_path = "natmarinn/sentimientos-milei"
model = XLMRobertaForSequenceClassification.from_pretrained(model_path)
tokenizer = XLMRobertaTokenizer.from_pretrained(model_path)
```

```python
# Texto de ejemplo
texto = "Milei presidente"
```

```python
# Tokenización
inputs = tokenizer(texto, return_tensors="pt", truncation=True)

# Predicción
with torch.no_grad():
    outputs = model(**inputs)
    logits = outputs.logits
    pred_class = torch.argmax(logits, dim=1).item()
```

```python
# Mostrar resultado
clases = ["negativo", "neutro", "positivo"]
print(f"El comentario es clasificado como: {clases[pred_class]}")
```

El comentario es clasificado como **positivo**

