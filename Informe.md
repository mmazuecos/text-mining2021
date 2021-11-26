# Tipificación de preguntas para mejorar un clasificador automático de preguntas

Informe del proyecto final de la materia de Text Mining 2021 de FAMAF-UNC.

## 1) Introducción

El dataset de [GuessWhat?!](https://arxiv.org/abs/1611.08481) contiene diálogos orientados a tarea contextualizados en imágenes. La tarea que se busca
resolver es la resolución de referencias. El problema está planteado como un juego de adivinanzas en el que dos jugadores, un *Questioner* que debe
hacer preguntas de tipo "sí/no" y un *Oracle* que las responde deben mantener un diálogo para tratar de identificar un objeto en una imagen (la identidad
de este objeto es conocida por el *Oracle* al principio del juego).

[Shekhar et al., (2019)](https://aclanthology.org/N19-1265/) propusieron una clasificación de preguntas para poder distinguir y diagnosticar modelos de
*Questioner* y sus capacidades lingüísticas. Esta clasificación, si bien útil, está basada en buscar *keywords* dentro de preguntas. Esto lleva a que haya
preguntas como *'is it red in color?'* que sean catalogadas como pregunta de 'color' y 'espacial'. Esta clasificación hace que diagnosticar qué capacidades
necesita un modelo de *Oracle* para responder ciertas preguntas se vea afectada por errores de clasificación.

En este proyecto se buscó entrenar un modelo no supervisado que pueda separar las preguntas en distintos clusters que sirvan para este análisis. Los resultados
no fueron prometedores y eso es un indicio de que muchos de los métodos propuestos no son aptos para agrupar las preguntas más allá de su forma.

## 2) Dataset y pregunta de investigación

El conjunto de datos de GuessWhat?! contiene los diálogos contextualizados en imágenes. Como en este trabajo se trabajó a nivel pregunta, un dato de este
conjunto tiene esta forma:

```
game_id                                        9225
answer                                          Yes
question                is it an electronic device?
dial_pos                                          6
question_id                                   30152
status                                      success
questioner_id                                  3118
target_object_id                            1111525
file_name           COCO_train2014_000000389775.jpg
qtype                                      <object>
num_objs                                          8
```

donde *game_id* es el identificador único para el juego, *question_id* el identificador único para esa pregunta, *target_object* el identificador único
para el objeto a adivinar, *status* cómo terminó el juego (puede ser *success*, *failure* o *incomplete*) y *qtype* el tipo de pregunta que el previo clasificador
le asigna a esa pregunta.

La pregunta de investigación fué: hay información en la forma de la pregunta que permita tipificarlas?

## 3) Metodología

Para el proyecto se usó *KMeans*, un algoritmo de clusterización que busca generar centroides que puedan usarse para diferenciar clusters minimizando su
distancia de cada dato a su correspondiente cluster. El método no requiere supervisión y es útil para realizar exploración o clasificación cuando no hay etiquetas
de supervisión.

Para la representación de los datos se probaron 3 tipos de embeddings:

- Embeddings basados en matriz de ocurrencia + reducción de dimensionalidad
- Word embeddings agregados
- Embeddings contextuales

Para el primer tipo de embeddings se computó una matriz de ocurrencia de unigramas. Esa matriz es luego reducida de dimensionalidad a 50 dimensiones usando SVD.
El resultado es un vector de 50 dimensiones que representa a la pregunta original. Las deciciones de usar unigramas y 50 dimensiones fué principalmente por el 
poder de cómputo disponible.

En el segundo tipo de embeddings se emplearon word embeddings de [GLoVE](https://nlp.stanford.edu/projects/glove/) de 50 dimensiones. A estos embeddings se les
agregar un vector aleatorio para palabras desconocidas. Luego, se transforman las oraciones a una lista de embeddings, estos se agregan y promedian resultando en
un vector de 50 dimensiones que tiene la representación de la pregunta.

El tercer y último tipo de embeddings son embeddings contextuales computados empleando BERT. Estos embeddings tienen 768 dimensiones que luego son reducidas a 50
para mantener la tratabilidad del problema.


