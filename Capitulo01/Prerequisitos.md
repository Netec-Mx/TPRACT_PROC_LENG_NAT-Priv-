# Prerrequisitos del laboratorio

Este archivo lo usa **quien prepara el equipo** antes de la clase. Los estudiantes no lo ejecutan y no instalan programas, librerías ni datos.

Al terminar, cada equipo con Windows Server abre Visual Studio Code en la carpeta del laboratorio y sigue [README.md](README.md).

## Resultado de la preparación

El equipo queda así:

| Elemento | Ubicación |
|---|---|
| Carpeta del laboratorio | `C:\Laboratorio\nlp-banca-lab` |
| Mensajes de práctica | `C:\Laboratorio\nlp-banca-lab\data\raw\mensajes_bancarios.csv` |
| Configuración | `C:\Laboratorio\nlp-banca-lab\config\config.json` |
| Python del laboratorio | `C:\Laboratorio\nlp-banca-lab\.venv` |
| Palabras vacías en español | `C:\Laboratorio\nltk_data` |
| Cuaderno que crearán los estudiantes | `C:\Laboratorio\nlp-banca-lab\notebooks` |

Los mensajes son inventados. Sirven para clasificar seis tipos de solicitud y para buscar casos parecidos.

## Qué debe tener el equipo

| Recurso | Mínimo |
|---|---|
| Sistema | Windows Server, 64 bits |
| Procesador | 4 núcleos |
| Memoria | 8 GB (16 GB recomendados) |
| Espacio libre | 10 GB |
| Pantalla | 1366 x 768 (1920 x 1080 recomendada) |
| Internet | Solo durante esta preparación |

Prepara el equipo **con la cuenta que usarán los estudiantes**, o en la imagen base antes de clonarla. Si cada estudiante entra con otra cuenta, repite en esa cuenta la instalación de extensiones de Visual Studio Code.

Abre **PowerShell como administrador** para los pasos de instalación.

## 1. Instalar Visual Studio Code

1. Descarga el instalador de 64 bits para Windows: [Visual Studio Code](https://code.visualstudio.com/download).
2. Instálalo para todos los usuarios.
3. En el asistente, deja marcada la opción de agregar Visual Studio Code a PATH.
4. Ábrelo una vez y ciérralo.

### Qué haces

Instalas dos extensiones. La primera permite usar Python. La segunda permite abrir cuadernos dentro de Visual Studio Code. El estudiante trabaja en ese cuaderno.

**Resultado esperado:** la terminal muestra que cada extensión quedó instalada.

En PowerShell:

```powershell
code --install-extension ms-python.python --force
code --install-extension ms-toolsai.jupyter --force
```

## 2. Instalar Python 3.11.9

1. Descarga el instalador de 64 bits: [Python 3.11.9 para Windows](https://www.python.org/ftp/python/3.11.9/python-3.11.9-amd64.exe).
2. En la primera pantalla marca **Add python.exe to PATH**.
3. Elige **Install for all users**.
4. Si al final ofrece desactivar el límite de longitud de rutas, acepta.

### Qué haces

Compruebas que Windows reconoce la versión exacta que usa este laboratorio.

**Resultado esperado:**

```text
Python 3.11.9
```

En PowerShell:

```powershell
py -3.11 --version
```

## 3. Crear las carpetas

### Qué haces

Creas la carpeta del laboratorio, las carpetas internas y la carpeta donde quedarán las palabras vacías. `New-Item` crea la carpeta si todavía no existe.

**Resultado esperado:** el comando termina y no muestra un error. La última línea deja la terminal dentro de `C:\Laboratorio\nlp-banca-lab`.

En PowerShell:

```powershell
New-Item -ItemType Directory -Force -Path @(
    "C:\Laboratorio\nlp-banca-lab\data\raw",
    "C:\Laboratorio\nlp-banca-lab\data\processed",
    "C:\Laboratorio\nlp-banca-lab\notebooks",
    "C:\Laboratorio\nlp-banca-lab\artifacts",
    "C:\Laboratorio\nlp-banca-lab\reports",
    "C:\Laboratorio\nlp-banca-lab\src",
    "C:\Laboratorio\nlp-banca-lab\config",
    "C:\Laboratorio\nlp-banca-lab\.vscode",
    "C:\Laboratorio\preparacion",
    "C:\Laboratorio\nltk_data"
) | Out-Null

Set-Location C:\Laboratorio\nlp-banca-lab
Get-Location
```

## 4. Crear el entorno de Python e instalar librerías

En Visual Studio Code, crea el archivo `C:\Laboratorio\nlp-banca-lab\requirements.txt`, pega el contenido y guárdalo. Visual Studio Code lo guarda en UTF-8.

```text
ipykernel==6.29.5
numpy==1.26.4
pandas==2.2.2
scikit-learn==1.5.1
scipy==1.13.1
matplotlib==3.9.1
seaborn==0.13.2
nltk==3.8.1
joblib==1.4.2
```

El cuaderno se abre en Visual Studio Code.

### Qué haces

El primer comando crea un Python propio del laboratorio, separado del resto del equipo. El segundo deja `pip` en la versión 24.0. El tercero instala las librerías del archivo, con la versión indicada. Esas librerías leen tablas, limpian texto, convierten palabras en números, entrenan los dos clasificadores, miden resultados, buscan mensajes parecidos y dibujan los gráficos.

**Resultado esperado:** `pip` termina con la lista de librerías instaladas y sin líneas que digan `ERROR`.

En PowerShell:

```powershell
py -3.11 -m venv C:\Laboratorio\nlp-banca-lab\.venv

& "C:\Laboratorio\nlp-banca-lab\.venv\Scripts\python.exe" -m pip install --upgrade pip==24.0

& "C:\Laboratorio\nlp-banca-lab\.venv\Scripts\python.exe" -m pip install -r C:\Laboratorio\nlp-banca-lab\requirements.txt
```

### Qué haces

Registras el Python del laboratorio para que Visual Studio Code lo ofrezca con el nombre **Python 3.11 - NLP Banca Lab**. El registro queda dentro de `.venv`, así varias cuentas del servidor pueden elegir ese mismo Python.

**Resultado esperado:** una línea que indica que el kernel quedó instalado.

En PowerShell:

```powershell
& "C:\Laboratorio\nlp-banca-lab\.venv\Scripts\python.exe" -m ipykernel install --sys-prefix --name nlp-banca-lab --display-name "Python 3.11 - NLP Banca Lab"
```

### Qué haces

Guardas la lista real de versiones y la versión de Python. Esos archivos sirven para comprobar que el equipo quedó igual al de los demás.

**Resultado esperado:** se crean dos archivos en `reports`. El segundo contiene `Python 3.11.9`.

En PowerShell:

```powershell
& "C:\Laboratorio\nlp-banca-lab\.venv\Scripts\python.exe" -m pip freeze > C:\Laboratorio\nlp-banca-lab\reports\requirements_instalados.txt

& "C:\Laboratorio\nlp-banca-lab\.venv\Scripts\python.exe" --version > C:\Laboratorio\nlp-banca-lab\reports\version_python.txt

Get-Content C:\Laboratorio\nlp-banca-lab\reports\version_python.txt
```

## 5. Dejar listas las palabras vacías en español

### Qué haces

Indicas una carpeta común para las listas de palabras de NLTK y descargas la lista de palabras vacías en español (palabras como «el», «la» y «de»). El taller las usa al limpiar los mensajes. La variable queda a nivel del equipo para que las cuentas de estudiantes la encuentren. Este comando se ejecuta en PowerShell **como administrador**.

**Resultado esperado:** el texto `True` o un mensaje de NLTK que indica que el paquete `stopwords` quedó descargado.

En PowerShell:

```powershell
$env:NLTK_DATA = "C:\Laboratorio\nltk_data"
[System.Environment]::SetEnvironmentVariable("NLTK_DATA", $env:NLTK_DATA, "Machine")

& "C:\Laboratorio\nlp-banca-lab\.venv\Scripts\python.exe" -c "import nltk; print(nltk.download('stopwords', download_dir=r'C:\Laboratorio\nltk_data'))"
```

Cierra Visual Studio Code si estaba abierto y ábrelo de nuevo después de este paso, para que tome la variable.

## 6. Decirle a Visual Studio Code qué Python usar

En Visual Studio Code, crea el archivo `C:\Laboratorio\nlp-banca-lab\.vscode\settings.json`, pega este contenido y guárdalo:

```json
{
  "python.defaultInterpreterPath": "C:\\Laboratorio\\nlp-banca-lab\\.venv\\Scripts\\python.exe",
  "python.terminal.activateEnvironment": false,
  "terminal.integrated.defaultProfile.windows": "PowerShell"
}
```

Con esto, al abrir la carpeta del laboratorio, el cuaderno usa el Python ya preparado. El estudiante elige el kernel **Python 3.11 - NLP Banca Lab** si Visual Studio Code lo pregunta.

## 7. Crear la configuración y los mensajes de práctica

En Visual Studio Code, crea el archivo `C:\Laboratorio\preparacion\crear_material.py`, pega el programa siguiente y guárdalo.

Ese programa escribe dos archivos:

- `config.json`: las reglas del ejercicio (qué parte se aparta para el examen, cuántos mensajes parecidos se buscan y dónde están los datos).
- `mensajes_bancarios.csv`: 180 mensajes útiles, 30 de cada categoría, más cuatro filas a propósito (un mensaje en blanco, uno que solo tiene palabras vacías y dos repetidos) para que en clase se vea la limpieza.

Las categorías son:

| Categoría | Significado |
|---|---|
| `cargo_no_reconocido` | Un cobro o una compra que la persona no reconoce |
| `transferencia_pendiente` | Un envío de dinero que todavía no llega |
| `problema_de_acceso` | No puede entrar a la aplicación o a la página |
| `bloqueo_tarjeta` | Perdió la tarjeta o pide bloquearla |
| `comision` | Una cuota o un cobro de manejo |
| `fraude` | Sospecha de que otra persona usó su producto |

```python
from pathlib import Path
import json
import pandas as pd

raiz = Path(r"C:\Laboratorio\nlp-banca-lab")

config = {
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
    "processed_path": "data/processed/mensajes_bancarios_procesados.csv",
}

(raiz / "config" / "config.json").write_text(
    json.dumps(config, ensure_ascii=False, indent=2),
    encoding="utf-8",
)

comercios = [
    "el supermercado", "la farmacia", "una tienda en línea", "el restaurante",
    "la estación de servicio", "una aplicación de transporte", "el almacén",
    "una página de ropa", "el cine", "una suscripción digital",
]
montos = [
    "veinte mil pesos", "cuarenta y cinco mil pesos", "ochenta mil pesos",
    "ciento veinte mil pesos", "quince mil pesos", "doscientos cincuenta mil pesos",
    "nueve mil pesos", "sesenta mil pesos", "treinta y cinco mil pesos",
    "quinientos mil pesos",
]
canales = [
    "la aplicación", "la página web", "la sucursal virtual", "el celular",
    "la oficina virtual",
]
destinatarios = [
    "mi hermano", "mi arrendador", "un proveedor", "la universidad", "mi mamá",
    "un amigo", "mi hijo", "un comercio", "mi cuenta de ahorros", "un familiar",
]

plantillas = {
    "cargo_no_reconocido": [
        "No reconozco un pago de {m} realizado en {c}",
        "Aparece una compra con mi tarjeta en {c} por {m} que yo no autoricé",
        "Hay un cargo desconocido de {m} y no sé de qué se trata",
        "Vi un débito en mi tarjeta por {m} en {c} y no fui yo",
        "Me descontaron {m} y no recuerdo esa compra en {c}",
        "No acepto el cobro de {m} que figura en {c}",
    ],
    "transferencia_pendiente": [
        "La transferencia de {m} enviada a {d} por {canal} sigue pendiente",
        "Envié {m} a {d} y el dinero todavía no llega a {canal}",
        "El giro de {m} para {d} aparece pendiente desde ayer en {canal}",
        "Hice una transferencia de {m} a {d} y continúa en proceso",
        "El envío de {m} hacia {d} no se ha completado en {canal}",
        "Mi transferencia de {m} a {d} no ha llegado",
    ],
    "problema_de_acceso": [
        "No puedo iniciar sesión en {canal}",
        "Olvidé la clave y no logro entrar a {canal}",
        "La aplicación no abre y no puedo ingresar por {canal}",
        "El código de verificación no llega y no entro a {canal}",
        "Mi usuario no funciona para entrar a {canal}",
        "Intento ingresar a {canal} y la página rechaza la clave",
    ],
    "bloqueo_tarjeta": [
        "Quiero bloquear mi tarjeta porque la perdí. Lo reporto en {canal}",
        "Me robaron la tarjeta y necesito bloquearla ya desde {canal}",
        "Perdí la tarjeta de débito y pido el bloqueo por {canal}",
        "Solicito bloquear la tarjeta de crédito. La extravié y aviso por {canal}",
        "No encuentro mi tarjeta y quiero que quede bloqueada en {canal}",
        "Por favor bloquea mi tarjeta. La perdí y escribo desde {canal}",
    ],
    "comision": [
        "Por qué me cobraron una comisión de {m} en {canal}",
        "Aparece una cuota de manejo de {m} y quiero la razón del cobro",
        "Me descontaron una comisión de mantenimiento de {m} en {canal}",
        "No entiendo el cobro de comisión por {m} que veo en {canal}",
        "Quiero saber por qué hay una cuota mensual de {m}",
        "La comisión de {m} en {canal} no estaba en lo que yo esperaba pagar",
    ],
    "fraude": [
        "Creo que hay fraude: aparece un movimiento de {m} en {c} que no realicé",
        "Sospecho suplantación. Hay un cargo de {m} en {c} y yo no fui",
        "Alerta de seguridad: un movimiento raro de {m} en {c}",
        "Alguien más pudo usar mi producto. Veo {m} en {c}",
        "Reporto un posible fraude por {m} registrado en {c}",
        "Hay movimientos extraños de {m} en {c} y temo que sea fraude",
    ],
}

ambiguos = {
    "cargo_no_reconocido": [
        "No reconozco un cobro y tampoco sé si entraron a mi cuenta",
        "Veo un cargo en la tarjeta y la aplicación no me deja revisar",
        "Hay un débito raro y quiero que revisen si es comisión o fraude",
    ],
    "transferencia_pendiente": [
        "La transferencia no llega y además veo un cobro que no ubico",
        "Envié el dinero, sigue pendiente y no puedo entrar a mirar el estado",
        "El giro está pendiente y la tarjeta aparece con un movimiento extraño",
    ],
    "problema_de_acceso": [
        "No puedo entrar a la aplicación y veo un cargo que no reconozco",
        "La clave no sirve y quiero bloquear la tarjeta por si acaso",
        "No ingreso al banco y hay una comisión nueva en la cuenta",
    ],
    "bloqueo_tarjeta": [
        "Bloqueen la tarjeta porque hay un cargo que no reconozco",
        "Perdí la tarjeta y también falló el ingreso a la aplicación",
        "Quiero bloquear la tarjeta: hay un movimiento que parece fraude",
    ],
    "comision": [
        "Me cobraron una comisión y no reconozco ese valor en la tarjeta",
        "La cuota de manejo parece un cargo fraudulento",
        "Por qué hay una comisión si la transferencia ni siquiera ha llegado",
    ],
    "fraude": [
        "No reconozco un pago y creo que otra persona entró a la cuenta",
        "Hay un movimiento extraño y no puedo iniciar sesión",
        "Sospecho fraude en la tarjeta y quiero bloquearla",
    ],
}

filas = []
for i in range(27):
    datos = {
        "m": montos[i % len(montos)],
        "c": comercios[i % len(comercios)],
        "canal": canales[i % len(canales)],
        "d": destinatarios[i % len(destinatarios)],
    }
    for categoria, textos in plantillas.items():
        filas.append({
            "message": textos[i % len(textos)].format(**datos),
            "category": categoria,
        })

for categoria, textos in ambiguos.items():
    for texto in textos:
        filas.append({"message": texto, "category": categoria})

filas.append(filas[0].copy())
filas.append(filas[27].copy())
filas.append({"message": "", "category": "cargo_no_reconocido"})
filas.append({
    "message": "el la de y que en con por",
    "category": "problema_de_acceso",
})

tabla = pd.DataFrame(filas)
tabla.insert(0, "message_id", range(1, len(tabla) + 1))
destino = raiz / "data" / "raw" / "mensajes_bancarios.csv"
tabla.to_csv(destino, index=False, encoding="utf-8")

print(f"Filas escritas: {len(tabla)}")
print(tabla["category"].value_counts().to_string())
print(f"Configuracion: {raiz / 'config' / 'config.json'}")
print(f"Mensajes: {destino}")
```

### Qué haces

Ejecutas ese programa con el Python del laboratorio. Crea la configuración y el archivo de mensajes en las carpetas que el taller espera.

**Resultado esperado:**

```text
Filas escritas: 184
```

Los conteos quedan así: `cargo_no_reconocido` 32, `bloqueo_tarjeta` 31, `problema_de_acceso` 31 y las otras tres categorías en 30. La suma es 184 porque incluye el mensaje en blanco, el mensaje de palabras vacías y dos repetidos. Abre el CSV en Visual Studio Code para ver las tildes.

En PowerShell:

```powershell
& "C:\Laboratorio\nlp-banca-lab\.venv\Scripts\python.exe" C:\Laboratorio\preparacion\crear_material.py
```

## 8. Permisos para las cuentas de la clase

### Qué haces

Das a los usuarios del servidor permiso para guardar el cuaderno, los reportes y los archivos que produce el taller. La carpeta `.venv` se conserva: ahí viven las librerías.

**Resultado esperado:** `icacls` confirma que procesó los archivos, sin un error de acceso.

En PowerShell como administrador:

```powershell
icacls C:\Laboratorio\nlp-banca-lab /grant "Users:(OI)(CI)M" /T
icacls C:\Laboratorio\nltk_data /grant "Users:(OI)(CI)RX" /T
```

## 9. Comprobar que el equipo quedó listo

Cierra PowerShell, ábrelo de nuevo y entra con una cuenta de estudiante si la clase usa varias cuentas.

### Qué haces

Compruebas versiones, la lectura del archivo de mensajes y que la lista de palabras vacías abre correctamente. `python -c` ejecuta una comprobación corta y termina.

**Resultado esperado:** aparecen las versiones `1.5.1`, `2.2.2` y `1.26.4`, la frase `Palabras vacias: listo` y las primeras líneas del CSV con columnas `message_id`, `message` y `category`.

En PowerShell:

```powershell
& "C:\Laboratorio\nlp-banca-lab\.venv\Scripts\python.exe" -c "import os; os.environ['NLTK_DATA']=r'C:\Laboratorio\nltk_data'; import sklearn, pandas, numpy; from nltk.corpus import stopwords; palabras=stopwords.words('spanish'); assert 'no' in palabras; print(sklearn.__version__, pandas.__version__, numpy.__version__); print('Palabras vacias: listo')"

Get-Content C:\Laboratorio\nlp-banca-lab\data\raw\mensajes_bancarios.csv -TotalCount 3 -Encoding UTF8
```

Abre Visual Studio Code, elige **Archivo > Abrir carpeta** y selecciona `C:\Laboratorio\nlp-banca-lab`. Crea un cuaderno de prueba y elige el kernel **Python 3.11 - NLP Banca Lab**.

### Qué haces

Esta celda comprueba que el cuaderno usa el Python del laboratorio. Imprime la versión de scikit-learn, la librería con la que se clasifica y se busca.

**Resultado esperado:** `1.5.1`. Después borra el cuaderno de prueba para que los estudiantes creen el suyo en `notebooks`.

```python
import os
os.environ["NLTK_DATA"] = r"C:\Laboratorio\nltk_data"
import nltk
if r"C:\Laboratorio\nltk_data" not in nltk.data.path:
    nltk.data.path.insert(0, r"C:\Laboratorio\nltk_data")
import sklearn
print(sklearn.__version__)
```

## Durante la clase

Los estudiantes abren `C:\Laboratorio\nlp-banca-lab` en Visual Studio Code y siguen el [README.md](README.md). Si falta el kernel, el archivo de mensajes o una librería, se corrige en el equipo con esta guía. El estudiante sigue en el cuaderno.
