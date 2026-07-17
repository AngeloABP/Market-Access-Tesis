# Market-Access-Tesis
Códigos utilizados en el trabajo de tesis de grado magíster en economía UC.
En la carpeta "Codes" se encuentran los notebooks utilizados para el trabajo de datos de la tesis. El orden de estimación es el siguiente:

## 1) networks_distance_finalversion
### Descripción: 
Notebook encargado de procesar los archivos geográficos regionales en formato .shp; verificar la consistencia de las redes tanto de manera individual como integrada; y generar las redes correspondientes a cada período (1970, 1986 y actualidad), calculando los tiempos de viaje asociados a cada camino de la red.

### Input:
* 16 redes regionales (nodos y aristas)
*  coeficientes de costos de transporte elaborados en el do-file "coeficientes de costo".

### Output:
Redes integradas para 1970, 1986 y la actualidad (nodos y aristas), con una columna que contiene el tiempo de transporte, medido en segundos, para cada camino.

## Importante:
Código utiliza API KEY google maps matrix, por lo cual necesita una cuenta de pago en Google Cloud para estimar variables duration y distance. 

## 2) Centros_de_gravedad_pob
### Descripción: 
Código encargado de trabajar islas de conectividad (trabajo manual en Qgis, se encuentran los caminos incongruentes según la cartografía histórica y se añaden a la base 1986). Posteriormente, a partír de la cartografía censal de 2024 se estiman los centros de gravedad poblacional que definen a las entidades comunales de 2024. Finalmente, se procede a unir estos nodos centro de gravedad mediante un proceso de homologación con el nodo de red en 1970 más cercano.


### Input: 
* Cartógrafías censales Chile 2024: Entidades y Manzanas.
* Redes Unificadas v1 1970, 1986 y actualidad.
### Output:
* Redes integradas para 1970, 1986 y actualidad con nodos centros de gravedad poblacional. Output con formato: del "edges_RED_unificado_v2.shp" y "nodes_RED_unificado_v2.shp".



## 3) esimacion ruta
### Descripción: 
Código encargado de procesar las redes v2, generando grafo geoespacial para cada red sobre el cual se procesan nuevas fronteras (si es que la unión de la red no fue efectiva en la etapa v1). Posterirmente se estiman rutas óptimas a partir del algóritmo de Djkstra utilizando como ponderador las variables de duración de cada red.


### Input:
* Redes Unificadas v2 1970, 1986 y actualidad.

### Output:
Matrices Origen destino (OD) con estimación de tiempo de viaje desde cada par de comunas identificadas como centros de gravedad poblacional para cada 1970, 1986 y actialidad. Output con formato:  "matriz_OD_FECHA.csv". 




## 4) mapping_1970_2024
### Descripción: 
Código encargado de construir una tabla de correspondencia (mapping) entre las comunas modernas (2024) y las comunas históricas de 1970, basándose en ubicación geográfica de los LPA de 1970 y 2024. Se calculan los centroides territoriales de las comunas de 2024 y posteriormente se vincula con el polígono histórico de 1970 de la comuna madre. 

### Input: 
* Cartografía LPA comunas 2024.
* Cartografía LPA comunas 1970.
  
### Output:
Matriz de relación comunas madres e hijas. Ejemplo: Comuna SAN JOAQUIN (hija) -> SAN MIGUEL (madre 1970). Ouput con formato: "mapping_1970_2024_v2.csv"

## 5) Estimacion_theta
### Descripción: 
Código encargado de estimar el parametro de elasticidad del comercio (Theta) que minimiza el error cuadrático siguiendo el setting empírico del trabajo. Primero se procesan las matrices OD para vincular los tiempos OD 1980 con las comunas históricas, luego se procesa el setting empírico par distintos valores de theta revisados en la literatura de trade. Se selecciona el valor de theta que minimiza la suma del error cuadrático. 


### Input: 
* Información electoral a nivel de comuna 1988.
* Mapping histórico.
* Información poblacional.
* Matrices OD 1970 y 1980 y 2023.
### Output:
Estimación de theta que mejor ajusta el setting empírico, en este caso theta = 4. 


## 6) DATA_market_access_comunas1970_RECONSTRUIDO
### Descripción: 
Notebook encargado de calcular los delta-Market Access para cada comuna histórica de 1970 en varias etapas.
Primero, se reconicila la información poblacional y matrices de tiempo origen-destino respecto a las entidades comunales históricas.
Segundo, se estiman los log-matket access para un panel en formato long y wide, junto con los delta market access.
Tercero, se realiza el mismo ejercicio para distintos valores de theta.
Posteriormente se realizan distintos mapas descriptivos de la estimación. 

### Input: 
* Mapping histórico.
* Censos Históricos.
* Flujos de comercio bilateral 1970.
* Deflactor del PIB.
* PIB percapital.
* Matrices OD 1970 y 1986 y 2023.
* Theta estimado (theta = 4)
  
### Output:
Estimación delta market access en formato para 1970-1986 y 1986-2024, para distintos valores de theta. Output con formato: "market_access_comunas1970_sensibilidad.csv"

## 7) elecciones_presidenciales_1970
### Descripción: 
Noteebok que se encarga de transformar la data elecotral de las elecciones extraida del pdf de FLACSO "elecciones 1970". 
### Input: 
* pdf "elecciones_1970"
* data bruta generadao "elecciones_1970_comunal_pdf.csv"
### Output:
Data final de elecciones 1970. Output con formato: "panel_con_elecciones_1970_corregidas.csv". 

## 8) Market_access_plus_output_comunas1970
### Descripción: 
### Input: 
### Output:


## 9) Construccion_Variables_Distancia_Puertos
### Descripción: 
### Input: 
### Output:


## 10) Construccion_Variables_Represion
### Descripción: 
### Input: 
### Output:

