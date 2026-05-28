# Predicción de la Calidad del Agua en India con PySpark y Machine Learning

**Asignatura:** Procesamiento de Alto Volumen de Datos (PAVD)  
**Universidad:** Pontificia Universidad Javeriana  
**Autor:** Jesús David Romero Melo  
**Entorno:** Jupyter Notebook · Apache Spark · Hadoop HDFS

---

## Descripción

Taller de implementación de un pipeline completo de Machine Learning sobre datos de calidad hídrica de ríos de la India. Cubre desde la carga de datos en HDFS hasta la comparación de tres modelos predictivos para estimar el Índice de Calidad del Agua (WQI), con énfasis en el uso del ecosistema distribuido Apache Spark.

---

## Dataset

**Fuente:** Sistema de monitoreo de ríos de la India (RiverIndia) — Central Pollution Control Board  
**Registros:** 534 estaciones de monitoreo distribuidas en el territorio nacional  
**Almacenamiento:** Hadoop HDFS

| Variable | Descripción | Tipo |
|----------|-------------|------|
| STATION CODE | Identificador de estación | String |
| LOCATIONS | Nombre de ubicación | String |
| STATE | Estado de India | String |
| TEMP | Temperatura del agua (°C) | Double |
| DO | Oxígeno disuelto (mg/L) | Double |
| pH | Potencial de hidrógeno | Double |
| CONDUCTIVITY | Conductividad eléctrica (µS/cm) | Double |
| BOD | Demanda Bioquímica de Oxígeno (mg/L) | Double |
| NITRATE_N_NITRITE_N | Nitratos y nitritos (mg/L) | Double |
| FECAL_COLIFORM | Coliformes fecales (UFC/100mL) | Double |
| TOTAL_COLIFORM | Coliformes totales (UFC/100mL) | Double |

---

## Estructura del Notebook

```
1. Configuración del entorno y semillas de reproducibilidad
2. Inicio de sesión Spark (SparkSession + SQLContext)
3. Carga del dataset desde HDFS
4. Limpieza y preprocesamiento
   ├── Eliminación de valores nulos
   ├── Conversión de tipos (String → Double)
   └── Descarte de columnas no predictivas
5. Cálculo de sub-índices de calidad (qrPH, qrDO, qrCOND, qrBOD, qrNN, qrFecal)
6. Cálculo del WQI y clasificación por categoría
7. Análisis Exploratorio de Datos (EDA)
   ├── Estadísticas descriptivas
   ├── Histogramas por parámetro
   ├── Mapa de correlación
   └── Visualización geoespacial por estado
8. Preparación del split train/test (unificado para todos los modelos)
9. Modelos de Machine Learning
   ├── Red Neuronal (Keras)
   ├── Random Forest Regressor (MLlib)
   └── Gradient Boosted Trees (MLlib)
10. Comparación de métricas y tabla resumen
```

---

## Modelos Implementados

### Red Neuronal — Keras/TensorFlow
- Arquitectura: 3 capas ocultas densas de 350 neuronas con activación ReLU
- Optimizador: Adam (lr=0.001, beta_1=0.9, beta_2=0.999)
- Función de pérdida: MSE · Épocas: 200 · Batch size: 81
- Limitación: el entrenamiento ocurre fuera de Spark (requiere `.toPandas()`)

### Random Forest Regressor — MLlib
- `numTrees=100`, `seed=1`
- Entrenamiento y predicción completamente distribuidos en Spark
- Ofrece métrica de importancia de variables (`featureImportances`)

### Gradient Boosted Trees — MLlib
- `maxIter=100`, `seed=1`
- Entrenamiento y predicción completamente distribuidos en Spark
- Nota: con este volumen de datos muestra sobreajuste (RMSE train << RMSE test)

---

## Resultados

| Modelo | RMSE Train | RMSE Test | R² | Tiempo (s) |
|--------|-----------|-----------|------|-----------|
| Red Neuronal | 0.0228 | 0.0228 | 0.9998 | 6.49 |
| Random Forest | 4.2709 | 5.2929 | 0.9053 | 1.25 |
| GBT | 0.1076 | 3.5074 | 0.9584 | 18.04 |

---

## Reproducibilidad

Las semillas están fijadas globalmente al inicio del notebook para garantizar resultados idénticos en cada ejecución:

```python
SEMILLA = 42
os.environ['PYTHONHASHSEED'] = str(SEMILLA)
random.seed(SEMILLA)
np.random.seed(SEMILLA)
tf.random.set_seed(SEMILLA)
```

El split train/test (80/20) se realiza una sola vez en Spark con `seed=42` y se comparte entre los tres modelos para garantizar una comparación justa.

---

## Requisitos

```
pyspark
tensorflow / keras
scikit-learn
numpy
pandas
matplotlib
seaborn
geopandas
adjustText
findspark
```

---

## Notas sobre integración con Spark

| Etapa | Red Neuronal | Random Forest | GBT |
|-------|-------------|--------------|-----|
| Carga de datos | ✓ Spark | ✓ Spark | ✓ Spark |
| Preprocesamiento | ✓ Spark | ✓ Spark | ✓ Spark |
| Entrenamiento | ✗ Local (Keras) | ✓ Spark | ✓ Spark |
| Predicción | ✗ Local | ✓ Spark | ✓ Spark |
| Escalabilidad real | Parcial | ✓ | ✓ |

La red neuronal requiere `.toPandas()` antes del entrenamiento, lo que representa un cuello de botella en escenarios de alto volumen de datos. RF y GBT operan de extremo a extremo dentro del ecosistema distribuido de Spark.

---

## Referencias

- Apache Spark MLlib: https://spark.apache.org/docs/latest/ml-guide.html
- Keras Documentation: https://keras.io
- Metodología WQI: https://www.intechopen.com/chapters/69568
- Central Pollution Control Board India: https://cpcb.nic.in
