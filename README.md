# Adaptación experimental de COMI para restauración de imágenes microscópicas

Este repositorio contiene la reproducción y adaptación experimental de
COMI (*Correction of Out-of-focus Microscopic Images by Deep Learning*)
realizada en el marco del Trabajo Fin de Máster:

**Desarrollo de métodos de corrección de artefactos en microscopía de
fluorescencia de células uterinas mediante técnicas de aprendizaje automático.**

## Contexto del TFM

El TFM fue desarrollado conjuntamente por:

- Mónica Fernández Navarro
- María Rodríguez Buitrago

El estudio completo evaluó cuatro enfoques:

- Noise2Void
- CARE
- COMI
- Restormer

Este repositorio contiene exclusivamente el módulo experimental de COMI.
No representa el código completo del TFM.

## Código original

Esta adaptación parte del repositorio original:

https://github.com/jiangdat/COMI

Artículo asociado:

Zhang, C. et al. (2022).
*Correction of out-of-focus microscopic images by deep learning*.
Computational and Structural Biotechnology Journal, 20, 1957–1966.

El código original se distribuye bajo licencia MIT.
Las modificaciones realizadas para el TFM se describen a continuación.

## Modificaciones realizadas

Se adaptó el script de evaluación para ejecutarlo en un entorno local
con recursos limitados:

- actualización de las funciones PSNR y SSIM de `scikit-image`;
- compatibilidad con `channel_axis=-1`;
- ejecución en CPU mediante `CUDA_VISIBLE_DEVICES=-1`;
- eliminación del envoltorio `multi_gpu_model` en la evaluación;
- reducción del tamaño de lote a 1;
- registro de PSNR, SSIM y correlación de Pearson;
- conservación de `InstanceNormalization` como objeto personalizado
  durante la carga de los modelos.

## Entorno utilizado

La evaluación se realizó en un entorno local bajo WSL:

- Python 3.7
- TensorFlow 1.14
- Keras 2.3.1
- keras-contrib
- NumPy 1.16
- scikit-image 0.19
- OpenCV 4.5
- SciPy 1.4
- ejecución en CPU

Las versiones heredadas son necesarias por la compatibilidad del
código original de COMI.

## Archivos principales

- `test.py`: script adaptado para evaluación en CPU.
- `test_backup.py`: copia de respaldo del código anterior.
- `deblur.py`: script de entrenamiento basado en la implementación original.
- `resultado_metricas.txt`: resultados de la reproducción funcional.
- `figure/`: figuras procedentes del proyecto original.
- `testPicture/`: imágenes de ejemplo del proyecto original.

Los pesos preentrenados y los datasets no se incluyen en el repositorio.

## Reproducción funcional

La prueba se realizó sobre el conjunto original de COMI:

- conjunto: `C0depth`;
- profundidad degradada: `Z005`;
- plano de referencia: `Z007`;
- tamaño de lote: 1;
- dispositivo: CPU.

### Resultados

| Comparación | PSNR (dB) | SSIM | Pearson |
|---|---:|---:|---:|
| Referencia nítida frente a entrada desenfocada | 19.95 | 0.325 | 0.920 |
| Referencia nítida frente a salida restaurada | 23.82 | 0.125 | 0.916 |
| Referencia desenfocada frente a dirección inversa | 18.40 | 0.621 | 0.863 |
| Referencia nítida frente a reconstrucción cíclica | 24.82 | 0.487 | 0.973 |

El generador principal aumentó el PSNR en aproximadamente 3.87 dB.
Sin embargo, la reducción de SSIM indica una modificación de la
estructura local. La reconstrucción cíclica obtuvo los valores más
altos, de acuerdo con el objetivo de consistencia cíclica del modelo.

Estos resultados corresponden a una reproducción funcional en el
dominio original y no demuestran una generalización favorable a las
imágenes de laboratorio del TFM.

## Datos

Las imágenes privadas del laboratorio no se incluyen por motivos de
confidencialidad.

Los datasets públicos y los pesos deben obtenerse desde sus fuentes
originales.

## Autoría de esta adaptación

**Mónica Fernández Navarro**

- configuración del entorno local;
- adaptación de la evaluación a CPU;
- actualización de las métricas;
- ejecución de las pruebas;
- análisis e interpretación de los resultados de COMI.

La definición general del estudio, la selección de métricas y la
interpretación global del TFM se realizaron conjuntamente con
María Rodríguez Buitrago.

## Licencia

El código original mantiene la licencia MIT y el copyright de sus
autores. Las modificaciones se publican respetando dicha licencia.
