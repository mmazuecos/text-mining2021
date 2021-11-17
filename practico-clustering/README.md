# Práctico Clustering
Para la materia [Text Mining 2021](https://sites.google.com/unc.edu.ar/textmining2021/inicio?authuser=0).

## Introducción
El práctico se basa en caracterización de las palabras dentro de un corpus usando herramientas de clustering. Para el corpus empleé los diálogos de [GuessWhat?!](https://arxiv.org/abs/1611.08481).

GuessWhat?! es un juego en el que dos jugadores, un *questioner* que realiza preguntas del tipo "sí/no" y un *oracle* que las responde, mantienen un diálogo cuyo objetivo es identificar un objeto secreto en la imagen. Es un corpus de diálogo visual orientado a tareas y todas las preguntas son exofóricas con respecto al objeto a adivinar. El corpus fué anotado por hablantes no nativos, asi que hay muchas preguntas que muestran inglés no nativo y el setting de diálogo hace que muchos "errores" de ortografía se hagan presentes.

## Procedimiento
Para extraer informacion del texto empleé [Stanza](https://stanfordnlp.github.io/stanza/), una herramienta de PLN basada en redes neuronales.

El procedimiento fué el siguiente:

1. Preprocesar los datos con Stanza.
2. Crear un vocabulario y diccionario de contextos.
3. Contar los contextos en los que ocurre cada palabra usando las dependencias extraidas por Stanza.
4. Mantener los contextos más frecuentes.
5. Vectorizar con respecto a la ocurrencia de los contextos más frecuentes.
6. Normalizar, aplicar LSA, Clusterizar y visualizar los clusters.
7. Analisar un subset de palabras.

Se usó el hardware del CCAD, en particular Nabucodonosor2 para los experimentos. Stanza corrió en una 1080ti. Solo se requirió una pasada por todo el dataset.
Para el punto 6 se usó la librería Scikit Learn. Se mantuvo 50 dimensiones para el espacio de representación de los vectores y se emplearon 20 clusters. La visualización fué generada con t-SNE.

Luego se hizo un análisis cualitativo de los clusters generados para ver si alguno estaba capturando características importantes del corpus.

Se hizo 3 experimentos:
1. Usar triplas de dependencia y contextos que ocurrieran más de 10 veces.
2. Usar triplas de dependencia y contextos que ocurrieran más de 100 veces.
3. Usar co ocurrencia.

El primero mostró que adjetivos y plurales suelen estar en sus propios clusters, pero las nociones de categoría y super categoría no tenian un cluster propio.

El segundo experimento apuntó a ver eso (ya que estas palabras suelen ser la raíz de una oración). Si bien habian clusters que podian considerarse como "clusters de categoría" seguian presentando mucho ruido dentro de lo que se veía.

El tercer experimento no aportó nada interesante. La co ocurrencia solo generó un enorme blob que contiene a todas las palabras en un mismo cluster.

## Resultados y conclusiones
No se pudo encontrar las features estuvieran capturando la noción de categoría y supercategoría muy empleada en la literatura del area para caracterizar palabras. Sí se pudo ver que cosas como adjetivos, numerales y plurales eran muy sobresalientes y usualmente tenian sus propios clusters dedicados. El corpus fué anotado principalmente por hablantes no nativos, por lo que muchas veces la sintaxis o la forma de las palabras escapa a la normalidad. Sin embargo se puede ver en algunos clusters que las mismas palabras escritas de maneras diferentes caen en un mismo cluster. Esto es atribuible a que los contextos son los mismos a pesar de que las palabras no sean iguales en forma.

A futuro es interesante emplear triplas de dependencia a nivel pregunta para clasificarlas de acuerdo al tipo de información por el que preguntas (por ejemplo, preguntas de *color*, *ubicación*, *categoría*, etc).
