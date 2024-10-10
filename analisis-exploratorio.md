---
description: Número de comentarios por red social - nube de palabras - bigramas
---

# 🔍 Análisis Exploratorio

El _**análisis de datos exploratorio**_ de los comentarios resulta una etapa fundamental en esta investigación, por un lado, proporciona una comprensión inicial de las palabras utilizadas referidas a cada candidato y por otro lado ha sido utilizado para mejorar y ajustar el paso anterior de limpieza de datos.

## <mark style="background-color:blue;">Librerías utilizadas</mark>

* _<mark style="color:blue;">**pandas:**</mark>_ esta librería fue diseñada para facilitar el análisis y manipulación de datos en forma tabular. Permite cargar datos de diversas fuentes (ej: CSV, Excel, SQL), limpiarlos, filtrar y seleccionar datos, realizar operaciones estadísticas y generar visualizaciones sencillas. Se destaca esta librería por su capacidad de trabajar con grandes volúmenes de datos y su integración con otras librerías de análisis y visualización de datos.
* _<mark style="color:blue;">**openpyxl:**</mark>_ es una herramienta para trabajar con archivos de Excel en formato .xlsx. Permite leer, escribir y manipular hojas de cálculo.&#x20;
* _<mark style="color:blue;">**nltk:**</mark>_ por sus siglas en inglés "Natural Language Toolkit"  es una librería muy importante para el procesamiento del lenguaje natural en Python. En esta sección utilizaremos "FreqDist" para calcular frecuencias y "ngrams" para generar n-gramas.
* <mark style="color:blue;">**nltk.corpus:**</mark> dentro de la librería nltk, "nltk.corpus" es un módulo que proporciona acceso a una variedad de corpus lingüísticos predefinidos. Dentro de esta variedad se encuentra una subcolección llamada _"stopwords"_. Las stopwords son palabras comunes que se eliminan durante el procesamiento de texto en NLP porque generalmente no aportan informacióin relevante para el análisis.
* _<mark style="color:blue;">**plotly.express:**</mark>_ facilita la creación de gráficos interactivos de alta calidad.
* _<mark style="color:blue;">**collections:**</mark>_ dentro de esta librería se encuentra una clase llamada "Counter" que se utiliza para contar la frecuencia de de elementos de una colección. Esta librería será útil para conocer la frecuencia de palabras en los comentarios de cada candidato y como paso previo a las visualizaciones que realizaremos.
* _<mark style="color:blue;">**wordcloud:**</mark>_ esta biblioteca se utiliza para crear visualizaciones de nube de palabras a partir de un conjunto de texto.&#x20;
* _<mark style="color:blue;">**matplotlib.pyplot:**</mark>_ es una biblioteca muy utilizada para la visualización de datos en Python.



### Carga de datos

Utilizaremos los datos obtenidos del apartado anterior de limpieza de datos

{% file src=".gitbook/assets/Datos_filtrados.zip" %}

```python
# Bregman
file_path1 = 'filtrado_bregman.xlsx'
df1 = pd.read_excel(file_path1)
# Bullrich
file_path2 = 'filtrado_bullrich.xlsx'
df2 = pd.read_excel(file_path2)
# Massa
file_path3 = 'filtrado_massa.xlsx'
df3= pd.read_excel(file_path3)
# Milei
file_path4 = 'filtrado_milei.xlsx'
df4 = pd.read_excel(file_path4)
# Schiaretti
file_path5 = 'filtrado_schiaretti.xlsx'
df5 = pd.read_excel(file_path5)
```

{% hint style="warning" %}
[**Al tratarse del mismo código para cada candidato, se utilizará como ejemplo el primer data frame df1, que corresponde a la candidata Myriam Bregman. Para realizar las visualizaciones de los restantes candidatos deberá reemplazarse df1 por el correspondiente data frame.**](#user-content-fn-1)[^1]
{% endhint %}

## <mark style="color:blue;">Número de comentarios por red social</mark>

Para observar con cuántos comentarios se cuenta para cada candidato en las distintas redes sociales, construímos gráficos de columnas. Para cada candidato se debe ejecutar el siguiente código:

```python
# Agrupar los datos por 'Fuente' y contar el número de ocurrencias
df_graf1 = df1.groupby('Fuente').count()['Texto_corregido'].reset_index()

# Crear la gráfica de barras
figura = px.bar(df_graf1, x='Fuente', y='Texto_corregido', title="Frecuencia de comentarios por red social sobre Myriam Bregman",
                labels={'Fuente': 'Red Social', 'Texto_corregido': 'Cantidad de comentarios'})

# Cambiar el color de fondo del gráfico a blanco
figura.update_layout(plot_bgcolor='white')

# Ajustar la altura del gráfico
figura.update_layout(height=400)  # Puedes ajustar el valor de altura según tus necesidades

# Ajustar el rango y las etiquetas del eje Y
figura.update_yaxes(range=[0, 1500], title_text='Cantidad de comentarios')

# Mostrar el gráfica
figura.show()
```

<figure><img src=".gitbook/assets/newplot (1).png" alt=""><figcaption><p>Ejemplo cantidad de comentarios por red social para la candidata Myriam Bregman</p></figcaption></figure>

### <mark style="color:blue;">Quitar stopwords</mark>

Se eliminan las “stopwords”, que son palabras como artículos, conectores, que aparecen con gran frecuencia en los comentarios analizados pero no apartan a la calidad del análisis.\
También se actualiza el diccionario de stopwords.

```python
stop = set(nltk.corpus.stopwords.words('spanish')) #carga de stopwords en español
stop.update(['.','q','pq','si','mas','xq','x',"vos",'?', '!','ja','debate','jajaja',"jeje","jaja","jajaj","jajajaja","jejeje","jajjajajajaja",'haber','decir',':', ';', '(', ')', '[', ']', '{', '},']) #actualización
```

```python
#Removemos las stopwords de la columna "Texto_corregido"
df1['Texto_corregido']=df1['Texto_corregido'].apply(lambda x: ' '.join([word for word in x.split() if word not in (stop)]))
```

### <mark style="color:blue;">Nube de palabras</mark>

Las nubes de palabras son una representación visual que muestra los términos más frecuentes en un conjunto de datos. Dichas palabras se presentan en tamaño proporcional a su frecuencia de aparición, es decir, las palabras más mencionadas serán aquellas de mayor dimensión.&#x20;

Las nubes de palabras permiten captar de forma rápida los términos más utilizados y relacionados cada candidato.

```python
# Unir todos los textos de los comentarios en una sola cadena
texto_completo1= ' '.join(df1['Texto_corregido'])

# Contar la frecuencia de cada palabra en el texto completo
counter = Counter(texto_completo1.split())

# Obtener las 50 palabras más comunes y almacenarlas en un diccionario
palabras_comunes = dict(counter.most_common(50))

# Crear la nube de palabras a partir de las frecuencias de las palabras
wordcloud = WordCloud(width=800, height=400, background_color='white').generate_from_frequencies(palabras_comunes)

# Mostrar la nube de palabras utilizando matplotlib
plt.figure(figsize=(10, 5))
plt.imshow(wordcloud, interpolation='bilinear')
plt.axis('off')
plt.title('Nube de Palabras: 50 palabras más utilizadas en comentarios sobre Myriam Bregman')
plt.show()
```

<figure><img src=".gitbook/assets/nubbe.png" alt=""><figcaption><p>Ejemplo nube de palabras de las 50 palabras más utilizadas en comentarios sobre la candidata Myriam Bregman</p></figcaption></figure>

### <mark style="color:blue;">Bigramas</mark>

Una técnica también muy utilizada es la utilización de n-gramas. Un n-grama se define como una secuencia de n-palabras consecutivas en un texto. Al analizar los n-gramas más frecuentes, podemos descubrir las combinaciones de palabras más utilizadas en los comentarios sobre cada candidato.

En este trabajo utilizamos n=2, que reciben el nombre de _**bigramas**_.

```python
# Dividir el texto en bigramas y contar su frecuencia
bigramas = list(ngrams(texto_completo1.split(), 2))
frecuencia_bigramas = Counter(bigramas)

# Obtener los 5 bigramas más comunes
top_bigramas = frecuencia_bigramas.most_common(5)

# Invertir el orden de los bigramas más comunes y sus frecuencias
top_bigramas.reverse()

# Separar los bigramas y sus frecuencias en listas separadas
bigram_labels, bigram_frequencies = zip(*top_bigramas)

# Crear un gráfico de barras horizontales con el orden invertido
plt.barh(range(len(top_bigramas)), bigram_frequencies, tick_label=[str(bigram) for bigram in bigram_labels])
plt.xlabel('Frecuencia')
plt.ylabel('Bigramas')
plt.title('Bigramas más comunes sobre comentarios de Myriam Bregman')

# Establecer el límite del eje X en 200
plt.xlim(0, 200)

plt.show()
```

<figure><img src=".gitbook/assets/muestrabi.png" alt=""><figcaption><p>Ejemplo 5 bigramas más frecuentes en comentarios de la candidata Myriam Bregman</p></figcaption></figure>

[^1]: 
