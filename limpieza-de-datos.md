---
description: >-
  eliminación de saltos de líneas - reemplazos -  eliminación de emojis - quita
  de hipervínculos - corrreción de ortografía - quita de caracteres especiales
cover: .gitbook/assets/Limpieza_imagen.png
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

# 🧹 Limpieza de datos

La **limpieza de datos** es un factor a considerar debido a que al tratarse de comentarios en redes sociales se debe lidiar con la presencia de errores ortográficos, emojis, hipervínculos y menciones a otros usuarios. Estos elementos, si no se manejan adecuadamente, pueden afectar la integridad semántica de los datos y comprometer la validez de los resultados.

## <mark style="background-color:green;">**Librerías utilizadas**</mark>

{% code overflow="wrap" lineNumbers="true" fullWidth="false" %}
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
{% endcode %}

## <mark style="background-color:green;">Importar archivos de excel</mark>

Los archivos de Excel de cada candidato se encuentran en el siguiente zip

{% file src=".gitbook/assets/Datos_candidatos (1).zip" %}

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

## <mark style="background-color:green;">Limpieza y filtrado de datos</mark>

### Eliminación de saltos de línea&#x20;

Se quitan los saltos de línea en los comentarios para eliminar el carácter especial \n

Definimos la función _eliminar\_saltos()_

> ```
> def eliminar_saltos(df):
> ```
>
> ```python
>     """
>     Función para eliminar saltos de línea de un DataFrame en la columna 'Texto'.
>     
>     Parámetros: df (pd.DataFrame): DataFrame que contiene la columna a procesar.
>
>     Retorna: pd.DataFrame: DataFrame con la columna 'Texto' sin saltos de línea
>
>     Esta función itera sobre cada fila del DataFrame y reemplaza los saltos de 
>     línea ("\n") en la columna 'Texto' por espacios en blanco. Además, utiliza 
>     html.unescape para decodificar cualquier entidad HTML en el texto.
>     """
>     for index, row in df.iterrows():
>         if isinstance(row['Texto'], str):  
>             # Reemplazar saltos de línea por espacios en blanco
>             x = row['Texto'].replace("\n", " ")  
>             # Decodificar entidades HTML
>             df.at[index, 'Texto'] = html.unescape(x)
> ```



Aplicamos la función eliminar\_saltos() a los dataframe de cada candidato

```python
eliminar_saltos(df1)
eliminar_saltos(df2)
eliminar_saltos(df3)
eliminar_saltos(df4)
eliminar_saltos(df5)
```

### <mark style="color:green;">Reemplazo de hastags (#) y menciones de usuarios</mark>

Se reemplazan los hashtags más utilizados y que aportan un significado relevante al posible sentimiento del comentario. Los hashtags menos frecuentes o que no aportaban un significado relevante al contexto se eliminan.

En cuanto a la menciones de usuarios se dividió el proceso en dos partes, dependiendo si la mención es a una cuenta oficial de alguno de los candidatos presidenciales o a otros usuarios. En los casos donde se menciona la cuenta oficial de alguno de los candidatos , esta mención se reemplaza por el apellido del candidato correspondiente, asegurando así que no se pierda información. Por otra parte, los comentarios que contienen menciones (@) a usuarios que no fueran los candidatos son eliminados del conjunto de datos. Esta decisión se toma para evitar incluir respuestas entre usuarios o comentarios que hicieran referencia principalmente a otros individuos en lugar de los candidatos presidenciales.

Se crea la función reemplazos()

```python
def reemplazos(df, columna='Texto', columna_limpia='Texto_limpio'):
    """
    Procesa y limpia el texto en una columna de un DataFrame, reemplazando menciones
    y hashtags específicos.

    Parámetros: df (pd.DataFrame): DataFrame que contiene la columna a procesar.
    columna (str): Nombre de la columna que contiene el texto original. 
    Por defecto es 'Texto'.
    columna_limpia (str): Nombre de la nueva columna donde se almacenará el texto
    limpio. Por defecto es 'Texto_limpio'.

    Retorna: pd.DataFrame: DataFrame con una nueva columna que contiene el texto
    procesado, y elimina filas donde el texto procesado es None o está vacío.
    """
    
    # Diccionario de reemplazos para menciones y hashtags
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
        """
        Limpia el texto aplicando los reemplazos definidos.

        Parámetros:
        texto (str): Texto a procesar.

        Retorna:
        str o None: Texto limpio o None si no se puede procesar.
        """
        
        if not isinstance(texto, str):
            return None
        
        texto_limpio = texto.lower()  # Convertir a minúsculas
        
        # Reemplazar menciones y hashtags
        for patron, reemplazo in reemplazos_dict.items():
            texto_limpio = re.sub(re.escape(patron), reemplazo, texto_limpio, 
            flags=re.IGNORECASE)
        
        # Verifica si quedan otras menciones o hashtags
        if re.search(r'@\w+|#\w+', texto_limpio):
            return None
        
        return texto_limpio.strip() if texto_limpio.strip() else None 
        #stip se utiliza para eliminar cualquier espacio blanco y al final de
        #la cadena. Asegura que no haya espacios innecesarios
    
    # Crear nueva columna con el texto procesado
    df[columna_limpia] = df[columna].apply(procesar_texto)
    
    # Eliminar filas donde el texto procesado es None o está vacío
    df = df.dropna(subset=[columna_limpia])
    df = df[df[columna_limpia].str.strip() != ''] 
    
    return df
```

Aplicamos la función reemplazos() a los dataframe de cada candidato

```python
df1 = reemplazos(df1)
df2 = reemplazos(df2)
df3 = reemplazos(df3)
df4 = reemplazos(df4)
df5 = reemplazos(df5)
```

### <mark style="color:green;">Eliminación de registros con hipervínculos</mark>

Se eliminan aquellos registros que contengan hipervínculos ya que la presencia de un enlace es un fuerte indicador de que el comentario podría dirigir hacia una página web que amplía como noticia lo mostrado en la publicación original. Este tipo de comentarios no expresan ningún tipo de juicio de valor, por lo que se eliminan.

Se crea la función quitar\_enlaces()

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

```python
df1 = quitar_enlaces(df1)
df2 = quitar_enlaces(df2)
df3 = quitar_enlaces(df3)
df4 = quitar_enlaces(df4)
df5 = quitar_enlaces(df5)
```

### <mark style="color:green;">Quitar caracteres especiales</mark>

Se eliminan caracteres como los signos de puntuación, numerales, astericos, ya que no aportan mucha información semántica y pueden producir ruido en el procesamiento.

Se define la función quitar\_caracteres()

```python
def quitar_caracteres(df):
    """
    Función para limpiar un DataFrame eliminando caracteres especiales, no 
    alfanuméricos y hashtags en el texto.

    Parámetros:
    - df: DataFrame de pandas. Se espera que contenga una columna llamada 
    'Texto_limpio'.

    La función itera sobre cada fila del DataFrame y elimina caracteres especiales, 
    no alfanuméricos y hashtags en el texto, manteniendo solo caracteres 
    alfanuméricos y espacios.

    Retorna:
    - df: El DataFrame modificado con los caracteres especiales, no alfanuméricos
    y hashtags eliminados del texto.
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

Aplicamos la función quitar\_caracteres() a los dataframe de cada candidato

```python
df1 = quitar_caracteres(df1)
df2 = quitar_caracteres(df2)
df3 = quitar_caracteres(df3)
df4 = quitar_caracteres(df4)
df5 = quitar_caracteres(df5)
```

### <mark style="color:green;">Eliminación de emojis</mark>

Se eliminan los emojis de los comentarios, ya que los mismos no pueden ser interpretados correctamente por los modelos de clasificación utilizados.

Se define la función eliminar\_emojis()

```python
def eliminar_emojis(texto):
    """
    Elimina todos los emojis de un texto dado.

    Esta función utiliza una expresión regular para identificar y eliminar 
    todos los emojis presentes en la cadena de texto proporcionada. Es útil 
    para limpiar textos que pueden contener caracteres no deseados, como 
    emojis, antes de realizar análisis de texto o procesamiento adicional.

    Parámetros:
    - texto : str (La cadena de texto de la cual se desean eliminar los emojis)

    Retorna:
    -str (Una nueva cadena de texto sin emojis)
    """
    return emoji.get_emoji_regexp().sub(r'', texto)
```

Se aplica la función eliminar\_emojis() a la columna "Texto" de cada dataframe&#x20;

```python
df1['Texto_limpio'] = df1['Texto_limpio'].apply(eliminar_emojis)
df2['Texto_limpio'] = df2['Texto_limpio'].apply(eliminar_emojis)
df3['Texto_limpio'] = df3['Texto_limpio'].apply(eliminar_emojis)
df4['Texto_limpio'] = df4['Texto_limpio'].apply(eliminar_emojis)
df5['Texto_limpio'] = df5['Texto_limpio'].apply(eliminar_emojis)
```

### <mark style="color:green;">Correción de errores de ortografía</mark>

Al tratarse de comentarios de redes sociales y no texto técnico, los comentarios contienen muchos errores de ortografía que resultan en un aumento de dimensionalidad. Las herramientas estándar de corrección ortográfica, como la librería de Python pyspellchecker, suelen basarse en diccionarios generales del español que no contemplan las particularidades del castellano argentino ni los nombres propios relevantes para este estudio.

Para abordar estos retos, se implementó un proceso de dos etapas:

* Inicialmente, se enriqueció el diccionario de pyspellchecker con términos específicos del contexto argentino y nombres propios frecuentes utilizados en el debate presidencial. Luego se aplicó la corrección ortográfica utilizando este nuevo diccionario personalizado.
* La limpieza de errores ortográficos mediante esta librería se basa en algoritmo de distancia de Levenshtein, proceso que comienza comparando cada palabra del texto con las entradas del diccionario incorporado. Cuando se detecta una palabra que no está en diccionario, se calcula la distancia de Levenshtein entre esta palabra y las del diccionario y el algoritmo sugiere como corrección la palabra con menor distancia.

```python
# Indicamos con language='es' que el idioma utilizado es el español
spellchecker = SpellChecker(language='es')

```

```python
# Agregar nuevas palabras al diccionario
nuevas_palabras = ['campora','vllc','bullrich','cavallo','prefiero','vamosnos','conozco','quisiera','venis','factos','charlatan','bla','zarasa','CFK','bregman',
                   'javier','gatito','eeuu','mirtha','fatima','schiaretti','jxc','bcra','voucher','clarin','fogonear','massita','dolarizar','rodrigazo','doparon',
                   'nazi','polarizacion','wacho','dolarizacion','pb','mb','messi', 'neoliberal','libertario', 'espert', 'donando', 'tribunera','tribuna', 'baradel', 
                   'piola', 'memes','fafafa', 'ñoquis', 'cachivache', 'punteros', 'alberto','fernandez', 'slogan', 'chavez', 'rusa', 'zelinski', 'votemos', 'pedo', 
                   'berreta', 'paseo', 'dubai', 'tucuman', 'porteños','bs','as', 'amba', 'cba', 'biden', 'falluto', 'batakis', 'petri', 'evita','mamarracho','hdmp',
                   'guzman', 'presidente', 'existe','planes', 'siendo','querer','cambio','cambiar', 'estuviste','cátedra','Eunerkian','vota','iphone','osde','voto',
                   'miryam','miriam','mirian','carajo','pelotudo','che','javo','oligarcas','recoleta','viale','barrabravas','feimann','motosierra','bananero','negativo',
                   'mate','garca', 'vino','narcotrafico','fundidos','reprimir','represion','lcdtm','fundir','pbi','rua','mamarracho','desquiciado','medicado','puede',
                   'humanizo','agustin','descoloco','descerebrados','chaborra','chupi','tomada','milei','borracha','montonera','barrionuevo','kirchnerismo','kicillof',
                   'bolsonaro','macri','macrismo','menem','menemismo','menemista','cambiemos','pro','trump','chanta','videla','tiktok','conviene','lla','shipeo','mword',
                   'bolas', 'chaco','jujuy','wakanda','sota', 'cararrota', 'nono', 'choreo','chorean', 'aguante', 'adoctrinamiento', 'uxp', 'che', 'narco','cancelado',
                   'massarasa', 'amba', 'esta', 'perdio', 'fmi', 'balotage','patri','siguen','chanta','eduque','pido', 'bendiga','récord','llego','mileli','ojala',
                   'hubieron', 'ex','siente','cínica','dio','panqueque','iba','chau','leliq','genia','dubai','perdiste','hubiera','veces','tengo','hablaba','crack',
                   'voten','robaste','juancito','pudo','mentiste','docentes','pibe','dale','crei','randazzo','bregman','negacionista','papelon','socialistas','UCR',
                   'facha','canchero','lta','liqui','troll','nazi','marra','massera','videla','sobrador','fachos','espert','corralito','porteño','molotov','negacionismo',
                   'patito','enserio','larreta','justicialismo','peron','empinar','malbec','libertarios','patagonia','grindetti','villaruel','tiene','piparo','fantino',
                   'melconian','piquete','queme','xq','kici','tachando','quiero','drogadicto','piqueteros','boludo','cgt','moyano','massa','pullaro','kirchner','hubo',
                   'esta', 'like', 'zurdita', 'Bonafini','mudarme','ladri','ucr', 'tribunero', 'doña', 'empobrecer','empobrecimiento', 'jeta','ventajita','cordobeza',
                   'zombie', 'manotazo', 'alberto', 'baradel','pandemia','pami', 'lcdtm','kirchneristas','kirchnerismo','peron','seremos','millones','chauuu','tomatela',
                   'tenemos','axel','sabemos','ñoqui','queremos','unicas','labura','laburar','fueron','lacra','comoda','clarisima','zarasa','melco','falacias','pinocho',
                   'cagador','peronismo','afjp','jeje','chupi','humille','meados','meada','boludito','boludita','tenes','tenés','jajaja','carajos','estamos','estan',
                   'rivotril','facebook','tinelli','directo','vamos','será','instagram','trosky','va','novaresio','martinez','hoz','bukele','terraplanista','dopar',
                   'dopado','presi','trosko','einstein','bue','crack','clona','petaca','transa','moyano','populista','90','30000','2023','30','3','vaga','ole', 'genocidio',
                   'socialista', 'cabida','chicana', 'conventillo','merquero','marbella','dope','invotable', 'groso','comunismo', 'paros', 'zurdita', 'culiao', 'yendo', 
                   'podes','myriam','malvinas','margaret','a','podría','tarado','bla','ubicando','prosti','sra','parripollo','Rigau','tarada','tarados','mamita',
                   'cris','salen','labura','laburador','chances','laburadora','tatcher','cinico','progre','gringo','boluda','dijera','explayo','pelotuda','bondi',
                   'comodin','fulero','rossi','dopar','platita','currar','dopado','ogt','hicieron','destruido','caradura','gps','escabio','cachivache','sarmiento',
                   'atorrante','fulmino','transa','perdiste','dijo','estuvo','malandra','maldonado','insauralde','córdoba','matanza','gil','infobae','villero',
                   'ensobrado','chanta','hacemos','melco','peronistas','hizo','jaja','carita','basado','sergio','tomas','diria','chamuyero','chamuyo','chamuyar',
                   'abortera','pedorro','zanganos','chusmerio','iphone','kukas','caradura','llaryora','sorete','interventor','lavagna','pinocho']


```

```python
# Agregar las nuevas palabras al diccionario de una sola vez
spellchecker.word_frequency.load_words(nuevas_palabras)
```

Se definen las funciones `spellcheck_text` y `clean_and_apply_spellcheck`para corregir la ortografía de textos en un DataFrame de pandas. La función `spellcheck_text` toma un texto como entrada, verifica si las palabras están en una lista de palabras aceptadas y utiliza un corrector ortográfico para sugerir correcciones cuando es necesario. Mantiene sin cambios las palabras nulas, vacías o no alfabéticas. Por otro lado, `clean_and_apply_spellcheck` se encarga de limpiar el DataFrame eliminando filas con valores nulos o vacíos en la columna 'Texto\_limpio' y aplica la función de corrección ortográfica a cada entrada, almacenando los resultados en una nueva columna llamada 'Texto\_corregido'.\


<pre class="language-python"><code class="lang-python">def spellcheck_text(texto):
    """
    Corrige la ortografía de un texto dado.

    Esta función toma un texto como entrada y corrige las palabras que no están en 
    la lista de palabras aceptadas (`nuevas_palabras`). Si una palabra no se 
    encuentra en la lista, se utiliza un corrector ortográfico para sugerir una 
    corrección. Las palabras que son nulas, vacías o que no son alfabéticas se
    mantienen sin cambios.

    Parámetros:
    -texto : str (El texto a corregir)

    Retorna:
    str (El texto corregido)
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
                palabras_corregidas.append(palabra)  # Mantener la palabra original
                #si no hay corrección
                
    return ' '.join(palabras_corregidas)


def clean_and_apply_spellcheck(df):
    """
    Limpia el DataFrame y aplica la corrección ortográfica a la columna 
    'Texto_limpio'.

    Esta función verifica si la columna 'Texto_limpio' existe en el DataFrame. 
    Si existe, elimina las filas con valores nulos o vacíos en esa columna y luego 
    aplica la función `spellcheck_text` a cada entrada de la columna para corregir
    su ortografía. Los resultados se almacenan en una nueva columna llamada 
    'Texto_corregido'.

    Parámetros:
    -df : pandas.DataFrame (contiene la columna 'Texto_limpio' que se desea limpiar
    y corregir)

    Retorna:
    -pandas.DataFrame (DataFrame original con una nueva columna 'Texto_corregido')
    
<strong>    """
</strong>
    if 'Texto_limpio' not in df.columns:
        print("La columna 'Texto_limpio' no existe en el DataFrame.")
        return df
    
    # Eliminar filas con valores nulos o vacíos en 'Texto_limpio'
    df = df.dropna(subset=['Texto_limpio'])
    df = df[df['Texto_limpio'].astype(str).str.strip() != '']
    
    # Aplicar la corrección ortográfica
    df['Texto_corregido'] = df['Texto_limpio'].apply(spellcheck_text)
    
    return df
</code></pre>

Aplicamos la función clean\_and\_apply\_spellcheck() a los dataframe de cada candidato

````
# lista de dataframes a los que vamos a aplicar la función clean_and_apply_spellcheck
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
La aplicación de estas dos funciones puede llevar más tiempo que otras funciones aplicadas previamente debido a que debe evaluar múltiples palabras y muchas filas.
{% endhint %}

### <mark style="color:green;">Diccionario de palabras</mark>

Se utiliza un diccionario de reemplazos para corregir automáticamente palabras y frases que pueden estar mal escritas o son abreviaciones, transformándolas en sus formas correctas y aceptadas. Sin embargo, es importante destacar que solo se corrigen aquellas instancias donde los nombres propios están mal escritos; las referencias correctas a la misma persona se mantienen intactas.\
\
Se define la función reemplazar\_palabras()

```python
def reemplazar_palabras(texto):
    """
    Función para reemplazar palabras específicas en el texto.

    Parámetros:
    -texto : str (donde se realizarán los reemplazos)

    Retorna:
    -str (El texto con las palabras reemplazadas según el diccionario definido)
    """

    # Verifica si el texto es nulo o no es una cadena, y devuelve el valor original 
    #si es así.
    if pd.isna(texto) or not isinstance(texto, str):
        return texto

    # Diccionario de reemplazos donde las claves son tuplas de variantes a ser 
    #reemplazadas y los valores son las palabras/frases por las que se reemplazarán.
    reemplazos = {
        ('miloski', 'mileli', 'miley'): 'milei',
        ('masa', 'masita'): 'massa',
        ('myriam', 'miria', 'miram', 'miryam', 'mirian', 'mb'): 'bregman',
        ('3'): 'tres',
        ('vllc'): 'viva la libertad carajo',
        ('30',): 'treinta',
        ('30000'): 'treinta mil',
        ('90',): 'noventa',
        ('eeuu'): 'estados unidos',
    }

    # Itera sobre cada grupo de variantes y su correspondiente reemplazo.
    for variantes, reemplazo in reemplazos.items():
        # Crea un patrón de expresión regular que coincide con cualquiera de las 
        #variantes.
        patron = r'\b(' + '|'.join(re.escape(v) for v in variantes) + r')\b'
        
        # Reemplaza las variantes encontradas en el texto con el término estándar.
        texto = re.sub(patron, reemplazo, texto, flags=re.IGNORECASE)

    return texto
```

```
Aplicamos la función reemplazar_palabras para los dataframes de cada candidato
```

````python
df1['Texto_limpio'] = df1['Texto_limpio'].apply(reemplazar_palabras)
df2['Texto_limpio'] = df2['Texto_limpio'].apply(reemplazar_palabras)
df3['Texto_limpio'] = df3['Texto_limpio'].apply(reemplazar_palabras)
df4['Texto_limpio'] = df4['Texto_limpio'].apply(reemplazar_palabras)
df5['Texto_limpio'] = df5['Texto_limpio'].apply(reemplazar_palabras)
```
````

### <mark style="color:green;">Eliminar finalmente los registros vacíos</mark>

Después de la limpieza de datos realizada algunos registros pueden resultar en registros vacíos, los mismos de eliminan ya que no podrán ser procesados en los modelos posteriores.

```python
dataframes = [df1, df2, df3, df4, df5]

for i, df in enumerate(dataframes, 1):
    df_cleaned = df[df['Texto_limpio'].notna() & (df['Texto_limpio'] != ' ')& 
    (df['Texto_limpio'] != '  ')]
    globals()[f'df{i}'] = df_cleaned
    print(f"Registros en df{i} antes: {len(df)}, después: {len(df_cleaned)}")
```

### <mark style="background-color:green;">Guardar archivos</mark>&#x20;

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

{% file src=".gitbook/assets/Datos_limpios_candidatos.zip" %}
