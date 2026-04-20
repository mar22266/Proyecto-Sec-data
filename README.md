# Analisis exploratorio, ingenieria de caracteristicas e implementacion de modelos sobre EMBER2024

## Descripcion general

Este proyecto se desarrollo en dos fases principales sobre el dataset EMBER2024, enfocado en archivos Windows PE para clasificacion de malware. La primera fase estuvo orientada al analisis exploratorio de datos e ingenieria de caracteristicas, mientras que la segunda fase se centro en la implementacion, comparacion, refinamiento y evaluacion final de modelos de clasificacion.

Dentro de los cuadernos, junto a las graficas y a los resultados obtenidos en cada corrida de codigo, se incluyen interpretaciones claras en cada etapa del proceso. Estas explicaciones permiten profundizar en el analisis exploratorio de datos, comprender mejor el comportamiento y la estructura del dataset, y tambien justificar las decisiones tomadas durante la fase de modelado.

## Estructura general del proyecto

El trabajo completo se organizo en dos etapas:

- **Fase 1:** Analisis exploratorio e ingenieria de caracteristicas
  - Cuaderno de Python: [Fase1.ipynb](./Fase1.ipynb)
  - Version en PDF: [Fase1.pdf](./Fase1.pdf)

- **Fase 2:** Implementacion y evaluacion del modelo
  - Cuaderno de Python: [Fase2.ipynb](./Fase2.ipynb)
  - Version en PDF: [Fase2.pdf](./Fase2.pdf)

La relacion entre ambas fases fue directa: la segunda reutiliza el dataset seleccionado, los indices generados y la lista de caracteristicas recomendadas producidas durante la primera.

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

## Fase 1. Analisis exploratorio e ingenieria de caracteristicas sobre EMBER2024

### Descripcion general

Esta fase del proyecto es para al analisis exploratorio de datos e ingenieria de caracteristicas sobre el dataset EMBER2024, enfocado en archivos Windows PE para clasificacion de malware. El objetivo principal fue transformar una coleccion muy grande de archivos JSON y JSONL en un conjunto de datos estructurado, escalable y util para la siguiente etapa de modelado. Debido al volumen del dataset, el proceso se diseño para trabajar por bloques, guardar avance en disco y permitir reanudacion.

Dentro del cuaderno, junto a las graficas y a los resultados obtenidos en cada corrida de codigo, se incluyen interpretaciones claras en cada etapa del proceso. Estas explicaciones permiten profundizar en el analisis exploratorio de datos y comprender mejor el comportamiento, la estructura y la logica del dataset.

### Proceso que se hizo

#### Deteccion de archivos fuente

El primer paso fue verificar que la estructura del dataset estuviera completa y que las carpetas esperadas existieran correctamente. A partir de esto, se construyo una tabla con informacion basica de cada archivo, como su nombre, arquitectura, particion, extension, tamano e identificador interno. Esta validacion inicial permitio confirmar que el dataset estaba bien organizado antes de comenzar con el procesamiento.

#### Conversion de JSON y JSONL a Parquet por chunks

Como los archivos originales eran demasiado grandes para cargarlos completos en memoria, se implemento un flujo de trabajo por bloques. En lugar de leer todo de una sola vez, se fueron procesando archivo por archivo y chunk por chunk, guardando cada bloque convertido en formato Parquet. Esto hizo posible avanzar de forma mas estable, evitar problemas de RAM y dejar guardado el progreso para no tener que iniciar desde cero si la ejecucion se interrumpia.

#### Construccion de muestra de trabajo

Una vez generados los chunks en Parquet, se construyo una muestra controlada para trabajar el analisis exploratorio de forma mas practica. Esta muestra sirvio para revisar la estructura real de los datos, obtener estadisticos descriptivos, generar graficas y probar las primeras transformaciones sin necesidad de cargar todo el dataset completo.

#### Analisis exploratorio de datos

En esta etapa se analizaron aspectos clave del dataset, como la distribucion de la variable objetivo, la diferencia entre arquitecturas y particiones, la presencia de malware y benignos, la cobertura de columnas como family y behavior, las familias y categorias mas frecuentes, el comportamiento de variables numericas y algunas relaciones entre caracteristicas resumidas. Esto permitio identificar que informacion parecia mas util para la siguiente fase.

#### Construccion de vocabularios frecuentes

Luego se construyeron vocabularios frecuentes a partir del dataset completo. Esto consistio en recorrer los chunks para identificar los valores que mas se repetian en columnas categoricas y estructuras complejas, como behavior, packer, exploit, nombres de secciones, DLLs, APIs importadas, tecnicas ttps, capacidades caps y claves frecuentes de strings. El objetivo fue quedarse con las categorias mas representativas y evitar una expansion excesiva de columnas poco utiles.

#### Ingenieria de caracteristicas

A partir de la informacion original, se aplico un proceso de transformacion para convertir listas y diccionarios en variables mas utiles para modelado. En esta parte se descompuso detection_ratio en componentes numericos, se generaron resumenes estadisticos de histogram y byteentropy, se extrajo informacion de strings, general, header, section e imports, y tambien se crearon indicadores para authenticode, caps, ttps, mbc, behavior, packer, exploit y group.

#### Seleccion de caracteristicas

Despues de construir el conjunto de variables, se realizo una fase de filtrado y priorizacion. Primero se eliminaron identificadores, luego se excluyeron variables con riesgo de fuga de informacion, se controlaron nulos, se quitaron columnas sin variacion y se redujo la redundancia entre variables muy correlacionadas. Finalmente, se genero un ranking combinando mutual information y ExtraTrees, del cual salio el conjunto de caracteristicas recomendadas para la siguiente fase.

#### Exportacion de resultados

Por ultimo, se exportaron los resultados mas importantes del proceso. Se guardaron tanto archivos intermedios como artefactos finales, de manera que la siguiente etapa no dependa otra vez de los datos fuente ni obligue a repetir todo el pipeline. Esto deja preparado un conjunto de salidas utiles para continuar con el modelado de forma mucho mas ordenada y reproducible.

### Carpetas generadas

Dentro de artefactos/ember2024/ se generaron varias carpetas. No todas estan visibles en el repositorio porque algunas son artefactos generados automaticamente durante la ejecucion del pipeline, como archivos Parquet y metadatos de avance. Estos archivos son pesados, y se pueden regenerarse a partir del cuaderno y del codigo fuente, y ademas almacenan estados locales de procesamiento que no forman parte de algo necesario para mostrar. Por esta razon, en el repositorio solo estan disponibles los ligeros y utiles para documentacion, reproducibilidad y continuidad para la proxima fase del proyecto.

#### metricas/

Contiene tablas y resúmenes del EDA y del proceso de ingeniería de características. Que sirve para documentar los resultados y justificar decisiones del análisis.

#### exportaciones/

Contiene los artefactos finales más útiles para documentación y para la siguiente etapa. Esto es util para consultar resultados rápidamente reutiliza muestas y rankings

### Exportaciones generadas

#### rankingCaracteristicas.csv

Contiene el ranking completo de variables según: Mutual information , Importancia del modelo ExtraTrees y esta sirve para revisar columnas que mas aportan y comparar la importancia entre características

#### caracteristicasRecomendadas.csv

Contiene la lista final de características sugeridas para el siguiente paso. Funciona como ayuda para construir el dataset final de entrenamiento, y evitar seleccionar columnas manualmente.

#### indiceChunksBrutos.csv

Índice de todos los chunks del dataset bruto. El cual facilita el ubicar bloques base y renudar o revisar el pipeline.

#### indiceChunksCompactos.csv

Índice de los chunks con ingeniería de características aplicada. Este ayuda a localizar datos transformados y usar el dataset compacto ordenadamente.

#### indiceDatasetSeleccionado.csv

Índice de los chunks finales con columnas seleccionadas. Ayuda a usar la salida final de pipeline, saber que chunks ya estan exportados.

### Principales métricas y CSV generados

Dentro de metricas/ se generan varios archivos. Los más relevantes son los siguientes.

#### Métricas de cobertura

Archivos como resumenCobertura.csv resumen qué tanto aparecen ciertas columnas en el dataset completo:

- Total de registros
- Cuántos tienen family
- Cuántos tienen behavior
- Cuántos tienen packer
- Cuántos tienen exploit
- Cuántos tienen group

Ayuda a entender qué variables tienen suficiente presencia como para ser útiles.

#### Distribuciones globales

Archivos:

- distribucionLabel.csv
- distribucionArquitecturaParticion.csv
- distribucionArquitecturaParticionLabel.csv
- alineacionDeteccionLabel.csv

Sirven para documentar el balance entre beningos y malware, su distrubucion entre Win32 y Win64, diferenciar train y test y su coherencia entre direction ratio label.

#### Frecuencias categóricas

Archivos:

- topFamily.csv
- topBehavior.csv
- topFileProperty.csv
- topPacker.csv
- topExploit.csv
- topGroup.csv

Ayudan para identificar las categorías más comunes del dataset.

#### Frecuencias de vocabularios

Archivos:

- frecuencias_behavior.csv
- frecuencias_importsDll.csv
- frecuencias_importsApi.csv
- frecuencias_sectionName.csv
- frecuencias_capsNamespace.csv
- frecuencias_ttspTechnique.csv
- frecuencias_mbcBehavior.csv

Sirven para justificar los vocabularios usados durante la expansión de categorías.

#### Estadísticos numéricos

Archivos:

- estadisticosNumericosRapidos.csv
- resumenOutliersRapidos.csv

Son importantes y sirven para documentar el comportamiento general de variables numéricas, sus percentiles y posibles valores extremos.

### Conclusiones

Se trabajó con un dataset grande de 5.12 millones de registros, por lo que fue necesario aplicar una estrategia por chunks para poder convertir, explorar y transformar la información sin topar la memoria. Este enfoque hizo viable el análisis y también dejó una base ordenada y reutilizable para las siguientes fases del proyecto.

En términos de distribución, la muestra quedó balanceada entre benignos y malware, con una proporción cercana al equilibrio. Esto es bueno para el modelado siguiente, ya que reduce el riesgo de sesgos fuertes por desbalance de clases y permite comparar mejor el comportamiento de las variables frente a ambas clases.

También se observó que variables como family y behavior aportan información importante para entender el dataset, pero su cobertura no es completa. Por eso, aunque pueden aportar al análisis y servir como apoyo, no conviene tratarlas como la base principal del modelo. En cambio, los hashes se conservaron solamente para trazabilidad e identificación, ya que no representan características útiles para generalizar la clasificación.

Por otro lado, detection_ratio resultó ser una variable muy alineada con la clase, lo cual la hace útil para el análisis exploratorio, pero también implica un riesgo claro de fuga de información. Por esa razón, se mantuvo para estudiar la coherencia del dataset, pero se excluyó del conjunto principal de modelado para evitar que el modelo aprenda una señal demasiado directa y poco realista.

Los resultados de la ingeniería y selección de características muestran que la señal más útil no está en los identificadores, sino en la estructura interna del ejecutable. Variables derivadas de secciones, imports, strings, medidas de entropía y metadatos PE fueron las que mostraron mayor capacidad para diferenciar entre archivos benignos y maliciosos. De esto se puede decir que que el comportamiento estructural del archivo ofrece mucha más información predictiva que campos superficiales o simplemente descriptivos.

Como resultado de esta fase, se determinó que las mejores características para implementar el modelo son aquellas relacionadas con la entropía del archivo, la estructura de secciones, los imports, las cadenas extraídas, ciertos metadatos del encabezado PE y otros indicadores estructurales derivados. Estas variables concentran la mayor señal analítica y constituyen la base más sólida para la siguiente etapa de entrenamiento.

En resumen, esta fase permitió convertir estructuras complejas en variables manejables y reducir el espacio original a un dataset compacto, filtrado y defendible para entrenar modelos.

## Fase 2. Implementacion y evaluacion del modelo

### Descripcion general

Esta fase del proyecto tuvo como objetivo implementar modelos de clasificacion sobre el dataset final recomendado por la fase previa, comparar lineas base, refinar los finalistas mas solidos y cerrar la evaluacion con metricas operativas, curva ROC, interpretacion y conclusiones.

A diferencia de la fase anterior, aqui ya no se trabajo sobre los archivos fuente originales ni sobre una muestra exploratoria, sino sobre el dataset final seleccionado y preparado especificamente para modelado. De esta forma, la segunda etapa reutiliza directamente los artefactos generados antes y convierte la salida del proceso de ingenieria de caracteristicas en una fase de entrenamiento y validacion formal.

### Base de datos utilizada para modelado

Se trabajo con parquetSeleccionado, con un total de 5,120,000 registros.

#### Resumen de particiones

- Train total: 4,160,000
- Test total: 960,000

#### Entrenamiento

No se uso todo el train para entrenar.
Se uso una muestra estratificada de 180,000 registros.

Esta muestra se dividio asi:

- Win32: 135,000
- Win64: 45,000

La razon fue reducir costo computacional y mantener representatividad de clases y arquitecturas.

#### Validacion interna

Sobre esos 180,000 registros se hizo una division adicional:

- Train interno 75%: 135,000
- Validacion interna 25%: 45,000

Esta division se uso para comparar configuraciones y ajustar el umbral sin tocar test.

#### Prueba final

La evaluacion final se hizo sobre todo el test, es decir:

- 960,000 registros
- 18.75% del total del dataset

La razon fue medir el rendimiento final sobre datos no vistos y sobre la particion completa de prueba.

### Variables usadas

Para modelar se usaron:

- 60 caracteristicas
- 1 variable objetivo: label
- 1 variable auxiliar: arquitectura

Ademas, detection_ratio fue excluida por riesgo de fuga de informacion.

### Resumen general de la fase

- Dataset total: 5,120,000
- Train total: 4,160,000
- Test total: 960,000
- Train usado para modelar: 180,000
- Train interno: 135,000
- Validacion interna: 45,000
- Test usado para evaluacion final: 960,000
- Caracteristicas usadas: 60

### Artefactos reutilizados de la fase anterior

La fase de implementacion aprovecha directamente los artefactos generados en la etapa de analisis e ingenieria de caracteristicas:

- artefactos/ember2024/parquetSeleccionado como fuente principal de modelado
- artefactos/ember2024/exportaciones/caracteristicasRecomendadas.csv para reutilizar las 60 variables sugeridas
- artefactos/ember2024/exportaciones/indiceDatasetSeleccionado.csv para respetar la particion original train y test
- artefactos/ember2024/exportaciones/muestraModeladoCompacto.parquet solo como referencia de control, no como base final, porque la muestra exploratoria esta concentrada en Win32 y no representa toda la mezcla de arquitecturas
- La exclusion de detection_ratio ya definida en la fase anterior por riesgo de fuga de informacion

### Proceso que se hizo

#### Validacion del flujo y de las rutas necesarias

Primero se definieron las rutas principales del proyecto, la semilla de trabajo y las funciones necesarias para validar archivos, cargar los datos y construir los modelos base. Esto permitio dejar preparado un flujo de trabajo reproducible, ordenado y consistente con los artefactos ya generados.

Tambien se validaron los archivos base del modelado y se confirmo que el proyecto estaba detectando correctamente la estructura esperada para esta fase.

#### Validacion del dataset final seleccionado

Antes de entrenar los modelos, se reviso un chunk de ejemplo del parquetSeleccionado para confirmar que el conjunto final contenia:

- la variable objetivo label
- las 60 caracteristicas recomendadas
- la exclusion efectiva de detection_ratio
- la separacion correcta entre columnas de modelado y columnas auxiliares o de trazabilidad

Esta revision permitio documentar que la fuente real para el modelado era la version seleccionada por chunks y no la muestra exploratoria usada antes para EDA.

#### Preparacion del conjunto de entrenamiento y prueba

Se construyo una muestra reproducible y proporcional por arquitectura desde train para hacer el entrenamiento sin romper el flujo por chunks. La evaluacion externa, en cambio, se mantuvo sobre la particion test completa.

Ademas, se reviso la distribucion por arquitectura y clase, el numero de valores nulos y el tipo de dato de las variables de modelado. Las caracteristicas quedaron en formato float32, lo cual ayudo a reducir el consumo de memoria y mantener el flujo mas estable.

#### Implementacion de modelos base

En esta etapa se entrenaron tres modelos de referencia consistentes con el tipo de dataset disponible:

**Regresion Logistica**

Se utilizo como linea base. Su funcion fue servir como punto de comparacion inicial, ya que es un modelo simple, rapido y facil de interpretar. Permitio verificar si una solucion lineal era suficiente o si el problema necesitaba un enfoque mas complejo.

**Extra Trees Base**

Se eligio porque era uno de los modelos mas coherentes con la fase anterior y porque suele rendir muy bien en datos tabulares. Su principal ventaja es que puede capturar relaciones no lineales e interacciones entre variables, algo importante en un problema de clasificacion de malware.

**Hist Gradient Boosting Base**

Se uso como una alternativa avanzada para comparar otra familia de modelos basados en arboles. Su inclusion permitio evaluar si un enfoque de boosting podia competir o superar al ensamblado de arboles de ExtraTrees en este conjunto de datos.

#### Evaluacion inicial de modelos

La comparacion inicial se realizo sobre el test externo completo. En esta parte no solo se midieron metricas generales, sino que tambien se reviso la matriz de confusion y el reporte de clasificacion del mejor modelo base para tener una referencia clara antes del refinamiento.

#### Resultados iniciales

| Modelo                   | Accuracy | Precision | Recall | F1     | AUC ROC |
| ------------------------ | -------- | --------- | ------ | ------ | ------- |
| ExtraTreesBase           | 0.9690   | 0.9772    | 0.9605 | 0.9688 | 0.9952  |
| HistGradientBoostingBase | 0.9604   | 0.9636    | 0.9570 | 0.9603 | 0.9936  |
| RegresionLogistica       | 0.9129   | 0.9228    | 0.9012 | 0.9119 | 0.9754  |

El mejor modelo base en esta primera evaluacion fue ExtraTreesBase, ya que alcanzo el mayor valor de AUC ROC, F1 y Accuracy frente a las otras alternativas. A partir de esa comparacion se tomo como referencia principal para la siguiente etapa de ajuste.

#### Refinamiento de modelos finalistas

Despues de la comparacion inicial, se refinaron las dos familias con mejor sentido para el problema: ExtraTrees y HistGradientBoosting.

En esta etapa se probaron cambios en hiperparametros como:

- numero de arboles
- profundidad
- cantidad minima de muestras por hoja
- proporcion de variables por division
- numero de iteraciones
- learning_rate
- limite de hojas
- regularizacion

Las configuraciones evaluadas fueron:

- ExtraTreesBaseAfinado
- ExtraTreesRefinado
- ExtraTreesProfundidadControlada
- HistGradientBoostingBaseAfinado
- HistGradientBoostingProfundo
- HistGradientBoostingHojaLimitada

#### Resultados del refinamiento

| Modelo                           | Accuracy | Precision | Recall | F1     | AUC ROC |
| -------------------------------- | -------- | --------- | ------ | ------ | ------- |
| ExtraTreesRefinado               | 0.9756   | 0.9807    | 0.9694 | 0.9750 | 0.9967  |
| ExtraTreesBaseAfinado            | 0.9744   | 0.9789    | 0.9689 | 0.9739 | 0.9965  |
| ExtraTreesProfundidadControlada  | 0.9734   | 0.9785    | 0.9671 | 0.9727 | 0.9960  |
| HistGradientBoostingBaseAfinado  | 0.9695   | 0.9758    | 0.9617 | 0.9687 | 0.9955  |
| HistGradientBoostingProfundo     | 0.9696   | 0.9763    | 0.9615 | 0.9688 | 0.9955  |
| HistGradientBoostingHojaLimitada | 0.9688   | 0.9755    | 0.9607 | 0.9680 | 0.9953  |

La mejor configuracion fue ExtraTreesRefinado, ya que alcanzo el mejor equilibrio entre AUC ROC, F1 y Accuracy en la validacion interna.

#### Revision de umbral

Despues de identificar el mejor modelo refinado, se realizo una revision de umbrales sobre validacion interna para no tomar la decision final directamente con el umbral fijo de 0.50.

#### Comparacion de umbrales

| Umbral | Accuracy | Precision | Recall | F1     |
| ------ | -------- | --------- | ------ | ------ |
| 0.45   | 0.9756   | 0.9764    | 0.9739 | 0.9752 |
| 0.50   | 0.9756   | 0.9807    | 0.9694 | 0.9750 |
| 0.40   | 0.9746   | 0.9706    | 0.9780 | 0.9743 |
| 0.55   | 0.9749   | 0.9843    | 0.9644 | 0.9742 |
| 0.60   | 0.9736   | 0.9875    | 0.9585 | 0.9728 |
| 0.35   | 0.9724   | 0.9629    | 0.9817 | 0.9722 |

El umbral de 0.45 fue el mas conveniente para el cierre final, porque ofrecio el mejor balance entre precision y recall.

#### Evaluacion final del modelo refinado

El modelo refinado ganador se volvio a entrenar usando toda la muestra de entrenamiento disponible para esta fase y luego se evaluo sobre el conjunto test completo.

#### Comparacion final: mejor base vs modelo refinado

| Escenario                   | Umbral | Accuracy | Precision | Recall | F1     | AUC ROC |
| --------------------------- | ------ | -------- | --------- | ------ | ------ | ------- |
| ExtraTreesBase              | 0.50   | 0.9690   | 0.9772    | 0.9605 | 0.9688 | 0.9952  |
| ExtraTreesRefinado refinado | 0.45   | 0.9709   | 0.9735    | 0.9682 | 0.9708 | 0.9949  |

#### Metricas finales del modelo refinado

- Modelo final: ExtraTreesRefinado
- Umbral final: 0.45
- Accuracy: 0.9709
- Precision: 0.9735
- Recall: 0.9682
- F1 score: 0.9708
- AUC ROC: 0.9949

#### Matriz de confusion final

|                | Pred 0 Benigno | Pred 1 Malware |
| -------------- | -------------- | -------------- |
| Real 0 Benigno | 467356         | 12644          |
| Real 1 Malware | 15270          | 464730         |

#### Reporte por clase

| Clase   | Precision | Recall | F1 score | Support |
| ------- | --------- | ------ | -------- | ------- |
| Benigno | 0.9684    | 0.9737 | 0.9710   | 480000  |
| Malware | 0.9735    | 0.9682 | 0.9708   | 480000  |

#### Metricas por arquitectura

| Arquitectura | Accuracy | Precision | Recall | F1     | AUC ROC |
| ------------ | -------- | --------- | ------ | ------ | ------- |
| Win32        | 0.9721   | 0.9752    | 0.9690 | 0.9721 | 0.9955  |
| Win64        | 0.9673   | 0.9686    | 0.9658 | 0.9672 | 0.9932  |

Estos resultados muestran que el modelo generaliza bien en ambas arquitecturas, aunque el subconjunto de Win64 sigue siendo un poco mas desafiante.

#### Curvas ROC y visualizaciones

Como parte del cierre de esta fase, se compararon las curvas ROC de los modelos base contra el refinado y se agrego la matriz de confusion final como apoyo visual para el analisis operativo. Esto permitio observar que la separacion entre clases se mantuvo alta y que el modelo refinado logro consolidar un rendimiento fuerte sobre el test completo.

### Interpretacion de resultados y conclusiones de la fase 2

El mejor resultado de esta fase se obtuvo con el modelo ExtraTreesRefinado, utilizando la configuracion:

```json
{
  "n_estimators": 420,
  "max_depth": null,
  "min_samples_leaf": 1,
  "max_features": 0.6,
  "class_weight": "balanced_subsample"
}
```

y un umbral de decision de 0.45.

Al evaluarlo sobre el conjunto test completo, el modelo alcanzo un Accuracy de 0.9709, una Precision de 0.9735, un Recall de 0.9682, un F1 score de 0.9708 y un AUC ROC de 0.9949. En terminos generales, estos resultados muestran un desempeño alto y bastante estable para el problema de clasificacion binaria entre archivos benignos y malware.

Si se observa el comportamiento por clase, el modelo mantiene un equilibrio adecuado. Para la clase malware, se obtuvo una Precision de 0.9735, un Recall de 0.9682 y un F1 score de 0.9708, lo que indica que el modelo logra recuperar la mayor parte de los archivos maliciosos sin disparar una cantidad excesiva de falsos positivos. Para la clase benigno, los resultados fueron igualmente fuertes, con una Precision de 0.9684, un Recall de 0.9737 y un F1 score de 0.9710. Esto sugiere que el clasificador no esta sesgado de forma marcada hacia una sola clase, sino que conserva un rendimiento bastante parejo en ambos sentidos.

Un punto importante es el analisis de errores. En el contexto de deteccion de malware, los errores mas delicados son los falsos negativos, porque representan archivos maliciosos que el modelo deja pasar como si fueran benignos. En la evaluacion final se registraron 15,270 falsos negativos frente a 12,644 falsos positivos. Aunque ambos tipos de error importan, los falsos negativos tienen un peso mayor desde la perspectiva de seguridad. Aun asi, la diferencia entre ambos no es desproporcionada y el comportamiento general del modelo sigue siendo consistente. El ajuste del umbral a 0.45 ayudo justamente a mejorar la capacidad de recuperacion de malware sin romper el balance global del clasificador.

Tambien es importante comparar el resultado final con el mejor modelo base para ver si el refinamiento realmente aporto algo. En este caso, el modelo refinado mejoro +0.0021 en F1 y +0.0077 en Recall, aunque con una ligera disminucion de -0.0037 en Precision. Esto quiere decir que el refinamiento si tuvo efecto y que la mejora fue real, sobre todo en la capacidad de detectar archivos maliciosos. No es una diferencia enorme, pero si es suficiente para justificar la seleccion de la version refinada como cierre de esta fase.

Desde la perspectiva practica, el modelo puede considerarse util como apoyo para la deteccion de malware. El nivel de separacion entre clases es alto y las metricas obtenidas indican que puede servir para tareas de filtrado, priorizacion o apoyo al analisis automatizado. De todas formas, no seria recomendable usarlo como unica capa de decision, ya que todavia existen errores restantes minimos, especialmente falsos negativos, que justifican mantenerlo dentro de un flujo mas amplio de evaluacion.

## Referencias

- joyce8. (2024). EMBER2024. Hugging Face. https://huggingface.co/datasets/joyce8/EMBER2024

- Joyce, R. J., Miller, G., Roth, P., Zak, R., Zaresky-Williams, E., Anderson, H., Raff, E., & Holt, J. (2025). EMBER2024: A benchmark dataset for holistic evaluation of malware classifiers. En Proceedings of the 31st ACM SIGKDD Conference on Knowledge Discovery and Data Mining.
