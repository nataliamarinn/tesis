---
description: >-
  eliminación de saltos de líneas - reemplazos -  eliminación de emojis - quita
  de hipervínculos - corrreción de ortografía - quita de caracteres especiales
layout:
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

# 🧹 Limpieza de datos

La **limpieza de datos** es un factor a considerar debido a que al tratarse de comentarios en redes sociales se debe lidiar con la presencia de errores ortográficos, emojis, hipervínculos y menciones a otros usuarios. Estos elementos, si no se manejan adecuadamente, pueden afectar la integridad semántica de los datos y comprometer la validez de los resultados.

## <mark style="background-color:blue;">**Librerías utilizadas**</mark>

* _<mark style="color:blue;">**pandas:**</mark>_ esta librería fue diseñada para facilitar el análisis y manipulación de datos en forma tabular. Permite cargar datos de diversas fuentes (ej: CSV, Excel, SQL), limpiarlos, filtrar y seleccionar datos, realizar operaciones estadísticas y generar visualizaciones sencillas. Se destaca esta librería por su capacidad de trabajar con grandes volúmenes de datos y su integración con otras librerías de análisis y visualización de datos.
* _<mark style="color:blue;">**openpyxl:**</mark>_ es una herramienta para trabajar con archivos de Excel en formato .xlsx. Permite leer, escribir y manipular hojas de cálculo.&#x20;
* _<mark style="color:blue;">**html:**</mark>_ esta librería permite analizar, manipular y generar contenido de HTML.
* _<mark style="color:blue;">**re:**</mark>_ es utilizada para trabajar con patrones de texto. Permite realizar búsquedas, extracciones y manipulaciones avanzadas de cadenas de texto, lo que resulta útil para las tareas de reemplazo y procesamiento de archivos de texto estructurados.&#x20;
* _<mark style="color:blue;">**nltk:**</mark>_ por sus siglas en inglés "Natural Language Toolkit"  es una librería muy importante para el procesamiento del lenguaje natural en Python. Proporciona funciones para la tokenización, lematización, análisis sintáctico, entre otras.&#x20;
* _<mark style="color:blue;">**unidecode:**</mark>_  se utiliza para transliterar cadenas de texto Unicode en ASCII, lo que significa que convierte caracteres Unicode (como las letras con tiles, diéresis, símbolos, o caracteres especiales) a sus equivalentes ASCII más cercanos. Esta librería resulta muy útil para normalizar cadenas de texto.
* _<mark style="color:blue;">**emoji:**</mark>_ esta librería proporciona herramientas para trabajar específicamente con emojis en Python, como la detección de emojis en cadenas de texto.&#x20;
* _<mark style="color:blue;">**spellchecker:**</mark>_ permite identificar y corregir errores ortográficos en texto, facilitando la tarea de asegurar la calidad del texto escrito en aplicaciones y procesos de análisis de dat

{% code overflow="wrap" lineNumbers="true" fullWidth="false" %}
````python
```python
import pandas as pd
import html
import re
import openpyxl
import nltk
from unidecode import unidecode
import emoji
from spellchecker import SpellChecker
```
````
{% endcode %}

## <mark style="background-color:blue;">Importar archivos de excel</mark>

Los archivos de Excel de cada candidato se encuentran en el siguiente zip

{% file src=".gitbook/assets/Datos.zip" %}

```python
# Definimos la ruta y nombre del archivo
# la función read_excel del paquete pandas lee el archivo

#Myriam Bregman
file_path1 = 'Bregman.xlsx'
df1 = pd.read_excel(file_path1)

#Patricia Bullrich
file_path2 = 'Bullrich.xlsx'
df2 = pd.read_excel(file_path1)


#Sergio Massa
file_path3 = 'Massa.xlsx'
df3 = pd.read_excel(file_path3)

# Javier Milei
file_path4 = 'Milei.xlsx'
df4 = pd.read_excel(file_path4)

# Juan Schiaretti
file_path5 = 'Schiaretti.xlsx'
df5 = pd.read_excel(file_path5)
```

## <mark style="background-color:blue;">Limpieza y filtrado de datos</mark>



### <mark style="color:blue;">Eliminación de saltos de línea</mark>&#x20;

Se quitan los saltos de línea en los comentarios para eliminar /n

Definimos la función _eliminar\_saltos()_

>
>
> ```python
> def eliminar_saltos(df):
>     """
>     Función para eliminar saltos de línea de un DataFrame en la columna 'Texto'.
>     
>     Parámetro de la función: df (DataFrame).Se espera que contenga una columna 
>     llamada 'Texto'.
>     
>     Esta función itera sobre cada fila del DataFrame y reemplaza los saltos de 
>     línea ("\n") en la columna 'Texto' por espacios en blanco. También utiliza 
>     html.unescape para decodificar cualquier entidad HTML en el texto.
>     """
>     
>     for index, row in df.iterrows():
>         if isinstance(row['Texto'], str):  
>             # Reemplazar saltos de línea por espacios en blanco
>             x = row['Texto'].replace("\n", " ")  
>             # Decodificar entidades HTML
>             df.at[index, 'Texto'] = html.unescape(x) 
>             
> ```



Aplicamos la función eliminar\_saltos() a los dataframe de cada candidato

```python
eliminar_saltos(df1)
eliminar_saltos(df2)
eliminar_saltos(df3)
eliminar_saltos(df4)
eliminar_saltos(df5)
```

### <mark style="color:blue;">Reemplazo de hastags y usuarios de cuentas oficiales</mark>



* _<mark style="color:blue;">Reemplazo de hastags:</mark>_ en esta etapa se reemplazan los hastags más utilizados por frases que hacen alusión al significado de cada hashtag. Este paso se realiza con el fin de retener más información.
* _<mark style="color:blue;">Reemplazo de menciones a cuentas oficiales:</mark>_ se reemplazan los usuarios de los cuentas oficiales de cada candidato arrobado por su apellido. Este paso asegura que no se elimine información relevante luego en otras etapas de la limpieza de datos.&#x20;

Se crea la función reemplazos()

````python
```python
def reemplazos(df, columna='Texto', columna_limpia='Texto_limpio'):
    # Diccionario de reemplazos para menciones y hashtags específicos
    reemplazos_dict = {
        '@Jmilei': 'Milei',
        '@myriambregman': 'Bregman',
        '@JMilei': 'Milei',
        '@PatoBullrich': 'Bullrich',
        '@SergioMassa': 'Massa',
        '@Schiaretti':'schiaretti',
        '@Javier': 'Javier',
        '@Milei': 'Milei',
        '#Argentina': "Argentina",
        '#Elecciones2023': 'Elecciones',
        "#Debate2023":"Debate",
        '#Milei': 'Milei',
        '#milei': 'milei',
        '#Massa': 'massa',
        '#massa': 'massa',
        "#MassaPresidente": 'massa presidente',
        '#Bullrich': 'Bullrich',
        '#bregman': "Bregman",
        '#Villaruel': 'Villaruel',
        '#Axel2023': "Kicillof",
        "#UniónPorLaPatria":"unión por la patria",
        'milei2023': 'milei presidente',
        'Milei2023': 'milei presidente',
        '#MileiesMassa': "Milei es Massa",
        "#MileiEsMassa": "Milei es Massa",
        "MyriamBregman2023": "Bregman presidente",
        "#massapresidente": "Massa presidente",
        "#SergioMassaPresidente": "Massa presidente",
        "#Debate2023": "Debate 2023",
        "#MileiPresidente": "Milei presidente",
        "#PatriciaBullrich2023": "Bullrich presidente",
        "#PatriciaEnPrimeraVuelta": "Bullrich presidente",
        "#PatriciaPresidente":"Bullrich presidente",
        "#Patopresidente": "Bullrich presidente",
        "#PatoPresidente": "Bullrich presidente",
        "#MassaPresidente2023": "Massa presidente",
        "#MassaPresidente": "Massa presidente",
        "#justiciaporsantiagomaldonado": "Justicia Maldonado",
        "#patopresidente": "Bullrich presidente",
        "#PatoPresidente": "Bullrich presidente",
        "#justicia":"justicia",
        "#patriciapresidenta": "Bullrich presidente",
        "#patobullrich": "Bullrich",
        "#MileiEnPrimeraVuelta": "Milei primera vuelta",
        "#Son30Mil": "son treinta mil",
        "#NuncaMás": "Nunca más",
        "30000": "treinta mil",
        "#Milei2023EnPrimeraVuelta": "Milei primera vuelta",
        '#mileipresidente': 'milei presidente',
        '#fuerontreintamil': 'fueron treinta mil',
        '#Nofueron30mil': 'no fueron treinta mil',
    }
    
    def procesar_texto(texto):
        if not isinstance(texto, str):
            return None
        
        texto_limpio = texto.lower()  # Convertir a minúsculas
        
        for patron, reemplazo in reemplazos_dict.items():
            texto_limpio = re.sub(re.escape(patron), reemplazo, texto_limpio, flags=re.IGNORECASE)
        
        # Verifica si quedan otras menciones o hashtags
        if re.search(r'@\w+|#\w+', texto_limpio):
            return None
        
        return texto_limpio if texto_limpio.strip() else None
    
    # Crear nueva columna con el texto procesado
    df[columna_limpia] = df[columna].apply(procesar_texto)
    
    # Eliminar filas donde el texto procesado es None o está vacío
    df = df.dropna(subset=[columna_limpia])
    df = df[df[columna_limpia].str.strip() != '']
    
    return df
```
````

Aplicamos la función reemplazos() a los dataframe de cada candidato

````python
```python
df1 = reemplazos(df1)
df2 = reemplazos(df2)
df3 = reemplazos(df3)
df4 = reemplazos(df4)
df5 = reemplazos(df5)
```
````

### <mark style="color:blue;">Eliminación de registros con hipervínculos</mark>

Se eliminan aquellos registros que contengan hipervínculos ya que la presencia de un enlace es un fuerte indicador que podría tratarse de un comentario que dirige hacia una noticia. En este caso, este tipo de comentarios no expresa ningún tipo de juicio de valor, por lo que eliminan.

````python
```python
def quitar_enlaces(df, columna='Texto_limpio'):
    """
    Función para limpiar un DataFrame eliminando registros que contienen hipervínculos,
    URLs acortadas, o están vacíos en la columna especificada.

    Parámetros:
    - df: DataFrame de pandas.
    - columna: Nombre de la columna que contiene el texto a procesar. Por defecto es 'Texto_limpio'.

    La función elimina registros si:
    1. El texto contiene un hipervínculo o URL acortada.
    2. El texto está vacío o solo contiene espacios en blanco.

    Retorna:
    - df: El DataFrame modificado con los registros no deseados eliminados.
    """
    def contiene_enlace_o_vacio(texto):
        if pd.isna(texto) or texto.strip() == '':
            return True
        
        # Convertir el valor a cadena
        texto = str(texto)
        
        # Patrón para detectar URLs, incluyendo URLs acortadas
        patron_url = r'(https?:\/\/)?[\w\-]+(\.[\w\-]+)+[/#?]?.*|\b\w+\.\w{2,3}\/\S*'
        
        # Verificar si el texto contiene una URL
        return bool(re.search(patron_url, texto))

    # Contar filas originales
    filas_originales = len(df)

    # Aplicar la función y filtrar el DataFrame
    df = df[~df[columna].apply(contiene_enlace_o_vacio)].copy()

    # Restablecer índices
    df.reset_index(drop=True, inplace=True)

    # Contar filas eliminadas
    filas_eliminadas = filas_originales - len(df)

    print(f"Filas originales: {filas_originales}")
    print(f"Filas eliminadas: {filas_eliminadas}")
    print(f"Filas restantes: {len(df)}")

    return df
```
````

````python
```python
df1 = quitar_enlaces(df1)
df2 = quitar_enlaces(df2)
df3 = quitar_enlaces(df3)
df4 = quitar_enlaces(df4)
df5 = quitar_enlaces(df5)
```
````

### <mark style="color:blue;">Quitar caracteres especiales</mark>

Se eliminan caracteres como los signos de puntuación, numerales, astericos, ya que no aportan mucha información semántica y pueden producir ruido en el procesamiento.

Se define la función quitar\_caracteres()

````python
```python
def quitar_caracteres(df):
    """
    Función para limpiar un DataFrame eliminando caracteres especiales, no alfanuméricos y hashtags en el texto.

    Parámetros:
    - df: DataFrame de pandas. Se espera que contenga una columna llamada 'Texto'.

    La función itera sobre cada fila del DataFrame y elimina caracteres especiales, no alfanuméricos y hashtags
    en el texto, manteniendo solo caracteres alfanuméricos y espacios.

    Retorna:
    - df: El DataFrame modificado con los caracteres especiales, no alfanuméricos y hashtags eliminados del texto.
    """
    # Iterar sobre las filas del DataFrame
    for index, row in df.iterrows():
        # Convertir el valor de 'Texto_limpio' a cadena
        text_value = str(row['Texto_limpio'])

        # Eliminar caracteres especiales, no alfanuméricos y hashtags
        clean_text = re.sub(r"[^\w\s]|#", "", text_value)

        # Actualizar el texto en el DataFrame
        df.at[index, 'Texto_limpio'] = clean_text

    return df
```
````

Aplicamos la función quitar\_caracteres() a los dataframe de cada candidato

````python
```python
df1 = quitar_caracteres(df1)
df2 = quitar_caracteres(df2)
df3 = quitar_caracteres(df3)
df4 = quitar_caracteres(df4)
df5 = quitar_caracteres(df5)
```
````

### <mark style="color:blue;">Eliminación de emojis</mark>

Se eliminan los emojis de los comentarios, ya que los mismos no pueden ser interpretados correctamente por los modelos de clasificación utilizados.

Se define la función eliminar\_emojis()

```python
def eliminar_emojis(texto):
    """
    Función para eliminar emojis de un texto.
    
    Parámetro de la función: texto, cadena de texto que puede contener emojis.

    La función retorna texto sin emojis.
    """
    return emoji.get_emoji_regexp().sub(r'', texto)
```

Se aplica la función eliminar\_emojis() a la columna "Texto" de cada dataframe&#x20;

````python
```python
df1['Texto_limpio'] = df1['Texto_limpio'].apply(eliminar_emojis)
df2['Texto_limpio'] = df2['Texto_limpio'].apply(eliminar_emojis)
df3['Texto_limpio'] = df3['Texto_limpio'].apply(eliminar_emojis)
df4['Texto_limpio'] = df4['Texto_limpio'].apply(eliminar_emojis)
df5['Texto_limpio'] = df5['Texto_limpio'].apply(eliminar_emojis)

```
````

### <mark style="color:blue;">Conversión de texto a minúscula</mark>

Este proceso reduce la dimensionalidad, estandarizando las palabras para evitar duplicidades innecesarias.

La función str.lower() es método integrado en Python que se utiliza para convertir una cadena de texto a minúsculas. Se aplica dicha función en la columna "Texto" de cada dataframe.

````
```python
df1['Texto_limpio'] = df1['Texto_limpio'].str.lower()
df2['Texto_limpio'] = df2['Texto_limpio'].str.lower()
df3['Texto_limpio'] = df3['Texto_limpio'].str.lower()
df4['Texto_limpio'] = df4['Texto_limpio'].str.lower()
df5['Texto_limpio'] = df5['Texto_limpio'].str.lower()
```
````

### <mark style="color:blue;">Correción de errores de ortografía</mark>

Al tratarse de comentarios de redes sociales y no texto técnico, los comentarios contienen muchos errores de ortografía que resultan en un aumento de dimensionalidad. Para evitar esto se realiza una limpieza de errores mediante la librería de Python pyspellchecker que utiliza el algoritmo de la distancia de Levenshtein para encondtrar variaciones de una palabra en 2 ediciones desde la palabra original para luego reemplazarlas.&#x20;

Si bien esta librería admite el idioma español, es necesario realizar una actualización del diccionario para incoporar palabras utilizadas en Argentina y también nombres propios mencionados en los comentarios

````python
# Indicamos con language='es' que el idioma utilizado es el español
```python
spellchecker = SpellChecker(language='es')
```
````

````python
# Agregar nuevas palabras al diccionario
```python
nuevas_palabras = ['campora','vllc','bullrich','cavallo','prefiero','vamosnos','conozco','quisiera','venis','factos','charlatan','bla','zarasa','CFK','bregman','javier','gatito','eeuu','mirtha','fatima','schiaretti','jxc','bcra','voucher','clarin','fogonear','massita','dolarizar','rodrigazo','doparon','nazi','polarizacion','wacho','dolarizacion','pb','mb','messi', 'neoliberal','libertario', 'espert', 'donando', 'tribunera','tribuna', 'baradel', 'piola', 'memes','fafafa', 'ñoquis', 'cachivache', 'punteros', 'alberto','fernandez', 'slogan', 'chavez', 'rusa', 'zelinski', 'votemos', 'pedo', 'berreta', 'paseo', 'dubai', 'tucuman', 'porteños','bs','as', 'amba', 'cba', 'biden', 'falluto', 'batakis', 'petri', 'evita','mamarracho','hdmp', 'guzman', 'presidente', 'existe','planes', 'siendo','querer','cambio','cambiar', 'estuviste','cátedra',
                   'Eunerkian','vota','iphone','osde','voto','miryam','miriam','mirian','carajo','pelotudo','che','javo','oligarcas','recoleta','viale','barrabravas','feimann','motosierra','bananero','negativo','mate','garca', 'vino','narcotrafico','fundidos','reprimir','represion','lcdtm','fundir','pbi','rua','mamarracho','desquiciado','medicado','puede','humanizo','agustin','descoloco','descerebrados','chaborra','chupi','tomada','milei','borracha','montonera','barrionuevo','kirchnerismo','kicillof','bolsonaro','macri','macrismo','menem','menemismo','menemista','cambiemos','pro','trump','chanta','videla','tiktok','conviene','lla','shipeo','mword','bolas', 'chaco','jujuy','wakanda','sota', 'cararrota', 'nono', 'choreo','chorean', 'aguante', 'adoctrinamiento', 'uxp', 'che', 'narco','cancelado','massarasa', 'amba', 'esta', 'perdio', 'fmi',
                   'balotage','patri','siguen','chanta','eduque','pido', 'bendiga','récord','llego','mileli','ojala','hubieron', 'ex','siente','cínica','dio','panqueque','iba','chau','leliq','genia','dubai','perdiste','hubiera','veces','tengo','hablaba','crack','voten','robaste','juancito','pudo','mentiste','docentes','pibe','dale','crei','randazzo','bregman','negacionista','papelon','socialistas','UCR','facha','canchero','lta','liqui','troll','nazi','marra','massera','videla','sobrador','fachos','espert','corralito','porteño','molotov','negacionismo','patito','enserio','larreta','justicialismo','peron','empinar','malbec','libertarios','patagonia','grindetti','villaruel','tiene','piparo','fantino','melconian','piquete','queme','xq','kici','tachando','quiero','drogadicto','piqueteros','boludo','cgt','moyano','massa','pullaro','kirchner','hubo','esta', 'like', 'zurdita', 'Bonafini','mudarme','ladri','ucr', 'tribunero', 'doña', 'empobrecer','empobrecimiento', 'jeta','ventajita','cordobeza', 'zombie', 'manotazo', 'alberto', 'baradel','pandemia','pami', 'lcdtm',
                   'kirchneristas','kirchnerismo','peron','seremos','millones','chauuu','tomatela','tenemos','axel','sabemos','ñoqui','queremos','unicas','labura','laburar','fueron','lacra','comoda','clarisima','zarasa','melco','falacias','pinocho','cagador','peronismo','afjp','jeje','chupi','humille','meados','meada','boludito','boludita','tenes','tenés','jajaja','carajos','estamos','estan','rivotril','facebook','tinelli','directo','vamos','será','instagram','trosky','va','novaresio','martinez','hoz','bukele','terraplanista','dopar','dopado','presi','trosko','einstein','bue','crack','clona','petaca','transa','moyano','populista','90','30000','2023','30','3','vaga','ole', 'genocidio','socialista', 'cabida','chicana', 'conventillo','merquero','marbella','dope','invotable', 'groso','comunismo', 'paros', 'zurdita', 'culiao', 'yendo', 'podes',
                   'myriam','malvinas','margaret','a','podría','tarado','bla','ubicando','prosti','sra','parripollo','Rigau','tarada','tarados','mamita','cris','salen','labura','laburador','chances','laburadora','tatcher','cinico','progre','gringo','boluda','dijera','explayo','pelotuda','bondi','comodin','fulero','rossi','dopar','platita','currar','dopado','ogt','hicieron','destruido','caradura','gps','escabio','cachivache','sarmiento','atorrante','fulmino','transa','perdiste','dijo','estuvo','malandra','maldonado','insauralde','córdoba','matanza','gil','infobae','villero','ensobrado','chanta','hacemos','melco','peronistas','hizo','jaja','carita','basado','sergio','tomas','diria','chamuyero','chamuyo','chamuyar','abortera','pedorro','zanganos','chusmerio','iphone','kukas','caradura','llaryora','sorete','interventor','lavagna','pinocho']

```
````

````python
```python
# Agregar las nuevas palabras al diccionario de una sola vez
spellchecker.word_frequency.load_words(nuevas_palabras)
```
````

````python
# Función para realizar el spellcheck en una fila
```python
def spellcheck_text(texto):
    """
    Función para corregir la ortografía de un texto.
    """
    if pd.isna(texto) or texto == '':
        return ''  # Devolver string vacío para valores nulos o vacíos
    if not isinstance(texto, str):
        texto = str(texto)  # Convertir a string si no lo es
    
    palabras = texto.split()
    palabras_corregidas = []
    for palabra in palabras:
        if palabra.lower() in nuevas_palabras or not palabra.isalpha():
            palabras_corregidas.append(palabra)
        else:
            palabra_corregida = spellchecker.correction(palabra)
            if palabra_corregida is not None:
                palabras_corregidas.append(palabra_corregida)
            else:
                palabras_corregidas.append(palabra)  # Mantener la palabra original si no hay corrección
    return ' '.join(palabras_corregidas)

def clean_and_apply_spellcheck(df):
    """
    Limpia el DataFrame y aplica la corrección ortográfica.
    """
    if 'Texto_limpio' not in df.columns:
        print("La columna 'Texto_limpio' no existe en el DataFrame.")
        return df
    
    # Eliminar filas con valores nulos o vacíos en 'Texto_limpio'
    df = df.dropna(subset=['Texto_limpio'])
    df = df[df['Texto_limpio'].astype(str).str.strip() != '']
    
    # Aplicar la corrección ortográfica
    df['Texto_corregido'] = df['Texto_limpio'].apply(spellcheck_text)
    return df
```
````

Se aplica la función spellcheck\_row() a la columna "Texto" de cada dataframe&#x20;

````python
```python
# Lista de DataFrames a procesar
dataframes = [df1, df2, df3, df4, df5]

# Aplicar la limpieza y corrección ortográfica a cada DataFrame
for i, df in enumerate(dataframes, 1):
    print(f"Procesando df{i}...")
    try:
        # Imprimir información de diagnóstico
        print(f"Filas originales en df{i}: {len(df)}")
        dataframes[i-1] = clean_and_apply_spellcheck(df)
        print(f"Filas después de la limpieza en df{i}: {len(dataframes[i-1])}")
        print(f"df{i} procesado exitosamente.")
    except Exception as e:
        print(f"Error al procesar df{i}: {str(e)}")

# Reasignar los DataFrames procesados a sus variables originales
df1, df2, df3, df4, df5 = dataframes
```
````

{% hint style="warning" %}
La aplicación de la función spellcheck\_row puede llevar más tiempo que otras funciones aplicadas previamente debido a que debe evaluar múltiples palabras y muchas filas.
{% endhint %}

### <mark style="color:blue;">Eliminación de registros vacíos</mark>

Después de completar los pasos anteriores, es crucial eliminar los registros vacíos que podrían haber quedado como resultado de este proceso.

```python
# Eliminar registros vacíos en la columna 'Texto_corregido'
df1.dropna(subset=['Texto_corregido'], inplace=True)
df2.dropna(subset=['Texto_corregido'], inplace=True)
df3.dropna(subset=['Texto_corregido'], inplace=True)
df4.dropna(subset=['Texto_corregido'], inplace=True)
df5.dropna(subset=['Texto_corregido'], inplace=True)ode
```

```python
# Restablecer índices después de eliminar filas
df1.reset_index(drop=True, inplace=True)
df2.reset_index(drop=True, inplace=True)
df3.reset_index(drop=True, inplace=True)
df4.reset_index(drop=True, inplace=True)
df5.reset_index(drop=True, inplace=True)
```

### <mark style="color:blue;">Diccionario de palabras</mark>

En este apartado se busca agupar palabras en un único término, con el fin de reducir la dimensionalidad.

````python
```python
def reemplazar_palabras(texto):
    """
    Función para reemplazar palabras específicas en el texto.
    """
    if pd.isna(texto) or not isinstance(texto, str):
        return texto

    # Diccionario de reemplazos
    reemplazos = {
        ('miloski','mileli','miley'): 'milei',
        ('masa','masita'): 'massa',
        ('myriam','miria', 'miram','miryam','mirian','mb'): 'bregman',
        ('3'): 'tres',
        ('vllc'): 'viva la libertad carajo',
        ('30',): 'treinta',
        ('30000'): 'treinta mil',
        ('90',): 'noventa',
        ('eeuu'): 'estados unidos',
    }

    for variantes, reemplazo in reemplazos.items():
        patron = r'\b(' + '|'.join(re.escape(v) for v in variantes) + r')\b'
        texto = re.sub(patron, reemplazo, texto, flags=re.IGNORECASE)

    return texto

```

````

````python
```python
df1['Texto_limpio'] = df1['Texto_limpio'].apply(reemplazar_palabras)
df2['Texto_limpio'] = df2['Texto_limpio'].apply(reemplazar_palabras)
df3['Texto_limpio'] = df3['Texto_limpio'].apply(reemplazar_palabras)
df4['Texto_limpio'] = df4['Texto_limpio'].apply(reemplazar_palabras)
df5['Texto_limpio'] = df5['Texto_limpio'].apply(reemplazar_palabras)
```
````

### <mark style="color:blue;">Eliminar registros vacíos</mark>

````python
```python
dataframes = [df1, df2, df3, df4, df5]

for i, df in enumerate(dataframes, 1):
    df_cleaned = df[df['Texto_limpio'].notna() & (df['Texto_limpio'] != ' ')& (df['Texto_limpio'] != '  ')]
    globals()[f'df{i}'] = df_cleaned
    print(f"Registros en df{i} antes: {len(df)}, después: {len(df_cleaned)}")
```
````

### <mark style="color:blue;">Guardar archivos</mark>&#x20;

Guardamos en Excel los archivos para el siguiente apartado "Análisis Exploratorio". \
Este paso es totalmente opcional.

```python
excel_file1= "filtrado_bullrich.xlsx"
df1.to_excel(excel_file1, index=False)
excel_file2= "filtrado_massa.xlsx"
df2.to_excel(excel_file2, index=False)
excel_file3= "filtrado_schiaretti.xlsx"
df3.to_excel(excel_file3, index=False)
excel_file4= "filtrado_milei.xlsx"
df4.to_excel(excel_file4, index=False)
excel_file5= "filtrado_bregman.xlsx"
df5.to_excel(excel_file5, index=False)
```

A continuación se adjuntan dichos archivos

{% file src=".gitbook/assets/Datos_limpios.zip" %}
