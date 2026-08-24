# Taller Práctico Procesamiento del lenguaje natural con clasificación y espacios vectoriales

En este taller, los participantes construirán una solución básica de procesamiento del lenguaje natural para analizar solicitudes, comentarios o mensajes recibidos por una institución bancaria. A partir de un conjunto de datos sintéticos, desarrollarán un clasificador que identifique si un mensaje expresa una experiencia positiva o negativa y complementarán la solución con un mecanismo de búsqueda que permita localizar textos similares dentro de una colección de casos previamente registrados.

La práctica integrará el preprocesamiento de texto, la extracción de características mediante frecuencias de palabras, la clasificación con regresión logística y Naive Bayes, la evaluación de predicciones y el análisis de errores. Posteriormente, los participantes representarán palabras o documentos mediante vectores y utilizarán similitud coseno, distancia euclidiana y vecinos más cercanos para recuperar solicitudes relacionadas. Como apoyo visual, podrán proyectar una selección de vectores en dos dimensiones mediante PCA.

## Estructura

- `CapituloXX/README.md`: guía de laboratorio por capítulo.

## Lista de laboratorios

### Capítulo 1

- [Clasificación y búsqueda de mensajes bancarios con NLP](Capitulo01/README.md#clasificación-y-búsqueda-de-mensajes-bancarios-con-nlp)
  - Descripción: Construir una solución de procesamiento del lenguaje natural que permita clasificar mensajes bancarios como positivos o negativos y localizar casos similares dentro de un conjunto de datos, aplicando preprocesamiento de texto, regresión logística, Naive Bayes, espacios vectoriales, similitud coseno y vecinos más cercanos.

Los participantes trabajarán en un notebook de Jupyter o Google Colab con un conjunto de mensajes sintéticos. Primero prepararán y transformarán los textos, luego entrenarán y compararán modelos de regresión logística y Naive Bayes. Finalmente, representarán los mensajes mediante vectores y utilizarán similitud coseno para recuperar los casos más relacionados con una nueva solicitud.
  - Duración estimada: 225 min

## Flujo de colaboración

- Trabajar en `changes_course`.
- Crear Pull Request hacia `main`.
- Merge por `Squash and merge`.
