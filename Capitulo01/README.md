# Clasificación y búsqueda de mensajes bancarios con NLP

## Metadatos

| Campo | Valor |
|---|---|
| Duración | 225 minutos |
| Complejidad | Media |
| Nivel de Bloom | Aplicar |
| Modalidad | Laboratorio individual con notebook reproducible |
| Directorio raíz obligatorio | `/home/<usuario>/nlp-banca-lab` |
| Notebook principal | `notebooks/01_clasificacion_busqueda_mensajes_bancarios.ipynb` |

## Descripción general

En este laboratorio construirás un flujo completo de procesamiento de lenguaje natural para mensajes bancarios. El sistema clasificará mensajes en categorías operativas, tales como transferencias, tarjetas, fraude, comisiones y problemas de acceso, y recuperará mensajes similares mediante vectores TF-IDF.

El notebook final integrará carga de datos, limpieza de texto en español, análisis exploratorio, vectorización, entrenamiento de modelos supervisados, evaluación, análisis de errores, búsqueda por similitud, visualización con PCA y persistencia de artefactos. El resultado debe permitir ingresar un mensaje nuevo, obtener una categoría predicha con confianza y recuperar los cinco mensajes más parecidos del corpus.

> **Nota de seguridad:** utiliza únicamente datos autorizados, anonimizados o sintéticos. No incluyas números reales de tarjeta, cuentas, documentos de identidad, correos personales ni información transaccional sensible en los artefactos, capturas o repositorios.

## Objetivos de aprendizaje

Al finalizar el laboratorio, podrás:

- [ ] Configurar un entorno Python reproducible y crear la estructura obligatoria del proyecto de PLN bancario.
- [ ] Preparar mensajes bancarios en español, conservando el texto original y generando una versión normalizada.
- [ ] Construir representaciones `CountVectorizer` y `TfidfVectorizer` sin fuga de información entre entrenamiento, validación y prueba.
- [ ] Entrenar, comparar y evaluar modelos `MultinomialNB` y `LogisticRegression` mediante exactitud, precisión, recall y F1 macro.
- [ ] Implementar consultas de similitud coseno, distancia euclidiana, vecinos más cercanos y visualización PCA, persistiendo los artefactos necesarios.

## Requisitos previos

### Conocimientos

Debes comprender los siguientes conceptos:

- Fundamentos de Python, `pandas`, `NumPy` y ejecución de celdas en Jupyter.
- Preprocesamiento de texto, tokenización y palabras vacías.
- Representación bag-of-words, TF-IDF y matrices dispersas.
- Regresión logística, Naive Bayes, similitud coseno, distancia euclidiana, vecinos más cercanos y PCA.
- Métricas de clasificación: exactitud, precisión, recall, F1 macro, matriz de confusión y reporte de clasificación.
- Diferencia entre **clasificación** (“¿a qué categoría pertenece?”) y **búsqueda de similitud** (“¿qué mensajes se parecen?”).

### Acceso y archivos necesarios

Debes contar con:

- Permisos para crear directorios, entornos virtuales e instalar paquetes.
- Conexión a Internet durante la instalación inicial.
- El archivo CSV UTF-8:

```text
/home/<usuario>/nlp-banca-lab/data/raw/mensajes_bancarios.csv
```

El archivo debe usar separador coma e incluir como mínimo estas columnas:

```text
message,category
```

La columna opcional recomendada es:

```text
message_id
```

Ejemplo de estructura válida:

```csv
message_id,message,category
1,"No reconozco un pago realizado con mi tarjeta",cargo_no_reconocido
2,"La transferencia enviada ayer todavía no llega",transferencia_pendiente
3,"No puedo iniciar sesión en la aplicación",problema_de_acceso
```

## Entorno del laboratorio

### Requisitos de hardware

| Recurso | Mínimo | Recomendado |
|---|---:|---:|
| Arquitectura | 64 bits | 64 bits |
| Procesador | 4 núcleos físicos | 4 o más núcleos físicos |
| Memoria RAM | 8 GB | 16 GB |
| Espacio libre | 10 GB | 10 GB o más |
| Pantalla | 1366x768 | 1920x1080 |
| Red | Durante instalación | Opcional tras descargar paquetes y datos |

### Software y versiones bloqueadas

| Componente | Versión |
|---|---:|
| Ubuntu Desktop | 22.04.4 LTS |
| Python | 3.11.9 |
| pip | 24.0 |
| venv | 3.11.9 |
| JupyterLab | 4.2.5 |
| NumPy | 1.26.4 |
| pandas | 2.2.2 |
| scikit-learn | 1.5.1 |
| SciPy | 1.13.1 |
| Matplotlib | 3.9.1 |
| Seaborn | 0.13.2 |
| NLTK | 3.8.1 |
| joblib | 1.4.2 |

> En Windows 11 23H2, sustituye la ruta raíz por `C:\Users\<usuario>\nlp-banca-lab`, manteniendo exactamente la misma estructura interna.

### Comandos iniciales

Abre una terminal y define tu usuario local:

```bash
export USUARIO="$(whoami)"
export PROYECTO="/home/${USUARIO}/nlp-banca-lab"
```

Crea el proyecto y el entorno virtual:

```bash
mkdir -p "$PROYECTO"
cd "$PROYECTO"

python3.11 -m venv .venv
source .venv/bin/activate

python -m pip install --upgrade pip==24.0
```

En Windows PowerShell:

```powershell
$PROYECTO = "C:\Users\$env:USERNAME\nlp-banca-lab"
New-Item -ItemType Directory -Force -Path $PROYECTO
Set-Location $PROYECTO

py -3.11 -m venv .venv
.\.venv\Scripts\Activate.ps1

python -m pip install --upgrade pip==24.0
```

---

## Procedimiento paso a paso

### Paso 1. Crear la estructura reproducible del proyecto

**Objetivo:** crear los directorios obligatorios y registrar las dependencias bloqueadas para asegurar que el laboratorio pueda repetirse.

**Instrucciones:**

1. Con el entorno virtual activado, crea la estructura requerida:

```bash
cd "$PROYECTO"

mkdir -p data/raw data/processed notebooks artifacts reports src config
```

2. Crea el archivo `requirements.txt`:

```bash
cat > requirements.txt <<'EOF'
jupyterlab==4.2.5
ipykernel==6.29.5
numpy==1.26.4
pandas==2.2.2
scikit-learn==1.5.1
scipy==1.13.1
matplotlib==3.9.1
seaborn==0.13.2
nltk==3.8.1
joblib==1.4.2
EOF
```

3. Instala las dependencias:

```bash
python -m pip install -r requirements.txt
```

4. Registra el kernel de Jupyter para este entorno:

```bash
python -m ipykernel install --user \
  --name nlp-banca-lab \
  --display-name "Python 3.11 - NLP Banca Lab"
```

5. Guarda las versiones efectivamente instaladas:

```bash
python -m pip freeze > reports/requirements_instalados.txt
python --version > reports/version_python.txt
```

6. Verifica la estructura:

```bash
find "$PROYECTO" -maxdepth 2 -type d | sort
```

**Resultado esperado:**

La estructura debe contener al menos:

```text
nlp-banca-lab/
├── artifacts/
├── config/
├── data/
│   ├── processed/
│   └── raw/
├── notebooks/
├── reports/
├── src/
├── .venv/
└── requirements.txt
```

**Verificación:**

Ejecuta:

```bash
python -c "import sklearn, pandas, numpy, nltk, joblib; print(sklearn.__version__, pandas.__version__, numpy.__version__)"
```

Debes observar las versiones `1.5.1`, `2.2.2` y `1.26.4`.

---

### Paso 2. Validar y ubicar el archivo de mensajes bancarios

**Objetivo:** confirmar que el conjunto de datos tiene codificación, columnas y contenido aptos para el flujo de clasificación y búsqueda.

**Instrucciones:**

1. Copia o mueve el archivo autorizado al directorio obligatorio:

```bash
cp /ruta/al/origen/mensajes_bancarios.csv \
  "$PROYECTO/data/raw/mensajes_bancarios.csv"
```

2. Revisa las primeras líneas sin modificar el archivo original:

```bash
head -n 5 "$PROYECTO/data/raw/mensajes_bancarios.csv"
```

3. Crea una configuración centralizada en `config/config.json`:

```bash
cat > config/config.json <<'EOF'
{
  "random_state": 42,
  "test_size": 0.20,
  "validation_size_relative": 0.25,
  "min_df": 1,
  "max_df": 0.95,
  "ngram_range": [1, 2],
  "max_features": 5000,
  "top_k": 5,
  "pca_max_features": 500,
  "pca_max_samples": 2000,
  "input_path": "data/raw/mensajes_bancarios.csv",
  "processed_path": "data/processed/mensajes_bancarios_procesados.csv"
}
EOF
```

4. Inicia JupyterLab desde la raíz del proyecto:

```bash
jupyter lab
```

5. En JupyterLab, selecciona el kernel **Python 3.11 - NLP Banca Lab** y crea el notebook:

```text
notebooks/01_clasificacion_busqueda_mensajes_bancarios.ipynb
```

6. En la primera celda del notebook, registra las importaciones y rutas. Ejecuta la siguiente celda:

```python
from pathlib import Path
import json
import re
import unicodedata
import warnings

import joblib
import matplotlib.pyplot as plt
import nltk
import numpy as np
import pandas as pd
import seaborn as sns

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

RUTA_RAIZ = Path.cwd().resolve()
if RUTA_RAIZ.name == "notebooks":
    RUTA_RAIZ = RUTA_RAIZ.parent

RUTA_CONFIG = RUTA_RAIZ / "config" / "config.json"
with open(RUTA_CONFIG, encoding="utf-8") as archivo:
    CONFIG = json.load(archivo)

RUTA_ENTRADA = RUTA_RAIZ / CONFIG["input_path"]
RUTA_PROCESADA = RUTA_RAIZ / CONFIG["processed_path"]
RUTA_ARTEFACTOS = RUTA_RAIZ / "artifacts"
RUTA_REPORTES = RUTA_RAIZ / "reports"

print(f"Raíz del proyecto: {RUTA_RAIZ}")
print(f"Archivo de entrada: {RUTA_ENTRADA}")
```

**Resultado esperado:**

La celda debe mostrar rutas similares a:

```text
Raíz del proyecto: /home/usuario/nlp-banca-lab
Archivo de entrada: /home/usuario/nlp-banca-lab/data/raw/mensajes_bancarios.csv
```

**Verificación:**

Ejecuta esta celda para validar que el archivo existe:

```python
assert RUTA_ENTRADA.exists(), (
    f"No se encontró el archivo requerido: {RUTA_ENTRADA}"
)
```

La celda debe finalizar sin errores.

---

### Paso 3. Cargar, inspeccionar y analizar exploratoriamente los datos

**Objetivo:** comprobar la calidad inicial del CSV, identificar valores nulos, duplicados, distribución de categorías y longitudes de mensajes.

**Instrucciones:**

1. Carga el archivo con UTF-8 y separador coma:

```python
df = pd.read_csv(RUTA_ENTRADA, encoding="utf-8", sep=",")

columnas_requeridas = {"message", "category"}
faltantes = columnas_requeridas - set(df.columns)
assert not faltantes, f"Faltan columnas obligatorias: {faltantes}"

if "message_id" not in df.columns:
    df.insert(0, "message_id", range(1, len(df) + 1))
    print("Se creó message_id porque no estaba presente en el archivo original.")

df["message"] = df["message"].astype("string")
df["category"] = df["category"].astype("string")

print("Dimensiones iniciales:", df.shape)
display(df.head())
display(df.info())
```

2. Calcula indicadores de calidad:

```python
resumen_calidad = pd.DataFrame({
    "indicador": [
        "filas_totales",
        "message_nulos",
        "category_nulos",
        "message_vacios",
        "duplicados_exactos_message_category",
        "categorias_unicas"
    ],
    "valor": [
        len(df),
        int(df["message"].isna().sum()),
        int(df["category"].isna().sum()),
        int(df["message"].fillna("").str.strip().eq("").sum()),
        int(df.duplicated(subset=["message", "category"]).sum()),
        int(df["category"].nunique())
    ]
})

display(resumen_calidad)
```

3. Examina la distribución de clases:

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
    palette="viridis"
)
plt.title("Distribución inicial de categorías")
plt.xlabel("Cantidad de mensajes")
plt.ylabel("Categoría")
plt.tight_layout()
plt.savefig(RUTA_REPORTES / "distribucion_clases_inicial.png", dpi=150)
plt.show()
```

4. Calcula la longitud aproximada de los mensajes:

```python
df["longitud_caracteres"] = df["message"].fillna("").str.len()
df["longitud_palabras"] = (
    df["message"]
    .fillna("")
    .str.findall(r"\b\w+\b")
    .str.len()
)

display(df[["longitud_caracteres", "longitud_palabras"]].describe())

plt.figure(figsize=(10, 5))
sns.histplot(
    data=df,
    x="longitud_palabras",
    hue="category",
    multiple="stack",
    bins=30
)
plt.title("Distribución de longitud de mensajes")
plt.xlabel("Número de palabras")
plt.tight_layout()
plt.savefig(RUTA_REPORTES / "longitud_mensajes_inicial.png", dpi=150)
plt.show()
```

5. Verifica la viabilidad de la partición estratificada. Cada categoría debe tener suficientes ejemplos. Para una división entrenamiento/validación/prueba, se recomienda un mínimo de 10 ejemplos por clase; idealmente, 20 o más.

```python
conteo_minimo = df["category"].value_counts()
clases_insuficientes = conteo_minimo[conteo_minimo < 10]

if not clases_insuficientes.empty:
    print("Advertencia: clases con menos de 10 ejemplos:")
    display(clases_insuficientes)
else:
    print("Todas las clases tienen al menos 10 ejemplos.")
```

**Resultado esperado:**

Debes identificar:

- El número total de mensajes y categorías.
- Mensajes vacíos o nulos que deberán eliminarse.
- Duplicados exactos por `message` y `category`.
- Categorías con baja representación.
- Distribución de longitudes que ayude a detectar mensajes anormalmente cortos o extensos.

**Verificación:**

Confirma que existen al menos dos categorías y que no hay una única clase:

```python
assert df["category"].nunique() >= 2, (
    "Se requieren al menos dos categorías para clasificación supervisada."
)
```

---

### Paso 4. Limpiar y normalizar texto en español

**Objetivo:** generar una columna normalizada de manera reproducible, conservando el mensaje original para auditoría, evaluación y recuperación de casos.

**Instrucciones:**

1. Descarga las palabras vacías de NLTK una sola vez. Este paso requiere Internet si el recurso no existe localmente:

```python
nltk.download("stopwords", quiet=True)
```

2. Define una función de normalización. La función realiza las siguientes acciones controladas:

   - Convierte a minúsculas.
   - Sustituye correos, números de tarjeta largos, números de cuenta o referencias extensas por marcadores.
   - Elimina URL.
   - Conserva letras españolas, incluidos caracteres acentuados y `ñ`.
   - Elimina signos de puntuación y espacios repetidos.
   - Elimina palabras vacías en español.
   - Conserva la negación `no`, importante para mensajes como “no reconozco este cargo”.

```python
STOPWORDS_ES = set(stopwords.words("spanish"))
STOPWORDS_ES.discard("no")

def normalizar_mensaje(texto: str) -> str:
    """Normaliza texto bancario en español sin alterar la columna original."""
    if pd.isna(texto):
        return ""

    texto = str(texto).lower().strip()
    texto = unicodedata.normalize("NFC", texto)

    # Minimización básica de datos potencialmente sensibles.
    texto = re.sub(
        r"\b[A-Z0-9._%+-]+@[A-Z0-9.-]+\.[A-Z]{2,}\b",
        " correo ",
        texto,
        flags=re.IGNORECASE
    )
    texto = re.sub(r"https?://\S+|www\.\S+", " url ", texto)
    texto = re.sub(r"\b(?:\d[ -]?){13,19}\b", " tarjeta_numero ", texto)
    texto = re.sub(r"\b\d{8,}\b", " referencia_numero ", texto)

    # Conserva letras, dígitos cortos, ñ y vocales con acento.
    texto = re.sub(r"[^a-záéíóúüñ0-9\s]", " ", texto)
    texto = re.sub(r"\s+", " ", texto).strip()

    tokens = [
        token for token in texto.split()
        if token not in STOPWORDS_ES and len(token) > 1
    ]

    return " ".join(tokens)
```

3. Elimina filas sin contenido útil o sin categoría y conserva una copia de la información original:

```python
df_limpio = df.copy()

df_limpio["message_original"] = df_limpio["message"].fillna("").str.strip()
df_limpio["category"] = df_limpio["category"].fillna("").str.strip()

df_limpio = df_limpio[
    (df_limpio["message_original"] != "") &
    (df_limpio["category"] != "")
].copy()

df_limpio["message_normalized"] = (
    df_limpio["message_original"].apply(normalizar_mensaje)
)

df_limpio = df_limpio[
    df_limpio["message_normalized"].str.strip() != ""
].copy()
```

4. Elimina duplicados exactos por texto normalizado y categoría. Esto evita que el mismo mensaje aparezca en entrenamiento y prueba:

```python
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
```

5. Guarda el conjunto procesado únicamente en el directorio requerido:

```python
columnas_salida = [
    "message_id",
    "message_original",
    "message_normalized",
    "category",
    "longitud_caracteres",
    "longitud_palabras"
]

columnas_existentes = [
    columna for columna in columnas_salida
    if columna in df_limpio.columns
]

df_limpio[columnas_existentes].to_csv(
    RUTA_PROCESADA,
    index=False,
    encoding="utf-8"
)

print(f"Archivo procesado guardado en: {RUTA_PROCESADA}")
```

**Resultado esperado:**

El DataFrame procesado debe incluir al menos:

| Columna | Descripción |
|---|---|
| `message_id` | Identificador del mensaje |
| `message_original` | Texto original para trazabilidad |
| `message_normalized` | Texto limpio usado para vectorización |
| `category` | Etiqueta operativa |

Ejemplo:

```text
message_original:   No reconozco un pago realizado con mi tarjeta.
message_normalized: no reconozco pago realizado tarjeta
category:           cargo_no_reconocido
```

**Verificación:**

Ejecuta:

```python
assert RUTA_PROCESADA.exists(), "No se creó el archivo procesado."
assert df_limpio["message_normalized"].str.len().gt(0).all()
assert not df_limpio.duplicated(
    subset=["message_normalized", "category"]
).any()

print("Preprocesamiento validado correctamente.")
```

---

### Paso 5. Separar entrenamiento, validación y prueba sin fuga de información

**Objetivo:** crear particiones estratificadas reproducibles y garantizar que los transformadores se ajusten solo con datos de entrenamiento.

**Instrucciones:**

1. Define variables de entrada y etiqueta:

```python
X = df_limpio["message_normalized"]
y = df_limpio["category"]

SEMILLA = CONFIG["random_state"]
```

2. Separa primero el 20 % para prueba. Después divide el 80 % restante en entrenamiento y validación.

La configuración `validation_size_relative=0.25` significa que el 25 % del bloque temporal será validación. Como el bloque representa el 80 % del total, la validación final equivale al 20 % del conjunto completo.

```python
X_temp, X_test, y_temp, y_test = train_test_split(
    X,
    y,
    test_size=CONFIG["test_size"],
    stratify=y,
    random_state=SEMILLA
)

X_train, X_val, y_train, y_val = train_test_split(
    X_temp,
    y_temp,
    test_size=CONFIG["validation_size_relative"],
    stratify=y_temp,
    random_state=SEMILLA
)

print("Tamaños de particiones:")
print(f"Entrenamiento: {len(X_train)}")
print(f"Validación:    {len(X_val)}")
print(f"Prueba:        {len(X_test)}")
```

3. Inspecciona la proporción de clases en cada partición:

```python
def resumen_particion(nombre, etiquetas):
    return (
        etiquetas.value_counts(normalize=True)
        .rename(nombre)
        .mul(100)
        .round(2)
    )

comparacion_particiones = pd.concat(
    [
        resumen_particion("train_%", y_train),
        resumen_particion("validation_%", y_val),
        resumen_particion("test_%", y_test),
    ],
    axis=1
).fillna(0)

display(comparacion_particiones)
```

4. Guarda los identificadores de prueba para reforzar la trazabilidad del experimento:

```python
ids_test = df_limpio.loc[X_test.index, ["message_id", "message_original", "category"]]
ids_test.to_csv(
    RUTA_REPORTES / "muestra_prueba_reproducible.csv",
    index=False,
    encoding="utf-8"
)
```

**Resultado esperado:**

Las particiones deben aproximarse a:

```text
Entrenamiento: 60 %
Validación:    20 %
Prueba:        20 %
```

Cada categoría debe aparecer en las tres particiones, siempre que el tamaño del conjunto lo permita.

**Verificación:**

```python
assert set(y_train.unique()) == set(y_val.unique()) == set(y_test.unique()), (
    "No todas las particiones contienen las mismas categorías. "
    "Revise el tamaño mínimo por clase."
)

assert set(X_train.index).isdisjoint(set(X_test.index))
assert set(X_val.index).isdisjoint(set(X_test.index))

print("Particiones estratificadas y disjuntas verificadas.")
```

---

### Paso 6. Construir representaciones bag-of-words y TF-IDF

**Objetivo:** comparar la representación de conteo con TF-IDF y crear la matriz TF-IDF que alimentará los clasificadores y el buscador.

**Instrucciones:**

1. Ajusta `CountVectorizer` exclusivamente con entrenamiento. Se usan unigramas y bigramas para capturar expresiones como “no reconozco”, “transferencia pendiente” o “bloquear tarjeta”.

```python
count_vectorizer = CountVectorizer(
    ngram_range=tuple(CONFIG["ngram_range"]),
    min_df=CONFIG["min_df"],
    max_df=CONFIG["max_df"],
    max_features=CONFIG["max_features"]
)

X_train_count = count_vectorizer.fit_transform(X_train)
X_val_count = count_vectorizer.transform(X_val)
X_test_count = count_vectorizer.transform(X_test)

print("Matriz bag-of-words de entrenamiento:", X_train_count.shape)
print("Vocabulario bag-of-words:", len(count_vectorizer.vocabulary_))
```

2. Ajusta `TfidfVectorizer` exclusivamente con entrenamiento. La normalización L2 permite comparar vectores por similitud coseno de forma consistente.

```python
tfidf_vectorizer = TfidfVectorizer(
    ngram_range=tuple(CONFIG["ngram_range"]),
    min_df=CONFIG["min_df"],
    max_df=CONFIG["max_df"],
    max_features=CONFIG["max_features"],
    sublinear_tf=True,
    norm="l2"
)

X_train_tfidf = tfidf_vectorizer.fit_transform(X_train)
X_val_tfidf = tfidf_vectorizer.transform(X_val)
X_test_tfidf = tfidf_vectorizer.transform(X_test)

print("Matriz TF-IDF de entrenamiento:", X_train_tfidf.shape)
print("Vocabulario TF-IDF:", len(tfidf_vectorizer.vocabulary_))
```

3. Revisa algunos términos del vocabulario:

```python
terminos_tfidf = tfidf_vectorizer.get_feature_names_out()

print("Primeros 30 términos del vocabulario TF-IDF:")
print(terminos_tfidf[:30])
```

4. Calcula la densidad de la matriz. Los textos suelen generar matrices dispersas; esto es esperado y eficiente.

```python
densidad = X_train_tfidf.nnz / (
    X_train_tfidf.shape[0] * X_train_tfidf.shape[1]
)

print(f"Densidad de la matriz TF-IDF: {densidad:.6f}")
```

**Resultado esperado:**

Debes obtener matrices dispersas similares a:

```text
Matriz bag-of-words de entrenamiento: (n_mensajes, n_terminos)
Matriz TF-IDF de entrenamiento: (n_mensajes, n_terminos)
Densidad de la matriz TF-IDF: valor pequeño
```

**Verificación:**

```python
assert X_train_tfidf.shape[0] == len(X_train)
assert X_val_tfidf.shape[0] == len(X_val)
assert X_test_tfidf.shape[0] == len(X_test)
assert X_train_tfidf.shape[1] == X_val_tfidf.shape[1] == X_test_tfidf.shape[1]

print("Vectorización validada sin fuga de información.")
```

---

### Paso 7. Entrenar y comparar Naive Bayes y regresión logística

**Objetivo:** entrenar dos clasificadores multiclase sobre TF-IDF y seleccionar el candidato con mayor F1 macro en validación.

**Instrucciones:**

1. Entrena el modelo Multinomial Naive Bayes:

```python
modelo_nb = MultinomialNB(alpha=1.0)
modelo_nb.fit(X_train_tfidf, y_train)

pred_val_nb = modelo_nb.predict(X_val_tfidf)
```

2. Entrena el modelo de regresión logística. El ajuste usa `class_weight="balanced"` para mitigar parcialmente distribuciones desbalanceadas y `max_iter=2000` para favorecer la convergencia.

```python
modelo_lr = LogisticRegression(
    max_iter=2000,
    class_weight="balanced",
    random_state=SEMILLA
)
modelo_lr.fit(X_train_tfidf, y_train)

pred_val_lr = modelo_lr.predict(X_val_tfidf)
```

3. Define una función de evaluación común:

```python
def calcular_metricas(y_real, y_pred):
    return {
        "accuracy": round(float(accuracy_score(y_real, y_pred)), 4),
        "precision_macro": round(
            float(precision_score(y_real, y_pred, average="macro", zero_division=0)),
            4
        ),
        "recall_macro": round(
            float(recall_score(y_real, y_pred, average="macro", zero_division=0)),
            4
        ),
        "f1_macro": round(
            float(f1_score(y_real, y_pred, average="macro", zero_division=0)),
            4
        )
    }
```

4. Compara ambos resultados de validación:

```python
resultados_validacion = pd.DataFrame([
    {"modelo": "MultinomialNB", **calcular_metricas(y_val, pred_val_nb)},
    {"modelo": "LogisticRegression", **calcular_metricas(y_val, pred_val_lr)}
]).sort_values("f1_macro", ascending=False)

display(resultados_validacion)

resultados_validacion.to_csv(
    RUTA_REPORTES / "comparacion_modelos_validacion.csv",
    index=False,
    encoding="utf-8"
)
```

5. Selecciona el mejor modelo según `f1_macro`. Si existe empate, selecciona el modelo con mayor `recall_macro` y documenta la decisión.

```python
mejor_nombre = resultados_validacion.iloc[0]["modelo"]

if mejor_nombre == "MultinomialNB":
    mejor_modelo = modelo_nb
else:
    mejor_modelo = modelo_lr

print(f"Modelo seleccionado por F1 macro: {mejor_nombre}")
```

**Resultado esperado:**

Debes contar con una tabla comparable:

| modelo | accuracy | precision_macro | recall_macro | f1_macro |
|---|---:|---:|---:|---:|
| LogisticRegression o MultinomialNB | valor | valor | valor | valor |
| Otro modelo | valor | valor | valor | valor |

No existe un modelo universalmente superior. La selección debe basarse en las métricas obtenidas en tu conjunto de validación y no en una suposición previa.

**Verificación:**

```python
assert resultados_validacion.shape[0] == 2
assert mejor_nombre in {"MultinomialNB", "LogisticRegression"}
assert 0 <= resultados_validacion["f1_macro"].max() <= 1

print("Comparación de modelos completada.")
```

---

### Paso 8. Evaluar el modelo seleccionado en prueba y analizar errores

**Objetivo:** medir el desempeño final en datos no usados durante selección, generar reportes y revisar al menos diez errores de clasificación.

**Instrucciones:**

1. Predice el conjunto de prueba con el modelo seleccionado:

```python
pred_test = mejor_modelo.predict(X_test_tfidf)
proba_test = mejor_modelo.predict_proba(X_test_tfidf)
confianza_test = proba_test.max(axis=1)

metricas_test = calcular_metricas(y_test, pred_test)
metricas_test["modelo_seleccionado"] = mejor_nombre

print(metricas_test)
```

2. Genera el reporte de clasificación:

```python
etiquetas_ordenadas = sorted(y.unique())

reporte_test = classification_report(
    y_test,
    pred_test,
    labels=etiquetas_ordenadas,
    zero_division=0
)

print(reporte_test)

with open(RUTA_REPORTES / "reporte_clasificacion_test.txt", "w", encoding="utf-8") as archivo:
    archivo.write(reporte_test)
```

3. Genera y guarda la matriz de confusión:

```python
matriz = confusion_matrix(
    y_test,
    pred_test,
    labels=etiquetas_ordenadas
)

plt.figure(figsize=(10, 8))
sns.heatmap(
    matriz,
    annot=True,
    fmt="d",
    cmap="Blues",
    xticklabels=etiquetas_ordenadas,
    yticklabels=etiquetas_ordenadas
)
plt.title(f"Matriz de confusión - {mejor_nombre}")
plt.xlabel("Categoría predicha")
plt.ylabel("Categoría real")
plt.xticks(rotation=45, ha="right")
plt.yticks(rotation=0)
plt.tight_layout()
plt.savefig(RUTA_REPORTES / "matriz_confusion_test.png", dpi=150)
plt.show()
```

4. Crea un DataFrame para análisis de errores. Usa el texto original, no solo el normalizado, para facilitar interpretación humana:

```python
resultados_test = df_limpio.loc[X_test.index, [
    "message_id",
    "message_original",
    "message_normalized",
    "category"
]].copy()

resultados_test = resultados_test.rename(columns={"category": "categoria_real"})
resultados_test["categoria_predicha"] = pred_test
resultados_test["confianza"] = confianza_test
resultados_test["es_error"] = (
    resultados_test["categoria_real"] != resultados_test["categoria_predicha"]
)

errores_test = resultados_test[
    resultados_test["es_error"]
].sort_values("confianza", ascending=False)

display(errores_test.head(10))

errores_test.to_csv(
    RUTA_REPORTES / "errores_clasificacion_test.csv",
    index=False,
    encoding="utf-8"
)
```

5. Analiza al menos diez errores, o todos los errores si existen menos de diez. Para cada caso, identifica una posible causa:

   - Vocabulario compartido entre categorías, por ejemplo “tarjeta” en fraude y bloqueo.
   - Mensaje demasiado breve o ambiguo.
   - Etiqueta original incorrecta o inconsistente.
   - Poca representación de una categoría.
   - Necesidad de contexto no disponible, como importe, comercio, estado de la transacción o canal de acceso.

6. Documenta tus observaciones en `reports/analisis_errores.md`:

```bash
cat > reports/analisis_errores.md <<'EOF'
# Análisis de errores de clasificación

## Modelo seleccionado
Completar con el nombre del modelo y la F1 macro de prueba.

## Observaciones de al menos diez errores
1. Mensaje ID: completar. Causa probable: ambigüedad entre categorías.
2. Mensaje ID: completar. Causa probable: vocabulario compartido.
3. Mensaje ID: completar. Causa probable: clase poco representada.
4. Mensaje ID: completar. Causa probable: etiqueta potencialmente inconsistente.
5. Mensaje ID: completar. Causa probable: falta de contexto.
6. Mensaje ID: completar. Causa probable: falta de contexto.
7. Mensaje ID: completar. Causa probable: mensaje demasiado corto.
8. Mensaje ID: completar. Causa probable: expresión poco frecuente.
9. Mensaje ID: completar. Causa probable: vocabulario compartido.
10. Mensaje ID: completar. Causa probable: ambigüedad operativa.

## Acción de mejora propuesta
Completar con una acción concreta: recolectar más ejemplos, revisar etiquetas,
agregar bigramas, ajustar clases operativas o incorporar reglas de escalamiento.
EOF
```

**Resultado esperado:**

Debes obtener:

- Métricas de prueba independientes de validación.
- Reporte de clasificación por categoría.
- Matriz de confusión guardada en `reports/`.
- Archivo CSV con errores.
- Análisis documentado de un mínimo de diez errores cuando estén disponibles.

**Verificación:**

```python
assert len(pred_test) == len(y_test)
assert len(resultados_test) == len(y_test)
assert resultados_test["confianza"].between(0, 1).all()

print("Evaluación de prueba y análisis de errores completados.")
```

---

### Paso 9. Construir la búsqueda de mensajes similares

**Objetivo:** recuperar los cinco mensajes más similares mediante similitud coseno, distancia euclidiana y `NearestNeighbors` sobre vectores TF-IDF.

**Instrucciones:**

1. Para el buscador final, ajusta un vectorizador TF-IDF sobre el corpus procesado completo. Esta decisión es válida para recuperación documental, porque no se usa para informar la métrica de prueba del clasificador.

> El vectorizador de clasificación se ajustó exclusivamente con entrenamiento para evitar fuga de información. El vectorizador persistido para uso operativo puede ajustarse con el corpus de referencia completo, porque su función es indexar todos los documentos disponibles para búsqueda.

```python
tfidf_vectorizer_search = TfidfVectorizer(
    ngram_range=tuple(CONFIG["ngram_range"]),
    min_df=CONFIG["min_df"],
    max_df=CONFIG["max_df"],
    max_features=CONFIG["max_features"],
    sublinear_tf=True,
    norm="l2"
)

X_corpus_tfidf = tfidf_vectorizer_search.fit_transform(
    df_limpio["message_normalized"]
)

referencia_busqueda = df_limpio[
    ["message_id", "message_original", "message_normalized", "category"]
].reset_index(drop=True)

print("Matriz de búsqueda:", X_corpus_tfidf.shape)
```

2. Construye el índice de vecinos con métrica coseno. `algorithm="brute"` es apropiado para matrices dispersas TF-IDF.

```python
indice_vecinos = NearestNeighbors(
    n_neighbors=CONFIG["top_k"],
    metric="cosine",
    algorithm="brute"
)

indice_vecinos.fit(X_corpus_tfidf)
```

3. Define una función de consulta que calcule similitud coseno, distancia euclidiana y vecinos más cercanos:

```python
def buscar_mensajes_similares(
    consulta: str,
    vectorizador,
    matriz_referencia,
    referencia,
    vecinos,
    top_k: int = 5
) -> pd.DataFrame:
    consulta_normalizada = normalizar_mensaje(consulta)

    if not consulta_normalizada:
        raise ValueError("La consulta no contiene términos útiles tras normalización.")

    vector_consulta = vectorizador.transform([consulta_normalizada])

    similitudes = cosine_similarity(
        vector_consulta,
        matriz_referencia
    ).ravel()

    distancias_euclidianas = euclidean_distances(
        vector_consulta,
        matriz_referencia
    ).ravel()

    distancias_coseno, indices = vecinos.kneighbors(
        vector_consulta,
        n_neighbors=top_k
    )

    resultado = referencia.iloc[indices[0]].copy()
    resultado["similitud_coseno"] = similitudes[indices[0]]
    resultado["distancia_euclidiana"] = distancias_euclidianas[indices[0]]
    resultado["distancia_coseno"] = distancias_coseno[0]

    resultado = resultado.sort_values(
        by=["similitud_coseno", "distancia_euclidiana"],
        ascending=[False, True]
    )

    return resultado[[
        "message_id",
        "message_original",
        "category",
        "similitud_coseno",
        "distancia_euclidiana",
        "distancia_coseno"
    ]]
```

4. Ejecuta una consulta de ejemplo:

```python
consulta_ejemplo = "Veo una compra con mi tarjeta que no autoricé"

similares = buscar_mensajes_similares(
    consulta=consulta_ejemplo,
    vectorizador=tfidf_vectorizer_search,
    matriz_referencia=X_corpus_tfidf,
    referencia=referencia_busqueda,
    vecinos=indice_vecinos,
    top_k=5
)

display(similares)
```

5. Interpreta las métricas:

   - **Similitud coseno:** valores más cercanos a `1` indican mayor similitud angular.
   - **Distancia coseno:** valores más cercanos a `0` indican mayor similitud.
   - **Distancia euclidiana:** valores más cercanos a `0` indican mayor proximidad. Como TF-IDF usa normalización L2, esta distancia está relacionada con la similitud coseno, aunque conserva una escala distinta.

6. Prueba consultas de diferentes categorías:

```python
consultas_prueba = [
    "La transferencia que envié sigue pendiente",
    "Quiero bloquear mi tarjeta porque la perdí",
    "No puedo entrar a la aplicación del banco",
    "Por qué me cobraron una comisión de mantenimiento"
]

for consulta in consultas_prueba:
    print(f"\nCONSULTA: {consulta}")
    display(
        buscar_mensajes_similares(
            consulta,
            tfidf_vectorizer_search,
            X_corpus_tfidf,
            referencia_busqueda,
            indice_vecinos,
            top_k=5
        )
    )
```

**Resultado esperado:**

Cada consulta debe retornar cinco mensajes ordenados, con sus categorías y métricas de proximidad. Para una consulta relacionada con fraude o cargo no reconocido, es razonable esperar que los resultados principales pertenezcan con frecuencia a categorías relacionadas, aunque no es una garantía absoluta.

**Verificación:**

```python
assert len(similares) == 5
assert similares["similitud_coseno"].between(0, 1).all()
assert (similares["distancia_euclidiana"] >= 0).all()
assert (similares["distancia_coseno"] >= 0).all()

print("Búsqueda por similitud validada.")
```

---

### Paso 10. Visualizar el corpus con PCA

**Objetivo:** reducir una representación TF-IDF a dos componentes para visualizar agrupamientos aproximados de categorías y la posición de una consulta.

**Instrucciones:**

1. PCA requiere una matriz densa. Para evitar un consumo excesivo de memoria, limita el número de características y, si es necesario, toma una muestra reproducible del corpus.

```python
max_caracteristicas_pca = min(
    CONFIG["pca_max_features"],
    X_corpus_tfidf.shape[1]
)

X_pca_base = X_corpus_tfidf[:, :max_caracteristicas_pca]

if X_pca_base.shape[0] > CONFIG["pca_max_samples"]:
    rng = np.random.default_rng(SEMILLA)
    indices_pca = np.sort(
        rng.choice(
            X_pca_base.shape[0],
            size=CONFIG["pca_max_samples"],
            replace=False
        )
    )
else:
    indices_pca = np.arange(X_pca_base.shape[0])

X_pca_denso = X_pca_base[indices_pca].toarray()
referencia_pca = referencia_busqueda.iloc[indices_pca].reset_index(drop=True)

print("Matriz densa para PCA:", X_pca_denso.shape)
```

2. Ajusta PCA con dos componentes:

```python
pca_model = PCA(n_components=2, random_state=SEMILLA)
coordenadas_pca = pca_model.fit_transform(X_pca_denso)

df_pca = referencia_pca.copy()
df_pca["pca_1"] = coordenadas_pca[:, 0]
df_pca["pca_2"] = coordenadas_pca[:, 1]

varianza_explicada = pca_model.explained_variance_ratio_.sum()
print(f"Varianza explicada por dos componentes: {varianza_explicada:.2%}")
```

3. Visualiza las categorías:

```python
plt.figure(figsize=(12, 8))
sns.scatterplot(
    data=df_pca,
    x="pca_1",
    y="pca_2",
    hue="category",
    alpha=0.75,
    s=55
)
plt.title("Mensajes bancarios en dos componentes PCA")
plt.xlabel("Componente principal 1")
plt.ylabel("Componente principal 2")
plt.legend(title="Categoría", bbox_to_anchor=(1.02, 1), loc="upper left")
plt.tight_layout()
plt.savefig(RUTA_REPORTES / "pca_corpus_categorias.png", dpi=150)
plt.show()
```

4. Visualiza una consulta junto con los mensajes recuperados:

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
    top_k=5
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
    legend=False
)

plt.scatter(
    puntos_similares["pca_1"],
    puntos_similares["pca_2"],
    s=130,
    facecolors="none",
    edgecolors="black",
    linewidths=1.5,
    label="Vecinos recuperados"
)

plt.scatter(
    coordenada_consulta[:, 0],
    coordenada_consulta[:, 1],
    marker="X",
    s=250,
    c="red",
    edgecolors="black",
    label="Consulta"
)

plt.title("Consulta y vecinos similares en espacio PCA")
plt.xlabel("Componente principal 1")
plt.ylabel("Componente principal 2")
plt.legend()
plt.tight_layout()
plt.savefig(RUTA_REPORTES / "pca_consulta_vecinos.png", dpi=150)
plt.show()
```

5. Interpreta el gráfico con precaución: PCA reduce muchos términos a dos dimensiones y puede perder información. La posición visual es una herramienta exploratoria; el orden oficial de recuperación debe basarse en las métricas calculadas sobre el espacio TF-IDF original.

**Resultado esperado:**

Debes obtener:

- Un gráfico de dispersión por categoría.
- Un gráfico con la consulta marcada y sus vecinos destacados.
- La proporción de varianza explicada por dos componentes.

**Verificación:**

```python
assert coordenadas_pca.shape[1] == 2
assert coordenada_consulta.shape == (1, 2)
assert (RUTA_REPORTES / "pca_corpus_categorias.png").exists()

print("Visualización PCA validada.")
```

---

### Paso 11. Reentrenar el clasificador final y persistir artefactos

**Objetivo:** reentrenar el modelo elegido con entrenamiento y validación, persistir componentes reutilizables y crear una función de consulta final.

**Instrucciones:**

1. Combina entrenamiento y validación. La prueba se mantiene separada y no se utiliza para el reentrenamiento final.

```python
X_train_final = pd.concat([X_train, X_val])
y_train_final = pd.concat([y_train, y_val])

tfidf_vectorizer_final = TfidfVectorizer(
    ngram_range=tuple(CONFIG["ngram_range"]),
    min_df=CONFIG["min_df"],
    max_df=CONFIG["max_df"],
    max_features=CONFIG["max_features"],
    sublinear_tf=True,
    norm="l2"
)

X_train_final_tfidf = tfidf_vectorizer_final.fit_transform(X_train_final)
```

2. Reentrena el modelo seleccionado:

```python
if mejor_nombre == "MultinomialNB":
    classifier_final = MultinomialNB(alpha=1.0)
else:
    classifier_final = LogisticRegression(
        max_iter=2000,
        class_weight="balanced",
        random_state=SEMILLA
    )

classifier_final.fit(X_train_final_tfidf, y_train_final)

print(f"Modelo final reentrenado: {mejor_nombre}")
```

3. Guarda los artefactos obligatorios. El archivo de referencia de búsqueda contiene la matriz TF-IDF, el índice, los mensajes y metadatos mínimos necesarios para recuperar resultados sin recalcular el corpus.

```python
joblib.dump(
    tfidf_vectorizer_final,
    RUTA_ARTEFACTOS / "tfidf_vectorizer.joblib"
)

joblib.dump(
    classifier_final,
    RUTA_ARTEFACTOS / "classifier.joblib"
)

joblib.dump(
    {
        "vectorizer": tfidf_vectorizer_search,
        "tfidf_matrix": X_corpus_tfidf,
        "reference": referencia_busqueda,
        "nearest_neighbors": indice_vecinos,
        "top_k_default": CONFIG["top_k"]
    },
    RUTA_ARTEFACTOS / "search_reference.joblib"
)

joblib.dump(
    {
        "pca_model": pca_model,
        "max_features_used": max_caracteristicas_pca,
        "sample_indices": indices_pca,
        "explained_variance_ratio_sum": float(varianza_explicada)
    },
    RUTA_ARTEFACTOS / "pca_model.joblib"
)
```

4. Guarda métricas de prueba y configuración:

```python
metricas_completas = {
    "lab_id": "01-00-01",
    "modelo_seleccionado": mejor_nombre,
    "metricas_test": metricas_test,
    "tamano_train": int(len(X_train)),
    "tamano_validation": int(len(X_val)),
    "tamano_test": int(len(X_test)),
    "categorias": sorted(y.unique().tolist()),
    "random_state": SEMILLA
}

with open(RUTA_ARTEFACTOS / "metrics_test.json", "w", encoding="utf-8") as archivo:
    json.dump(metricas_completas, archivo, ensure_ascii=False, indent=2)

with open(RUTA_ARTEFACTOS / "config_used.json", "w", encoding="utf-8") as archivo:
    json.dump(CONFIG, archivo, ensure_ascii=False, indent=2)
```

5. Define la función final de predicción y búsqueda para un nuevo mensaje:

```python
def predecir_y_buscar(
    mensaje_nuevo: str,
    vectorizador_clasificacion,
    clasificador,
    recurso_busqueda,
    top_k: int = 5
):
    mensaje_normalizado = normalizar_mensaje(mensaje_nuevo)

    if not mensaje_normalizado:
        raise ValueError("El mensaje no contiene texto útil para procesar.")

    vector_clasificacion = vectorizador_clasificacion.transform(
        [mensaje_normalizado]
    )

    categoria_predicha = clasificador.predict(vector_clasificacion)[0]
    probabilidades = clasificador.predict_proba(vector_clasificacion)[0]
    confianza = float(probabilidades.max())

    similares = buscar_mensajes_similares(
        consulta=mensaje_nuevo,
        vectorizador=recurso_busqueda["vectorizer"],
        matriz_referencia=recurso_busqueda["tfidf_matrix"],
        referencia=recurso_busqueda["reference"],
        vecinos=recurso_busqueda["nearest_neighbors"],
        top_k=top_k
    )

    return {
        "mensaje_original": mensaje_nuevo,
        "mensaje_normalizado": mensaje_normalizado,
        "categoria_predicha": categoria_predicha,
        "confianza": confianza,
        "mensajes_similares": similares
    }
```

6. Prueba la experiencia final:

```python
recurso_busqueda = joblib.load(
    RUTA_ARTEFACTOS / "search_reference.joblib"
)

resultado_final = predecir_y_buscar(
    mensaje_nuevo="Aparece una compra en mi tarjeta que yo no hice",
    vectorizador_clasificacion=tfidf_vectorizer_final,
    clasificador=classifier_final,
    recurso_busqueda=recurso_busqueda,
    top_k=5
)

print("Categoría predicha:", resultado_final["categoria_predicha"])
print(f"Confianza: {resultado_final['confianza']:.2%}")
display(resultado_final["mensajes_similares"])
```

7. Registra una advertencia operativa en una celda Markdown del notebook:

> Una confianza alta es una puntuación del modelo, no una autorización para ejecutar automáticamente acciones de alto impacto. Mensajes relacionados con fraude, bloqueos, movimientos financieros o seguridad deben seguir reglas de negocio, umbrales, trazabilidad y revisión humana cuando corresponda.

**Resultado esperado:**

El directorio `artifacts/` debe contener al menos:

```text
artifacts/
├── classifier.joblib
├── config_used.json
├── metrics_test.json
├── pca_model.joblib
├── search_reference.joblib
└── tfidf_vectorizer.joblib
```

**Verificación:**

```python
artefactos_obligatorios = [
    "tfidf_vectorizer.joblib",
    "classifier.joblib",
    "search_reference.joblib",
    "pca_model.joblib",
    "metrics_test.json"
]

for nombre in artefactos_obligatorios:
    ruta = RUTA_ARTEFACTOS / nombre
    assert ruta.exists(), f"Falta el artefacto: {ruta}"

with open(RUTA_ARTEFACTOS / "metrics_test.json", encoding="utf-8") as archivo:
    metricas_cargadas = json.load(archivo)

assert "metricas_test" in metricas_cargadas
assert len(resultado_final["mensajes_similares"]) == 5

print("Persistencia y consulta final verificadas.")
```

---

## Validación y pruebas

Realiza la siguiente lista final antes de entregar el laboratorio:

- [ ] El notebook existe en `notebooks/01_clasificacion_busqueda_mensajes_bancarios.ipynb`.
- [ ] El CSV original está en `data/raw/mensajes_bancarios.csv`.
- [ ] El CSV procesado existe en `data/processed/mensajes_bancarios_procesados.csv`.
- [ ] Se conservan tanto `message_original` como `message_normalized`.
- [ ] La separación entrenamiento, validación y prueba usa `random_state=42` y estratificación.
- [ ] `CountVectorizer` y `TfidfVectorizer` de evaluación se ajustaron solamente con entrenamiento.
- [ ] Se compararon `MultinomialNB` y `LogisticRegression` mediante F1 macro de validación.
- [ ] Se generaron métricas, reporte de clasificación y matriz de confusión de prueba.
- [ ] Se analizaron al menos diez errores de prueba o todos los disponibles.
- [ ] La búsqueda devuelve cinco resultados con similitud coseno y distancia euclidiana.
- [ ] Se generaron visualizaciones PCA en `reports/`.
- [ ] Se guardaron todos los artefactos obligatorios en `artifacts/`.

Ejecuta además esta prueba de carga independiente en una nueva celda o sesión de kernel:

```python
from pathlib import Path
import joblib

ruta_artifacts = Path("/home/<usuario>/nlp-banca-lab/artifacts")

vectorizador_cargado = joblib.load(ruta_artifacts / "tfidf_vectorizer.joblib")
clasificador_cargado = joblib.load(ruta_artifacts / "classifier.joblib")
buscador_cargado = joblib.load(ruta_artifacts / "search_reference.joblib")

mensaje_prueba = "No puedo acceder a mi cuenta desde la aplicación"
mensaje_normalizado = normalizar_mensaje(mensaje_prueba)

vector = vectorizador_cargado.transform([mensaje_normalizado])
categoria = clasificador_cargado.predict(vector)[0]
confianza = clasificador_cargado.predict_proba(vector).max()

print("Categoría:", categoria)
print(f"Confianza: {confianza:.2%}")

resultado_busqueda = buscar_mensajes_similares(
    mensaje_prueba,
    buscador_cargado["vectorizer"],
    buscador_cargado["tfidf_matrix"],
    buscador_cargado["reference"],
    buscador_cargado["nearest_neighbors"],
    top_k=5
)

display(resultado_busqueda)
```

La prueba es satisfactoria si no requiere reentrenar modelos, devuelve una categoría válida, una confianza entre 0 y 1 y exactamente cinco mensajes similares.

## Solución de problemas

### Problema 1: falla la división estratificada con un error relacionado con la clase menos poblada

**Síntoma:**

Aparece un error similar a:

```text
ValueError: The least populated class in y has only 1 member
```

o alguna categoría no aparece en entrenamiento, validación o prueba.

**Causa:**

Una o más categorías contienen muy pocos mensajes para realizar dos divisiones estratificadas. Esto ocurre con frecuencia cuando una categoría tiene menos de 10 ejemplos o existen etiquetas con variaciones innecesarias, por ejemplo `fraude`, `Fraude` y `fraude `.

**Solución:**

1. Normaliza las etiquetas antes de dividir:

```python
df_limpio["category"] = (
    df_limpio["category"]
    .str.strip()
    .str.lower()
)
```

2. Revisa los conteos:

```python
display(df_limpio["category"].value_counts())
```

3. Recolecta más ejemplos para las clases escasas o agrupa categorías únicamente si existe una justificación operativa documentada.
4. Como mínimo didáctico, conserva 10 ejemplos por categoría; para una evaluación más estable, utiliza 20 o más por clase.

### Problema 2: PCA produce un error de memoria o el kernel se reinicia

**Síntoma:**

El notebook se detiene durante:

```python
X_pca_base.toarray()
```

o muestra un error similar a:

```text
MemoryError
```

**Causa:**

La matriz TF-IDF es dispersa, pero PCA necesita una matriz densa. Convertir un corpus grande con miles de mensajes y características a una matriz densa puede consumir más memoria de la disponible.

**Solución:**

Reduce de forma controlada la cantidad de filas y características usadas para visualización, sin modificar el índice TF-IDF de búsqueda:

```python
CONFIG["pca_max_features"] = 200
CONFIG["pca_max_samples"] = 500
```

Vuelve a ejecutar únicamente las celdas del Paso 10. La visualización PCA es exploratoria; el buscador debe seguir utilizando la matriz TF-IDF dispersa completa y no la representación reducida.

## Limpieza

**Objetivo:** detener procesos del entorno y conservar únicamente los artefactos necesarios para reproducibilidad.

1. Guarda el notebook desde JupyterLab:

```text
File > Save Notebook
```

2. Cierra el kernel desde JupyterLab:

```text
Kernel > Shut Down Kernel
```

3. En la terminal donde ejecutaste JupyterLab, presiona `Ctrl+C` y confirma el cierre si es solicitado.

4. Desactiva el entorno virtual:

```bash
deactivate
```

5. No elimines los siguientes directorios, ya que contienen la evidencia y artefactos del laboratorio:

```text
data/raw/
data/processed/
notebooks/
artifacts/
reports/
config/
```

6. Si debes liberar espacio de caché de paquetes, hazlo solo después de validar que el laboratorio está completo:

```bash
python -m pip cache purge
```

> No elimines `.venv` si necesitarás volver a ejecutar el notebook. El entorno virtual forma parte de la reproducibilidad local.

## Resumen

En este laboratorio implementaste un sistema reproducible de PLN para mensajes bancarios. Preparaste un conjunto de datos conservando texto original y normalizado, construiste representaciones bag-of-words y TF-IDF, y entrenaste modelos Multinomial Naive Bayes y regresión logística.

También evaluaste el modelo seleccionado con métricas de clasificación y análisis de errores, construiste una búsqueda de mensajes similares mediante similitud coseno, distancia euclidiana y vecinos más cercanos, y visualizaste el corpus con PCA. Finalmente, persististe el vectorizador, clasificador, índice de búsqueda, modelo PCA y métricas para realizar predicciones y consultas reproducibles.

Como siguiente mejora, considera ampliar el conjunto de datos, revisar etiquetas ambiguas, incorporar reglas de escalamiento para casos sensibles y evaluar representaciones vectoriales más avanzadas solo cuando exista una base de evaluación, privacidad y trazabilidad adecuada.

### Recursos opcionales

- [Documentación de scikit-learn: clasificación de texto](https://scikit-learn.org/stable/tutorial/text_analytics/working_with_text_data.html)
- [Documentación de TfidfVectorizer](https://scikit-learn.org/stable/modules/generated/sklearn.feature_extraction.text.TfidfVectorizer.html)
- [Documentación de NearestNeighbors](https://scikit-learn.org/stable/modules/generated/sklearn.neighbors.NearestNeighbors.html)
- [Documentación de PCA](https://scikit-learn.org/stable/modules/generated/sklearn.decomposition.PCA.html)
- [NIST SP 800-122: protección de información personal identificable](https://csrc.nist.gov/pubs/sp/800/122/final)
