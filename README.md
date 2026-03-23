# Analisis exploratorio e ingenieria de caracteristicas sobre EMBER2024

## Descripcion general

Esta fase del proyecto es para al analisis exploratorio de datos e ingenieria de caracteristicas sobre el dataset EMBER2024, enfocado en archivos Windows PE para clasificacion de malware. El objetivo principal fue transformar una coleccion muy grande de archivos JSON y JSONL en un conjunto de datos estructurado, escalable y util para la siguiente etapa de modelado. Debido al volumen del dataset, el proceso se diseño para trabajar por bloques, guardar avance en disco y permitir reanudacion.

Dentro del cuaderno, junto a las graficas y a los resultados obtenidos en cada corrida de codigo, se incluyen interpretaciones claras en cada etapa del proceso. Estas explicaciones permiten profundizar en el analisis exploratorio de datos y comprender mejor el comportamiento, la estructura y la logica del dataset.

## Estructura general del dataset fuente

El dataset original se organiza en cuatro carpetas (no etan subidos en este repo pero se pueden descargar se adjunta link en la referencia):

```text
data/
└── ember2024/
    ├── Win32_test
    ├── Win32_train
    ├── Win64_test
    └── Win64_train

```

## Proceso que se hizo

### Deteccion de archivos fuente

El primer paso fue verificar que la estructura del dataset estuviera completa y que las carpetas esperadas existieran correctamente. A partir de esto, se construyo una tabla con informacion basica de cada archivo, como su nombre, arquitectura, particion, extension, tamano e identificador interno. Esta validacion inicial permitio confirmar que el dataset estaba bien organizado antes de comenzar con el procesamiento.

### Conversion de JSON y JSONL a Parquet por chunks

Como los archivos originales eran demasiado grandes para cargarlos completos en memoria, se implemento un flujo de trabajo por bloques. En lugar de leer todo de una sola vez, se fueron procesando archivo por archivo y chunk por chunk, guardando cada bloque convertido en formato Parquet. Esto hizo posible avanzar de forma mas estable, evitar problemas de RAM y dejar guardado el progreso para no tener que iniciar desde cero si la ejecucion se interrumpia.

### Construccion de muestra de trabajo

Una vez generados los chunks en Parquet, se construyo una muestra controlada para trabajar el analisis exploratorio de forma mas practica. Esta muestra sirvio para revisar la estructura real de los datos, obtener estadisticos descriptivos, generar graficas y probar las primeras transformaciones sin necesidad de cargar todo el dataset completo.

### Analisis exploratorio de datos

En esta etapa se analizaron aspectos clave del dataset, como la distribucion de la variable objetivo, la diferencia entre arquitecturas y particiones, la presencia de malware y benignos, la cobertura de columnas como `family` y `behavior`, las familias y categorias mas frecuentes, el comportamiento de variables numericas y algunas relaciones entre caracteristicas resumidas. Esto permitio identificar que informacion parecia mas util para la siguiente fase.

### Construccion de vocabularios frecuentes

Luego se construyeron vocabularios frecuentes a partir del dataset completo. Esto consistio en recorrer los chunks para identificar los valores que mas se repetian en columnas categoricas y estructuras complejas, como `behavior`, `packer`, `exploit`, nombres de secciones, DLLs, APIs importadas, tecnicas `ttps`, capacidades `caps` y claves frecuentes de `strings`. El objetivo fue quedarse con las categorias mas representativas y evitar una expansion excesiva de columnas poco utiles.

### Ingenieria de caracteristicas

A partir de la informacion original, se aplico un proceso de transformacion para convertir listas y diccionarios en variables mas utiles para modelado. En esta parte se descompuso `detection_ratio` en componentes numericos, se generaron resumenes estadisticos de `histogram` y `byteentropy`, se extrajo informacion de `strings`, `general`, `header`, `section` e `imports`, y tambien se crearon indicadores para `authenticode`, `caps`, `ttps`, `mbc`, `behavior`, `packer`, `exploit` y `group`.

### Seleccion de caracteristicas

Despues de construir el conjunto de variables, se realizo una fase de filtrado y priorizacion. Primero se eliminaron identificadores, luego se excluyeron variables con riesgo de fuga de informacion, se controlaron nulos, se quitaron columnas sin variacion y se redujo la redundancia entre variables muy correlacionadas. Finalmente, se genero un ranking combinando `mutual information` y `ExtraTrees`, del cual salio el conjunto de caracteristicas recomendadas para la siguiente fase.

### Exportacion de resultados

Por ultimo, se exportaron los resultados mas importantes del proceso. Se guardaron tanto archivos intermedios como artefactos finales, de manera que la siguiente etapa no dependa otra vez de los datos fuente ni obligue a repetir todo el pipeline. Esto deja preparado un conjunto de salidas utiles para continuar con el modelado de forma mucho mas ordenada y reproducible.

## Carpetas generadas

Dentro de `artefactos/ember2024/` se generaron varias carpetas. No todas estan visibles en el repositorio porque algunas son artefactos generados automaticamente durante la ejecucion del pipeline, como archivos Parquet y metadatos de avance. Estos archivos son pesados, y se pueden regenerarse a partir del cuaderno y del codigo fuente, y ademas almacenan estados locales de procesamiento que no forman parte de algo necesario para mostrar. Por esta razon, en el repositorio solo estan disponibles los ligeros y utiles para documentacion, reproducibilidad y continuidad para la proxima fase del proyecto.

### `metricas/`

Contiene tablas y resúmenes del EDA y del proceso de ingeniería de características. Que sirve para documentar los resultados y justificar decisiones del análisis.

### `exportaciones/`

Contiene los artefactos finales más útiles para documentación y para la siguiente etapa. Esto es util para consultar resultados rápidamente reutiliza muestas y rankings

## Exportaciones generadas

### `rankingCaracteristicas.csv`

Contiene el ranking completo de variables según: _Mutual information_ , Importancia del modelo _ExtraTrees_ y esta sirve para revisar columnas que mas aportan y comparar la importancia entre características

### `caracteristicasRecomendadas.csv`

Contiene la lista final de características sugeridas para el siguiente paso. Funciona como ayuda para construir el dataset final de entrenamiento, y evitar seleccionar columnas manualmente.

### `indiceChunksBrutos.csv`

Índice de todos los chunks del dataset bruto. El cual facilita el ubicar bloques base y renudar o revisar el pipeline.

### `indiceChunksCompactos.csv`

Índice de los chunks con ingeniería de características aplicada. Este ayuda a localizar datos transformados y usar el dataset compacto ordenadamente.

### `indiceDatasetSeleccionado.csv`

Índice de los chunks finales con columnas seleccionadas. Ayuda a usar la salida final de pipeline, saber que chunks ya estan exportados.

## Principales métricas y CSV generados

Dentro de `metricas/` se generan varios archivos. Los más relevantes son los siguientes.

### Métricas de cobertura

Archivos como `resumenCobertura.csv` resumen qué tanto aparecen ciertas columnas en el dataset completo:

- Total de registros
- Cuántos tienen `family`
- Cuántos tienen `behavior`
- Cuántos tienen `packer`
- Cuántos tienen `exploit`
- Cuántos tienen `group`

Ayuda a entender qué variables tienen suficiente presencia como para ser útiles.

### Distribuciones globales

Archivos:

- `distribucionLabel.csv`
- `distribucionArquitecturaParticion.csv`
- `distribucionArquitecturaParticionLabel.csv`
- `alineacionDeteccionLabel.csv`

Sirven para documentar el balance entre beningos y malware, su distrubucion entre Win32 y Win64, diferenciar train y test y su coherencia entre direction ratio label.

### Frecuencias categóricas

Archivos:

- `topFamily.csv`
- `topBehavior.csv`
- `topFileProperty.csv`
- `topPacker.csv`
- `topExploit.csv`
- `topGroup.csv`

Ayudan para identificar las categorías más comunes del dataset.

### Frecuencias de vocabularios

Archivos:

- `frecuencias_behavior.csv`
- `frecuencias_importsDll.csv`
- `frecuencias_importsApi.csv`
- `frecuencias_sectionName.csv`
- `frecuencias_capsNamespace.csv`
- `frecuencias_ttspTechnique.csv`
- `frecuencias_mbcBehavior.csv`

Sirven para justificar los vocabularios usados durante la expansión de categorías.

### Estadísticos numéricos

Archivos:

- `estadisticosNumericosRapidos.csv`
- `resumenOutliersRapidos.csv`

Son importantes y sirven para documentar el comportamiento general de variables numéricas, sus percentiles y posibles valores extremos.

---

## Conclusiones

- Se trabajó con un dataset grande de **5.12 millones de registros**, por lo que fue necesario aplicar una estrategia por _chunks_ para poder convertir, explorar y transformar la información sin topar la memoria. Este enfoque hizo viable el análisis y también dejó una base ordenada y reutilizable para las siguientes fases del proyecto.

- En términos de distribución, la muestra quedó **balanceada entre benignos y malware**, con una proporción cercana al equilibrio. Esto es bueno para el modelado siguiente, ya que reduce el riesgo de sesgos fuertes por desbalance de clases y permite comparar mejor el comportamiento de las variables frente a ambas clases.

- También se observó que variables como `family` y `behavior` aportan información importante para entender el dataset, pero su cobertura no es completa. Por eso, aunque pueden aportar al análisis y servir como apoyo, no conviene tratarlas como la base principal del modelo. En cambio, los _hashes_ se conservaron solamente para trazabilidad e identificación, ya que no representan características útiles para generalizar la clasificación.

- Por otro lado, `detection_ratio` resultó ser una variable muy alineada con la clase, lo cual la hace útil para el análisis exploratorio, pero también implica un riesgo claro de fuga de información. Por esa razón, se mantuvo para estudiar la coherencia del dataset, pero se excluyó del conjunto principal de modelado para evitar que el modelo aprenda una señal demasiado directa y poco realista.

- Los resultados de la ingeniería y selección de características muestran que la señal más útil no está en los identificadores, sino en la estructura interna del ejecutable. Variables derivadas de secciones, _imports_, _strings_, medidas de entropía y metadatos PE fueron las que mostraron mayor capacidad para diferenciar entre archivos benignos y maliciosos. De esto se puede decir que que el comportamiento estructural del archivo ofrece mucha más información predictiva que campos superficiales o simplemente descriptivos.

- Como resultado de esta fase, se determinó que las mejores características para implementar el modelo son aquellas relacionadas con la **entropía del archivo**, la **estructura de secciones**, los **imports**, las **cadenas extraídas**, ciertos **metadatos del encabezado PE** y otros indicadores estructurales derivados. Estas variables concentran la mayor señal analítica y constituyen la base más sólida para la siguiente etapa de entrenamiento.

- En resumen, esta fase permitió convertir estructuras complejas en variables manejables y reducir el espacio original a un dataset compacto, filtrado y defendible para entrenar modelos.

## Referencias

- joyce8. (2024). EMBER2024. Hugging Face. https://huggingface.co/datasets/joyce8/EMBER2024

- Joyce, R. J., Miller, G., Roth, P., Zak, R., Zaresky-Williams, E., Anderson, H., Raff, E., & Holt, J. (2025). EMBER2024: A benchmark dataset for holistic evaluation of malware classifiers. En Proceedings of the 31st ACM SIGKDD Conference on Knowledge Discovery and Data Mining.
