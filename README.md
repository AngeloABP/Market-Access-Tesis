# Market-Access-Tesis

Códigos utilizados en el trabajo de tesis de grado magíster en economía UC. La tesis estudia el efecto del acceso a mercados sobre resultados electorales a nivel comunal en Chile, construyendo medidas de *market access* a partir de redes viales históricas reconstruidas para 1970, 1986 y la actualidad.

En la carpeta `Codes_final` se encuentran los notebooks utilizados para el trabajo de datos de la tesis. El flujo completo va desde la construcción de las redes viales geoespaciales hasta la consolidación de la base de datos analítica principal. El orden de estimación es el siguiente:

---

## 1) `networks_distance_finalversion`

### Descripción
Notebook encargado de procesar los archivos geográficos regionales en formato `.shp`; verificar la consistencia de las redes tanto de manera individual como integrada; y generar las redes correspondientes a cada período (1970, 1986 y actualidad), calculando los tiempos de viaje asociados a cada camino de la red.

El script carga los shapefiles regionales, los inspecciona sistemáticamente para identificar y corregir problemas comunes (valores faltantes, inconsistencias geométricas, errores tipográficos), analiza la conectividad del grafo para garantizar una red completamente conectada, y computa distancias y tiempos de viaje vía la API de Google Maps para cada arista de la red.

### Input
- 16 redes regionales (nodos y aristas) en formato `.shp`
- Coeficientes de costos de transporte elaborados en el do-file `coeficientes de costo`

### Output
Redes integradas para 1970, 1986 y la actualidad (nodos y aristas), con una columna que contiene el tiempo de transporte, medido en segundos, para cada camino.

> ** Importante:** El código utiliza la API de Google Maps Distance Matrix, por lo que requiere una cuenta de pago activa en Google Cloud para estimar las variables `duration` y `distance`.

---

## 2) `Centros_de_gravedad_pob`

### Descripción
Notebook encargado de trabajar las islas de conectividad (trabajo manual en QGIS, donde se identifican los caminos incongruentes según la cartografía histórica y se añaden a la base 1986). Posteriormente, a partir de la cartografía censal de 2024 se estiman los centros de gravedad poblacional que definen a las entidades comunales de 2024. Finalmente, se procede a unir estos nodos centro de gravedad mediante un proceso de homologación con el nodo de red en 1970 más cercano.

### Input
- Cartografías censales Chile 2024: Entidades y Manzanas
- Redes Unificadas v1 1970, 1986 y actualidad

### Output
Redes integradas para 1970, 1986 y actualidad con nodos centros de gravedad poblacional. Formato: `edges_RED_unificado_v2.shp` y `nodes_RED_unificado_v2.shp`.

---

## 3) `esimacion ruta`

### Descripción
Notebook encargado de procesar las redes v2, generando un grafo geoespacial para cada red sobre el cual se procesan nuevas fronteras (si es que la unión de la red no fue efectiva en la etapa v1). Posteriormente se estiman rutas óptimas a partir del algoritmo de Dijkstra, utilizando como ponderador las variables de duración de cada red.

### Input
- Redes Unificadas v2 1970, 1986 y actualidad

### Output
Matrices Origen-Destino (OD) con estimación de tiempo de viaje desde cada par de comunas identificadas como centros de gravedad poblacional, para 1970, 1986 y actualidad. Formato: `matriz_OD_FECHA.csv`.

---

## 4) `mapping_1970_2024`

### Descripción
Notebook encargado de construir una tabla de correspondencia (*mapping*) entre las comunas modernas (2024) y las comunas históricas de 1970, basándose en la ubicación geográfica de los LPA de 1970 y 2024. Se calculan los centroides territoriales de las comunas de 2024 y posteriormente se vinculan con el polígono histórico de 1970 de la comuna madre.

### Input
- Cartografía LPA comunas 2024
- Cartografía LPA comunas 1970

### Output
Matriz de relación comunas madres e hijas. Ejemplo: comuna `SAN JOAQUIN` (hija) → `SAN MIGUEL` (madre 1970). Formato: `mapping_1970_2024_v2.csv`.

---

## 5) `Estimacion_theta`

### Descripción
Notebook encargado de estimar el parámetro de elasticidad del comercio (Theta) que minimiza el error cuadrático, siguiendo el *setting* empírico del trabajo. Primero se procesan las matrices OD para vincular los tiempos OD 1980 con las comunas históricas; luego se procesa el *setting* empírico para distintos valores de theta revisados en la literatura de comercio internacional. Se selecciona el valor de theta que minimiza la suma del error cuadrático.

### Input
- Información electoral a nivel de comuna 1988
- Mapping histórico
- Información poblacional
- Matrices OD 1970, 1980 y 2023

### Output
Estimación de theta que mejor ajusta el *setting* empírico. En este caso: **θ = 4**.

---

## 6) `DATA_market_access_comunas1970_RECONSTRUIDO`

### Descripción
Notebook encargado de calcular los Δ-*Market Access* para cada comuna histórica de 1970 en varias etapas:

1. Se reconcilia la información poblacional y las matrices de tiempo origen-destino respecto a las entidades comunales históricas.
2. Se estiman los log-*market access* para un panel en formato *long* y *wide*, junto con los Δ *market access*.
3. Se repite el ejercicio para distintos valores de theta (análisis de sensibilidad).
4. Se generan mapas descriptivos de la estimación.

### Input
- Mapping histórico
- Censos Históricos
- Flujos de comercio bilateral 1970
- Deflactor del PIB
- PIB per cápita
- Matrices OD 1970, 1986 y 2023
- Theta estimado (θ = 4)

### Output
Estimación Δ *market access* para los períodos 1970–1986 y 1986–2024, para distintos valores de theta. Formato: `market_access_comunas1970_sensibilidad.csv`.

---

## 7) `elecciones_presidenciales_1970`

### Descripción
Notebook que se encarga de transformar la data electoral de las elecciones presidenciales de 1970, extraída del PDF de FLACSO *"Elecciones 1970"*. Realiza la homologación y normalización de los nombres comunales, agrega los resultados electorales al nivel comunal y los vincula al panel comunal histórico.

### Input
- PDF `elecciones_1970`
- Data bruta generada: `elecciones_1970_comunal_pdf.csv`

### Output
Data final de elecciones 1970 integrada al panel comunal. Formato: `panel_con_elecciones_1970_corregidas.csv`.

---

## 8) `Market_access_plus_output_comunas1970`

### Descripción
Notebook encargado de unir las bases de *Market Access*, elecciones (plebiscito 1988, 2020 y 2022; presidenciales 1989–2009 y diputados 1989–2005), controles (geográficos, demográficos y elecciones Alessandri) y estimaciones de *market access* extralocal (variable instrumental).

### Input
- DataFrame sección 6: `market_access_comunas1970_sensibilidad.csv`
- DataFrame Stata con elecciones
- DataFrames plebiscitos: `Resultados_plebiscito_2022`, `Resultados_plebiscito_88` y `Resultados-Plebiscito-Constitucion-Politica-2020`
- DataFrame mapping
- DataFrame población

### Output
DataFrame principal de la investigación en formato `.dta`: `base_unificada_ma_elecciones_comunas1970.dta`. Sobre esta base se realizan los principales análisis y se unen las bases auxiliares de los ejercicios de mecanismos.

---

## 9) `Construccion_Variables_Distancia_Puertos`

### Descripción
Notebook encargado de generar las variables de proximidad (en términos de distancia y tiempo base) con el puerto principal de Chile en dirección del EMPORCHI en 1970 más cercano.  
Los puertos de estudio corresponden a: 
* Valparaiso
* San Antonio
* San Vicente
* Antofagasta
* Iquique
* Puerto Montt
* Punta Arenas
* Arica
* Coquimbo
* Valdivia
  

### Input
* Listado y coordenadas de cada puerto.
* Dataframe sección 7: `base_unificada_ma_elecciones_comunas1970.dta`


### Output
DataFrame mecanismo represión en formato `.dta`: `mecanismos_distancia_puertos`. La variable relevante en el estudio corresponde a `distancia_puerto_km`.

---

## 10) `Construccion_Variables_Represion`

### Descripción
Notebook complementario al anterior (sección 9), encargado de construir variables adicionales de represión política para el análisis de mecanismos. El proceso consta de cuatro etapas principales:

1. **Extracción y clasificación de recintos:** Mediante *web scraping* sobre la web oficial del Programa de Derechos Humanos del Ministerio de Justicia, se extraen los recintos de detención, asignándoles un tipo institucional (DINA/CNI, Regimiento, Comisaría, Campamento, Cárcel u Otro) y georreferenciándolos a las unidades comunales históricas de 1970. Se utiliza un diccionario de homologación (`MODERNO_A_HISTORICO`) que vincula nombres modernos de comunas con sus equivalentes históricos de 1970.

2. **Agregación comunal y definición de Zonas Principales:** Se cuenta el número de recintos por tipo en cada comuna histórica de 1970 y se define una variable binaria `es_zona_principal` = 1 si la comuna albergó al menos uno de los recintos de mayor peso institucional: DINA/CNI, Campamento de Prisioneros o Regimiento/Base Militar.

3. **Cálculo de distancias y tiempo de viaje:** Se calcula la distancia euclidiana en kilómetros (ajustada por latitud media) desde el centroide de cada comuna hasta la Zona Principal de Detención más cercana dentro de su región. Complementariamente, se estima el tiempo de viaje usando la **Matriz OD 1970**, midiendo la accesibilidad en términos de minutos de viaje por la red vial histórica. Se construyen variables auxiliares como `distancia_100km` (distancia normalizada a 100 km) y la dummy `cerca_zona_represion_50km` (= 1 si la comuna está a ≤ 50 km de una Zona Principal).

4. **Exportación:** Se seleccionan las variables relevantes para los análisis de mecanismos y se exporta la base en formato Stata.

### Input
- DataFrame sección 8: `base_unificada_ma_elecciones_comunas1970.csv` (contiene coordenadas y región de cada comuna de 1970)
- URL Web Ministerio de Justicia: https://pdh.minjusticia.gob.cl/recintos-de-detencion/
- Matriz OD 1970: `matriz_OD_1970.csv`

### Output
DataFrame de variables de represión en formato `.dta`: `mecanismos_recintos_de_detencion.dta`. Las variables construidas incluyen:

| Variable | Descripción |
|---|---|
| `n_recintos_total` | Número total de recintos de detención en la comuna |
| `n_dina_cni` | Número de centros DINA/CNI |
| `n_regimientos` | Número de regimientos o bases militares |
| `n_comisarias` | Número de comisarías |
| `n_campos` | Número de campamentos de prisioneros |
| `n_carceles` | Número de cárceles |
| `n_otros` | Número de otros recintos |
| `es_zona_principal` | Dummy: 1 si la comuna es Zona Principal de Represión |
| `distancia_zona_principal_km` | Distancia euclidiana (km) a la Zona Principal más cercana |
| `distancia_100km` | Distancia normalizada en unidades de 100 km |
| `cerca_zona_represion_50km` | Dummy: 1 si la comuna está a ≤ 50 km de una Zona Principal |
| `tiempo_comuna_zona_represion_1970` | Tiempo de viaje (red vial 1970) a la Zona Principal más cercana |

---

## Dependencias principales

Los notebooks requieren las siguientes librerías de Python:

```
pandas, geopandas, numpy, networkx, shapely, matplotlib,
requests, beautifulsoup4, scipy, unicodedata, re
```

Adicionalmente, el notebook `networks_distance_finalversion` requiere acceso a la **API de Google Maps Distance Matrix** (cuenta de pago en Google Cloud).

---

## Flujo General del Proyecto

```
[Redes .shp regionales]
        |
        v
 1. networks_distance_finalversion   -->  Redes v1 (nodos + aristas + tiempos)
        |
        v
 2. Centros_de_gravedad_pob          -->  Redes v2 (+ centros de gravedad comunal)
        |
        v
 3. esimacion ruta                   -->  Matrices OD (1970, 1986, 2023)
        |
        v
 4. mapping_1970_2024                -->  Tabla de correspondencia comunas
        |
        v
 5. Estimacion_theta                 -->  theta = 4
        |
        v
 6. DATA_market_access_comunas1970   -->  Delta Market Access por comuna
        |
        v
 7. elecciones_presidenciales_1970   -->  Data electoral 1970
        |
        v
 8. Market_access_plus_output        -->  Base unificada principal (.dta)
        |
        v
 9. Construccion_Variables_Distancia_Puertos  -->  Variables represion (distancia)
        |
        v
10. Construccion_Variables_Represion         -->  Variables represion (tiempo OD + indices)
```


