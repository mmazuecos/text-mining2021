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
poder de cómputo disponible. Tambien se probó usar los one-hot provenientes de la matriz de ocurrencia como features.

En el segundo tipo de embeddings se emplearon word embeddings de [GLoVE](https://nlp.stanford.edu/projects/glove/) de 50 y 300 dimensiones. A estos embeddings se les
agregar un vector aleatorio para palabras desconocidas. Luego, se transforman las oraciones a una lista de embeddings, estos se agregan y promedian resultando en
un vector de 50 dimensiones que tiene la representación de la pregunta.

El tercer y último tipo de embeddings son embeddings contextuales computados empleando BERT. Se hicieron dos experimentos: uno reduciendo la dimensionalidad a 50 dimensiones y otro manteniendo las 768 dimensiones del embedding original.

# 4) Experimentos y Resultados

Para evaluar los clusters obtenidos se usaron matrices de confución y la métrica de pureza ([Purity](https://nlp.stanford.edu/IR-book/html/htmledition/evaluation-of-clustering-1.html)). Para cada uno de los experimentos generamos una matriz de confución *Question Type x Cluster* para ver cómo estos agrupan las preguntas y retornamos su puntaje de pureza.

|      Feature       |       Pureza       |
|:------------------:|:------------------:|
|   Ocurrencia 50d   |  0.5654            |
| Ocurrencia One-hot | 0.5504             |
|      Glove 50d     |  0.6345            |
|     Glove 300d     | 0.6236             |
|    BERT Embs 50d   | 0.5643             |
| BERT Embs          | 0.5487             |

En general la pureza de los clusters no es muy alta. Los mejores resultados se obtienen con los word embeddings agregados de Glove, donde los clusters son un poco más especializados.

![image](https://user-images.githubusercontent.com/8678939/144768457-c7f1741b-b3d9-4a2f-96b3-9d5eb7b2a4a1.png)

En la matriz de confusion del modelo que usar Glove 50d se ve que los tipos de preguntas se distribuyen entre todos los clusters, con más o menos presencia de algunas de las clases. Es importante notar que la tipificación es una tarea de clasificación multiclase y, por lo tanto, una pregunta se cuenta en una vez para cada uno de sus tipos de pregunta (i.e. si una pregunta es de tipo color y spatial, se cuenta en ambas filas). En esta matriz se ve una presencia más marcada de las preguntas de objeto en el cluster 6, mientras que otros modelos suelen distribuirlas casi uniformemente entre todos los clusters.

![image](https://user-images.githubusercontent.com/8678939/144768537-6200456f-c8b6-4b39-a38a-fe47d21abf80.png)

En esta matriz de confusion se muestran los resultados usando BERT Embs 50d. Las preguntas de color, spatial y object son distribuidas mas uniformemente entre los distintos clusters.

# 4.1) Análisis Cualitativo de Clusters

Para el primer experimento se usaron embeddings basados en matriz de ocurrencia. Luego de ajustar un modelo de KMeans se observaron manualmente los clusters generados.
Principalmente, los clusters mostraban un agrupamiento por la forma de la oración más allá de su tipo de pregunta o algún tipo emergente de pregunta:

```
Cluster 0
                            question  distance                                  qtype  
60490       is it man's suit jacket?  0.090877                            ['<color>']
37133   is it paisley pattern cloth?  0.092710                                <other>
5580        it is non living things?  0.092736                          ['<spatial>']
36899      is it like small glasses?  0.093030                               <object> 
3455         is it giving off smoke?  0.093278                          ['<spatial>']
9960           is it fish floor mat?  0.093372                       <super-category>
69627      is it shorter then jeans?  0.093413  ['<action>', '<action>', '<spatial>']
78991      is it over someones head?  0.093718                       <super-category>
69660  is it shiny metallic colored?  0.093828                          ['<spatial>']
16892           is it used for fire?  0.093895               ['<action>', '<action>']

Cluster 5
                              question  distance                     qtype
2449        does it protect your eyes?  0.176699                  <object>
19110         does it look like human?  0.177666             ['<spatial>']
58973      does it look like clothing?  0.178193             ['<spatial>']
2450     does it keep your hands warm?  0.178687  ['<action>', '<action>']
26315          does it look like wood?  0.179186  ['<color>', '<spatial>']
99720      does it go round your neck?  0.179453                  <object>
266           does it carry many kids?  0.179454               ['<color>']
3373      does it go around your neck?  0.179542                   <other>
34566   does it look like small grape?  0.179646             ['<spatial>']
108575          does it say 3 o'clock?  0.179844               ['<color>']
```

El siguiente experimento implicó usar word embeddings, pero los resultados fueron similares. Sin embargo, uno de los clusters (el 16) parece capturar un tipo de
pregunta acerca de orden dentro de un agrupamiento de objetos en la imagen. Este tipo de preguntas se suele denominar preguntas de grupo y fueron estudiadas
también en [trabajo previo](https://www.aclweb.org/anthology/2020.splu-1.4/).

```
Cluster 0
                        question  distance                     qtype
72396   is it on right..red one?  0.096583                  <object>
97335    is it on top...red one?  0.096583  ['<color>', '<spatial>']
93454       is it in ladys hand?  0.119337          <super-category>
56041       is it in man's hand?  0.119337          <super-category>
25483    is it in anyone's hand?  0.119337          <super-category>
56042     is it in women's hand?  0.119337                  <object>
66499   is it in someone's hand?  0.119337          <super-category>
51917  is it the one...leftmost?  0.121726             ['<spatial>']
91430      is it the tea/coffee?  0.121726               ['<color>']
40686  is it the table...itself?  0.121726               ['<color>']

Cluster 16

                        question  distance                                  qtype
59388     is it 1st one in back?  0.145799                               <object>  
82673        2nd one from right?  0.146202  ['<action>', '<action>', '<spatial>']
74477          is 2nd from left?  0.146218                          ['<spatial>']
33853    is it 2nd row from top?  0.147468                               <object>
54936   is it 3rd one from left?  0.149343               ['<action>', '<action>']
98871    is it the 2nd one down?  0.149889                       <super-category> 
3881        is it 2nd from left?  0.150107                               <object>
98844          is 4th from left?  0.150454                            ['<color>']
77358        is it 2nd from top?  0.150704               ['<action>', '<action>']  
102172  is the 2nd 1 from right?  0.152405                          ['<spatial>']
```

Finalmente se emplearon los embeddings contextuales y los resultados no variaron mucho pero mostraron mejores resultados con respecto a los tipos de preguntas
que terminabas agrupadas. En los siguientes ejemplos se ven ejemplos de clusters que agrupan preguntas de tipo color

```
Cluster 1
                             question  distance  
88067       is it the jet blue plane?  0.055574                                  <super-category>
14145   is it the red and blue thing?  0.057499  ['<color>', '<action>', '<action>', '<spatial>']
96335   is the blue piece really big?  0.058215                                  <super-category>
89167            is it the white bar?  0.059133                                     ['<spatial>']
70862             is it the near one?  0.059142                                     ['<spatial>']
16185  is it the black and white one?  0.059268                                           <other>
38942            is it a glass plate?  0.059307                                          <object>
51545             is it a blue chair?  0.060179                                     ['<spatial>']
5738              is it a scat board?  0.060455                                     ['<spatial>']
80870            is it a white color?  0.061125                                           <other>
```

Otro experimento que se llevó a cabo fué mirar solo el conjunto de preguntas que se clasificaban como "other". Estas son preguntas que no contenian keywords de las usadas para clasificar las preguntas. Este experimento fué interesante porque permitió encontrar nuevas preguntas de objeto y encontrar un cluster que parece preguntar por capacidades (sea del otro jugador o del objecto a adivinar). Para este experimento se usaron word embeddings.

```
Cluster 0
                   question  distance    qtype
27135     is it the butter?  0.141605  <other>
34151       is it a potato?  0.150702  <other>
99036      is it the pasta?  0.151131  <other>
67347      is it a pumpkin?  0.151912  <other>
12592     is it the cheese?  0.152723  <other>
59813    is it the chicken?  0.153262  <other>
88490    it is the chicken?  0.153262  <other>
100094     is it a chicken?  0.157563  <other>
28661   is it a baked good?  0.158232  <other>
62121      is it the lemon?  0.158707  <other>

Cluster 9
                              question  distance    qtype
10424           can you see all of it?  0.108983  <other>
95372                 can i go for it?  0.129123  <other>
13469          can you see most of it?  0.129953  <other>
116117            can i see all of it?  0.130363  <other>
28439          can you see all of him?  0.130392  <other>
77182             can you see it face?  0.131315  <other>
109396  can you easily see what it is?  0.132339  <other>
2311           can you see through it?  0.135071  <other>
86291       can you just see the face?  0.137291  <other>
51683   can you see clearly who is it?  0.137365  <other>
```

Finalmente se intentó agregar el tipo de la pregunta como informacion adicional a cada embedding de pregunta. A las 50 features de los word embeddings se le
concatenaron vectores de 8 dimensiones que tienen un 1 en la posición correspondiente a su categoría de acuerdo al script previo de clasificación y 0
en las otras posiciones. La forma del vector es la siguiente: \['<color>', '<shape>', '<spatial>', 'other', '<action>', 'object', '<texture>', '<size>'].
  
Con estos nuevos vectores de 58 dimensiones se intentó ver si los resultados mejoraban. 

![image](https://user-images.githubusercontent.com/8678939/143623860-efe55b8a-a7ec-42f5-b255-df0bd4c6a574.png)
  
Si bien las separaciones se ven más marcadas aplicando t-SNE, los resultados fueron análogos al experimento con word embeddings. Esto muestra que el tipo de pregunta no es un importante indicador o que la representación elegida no fué suficiente.
  
### 4.2) Clasificadores neuronales supervisados

   Además de los modelos basados en clustering se entrenaron unos clasificadores basados en redes neuronales. Cada pregunta tiene un vector de 8 dimensiones como en la sección anterior que juega el rol de supervisión. Con esto podemos plantear el problema de clasificación como un problema de clasificación multi-etiqueta. Cada clasificador debe retornar un puntaje por cada uno de los tipos de pregunta para decir si la pregunta pertenece a cada una de esas categorías. Se prueban dos tipos de redes: uno que usa los features del Glove 50d, ya que su mayor score de pureza facilita la separabilidad para el modelo; el segundo simplemente usa embeddings de glove de 50 dimensiones con una GRU.
  Cada modelo entrena por 5 épocas sobre el conjunto de datos de entrenamiento del GuessWhat?! optimizando la función de pérdida de [Entropía Cruzada Binaria](https://pytorch.org/docs/stable/generated/torch.nn.BCEWithLogitsLoss.html). Luego se calcula el [*average precision score*](https://scikit-learn.org/stable/modules/generated/sklearn.metrics.average_precision_score.html) que es un resumen de las métricas de precision y recall para distindos umbrales de clasificación.
  
|      Modelo        |          AVP       |
|:------------------:|:------------------:|
|   MLP + Glove 50d  |       0.622        |
|   GRU + word embs  |       0.744        |
  
  Usar GRU con la secuencia de palabras convertida a word embeddings retorna la mejor performance en la tarea de clasificación. Sin embargo la performance sigue estando lejos de ser satisfactoria para emplear el clasificador para clasificar cualquier pregunta que pueda surgir en un diálogo. Cabe aclarar que parte de esto se puede deber a la gran cantidad de preguntas clasificadas como "*other*" que pueden estar siendo mal clasificadas.
  
## 5) Conclusiones
  
En este trabajo se empleó técnicas no supervisadas para intentar obtener una mejor clasificación de las preguntas en el conjunto de datos de GuessWhat?!. Los resultados de algunos métodos mostraron tipos de preguntas interesantes pero en su mayoría los algoritmos basados en clustering agrupan las preguntas por su forma sintáctica. Esto puede ser un indicio de que hay información comunicada que escapa a la forma de la pregunta si se quiere objener una clasificación de preguntas por la información que solicitan o por la intención comunicativa.
  
En una segunda aproximación se emplearon redes neuronales (MLP y GRU) para la tarea de clasificación multietiqueta, usando la clasificación del script previo como supervisión. En este contexto, usar un modelo simple de RNNs y word embeddings mostró los mejores resultados.
  
Trabajo futuro sería entrenar un modelo de *Questioner* que pueda explicar su estrategia de preguntas mostrando qué tipo de pregunta considera más probable realizar en cada turno de diálogo.
