# Clasificación y búsqueda de mensajes bancarios con NLP

## Datos del taller

| | |
|---|---|
| Duración | 225 minutos |
| Dónde se trabaja | Windows Server |
| Programa | Visual Studio Code |
| Carpeta | `C:\Laboratorio\nlp-banca-lab` |
| Tu archivo de trabajo | `notebooks\01_clasificacion_busqueda_mensajes_bancarios.ipynb` |

El equipo ya tiene los programas, las librerías y los mensajes de práctica. Quien prepara la sala usa [Prerrequisitos.md](Prerrequisitos.md). Tú abres Visual Studio Code y sigues esta guía.

Los mensajes son inventados. Durante el taller usa solo ese archivo. Números de tarjeta, cuentas, documentos y correos reales se quedan fuera del cuaderno, de las capturas y de los archivos que guardes.

## Qué vas a construir

Al final, escribes un mensaje nuevo y el cuaderno hace dos cosas:

1. **Clasifica:** dice a qué tipo de solicitud se parece y con qué seguridad.
2. **Busca:** trae los cinco mensajes guardados que más se le parecen.

Esas dos tareas son distintas. Clasificar responde «¿de qué se trata?». Buscar responde «¿qué casos ya vistos se parecen a este?».

## Palabras que vas a usar

| Palabra | En este taller significa |
|---|---|
| Categoría | El tipo de solicitud. Hay seis. |
| Texto original | El mensaje tal como llegó. Se conserva para poder leerlo. |
| Texto limpio | El mismo mensaje en minúsculas, sin palabras de relleno. Con este trabaja el programa. |
| Aprendizaje | Mensajes con los que el programa estudia. |
| Ensayo | Mensajes con los que se elige el mejor de los dos programas. |
| Examen | Mensajes que se apartan hasta el final, para medir sin hacer trampa. |
| Bolsa de palabras | Cuenta cuántas veces aparece cada palabra. |
| TF-IDF | Les da más valor a las palabras que distinguen un mensaje y menos a las que están en casi todos. |
| Naive Bayes | Programa que mira qué palabras suelen acompañar a cada categoría. |
| Regresión logística | Programa que combina las palabras y les pone un peso. |
| Exactitud | De todos los mensajes, cuántos quedaron en la categoría correcta. |
| Precisión | Cuando el programa dice una categoría, cuántas veces acierta. |
| Cobertura (recall) | De los mensajes que sí eran de una categoría, cuántos encontró. |
| F1 macro | Equilibrio entre precisión y cobertura. Cada categoría pesa igual, aunque tenga menos mensajes. |
| Similitud coseno | Qué tan parecida es la dirección de dos mensajes. 1 es muy parecido. 0 es distinto. |
| Distancia euclidiana | Qué tan lejos quedan dos mensajes. 0 es muy cerca. |
| Vecinos más cercanos | Los mensajes más parecidos al que escribiste. |
| PCA | Un mapa de dos ejes para mirar los grupos. Sirve para ver, y el orden oficial sale de la tabla de similitud. |

## Las seis categorías

| Categoría | Qué está pidiendo la persona |
|---|---|
| `cargo_no_reconocido` | Un cobro o una compra que no reconoce |
| `transferencia_pendiente` | Un envío de dinero que todavía no llega |
| `problema_de_acceso` | No puede entrar a la aplicación o a la página |
| `bloqueo_tarjeta` | Perdió la tarjeta o pide bloquearla |
| `comision` | Una cuota o un cobro de manejo |
| `fraude` | Cree que otra persona usó su producto |

Varias categorías comparten palabras como «tarjeta» o «cobro». Por eso el programa a veces se confunde, y por eso al final se revisan los errores.

## Cómo ejecutar cada bloque

1. Abre **Visual Studio Code**.
2. Menú **Archivo > Abrir carpeta**. Elige `C:\Laboratorio\nlp-banca-lab`.
3. Si pregunta si confías en los autores de la carpeta, elige **Sí, confío en los autores**.
4. Pulsa `Ctrl+Shift+P`, escribe `Create: New Jupyter Notebook` y elige esa opción. Si prefieres el menú: **Archivo > Nuevo archivo** y elige **Jupyter Notebook**.
5. **Archivo > Guardar como**. Entra a la carpeta `notebooks` y guarda el archivo con este nombre:

```text
01_clasificacion_busqueda_mensajes_bancarios.ipynb
```

6. En la esquina superior derecha del cuaderno, elige el kernel **Python 3.11 - NLP Banca Lab**.

Cada vez que esta guía diga que pegues un bloque:

1. Pulsa **+ Código** para crear una celda nueva.
2. Pega el bloque.
3. Pulsa el triángulo de la izquierda de la celda, o `Mayús+Enter`.
4. Espera a que el asterisco `[*]` se convierta en un número.

Ejecuta las celdas de arriba hacia abajo. Cada una usa lo que crearon las anteriores. El código se copia: tu trabajo es ejecutarlo, leer el resultado y reconocer qué decisión tomó el programa.

Para abrir la terminal: menú **Terminal > Nueva terminal**. El tipo de terminal es **PowerShell**. Si dice Símbolo del sistema, pulsa la flecha junto al signo **+** y elige **PowerShell**.

## Ritmo sugerido

| Parte | Pasos | Tiempo |
|---|---|---|
| Conocer y limpiar los mensajes | 1 a 4 | 70 minutos |
| Enseñar y medir la clasificación | 5 a 8 | 90 minutos |
| Buscar parecidos, ver el mapa y guardar | 9 a 11 | 65 minutos |

## Paso 1. Abrir la carpeta y mirar los mensajes

**En este paso:** confirmas que estás en la carpeta del laboratorio y que el archivo de mensajes abre.

### Qué haces

La terminal entra a `C:\Laboratorio\nlp-banca-lab`, muestra las carpetas y enseña las primeras líneas del archivo. La línea del encoding hace que las tildes se vean bien en la terminal. `Get-Content` solo lee: no modifica el archivo.

**Resultado esperado:** ves carpetas como `data`, `notebooks`, `config`, `reports` y `artifacts`. Las primeras líneas del archivo tienen las columnas `message_id`, `message` y `category`, y un mensaje en español.

**Pega esto en la terminal de PowerShell y pulsa Enter:**

```powershell
Set-Location C:\Laboratorio\nlp-banca-lab
[Console]::OutputEncoding = [System.Text.Encoding]::UTF8
Get-ChildItem
Get-Content .\data\raw\mensajes_bancarios.csv -TotalCount 5 -Encoding UTF8
```

Si la terminal muestra símbolos raros en lugar de tildes, abre `data\raw\mensajes_bancarios.csv` desde el explorador de Visual Studio Code. Ahí el texto se lee bien.

**Sigue cuando:** viste las tres columnas y al menos un mensaje.

## Paso 2. Preparar el cuaderno

**En este paso:** el cuaderno localiza la carpeta, lee las reglas del ejercicio y comprueba que Python es el del laboratorio.

### Qué haces

Este bloque carga las herramientas que leen tablas, limpian texto, convierten palabras en números, comparan dos programas, buscan mensajes parecidos y dibujan gráficos. Después fija la carpeta `C:\Laboratorio\nlp-banca-lab`, lee `config\config.json` y comprueba que el archivo de mensajes existe. También indica dónde están las palabras vacías que ya quedaron preparadas.

**Resultado esperado:** la ruta de Python incluye `.venv\Scripts\python.exe`. Aparecen las versiones `1.5.1`, `2.2.2` y `1.26.4`. La última línea dice que el archivo de mensajes está listo. Si la ruta de Python no incluye `.venv`, elige otra vez el kernel **Python 3.11 - NLP Banca Lab** y ejecuta la celda de nuevo.

**Pega esto en una celda nueva y ejecútala:**

```python
import os
os.environ["NLTK_DATA"] = r"C:\Laboratorio\nltk_data"

import json
import re
import sys
import unicodedata
import warnings
from pathlib import Path

import joblib
import matplotlib.pyplot as plt
import nltk
if r"C:\Laboratorio\nltk_data" not in nltk.data.path:
    nltk.data.path.insert(0, r"C:\Laboratorio\nltk_data")

import numpy as np
import pandas as pd
import seaborn as sns
from IPython.display import display
from nltk.corpus import stopwords
from sklearn.decomposition import PCA
from sklearn.feature_extraction.text import CountVectorizer, TfidfVectorizer
from sklearn.linear_model import LogisticRegression
from sklearn.metrics import (
    accuracy_score,
    classification_report,
    confusion_matrix,
    f1_score,
    precision_score,
    recall_score,
)
from sklearn.metrics.pairwise import cosine_similarity, euclidean_distances
from sklearn.model_selection import train_test_split
from sklearn.naive_bayes import MultinomialNB
from sklearn.neighbors import NearestNeighbors

warnings.filterwarnings("ignore")
pd.set_option("display.max_colwidth", 120)

RUTA_RAIZ = Path(r"C:\Laboratorio\nlp-banca-lab")
RUTA_CONFIG = RUTA_RAIZ / "config" / "config.json"
RUTA_ARTEFACTOS = RUTA_RAIZ / "artifacts"
RUTA_REPORTES = RUTA_RAIZ / "reports"
RUTA_ARTEFACTOS.mkdir(parents=True, exist_ok=True)
RUTA_REPORTES.mkdir(parents=True, exist_ok=True)

print("Python en uso:")
print(sys.executable)

import sklearn
print("scikit-learn:", sklearn.__version__)
print("pandas:", pd.__version__)
print("numpy:", np.__version__)

if not RUTA_CONFIG.exists():
    raise FileNotFoundError(
        "Falta config\\config.json. Avisa a quien dirige el laboratorio."
    )

with open(RUTA_CONFIG, encoding="utf-8") as archivo:
    CONFIG = json.load(archivo)

RUTA_ENTRADA = RUTA_RAIZ / CONFIG["input_path"]
RUTA_PROCESADA = RUTA_RAIZ / CONFIG["processed_path"]
RUTA_PROCESADA.parent.mkdir(parents=True, exist_ok=True)

print("Carpeta del laboratorio:", RUTA_RAIZ)
print("Archivo de mensajes:", RUTA_ENTRADA)
print("Semilla para repetir el mismo resultado:", CONFIG["random_state"])
print("Parte que se aparta para el examen:", f"{CONFIG['test_size']:.0%}")
print("Mensajes parecidos que se van a buscar:", CONFIG["top_k"])

if RUTA_ENTRADA.exists():
    print("El archivo de mensajes está listo.")
else:
    raise FileNotFoundError(
        "Falta el archivo de mensajes. Avisa a quien dirige el laboratorio."
    )
```

**Sigue cuando:** leíste «El archivo de mensajes está listo.»

## Paso 3. Conocer los mensajes

**En este paso:** miras cuántos mensajes hay, si alguno está vacío o repetido, cómo se reparten las categorías y qué tan largos son.

### Qué haces

Este bloque abre el archivo y comprueba que existan las columnas `message` y `category`. Si faltara el identificador, lo crea. Después arma una tabla de calidad: cuántas filas hay, cuántos mensajes están vacíos, cuántos están repetidos y cuántas categorías distintas existen.

**Resultado esperado:** la tabla de mensajes muestra texto y categoría. La tabla de calidad, con el archivo de este laboratorio, queda así:

| Indicador | Valor |
|---|---:|
| filas_totales | 184 |
| message_nulos | 1 |
| category_nulos | 0 |
| message_vacios | 1 |
| duplicados_exactos_message_category | 2 |
| categorias_unicas | 6 |

El vacío y el nulo son la misma fila en blanco. Los dos repetidos y esa fila en blanco están puestos a propósito, para ver la limpieza del siguiente paso.

**Pega esto en una celda nueva y ejecútala:**

```python
df = pd.read_csv(RUTA_ENTRADA, encoding="utf-8", sep=",")

columnas_requeridas = {"message", "category"}
faltantes = columnas_requeridas - set(df.columns)
if faltantes:
    raise ValueError(f"Faltan columnas: {faltantes}")

if "message_id" not in df.columns:
    df.insert(0, "message_id", range(1, len(df) + 1))
    print("Se creó message_id porque el archivo no lo traía.")

df["message"] = df["message"].astype("string")
df["category"] = df["category"].astype("string")

print("Cantidad de filas y columnas:", df.shape)
display(df.head())

resumen_calidad = pd.DataFrame({
    "indicador": [
        "filas_totales",
        "message_nulos",
        "category_nulos",
        "message_vacios",
        "duplicados_exactos_message_category",
        "categorias_unicas",
    ],
    "valor": [
        len(df),
        int(df["message"].isna().sum()),
        int(df["category"].isna().sum()),
        int(df["message"].fillna("").str.strip().eq("").sum()),
        int(df.duplicated(subset=["message", "category"]).sum()),
        int(df["category"].nunique()),
    ],
})
display(resumen_calidad)
```

### Qué haces

Este bloque cuenta cuántos mensajes hay en cada categoría, calcula el porcentaje y dibuja una barra por categoría. El gráfico se guarda en `reports`.

**Resultado esperado:** una tabla con las seis categorías y un gráfico de barras debajo de la celda. `cargo_no_reconocido` queda en 32. `bloqueo_tarjeta` y `problema_de_acceso` quedan en 31. Las otras tres quedan en 30. Esa diferencia son las filas de práctica que el paso siguiente retira.

**Pega esto en una celda nueva y ejecútala:**

```python
distribucion_clases = (
    df["category"]
    .value_counts(dropna=False)
    .rename_axis("category")
    .reset_index(name="cantidad")
)
distribucion_clases["porcentaje"] = (
    100 * distribucion_clases["cantidad"] / len(df)
).round(2)

display(distribucion_clases)

plt.figure(figsize=(10, 5))
sns.barplot(
    data=distribucion_clases,
    x="cantidad",
    y="category",
    hue="category",
    legend=False,
    palette="viridis",
)
plt.title("Distribución inicial de categorías")
plt.xlabel("Cantidad de mensajes")
plt.ylabel("Categoría")
plt.tight_layout()
plt.savefig(RUTA_REPORTES / "distribucion_clases_inicial.png", dpi=150)
plt.show()
```

### Qué haces

Este bloque cuenta letras y palabras de cada mensaje, muestra un resumen y dibuja esas longitudes por categoría. También avisa si alguna categoría tuviera menos de 10 mensajes. Con tan pocos ejemplos, el examen de esa categoría no sería confiable.

**Resultado esperado:** una tabla de longitudes, un gráfico de barras apiladas y la frase «Todas las categorías tienen al menos 10 mensajes.»

**Pega esto en una celda nueva y ejecútala:**

```python
df["longitud_caracteres"] = df["message"].fillna("").str.len()
df["longitud_palabras"] = (
    df["message"].fillna("").str.findall(r"\b\w+\b").str.len()
)

display(df[["longitud_caracteres", "longitud_palabras"]].describe())

plt.figure(figsize=(10, 5))
sns.histplot(
    data=df,
    x="longitud_palabras",
    hue="category",
    multiple="stack",
    bins=20,
)
plt.title("Distribución de longitud de mensajes")
plt.xlabel("Número de palabras")
plt.tight_layout()
plt.savefig(RUTA_REPORTES / "longitud_mensajes_inicial.png", dpi=150)
plt.show()

conteo_minimo = df["category"].value_counts()
clases_insuficientes = conteo_minimo[conteo_minimo < 10]
if clases_insuficientes.empty:
    print("Todas las categorías tienen al menos 10 mensajes.")
else:
    print("Hay categorías con menos de 10 mensajes:")
    display(clases_insuficientes)
```

**Sigue cuando:** reconoces las seis categorías y viste el mensaje en blanco y los repetidos en la tabla de calidad.

## Paso 4. Limpiar el texto y conservar el original

**En este paso:** creas una versión limpia de cada mensaje. El texto original se queda guardado para que una persona pueda leerlo.

Limpiar, en este taller, hace esto:

- Pasa todo a minúsculas.
- Tapa correos, páginas web y números largos, para reducir datos sensibles.
- Quita signos de puntuación.
- Conserva las letras con tilde y la eñe.
- Quita palabras de relleno como «el», «la» y «de».
- Conserva la palabra «no», porque cambia el sentido: «no reconozco» es distinto de «reconozco».

### Qué haces

Este bloque señala la carpeta donde el laboratorio ya dejó las palabras vacías, carga esa lista y define la limpieza. Luego la prueba con dos frases: una solicitud normal y otra con un correo y un número largo inventados. Así ves el efecto antes de aplicarlo a todo el archivo.

**Resultado esperado:**

```text
no reconozco pago realizado tarjeta
escriban correo revisen tarjeta_numero
```

La segunda frase ya no muestra el correo ni el número. Esos datos quedaron reemplazados por `correo` y `tarjeta_numero`.

**Pega esto en una celda nueva y ejecútala:**

```python
os.environ["NLTK_DATA"] = r"C:\Laboratorio\nltk_data"
if r"C:\Laboratorio\nltk_data" not in nltk.data.path:
    nltk.data.path.insert(0, r"C:\Laboratorio\nltk_data")

try:
    STOPWORDS_ES = set(stopwords.words("spanish"))
except LookupError:
    raise LookupError(
        "Faltan las palabras vacías en español. Avisa a quien dirige el laboratorio."
    )

STOPWORDS_ES.discard("no")

def normalizar_mensaje(texto):
    """Deja el mensaje en minúsculas, sin signos ni palabras de relleno."""
    if pd.isna(texto):
        return ""

    texto = str(texto).lower().strip()
    texto = unicodedata.normalize("NFC", texto)

    # Tapa correos, páginas web y números largos.
    texto = re.sub(
        r"\b[A-Z0-9._%+-]+@[A-Z0-9.-]+\.[A-Z]{2,}\b",
        " correo ",
        texto,
        flags=re.IGNORECASE,
    )
    texto = re.sub(r"https?://\S+|www\.\S+", " url ", texto)
    texto = re.sub(r"\b(?:\d[ -]?){13,19}\b", " tarjeta_numero ", texto)
    texto = re.sub(r"\b\d{8,}\b", " referencia_numero ", texto)

    # Quita signos y conserva letras, números, tildes, eñe y las etiquetas.
    texto = re.sub(r"[^a-záéíóúüñ0-9_\s]", " ", texto)
    texto = re.sub(r"\s+", " ", texto).strip()

    tokens = [
        token for token in texto.split()
        if token not in STOPWORDS_ES and len(token) > 1
    ]
    return " ".join(tokens)

print(normalizar_mensaje("No reconozco un pago realizado con mi tarjeta."))
print(normalizar_mensaje(
    "Escriban a cliente@correo.com o revisen 1234567890123456"
))
```

### Qué haces

Este bloque copia los datos, guarda el texto original, bota las filas vacías y crea la columna limpia. Una frase hecha solo de palabras como «el» y «la» queda vacía después de limpiar, y también se retira. Después quita los mensajes repetidos en texto limpio y categoría. Un repetido podría caer en el aprendizaje y en el examen a la vez, y el examen saldría más fácil de lo real.

**Resultado esperado:** verás «Filas antes de deduplicar: 182», «Filas después de deduplicar: 180» y «Duplicados eliminados: 2». La tabla muestra el original, el limpio y la categoría. Cada categoría queda con 30 mensajes.

**Pega esto en una celda nueva y ejecútala:**

```python
df_limpio = df.copy()
df_limpio["message_original"] = df_limpio["message"].fillna("").str.strip()
df_limpio["category"] = df_limpio["category"].fillna("").str.strip()

df_limpio = df_limpio[
    (df_limpio["message_original"] != "") &
    (df_limpio["category"] != "")
].copy()

df_limpio["message_normalized"] = df_limpio["message_original"].apply(
    normalizar_mensaje
)
df_limpio = df_limpio[
    df_limpio["message_normalized"].str.strip() != ""
].copy()

filas_antes_duplicados = len(df_limpio)
df_limpio = df_limpio.drop_duplicates(
    subset=["message_normalized", "category"]
).reset_index(drop=True)

print(f"Filas antes de deduplicar: {filas_antes_duplicados}")
print(f"Filas después de deduplicar: {len(df_limpio)}")
print(f"Duplicados eliminados: {filas_antes_duplicados - len(df_limpio)}")

display(
    df_limpio[
        ["message_id", "message_original", "message_normalized", "category"]
    ].head(10)
)
print(df_limpio["category"].value_counts().sort_index())
```

### Qué haces

Este bloque guarda el archivo limpio en `data\processed` y comprueba que cada mensaje limpio tenga texto y que ya no haya repetidos.

**Resultado esperado:** la ruta del archivo procesado y la frase «Limpieza lista.»

**Pega esto en una celda nueva y ejecútala:**

```python
columnas_salida = [
    "message_id",
    "message_original",
    "message_normalized",
    "category",
    "longitud_caracteres",
    "longitud_palabras",
]
columnas_existentes = [
    columna for columna in columnas_salida if columna in df_limpio.columns
]
df_limpio[columnas_existentes].to_csv(
    RUTA_PROCESADA,
    index=False,
    encoding="utf-8",
)

limpieza_ok = (
    RUTA_PROCESADA.exists()
    and df_limpio["message_normalized"].str.len().gt(0).all()
    and not df_limpio.duplicated(
        subset=["message_normalized", "category"]
    ).any()
)
print("Archivo limpio:", RUTA_PROCESADA)
print("Limpieza lista." if limpieza_ok else "Revisa la celda anterior.")
```

**Sigue cuando:** el archivo limpio existe y cada categoría tiene 30 mensajes.

## Paso 5. Apartar aprendizaje, ensayo y examen

**En este paso:** separas los mensajes en tres grupos antes de enseñarle algo al programa.

| Grupo | Para qué sirve | Tamaño con este archivo |
|---|---|---|
| Aprendizaje | El programa estudia aquí | 108 mensajes, 60 % |
| Ensayo | Sirve para elegir entre los dos programas | 36 mensajes, 20 % |
| Examen | Se mide al final y no se usa para estudiar ni para elegir | 36 mensajes, 20 % |

La separación es **estratificada**: cada categoría queda presente en los tres grupos, en una proporción parecida. La semilla `42` hace que, si repites el taller, los grupos salgan iguales.

El diccionario de palabras del clasificador se arma solo con el aprendizaje. Si se armara con el examen, el programa habría visto esas palabras antes de presentar la prueba.

### Qué haces

Este bloque separa primero el 20 % del examen. Del 80 % restante aparta un cuarto para el ensayo. Ese cuarto del 80 % es el 20 % del total. Después muestra el tamaño de cada grupo y el porcentaje de cada categoría. También guarda los identificadores del examen en `reports`, para saber después cuáles mensajes se usaron en la medición.

**Resultado esperado:**

```text
Aprendizaje: 108
Ensayo:      36
Examen:      36
```

La tabla de porcentajes muestra las seis categorías en los tres grupos. La última frase confirma que los grupos quedaron separados y con las mismas categorías.

**Pega esto en una celda nueva y ejecútala:**

```python
X = df_limpio["message_normalized"]
y = df_limpio["category"]
SEMILLA = CONFIG["random_state"]

X_temp, X_test, y_temp, y_test = train_test_split(
    X,
    y,
    test_size=CONFIG["test_size"],
    stratify=y,
    random_state=SEMILLA,
)
X_train, X_val, y_train, y_val = train_test_split(
    X_temp,
    y_temp,
    test_size=CONFIG["validation_size_relative"],
    stratify=y_temp,
    random_state=SEMILLA,
)

print("Tamaño de cada grupo:")
print(f"Aprendizaje: {len(X_train)}")
print(f"Ensayo:      {len(X_val)}")
print(f"Examen:      {len(X_test)}")

def resumen_particion(nombre, etiquetas):
    return (
        etiquetas.value_counts(normalize=True)
        .rename(nombre)
        .mul(100)
        .round(2)
    )

comparacion_particiones = pd.concat(
    [
        resumen_particion("aprendizaje_%", y_train),
        resumen_particion("ensayo_%", y_val),
        resumen_particion("examen_%", y_test),
    ],
    axis=1,
).fillna(0)
display(comparacion_particiones)

ids_test = df_limpio.loc[
    X_test.index, ["message_id", "message_original", "category"]
]
ids_test.to_csv(
    RUTA_REPORTES / "muestra_examen.csv",
    index=False,
    encoding="utf-8",
)

mismas_categorias = (
    set(y_train.unique()) == set(y_val.unique()) == set(y_test.unique())
)
grupos_separados = (
    set(X_train.index).isdisjoint(set(X_test.index))
    and set(X_val.index).isdisjoint(set(X_test.index))
)
if mismas_categorias and grupos_separados:
    print("Los tres grupos quedaron separados y con las mismas categorías.")
else:
    print("La separación no quedó bien. Avisa a quien dirige el laboratorio.")
```

**Sigue cuando:** viste 108, 36 y 36, y la frase de grupos separados.

## Paso 6. Convertir el texto en números

**En este paso:** comparas dos formas de representar el texto. Los programas de este taller trabajan con números.

**Bolsa de palabras.** El mensaje «no reconozco el pago» se vuelve un conteo: no = 1, reconozco = 1, pago = 1. También se cuentan pares de palabras seguidas, como «no reconozco» o «bloquear tarjeta».

**TF-IDF.** Una palabra que aparece en casi todos los mensajes aporta poco. Una palabra que aparece en pocos mensajes y ayuda a distinguir la categoría aporta más. Los números quedan en una escala que después permite comparar qué tan parecidos son dos mensajes.

Las dos representaciones se ajustan solo con el aprendizaje. El ensayo y el examen se transforman con ese mismo diccionario, sin agregar palabras nuevas.

### Qué haces

Este bloque arma la bolsa de palabras con el aprendizaje y convierte los tres grupos. Muestra cuántos mensajes y cuántas palabras quedaron en la tabla de aprendizaje.

**Resultado esperado:** una línea con la forma `(108, cantidad de palabras)` y el tamaño del vocabulario. La cantidad de palabras depende de los textos; con este archivo es mucho menor que 5000.

**Pega esto en una celda nueva y ejecútala:**

```python
count_vectorizer = CountVectorizer(
    ngram_range=tuple(CONFIG["ngram_range"]),
    min_df=CONFIG["min_df"],
    max_df=CONFIG["max_df"],
    max_features=CONFIG["max_features"],
)
X_train_count = count_vectorizer.fit_transform(X_train)
X_val_count = count_vectorizer.transform(X_val)
X_test_count = count_vectorizer.transform(X_test)

print("Bolsa de palabras, aprendizaje:", X_train_count.shape)
print("Palabras y pares en la bolsa:", len(count_vectorizer.vocabulary_))
```

### Qué haces

Este bloque arma TF-IDF de la misma manera y muestra algunas palabras del diccionario. Después calcula qué tan llena está la tabla. Casi todas las casillas valen cero, porque cada mensaje usa pocas palabras del diccionario. Eso es normal y el programa lo guarda de forma ligera.

**Resultado esperado:** la tabla TF-IDF tiene 108 filas y la misma cantidad de columnas para aprendizaje, ensayo y examen. La densidad es un número pequeño, cercano a cero. La última frase dice que la conversión quedó lista.

**Pega esto en una celda nueva y ejecútala:**

```python
tfidf_vectorizer = TfidfVectorizer(
    ngram_range=tuple(CONFIG["ngram_range"]),
    min_df=CONFIG["min_df"],
    max_df=CONFIG["max_df"],
    max_features=CONFIG["max_features"],
    sublinear_tf=True,
    norm="l2",
)
X_train_tfidf = tfidf_vectorizer.fit_transform(X_train)
X_val_tfidf = tfidf_vectorizer.transform(X_val)
X_test_tfidf = tfidf_vectorizer.transform(X_test)

print("TF-IDF, aprendizaje:", X_train_tfidf.shape)
print("Palabras y pares en TF-IDF:", len(tfidf_vectorizer.vocabulary_))
print("Algunas palabras del diccionario:")
print(tfidf_vectorizer.get_feature_names_out()[:30])

densidad = X_train_tfidf.nnz / (
    X_train_tfidf.shape[0] * X_train_tfidf.shape[1]
)
print(f"Densidad de TF-IDF: {densidad:.6f}")

formas_ok = (
    X_train_tfidf.shape[0] == len(X_train)
    and X_val_tfidf.shape[0] == len(X_val)
    and X_test_tfidf.shape[0] == len(X_test)
    and X_train_tfidf.shape[1] == X_val_tfidf.shape[1] == X_test_tfidf.shape[1]
)
print(
    "Conversión lista. El diccionario salió del aprendizaje."
    if formas_ok
    else "Las tablas no coinciden. Ejecuta otra vez desde el paso 5."
)
```

**Sigue cuando:** viste las dos tablas y una densidad pequeña.

## Paso 7. Enseñar dos programas y elegir uno

**En este paso:** entrenas Naive Bayes y regresión logística con TF-IDF. Los dos ven los mismos mensajes de aprendizaje. El ensayo decide cuál se queda.

Naive Bayes se apoya en las palabras típicas de cada categoría. La regresión logística combina esas señales y equilibra un poco las categorías con menos ejemplos. Se elige el que tenga mayor **F1 macro** en el ensayo. Si empatan, se queda el de mayor cobertura.

Un número más cercano a 1 es mejor. Un número más cercano a 0 es peor.

### Qué haces

Este bloque enseña los dos programas, calcula exactitud, precisión, cobertura y F1 macro en el ensayo, ordena la tabla y guarda la comparación. La primera fila es la elegida.

**Resultado esperado:** una tabla con dos filas. Arriba queda el programa con mejor F1 macro. Debajo lees cuál fue el elegido. Los números cambian si cambia el archivo; con el archivo de este laboratorio los dos programas responden y sus notas quedan entre 0 y 1.

**Pega esto en una celda nueva y ejecútala:**

```python
modelo_nb = MultinomialNB(alpha=1.0)
modelo_nb.fit(X_train_tfidf, y_train)
pred_val_nb = modelo_nb.predict(X_val_tfidf)

modelo_lr = LogisticRegression(
    max_iter=2000,
    class_weight="balanced",
    random_state=SEMILLA,
)
modelo_lr.fit(X_train_tfidf, y_train)
pred_val_lr = modelo_lr.predict(X_val_tfidf)

def calcular_metricas(y_real, y_pred):
    return {
        "exactitud": round(float(accuracy_score(y_real, y_pred)), 4),
        "precision": round(
            float(precision_score(y_real, y_pred, average="macro", zero_division=0)),
            4,
        ),
        "cobertura": round(
            float(recall_score(y_real, y_pred, average="macro", zero_division=0)),
            4,
        ),
        "f1_macro": round(
            float(f1_score(y_real, y_pred, average="macro", zero_division=0)),
            4,
        ),
    }

resultados_validacion = pd.DataFrame([
    {"modelo": "MultinomialNB", **calcular_metricas(y_val, pred_val_nb)},
    {"modelo": "LogisticRegression", **calcular_metricas(y_val, pred_val_lr)},
]).sort_values(
    by=["f1_macro", "cobertura"],
    ascending=[False, False],
)
display(resultados_validacion)
resultados_validacion.to_csv(
    RUTA_REPORTES / "comparacion_modelos_ensayo.csv",
    index=False,
    encoding="utf-8",
)

mejor_nombre = resultados_validacion.iloc[0]["modelo"]
mejor_modelo = modelo_nb if mejor_nombre == "MultinomialNB" else modelo_lr
print("Modelo elegido por F1 macro del ensayo:", mejor_nombre)
```

**Sigue cuando:** anotaste el nombre del modelo elegido y su F1 macro.

## Paso 8. Medir el examen y mirar los errores

**En este paso:** el programa elegido clasifica el examen. Esos mensajes no se usaron para estudiar ni para elegir. Después miras en qué se equivocó.

### Qué haces

Este bloque predice la categoría del examen, calcula la seguridad de cada predicción y muestra las cuatro medidas. También escribe el detalle por categoría: en cada fila ves cómo le fue a esa solicitud.

**Resultado esperado:** las cuatro medidas del examen, entre 0 y 1, y una tabla de texto con una fila por categoría. Anota el F1 macro del examen. Puede ser distinto del F1 del ensayo.

**Pega esto en una celda nueva y ejecútala:**

```python
pred_test = mejor_modelo.predict(X_test_tfidf)
confianza_test = mejor_modelo.predict_proba(X_test_tfidf).max(axis=1)
metricas_test = calcular_metricas(y_test, pred_test)
metricas_test["modelo_seleccionado"] = mejor_nombre

print("Resultado del examen:")
for nombre, valor in metricas_test.items():
    print(f"  {nombre}: {valor}")

etiquetas_ordenadas = sorted(y.unique())
reporte_test = classification_report(
    y_test,
    pred_test,
    labels=etiquetas_ordenadas,
    zero_division=0,
)
print(reporte_test)
with open(
    RUTA_REPORTES / "reporte_clasificacion_examen.txt",
    "w",
    encoding="utf-8",
) as archivo:
    archivo.write(reporte_test)
```

### Qué haces

Este bloque dibuja la matriz de confusión y la guarda en `reports`. Cada fila es la categoría real. Cada columna es la categoría que dijo el programa. La diagonal, de arriba izquierda a abajo derecha, son los aciertos. Los números fuera de la diagonal son confusiones.

**Resultado esperado:** un gráfico con las seis categorías en los dos ejes y la frase «Matriz lista.» El archivo queda en `reports\matriz_confusion_examen.png`.

**Pega esto en una celda nueva y ejecútala:**

```python
matriz = confusion_matrix(y_test, pred_test, labels=etiquetas_ordenadas)

plt.figure(figsize=(10, 8))
sns.heatmap(
    matriz,
    annot=True,
    fmt="d",
    cmap="Blues",
    xticklabels=etiquetas_ordenadas,
    yticklabels=etiquetas_ordenadas,
)
plt.title(f"Matriz de confusión - {mejor_nombre}")
plt.xlabel("Categoría que dijo el programa")
plt.ylabel("Categoría real")
plt.xticks(rotation=45, ha="right")
plt.yticks(rotation=0)
plt.tight_layout()
plt.savefig(RUTA_REPORTES / "matriz_confusion_examen.png", dpi=150)
plt.show()
print("Matriz lista.")
```

### Qué haces

Este bloque junta el mensaje original, la categoría real, la categoría predicha y la seguridad. Se queda con los errores y los ordena de mayor a menor seguridad. Un error con seguridad alta merece una mirada especial: el programa estaba seguro y aun así se equivocó.

**Resultado esperado:** la frase `Errores en el examen: 4` y una tabla con esos cuatro mensajes. Son pocos porque varias categorías usan palabras distintas. Alcanzan para ver confusiones reales, por ejemplo entre fraude y bloqueo de tarjeta, o entre un cargo no reconocido y fraude.

**Pega esto en una celda nueva y ejecútala:**

```python
resultados_test = df_limpio.loc[
    X_test.index,
    ["message_id", "message_original", "message_normalized", "category"],
].copy()
resultados_test = resultados_test.rename(columns={"category": "categoria_real"})
resultados_test["categoria_predicha"] = pred_test
resultados_test["seguridad"] = confianza_test.round(2)
resultados_test["es_error"] = (
    resultados_test["categoria_real"] != resultados_test["categoria_predicha"]
)

errores_test = resultados_test[resultados_test["es_error"]].sort_values(
    "seguridad",
    ascending=False,
)
errores_test.to_csv(
    RUTA_REPORTES / "errores_clasificacion_examen.csv",
    index=False,
    encoding="utf-8",
)

print(f"Errores en el examen: {len(errores_test)}")
display(errores_test.head(10))
```

### Escribe lo que viste

En el explorador de Visual Studio Code, clic derecho en la carpeta `reports`, elige **Nuevo archivo** y nómbralo `analisis_errores.md`. Pega esta plantilla, complétala y guárdala.

Revisa los cuatro errores. Si en tu pantalla aparece otro número, comenta todos los que salgan. Completa hasta diez líneas: las que falten se arman con mensajes del archivo limpio que usen palabras parecidas y pertenezcan a categorías distintas. «Tarjeta», por ejemplo, aparece en fraude, en bloqueo y en un cargo no reconocido.

Para cada caso elige una causa:

- Las dos categorías usan palabras parecidas.
- El mensaje es muy corto.
- La categoría original parece dudosa.
- Hay pocos ejemplos de esa categoría.
- Falta un dato que el texto no trae, como el valor, el comercio o el canal.

```markdown
# Análisis de errores

## Modelo elegido
Nombre:
F1 macro del examen:

## Casos
1. Id del mensaje:  Categoría real:  Categoría predicha:  Causa:
2. Id del mensaje:  Categoría real:  Categoría predicha:  Causa:
3. Id del mensaje:  Categoría real:  Categoría predicha:  Causa:
4. Id del mensaje:  Categoría real:  Categoría predicha:  Causa:
5. Id del mensaje:  Categoría real:  Categoría predicha:  Causa:
6. Id del mensaje:  Categoría real:  Categoría predicha:  Causa:
7. Id del mensaje:  Categoría real:  Categoría predicha:  Causa:
8. Id del mensaje:  Categoría real:  Categoría predicha:  Causa:
9. Id del mensaje:  Categoría real:  Categoría predicha:  Causa:
10. Id del mensaje:  Categoría real:  Categoría predicha:  Causa:

## Mejora que yo propondría
Escribe una acción concreta: reunir más ejemplos, revisar etiquetas dudosas o separar mejor dos categorías que comparten palabras.
```

**Sigue cuando:** la matriz está guardada y `analisis_errores.md` tiene diez líneas completas.

## Paso 9. Buscar los cinco mensajes más parecidos

**En este paso:** armas un buscador. El clasificador se midió con un diccionario hecho solo del aprendizaje. El buscador es otro trabajo: su función es mirar toda la colección, así que su diccionario TF-IDF se arma con los 180 mensajes limpios. Esa decisión no cambia la nota del examen.

Vas a ver tres medidas del mismo parecido:

| Medida | Cómo se lee |
|---|---|
| Similitud coseno | Más cerca de 1, más parecido |
| Distancia coseno | Más cerca de 0, más parecido |
| Distancia euclidiana | Más cerca de 0, más cerca |

### Qué haces

Este bloque convierte toda la colección limpia a TF-IDF, guarda una tabla de consulta con el texto original y prepara el índice de vecinos. El índice busca los cinco más cercanos usando la distancia coseno.

**Resultado esperado:** una línea `Colección para buscar: (180, cantidad de palabras)`.

**Pega esto en una celda nueva y ejecútala:**

```python
tfidf_vectorizer_search = TfidfVectorizer(
    ngram_range=tuple(CONFIG["ngram_range"]),
    min_df=CONFIG["min_df"],
    max_df=CONFIG["max_df"],
    max_features=CONFIG["max_features"],
    sublinear_tf=True,
    norm="l2",
)
X_corpus_tfidf = tfidf_vectorizer_search.fit_transform(
    df_limpio["message_normalized"]
)
referencia_busqueda = df_limpio[
    ["message_id", "message_original", "message_normalized", "category"]
].reset_index(drop=True)

indice_vecinos = NearestNeighbors(
    n_neighbors=CONFIG["top_k"],
    metric="cosine",
    algorithm="brute",
)
indice_vecinos.fit(X_corpus_tfidf)
print("Colección para buscar:", X_corpus_tfidf.shape)
```

### Qué haces

Este bloque define la búsqueda y la prueba con una frase de una compra no autorizada. La función limpia la frase, la convierte con el diccionario del buscador y devuelve cinco mensajes, ordenados del más parecido al menos parecido. Las medidas se redondean a dos decimales para leerlas.

**Resultado esperado:** cinco filas. En una consulta de compra no autorizada, varias filas suelen ser `cargo_no_reconocido` o `fraude`. Puede aparecer otra categoría: la similitud mira las palabras, y algunas se comparten.

**Pega esto en una celda nueva y ejecútala:**

```python
def buscar_mensajes_similares(
    consulta,
    vectorizador,
    matriz_referencia,
    referencia,
    vecinos,
    top_k=5,
):
    consulta_normalizada = normalizar_mensaje(consulta)
    if not consulta_normalizada:
        raise ValueError(
            "La consulta quedó vacía después de quitar las palabras de relleno."
        )

    vector_consulta = vectorizador.transform([consulta_normalizada])
    similitudes = cosine_similarity(vector_consulta, matriz_referencia).ravel()
    distancias_euclidianas = euclidean_distances(
        vector_consulta, matriz_referencia
    ).ravel()
    distancias_coseno, indices = vecinos.kneighbors(
        vector_consulta,
        n_neighbors=top_k,
    )

    resultado = referencia.iloc[indices[0]].copy()
    resultado["similitud_coseno"] = similitudes[indices[0]]
    resultado["distancia_euclidiana"] = distancias_euclidianas[indices[0]]
    resultado["distancia_coseno"] = distancias_coseno[0]
    resultado = resultado.sort_values(
        by=["similitud_coseno", "distancia_euclidiana"],
        ascending=[False, True],
    )
    return resultado[
        [
            "message_id",
            "message_original",
            "category",
            "similitud_coseno",
            "distancia_euclidiana",
            "distancia_coseno",
        ]
    ]

def mostrar_busqueda(tabla):
    vista = tabla.copy()
    for columna in ("similitud_coseno", "distancia_euclidiana", "distancia_coseno"):
        vista[columna] = vista[columna].round(2)
    display(vista)

consulta_ejemplo = "Veo una compra con mi tarjeta que no autoricé"
similares = buscar_mensajes_similares(
    consulta_ejemplo,
    tfidf_vectorizer_search,
    X_corpus_tfidf,
    referencia_busqueda,
    indice_vecinos,
    top_k=5,
)
print("Consulta:", consulta_ejemplo)
mostrar_busqueda(similares)

busqueda_ok = (
    len(similares) == 5
    and similares["similitud_coseno"].between(0, 1).all()
    and (similares["distancia_euclidiana"] >= 0).all()
    and (similares["distancia_coseno"] >= 0).all()
)
print("Búsqueda lista." if busqueda_ok else "La búsqueda no devolvió cinco mensajes.")
```

### Qué haces

Este bloque repite la búsqueda con cuatro solicitudes distintas: una transferencia, un bloqueo, un problema para entrar y una comisión. Así comparas si los vecinos cambian cuando cambia el tema.

**Resultado esperado:** cuatro títulos `CONSULTA` y, debajo de cada uno, cinco mensajes. La categoría más repetida en cada tabla suele corresponder al tema de la frase.

**Pega esto en una celda nueva y ejecútala:**

```python
consultas_prueba = [
    "La transferencia que envié sigue pendiente",
    "Quiero bloquear mi tarjeta porque la perdí",
    "No puedo entrar a la aplicación del banco",
    "Por qué me cobraron una comisión de mantenimiento",
]
for consulta in consultas_prueba:
    print(f"\nCONSULTA: {consulta}")
    mostrar_busqueda(
        buscar_mensajes_similares(
            consulta,
            tfidf_vectorizer_search,
            X_corpus_tfidf,
            referencia_busqueda,
            indice_vecinos,
            top_k=5,
        )
    )
```

**Sigue cuando:** cada consulta mostró cinco mensajes con similitud entre 0 y 1.

## Paso 10. Ver los mensajes en un mapa

**En este paso:** PCA reduce muchas palabras a dos ejes para dibujar un mapa. Los colores son las categorías. El mapa es una ayuda visual: al aplastar tantas palabras en dos ejes se pierde detalle. El orden oficial de la búsqueda sigue siendo la tabla del paso 9.

Para dibujar, el programa usa una parte de las palabras y, si hiciera falta, una muestra de mensajes. Así el equipo puede armar el gráfico.

### Qué haces

Este bloque prepara la tabla del mapa, calcula los dos ejes y dibuja un punto por mensaje. También imprime qué parte de la información original conservan esos dos ejes.

**Resultado esperado:** un porcentaje de varianza explicada y un gráfico de puntos de colores. El porcentaje puede ser bajo. Eso significa que dos ejes no alcanzan para contar todas las palabras. El gráfico queda en `reports\pca_corpus_categorias.png`.

**Pega esto en una celda nueva y ejecútala:**

```python
max_caracteristicas_pca = min(
    CONFIG["pca_max_features"],
    X_corpus_tfidf.shape[1],
)
X_pca_base = X_corpus_tfidf[:, :max_caracteristicas_pca]

if X_pca_base.shape[0] > CONFIG["pca_max_samples"]:
    rng = np.random.default_rng(SEMILLA)
    indices_pca = np.sort(
        rng.choice(
            X_pca_base.shape[0],
            size=CONFIG["pca_max_samples"],
            replace=False,
        )
    )
else:
    indices_pca = np.arange(X_pca_base.shape[0])

X_pca_denso = X_pca_base[indices_pca].toarray()
referencia_pca = referencia_busqueda.iloc[indices_pca].reset_index(drop=True)
print("Tabla usada para el mapa:", X_pca_denso.shape)

pca_model = PCA(n_components=2, random_state=SEMILLA)
coordenadas_pca = pca_model.fit_transform(X_pca_denso)
df_pca = referencia_pca.copy()
df_pca["pca_1"] = coordenadas_pca[:, 0]
df_pca["pca_2"] = coordenadas_pca[:, 1]
varianza_explicada = float(pca_model.explained_variance_ratio_.sum())
print(f"Información que conservan los dos ejes: {varianza_explicada:.0%}")

plt.figure(figsize=(12, 8))
sns.scatterplot(
    data=df_pca,
    x="pca_1",
    y="pca_2",
    hue="category",
    alpha=0.75,
    s=55,
)
plt.title("Mensajes en dos ejes")
plt.xlabel("Eje 1")
plt.ylabel("Eje 2")
plt.legend(title="Categoría", bbox_to_anchor=(1.02, 1), loc="upper left")
plt.tight_layout()
plt.savefig(RUTA_REPORTES / "pca_corpus_categorias.png", dpi=150)
plt.show()
```

### Qué haces

Este bloque ubica una consulta nueva en el mismo mapa, marca con una X el lugar de la consulta y rodea los cinco mensajes que el buscador recuperó. Si un vecino queda lejos de la X, recuerda que el mapa está simplificado: manda la tabla de similitud.

**Resultado esperado:** el mismo mapa, con una X y cinco círculos negros. El archivo queda en `reports\pca_consulta_vecinos.png`. La última frase dice «Mapa listo.»

**Pega esto en una celda nueva y ejecútala:**

```python
consulta_visual = "Tengo un cobro con tarjeta que no reconozco"
consulta_visual_normalizada = normalizar_mensaje(consulta_visual)
vector_consulta_pca = tfidf_vectorizer_search.transform(
    [consulta_visual_normalizada]
)[:, :max_caracteristicas_pca].toarray()
coordenada_consulta = pca_model.transform(vector_consulta_pca)

similares_visual = buscar_mensajes_similares(
    consulta_visual,
    tfidf_vectorizer_search,
    X_corpus_tfidf,
    referencia_busqueda,
    indice_vecinos,
    top_k=5,
)
ids_similares = set(similares_visual["message_id"])
puntos_similares = df_pca[df_pca["message_id"].isin(ids_similares)]

plt.figure(figsize=(12, 8))
sns.scatterplot(
    data=df_pca,
    x="pca_1",
    y="pca_2",
    hue="category",
    alpha=0.35,
    s=40,
    legend=False,
)
plt.scatter(
    puntos_similares["pca_1"],
    puntos_similares["pca_2"],
    s=130,
    facecolors="none",
    edgecolors="black",
    linewidths=1.5,
    label="Parecidos recuperados",
)
plt.scatter(
    coordenada_consulta[:, 0],
    coordenada_consulta[:, 1],
    marker="X",
    s=250,
    c="red",
    edgecolors="black",
    label="Consulta",
)
plt.title("Consulta y mensajes parecidos en el mapa")
plt.xlabel("Eje 1")
plt.ylabel("Eje 2")
plt.legend()
plt.tight_layout()
plt.savefig(RUTA_REPORTES / "pca_consulta_vecinos.png", dpi=150)
plt.show()

mapa_ok = (
    coordenadas_pca.shape[1] == 2
    and coordenada_consulta.shape == (1, 2)
    and (RUTA_REPORTES / "pca_corpus_categorias.png").exists()
)
print("Mapa listo." if mapa_ok else "El mapa no se guardó. Ejecuta otra vez este paso.")
```

**Sigue cuando:** viste los dos gráficos y la frase «Mapa listo.»

## Paso 11. Guardar el sistema y probar un mensaje nuevo

**En este paso:** el programa elegido se vuelve a enseñar con el aprendizaje y el ensayo, para aprovechar más ejemplos. El examen sigue fuera de esa enseñanza. Después se guardan las piezas para usarlas sin volver a enseñar.

| Archivo | Qué guarda |
|---|---|
| `tfidf_vectorizer.joblib` | El diccionario de palabras del clasificador |
| `classifier.joblib` | El programa que asigna la categoría |
| `search_reference.joblib` | El buscador y los mensajes de referencia |
| `pca_model.joblib` | El mapa |
| `metrics_test.json` | Las notas del examen |
| `config_used.json` | Las reglas con las que se trabajó |

### Qué haces

Este bloque arma de nuevo el TF-IDF con aprendizaje y ensayo, enseña otra vez el programa elegido y guarda las seis piezas.

**Resultado esperado:** el nombre del modelo y la frase «Piezas guardadas.»

**Pega esto en una celda nueva y ejecútala:**

```python
X_train_final = pd.concat([X_train, X_val])
y_train_final = pd.concat([y_train, y_val])

tfidf_vectorizer_final = TfidfVectorizer(
    ngram_range=tuple(CONFIG["ngram_range"]),
    min_df=CONFIG["min_df"],
    max_df=CONFIG["max_df"],
    max_features=CONFIG["max_features"],
    sublinear_tf=True,
    norm="l2",
)
X_train_final_tfidf = tfidf_vectorizer_final.fit_transform(X_train_final)

if mejor_nombre == "MultinomialNB":
    classifier_final = MultinomialNB(alpha=1.0)
else:
    classifier_final = LogisticRegression(
        max_iter=2000,
        class_weight="balanced",
        random_state=SEMILLA,
    )
classifier_final.fit(X_train_final_tfidf, y_train_final)
print("Modelo guardado:", mejor_nombre)

joblib.dump(tfidf_vectorizer_final, RUTA_ARTEFACTOS / "tfidf_vectorizer.joblib")
joblib.dump(classifier_final, RUTA_ARTEFACTOS / "classifier.joblib")
joblib.dump(
    {
        "vectorizer": tfidf_vectorizer_search,
        "tfidf_matrix": X_corpus_tfidf,
        "reference": referencia_busqueda,
        "nearest_neighbors": indice_vecinos,
        "top_k_default": CONFIG["top_k"],
    },
    RUTA_ARTEFACTOS / "search_reference.joblib",
)
joblib.dump(
    {
        "pca_model": pca_model,
        "max_features_used": max_caracteristicas_pca,
        "sample_indices": indices_pca,
        "explained_variance_ratio_sum": varianza_explicada,
    },
    RUTA_ARTEFACTOS / "pca_model.joblib",
)

metricas_completas = {
    "lab_id": "01-00-01",
    "modelo_seleccionado": mejor_nombre,
    "metricas_test": metricas_test,
    "tamano_aprendizaje": int(len(X_train)),
    "tamano_ensayo": int(len(X_val)),
    "tamano_examen": int(len(X_test)),
    "categorias": sorted(y.unique().tolist()),
    "random_state": SEMILLA,
}
with open(RUTA_ARTEFACTOS / "metrics_test.json", "w", encoding="utf-8") as archivo:
    json.dump(metricas_completas, archivo, ensure_ascii=False, indent=2)
with open(RUTA_ARTEFACTOS / "config_used.json", "w", encoding="utf-8") as archivo:
    json.dump(CONFIG, archivo, ensure_ascii=False, indent=2)

print("Piezas guardadas.")
```

### Qué haces

Este bloque define la consulta final: limpia el mensaje, predice la categoría, calcula la seguridad y trae cinco parecidos. Luego lo prueba con una compra que la persona dice no haber hecho. La seguridad es una nota del programa. En casos de fraude, bloqueo, dinero o acceso, la decisión del banco sigue las reglas del negocio y la revisión de una persona cuando corresponda.

**Resultado esperado:** la categoría `cargo_no_reconocido`, un porcentaje de seguridad y cinco mensajes de compras parecidas.

**Pega esto en una celda nueva y ejecútala:**

```python
def predecir_y_buscar(
    mensaje_nuevo,
    vectorizador_clasificacion,
    clasificador,
    recurso_busqueda,
    top_k=5,
):
    mensaje_normalizado = normalizar_mensaje(mensaje_nuevo)
    if not mensaje_normalizado:
        raise ValueError("El mensaje no tiene palabras útiles para procesar.")

    vector_clasificacion = vectorizador_clasificacion.transform(
        [mensaje_normalizado]
    )
    categoria_predicha = clasificador.predict(vector_clasificacion)[0]
    seguridad = float(clasificador.predict_proba(vector_clasificacion).max())
    similares_nuevos = buscar_mensajes_similares(
        mensaje_nuevo,
        recurso_busqueda["vectorizer"],
        recurso_busqueda["tfidf_matrix"],
        recurso_busqueda["reference"],
        recurso_busqueda["nearest_neighbors"],
        top_k=top_k,
    )
    return {
        "mensaje_original": mensaje_nuevo,
        "mensaje_normalizado": mensaje_normalizado,
        "categoria_predicha": categoria_predicha,
        "seguridad": seguridad,
        "mensajes_similares": similares_nuevos,
    }

recurso_busqueda = joblib.load(RUTA_ARTEFACTOS / "search_reference.joblib")
resultado_final = predecir_y_buscar(
    "Aparece una compra en mi tarjeta que yo no hice",
    tfidf_vectorizer_final,
    classifier_final,
    recurso_busqueda,
    top_k=5,
)
print("Categoría:", resultado_final["categoria_predicha"])
print(f"Seguridad del modelo: {resultado_final['seguridad']:.0%}")
mostrar_busqueda(resultado_final["mensajes_similares"])
```

### Qué haces

Este bloque abre las piezas desde el disco y responde un mensaje de acceso, sin volver a enseñar al programa. Comprueba que lo guardado sirve por sí solo. Las funciones de limpiar y de buscar siguen siendo las de este cuaderno; el diccionario, el clasificador y el índice salen de los archivos.

**Resultado esperado:** la categoría `problema_de_acceso`, una seguridad entre 0 % y 100 %, cinco mensajes y la frase «El sistema guardado respondió.»

**Pega esto en una celda nueva y ejecútala:**

```python
vectorizador_cargado = joblib.load(RUTA_ARTEFACTOS / "tfidf_vectorizer.joblib")
clasificador_cargado = joblib.load(RUTA_ARTEFACTOS / "classifier.joblib")
buscador_cargado = joblib.load(RUTA_ARTEFACTOS / "search_reference.joblib")

mensaje_prueba = "No puedo acceder a mi cuenta desde la aplicación"
mensaje_normalizado = normalizar_mensaje(mensaje_prueba)
vector = vectorizador_cargado.transform([mensaje_normalizado])
categoria = clasificador_cargado.predict(vector)[0]
seguridad = float(clasificador_cargado.predict_proba(vector).max())
resultado_busqueda = buscar_mensajes_similares(
    mensaje_prueba,
    buscador_cargado["vectorizer"],
    buscador_cargado["tfidf_matrix"],
    buscador_cargado["reference"],
    buscador_cargado["nearest_neighbors"],
    top_k=5,
)

print("Mensaje:", mensaje_prueba)
print("Categoría:", categoria)
print(f"Seguridad del modelo: {seguridad:.0%}")
mostrar_busqueda(resultado_busqueda)

with open(RUTA_ARTEFACTOS / "metrics_test.json", encoding="utf-8") as archivo:
    metricas_cargadas = json.load(archivo)

piezas = [
    "tfidf_vectorizer.joblib",
    "classifier.joblib",
    "search_reference.joblib",
    "pca_model.joblib",
    "metrics_test.json",
    "config_used.json",
]
faltan = [nombre for nombre in piezas if not (RUTA_ARTEFACTOS / nombre).exists()]
if not faltan and "metricas_test" in metricas_cargadas and len(resultado_busqueda) == 5:
    print("El sistema guardado respondió.")
else:
    print("Falta alguna pieza:", faltan)
```

### Qué haces

Este bloque, de vuelta en PowerShell, lista los archivos que produjo el taller: las piezas guardadas, los gráficos, el análisis y el archivo limpio.

**Resultado esperado:** en `artifacts` están los seis archivos de la tabla de este paso. En `reports` están los gráficos, la comparación de modelos, los errores y `analisis_errores.md`. En `data\processed` está `mensajes_bancarios_procesados.csv`.

**Pega esto en la terminal de PowerShell y pulsa Enter:**

```powershell
Set-Location C:\Laboratorio\nlp-banca-lab
Write-Output "--- artifacts ---"
Get-ChildItem .\artifacts -Name
Write-Output "--- reports ---"
Get-ChildItem .\reports -Name
Write-Output "--- data\processed ---"
Get-ChildItem .\data\processed -Name
```

**Sigue cuando:** leíste «El sistema guardado respondió.» y la terminal muestra los archivos.

## Lista de cierre

- [ ] El cuaderno está en `notebooks\01_clasificacion_busqueda_mensajes_bancarios.ipynb`.
- [ ] El archivo limpio conserva el mensaje original y el mensaje limpio.
- [ ] Aprendizaje, ensayo y examen quedaron en 108, 36 y 36.
- [ ] La bolsa de palabras y TF-IDF del clasificador se armaron con el aprendizaje.
- [ ] Comparaste Naive Bayes y regresión logística con el F1 macro del ensayo.
- [ ] Guardaste la matriz del examen y completaste diez observaciones.
- [ ] La búsqueda devuelve cinco mensajes, con similitud coseno y distancia euclidiana.
- [ ] Guardaste los dos mapas.
- [ ] Un mensaje nuevo recibe categoría, seguridad y cinco parecidos desde los archivos guardados.

## Si algo sale distinto

### La celda dice que falta un archivo, una librería o las palabras vacías

Eso se corrige en la preparación del equipo. Avisa a quien dirige el laboratorio y señala el mensaje. Sigue cuando esa persona confirme que la carpeta quedó lista. Después ejecuta otra vez desde la primera celda.

### Aparece un aviso de que una categoría tiene muy pocos mensajes

Ese texto significa que alguna categoría no alcanza para armar aprendizaje, ensayo y examen. Con el archivo de este laboratorio no ocurre. Si cambiaste el CSV, avisa a quien dirige el laboratorio.

### El mapa se detiene o Visual Studio Code pide reiniciar

El mapa necesita una tabla completa en memoria. Puedes dibujarlo con menos palabras y menos mensajes. El buscador sigue usando la colección completa.

#### Qué haces

Estas dos líneas reducen el mapa a 200 palabras y 500 mensajes como máximo. Después vuelves a ejecutar solo las dos celdas del paso 10.

**Resultado esperado:** el paso 10 termina y muestra los dos gráficos.

**Pega esto en una celda nueva, ejecútala y luego ejecuta otra vez las dos celdas del paso 10:**

```python
CONFIG["pca_max_features"] = 200
CONFIG["pca_max_samples"] = 500
print("El mapa usará menos palabras y, si hace falta, menos mensajes.")
```

### Una celda dice que un nombre no existe

Ejecuta las celdas desde el inicio del cuaderno hasta la que falló. El nombre se crea en una celda anterior.

## Al cerrar

1. Menú **Archivo > Guardar**, o `Ctrl+S`.
2. Cierra el cuaderno y Visual Studio Code.

Estas carpetas se quedan en el equipo. Ahí están tu cuaderno, los mensajes, los gráficos y el sistema guardado:

```text
data\raw\
data\processed\
notebooks\
artifacts\
reports\
config\
```

## Qué te llevas

Limpiaste mensajes en español y conservaste el texto original. Convertiste ese texto en conteos y en TF-IDF. Enseñaste dos programas, elegiste uno con el ensayo y lo mediste con un examen que no había participado en la elección.

Revisaste errores, casi siempre entre categorías que comparten palabras. Buscaste cinco mensajes parecidos con similitud coseno, distancia euclidiana y vecinos más cercanos. Miraste la colección en un mapa de dos ejes y guardaste el clasificador y el buscador para consultar un mensaje nuevo.

Una mejora posterior, ya fuera de este taller, puede ser reunir más ejemplos, revisar las categorías que se confunden y mantener la revisión humana en los casos de fraude, bloqueo, dinero y acceso.

## Si quieres leer más

- [Clasificación de texto en scikit-learn](https://scikit-learn.org/stable/tutorial/text_analytics/working_with_text_data.html)
- [TfidfVectorizer](https://scikit-learn.org/stable/modules/generated/sklearn.feature_extraction.text.TfidfVectorizer.html)
- [Vecinos más cercanos](https://scikit-learn.org/stable/modules/generated/sklearn.neighbors.NearestNeighbors.html)
- [PCA](https://scikit-learn.org/stable/modules/generated/sklearn.decomposition.PCA.html)
