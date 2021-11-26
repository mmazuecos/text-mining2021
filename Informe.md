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

# 4) Experimentos y Resultados

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
  
## 5) Conclusiones
  
En este trabajo se empleó técnicas no supervisadas para intentar obtener una mejor clasificación de las preguntas en el conjunto de datos de GuessWhat?!. Los resultados de algunos métodos mostraron tipos de preguntas interesantes pero en su mayoría el algoritmo estaba agrupando las preguntas por su forma sintáctica. Esto puede ser un indicio de que hay información comunicada que escapa a la forma de la pregunta si se quiere objener una clasificación de preguntas por la información que solicitan o por la intención comunicativa.
  
Trabajo futuro sería, dada una clasificacion confiable del tipo de pregunta, entrenar un clasificador de las mismas de modo supervisado y posteriormente un modelo de *Questioner* que pueda explicar su estrategia de preguntas mostrando qué tipo de pregunta considera más probable realizar en cada turno de diálogo.
