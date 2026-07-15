# Adaptación experimental de COMI para restauración de imágenes microscópicas

Este repositorio contiene la reproducción y adaptación experimental de **COMI** (*Correction of Out-of-focus Microscopic Images by Deep Learning*) realizada en el marco del Trabajo Fin de Máster:

> **Desarrollo de métodos de corrección de artefactos en microscopía de fluorescencia de células uterinas mediante técnicas de aprendizaje automático**

## Alcance del repositorio

El TFM fue desarrollado conjuntamente por **Mónica Fernández Navarro** y **María Rodríguez Buitrago** e incluyó cuatro enfoques: Noise2Void, CARE, COMI y Restormer.

Este repositorio contiene **únicamente el módulo experimental de COMI**. No representa el código completo del TFM. La definición general del protocolo experimental, la selección de métricas y la interpretación global del estudio se realizaron de forma conjunta por ambas autoras.

## Código original y licencia

La adaptación parte del repositorio original de COMI:

- [Repositorio original de COMI](https://github.com/jiangdat/COMI)
- Zhang, C. et al. (2022). *Correction of out-of-focus microscopic images by deep learning*. Computational and Structural Biotechnology Journal, 20, 1957–1966.

El código original se distribuye bajo licencia MIT. Este repositorio conserva la licencia y la atribución correspondientes.

## Cambios realizados para el TFM

El archivo `test.py` fue adaptado para ejecutar la evaluación en un entorno local con recursos limitados:

- sustitución de las funciones obsoletas `skimage.measure.compare_psnr` y `compare_ssim` por `skimage.metrics.peak_signal_noise_ratio` y `structural_similarity`;
- uso explícito de `data_range=1` en PSNR y SSIM;
- compatibilidad de SSIM mediante `channel_axis=-1`, con alternativa `multichannel=True`;
- ejecución forzada en CPU con `CUDA_VISIBLE_DEVICES=-1`;
- eliminación del envoltorio multi-GPU en los modelos combinados;
- reducción del tamaño de lote de 4 a 1;
- carga de los modelos `.h5` mediante el objeto personalizado `InstanceNormalization`;
- cálculo de PSNR, SSIM y correlación de Pearson;
- generación de imágenes restauradas en la carpeta de predicciones.

## Archivos relevantes

- `test.py`: script modificado y utilizado para la reproducción funcional.
- `test_backup.py`: copia de respaldo del script anterior.
- `deblur.py`: script de entrenamiento procedente de la implementación original; requiere adaptar rutas y configuración de GPU antes de ejecutarlo.
- `requirements.txt`: dependencias del entorno utilizado por `test.py`.
- `resultados_COMI_dataset_original.txt`: registro de métricas obtenido durante la reproducción. Este archivo no formaba parte del repositorio original.

## Entorno de ejecución

La prueba se realizó en un entorno local bajo WSL, con ejecución en CPU:

- Python 3.7
- TensorFlow 1.14.0
- Keras 2.3.1
- `keras-contrib`
- NumPy
- SciPy
- scikit-image
- scikit-learn
- OpenCV
- Pillow

Las versiones exactas se encuentran en `requirements.txt`. Se trata de un entorno heredado debido a las dependencias del código original de COMI.

## Instalación

```bash
git clone https://github.com/Monicafn/COMI.git
cd COMI

python3.7 -m venv .venv
source .venv/bin/activate
python -m pip install -r requirements.txt
```

## Datos y pesos necesarios

Los datasets, las imágenes privadas del laboratorio y los pesos preentrenados no se distribuyen en este repositorio.

Para reproducir la prueba, `test.py` espera una estructura equivalente a:

```text
COMI/
├── confocal/
│   └── C0depth/
│       ├── Z005/
│       └── Z007/
├── deblursrgan4_C0depth_Z005_weights/
│   ├── generator_l2h.h5
│   └── generator_h2l.h5
├── model/
│   └── vgg19_weights_tf_dim_ordering_tf_kernels_notop.h5
├── test.py
└── requirements.txt
```

## Ejecución

Para ejecutar la evaluación y guardar simultáneamente la salida en un archivo:

```bash
python test.py 2>&1 | tee resultados_COMI_dataset_original.txt
```

Configuración empleada:

- conjunto: `C0depth`;
- entrada desenfocada: `Z005`;
- referencia en foco: `Z007`;
- tamaño de lote: 1;
- dispositivo: CPU.

## Resultados de la reproducción funcional

| Comparación | PSNR (dB) | SSIM | Pearson |
|---|---:|---:|---:|
| Referencia nítida vs. entrada desenfocada | 19.95 | 0.325 | 0.920 |
| Referencia nítida vs. salida restaurada (`lq2hq`) | 23.82 | 0.125 | 0.916 |
| Referencia desenfocada vs. dirección inversa (`hq2lq`) | 18.40 | 0.621 | 0.863 |
| Referencia nítida vs. reconstrucción cíclica | 24.82 | 0.487 | 0.973 |

El generador principal aumentó el PSNR en aproximadamente **3.87 dB** respecto a la entrada desenfocada. Sin embargo, la reducción de SSIM indica que la salida modifica la estructura local. La reconstrucción cíclica obtuvo los valores más altos, de acuerdo con el objetivo de consistencia cíclica de la arquitectura.

Estos resultados corresponden a una reproducción funcional en el dominio original de COMI y **no demuestran una generalización favorable** a las imágenes de laboratorio del TFM.

## Autoría de la adaptación

**Mónica Fernández Navarro**

- configuración del entorno local bajo WSL;
- adaptación de `test.py` para CPU;
- actualización de las métricas PSNR y SSIM;
- ejecución de las pruebas;
- registro, análisis e interpretación de los resultados del módulo COMI.

La definición general del estudio, la selección de métricas y la interpretación global del TFM se realizaron conjuntamente con **María Rodríguez Buitrago**.

## Confidencialidad de los datos

Las imágenes privadas del laboratorio no se publican por motivos de confidencialidad. El repositorio contiene únicamente código, documentación y resultados que pueden compartirse públicamente.
