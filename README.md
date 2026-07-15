# Adaptación experimental de COMI para restauración de imágenes microscópicas

Este repositorio contiene la reproducción y adaptación experimental de **COMI**
(*Correction of Out-of-focus Microscopic Images by Deep Learning*), realizada en
el marco del Trabajo Fin de Máster:

> **Desarrollo de métodos de corrección de artefactos en microscopía de
> fluorescencia de células uterinas mediante técnicas de aprendizaje automático**

## Alcance del repositorio

El TFM fue desarrollado conjuntamente por:

- **Mónica Fernández Navarro**
- **María Rodríguez Buitrago**

El estudio completo evaluó cuatro enfoques de restauración:

- Noise2Void (N2V)
- CARE
- COMI
- Restormer

Este repositorio contiene **únicamente el módulo experimental de COMI** y no
representa el código completo del TFM.

La definición general del protocolo experimental, la selección de métricas y la
interpretación global del estudio se realizaron de forma conjunta por ambas
autoras.

## Código original y licencia

Esta adaptación parte del repositorio original de COMI:

- [Repositorio original de COMI](https://github.com/jiangdat/COMI)
- Zhang, C. et al. (2022). *Correction of out-of-focus microscopic images by
  deep learning*. Computational and Structural Biotechnology Journal, 20,
  1957–1966.
- [Artículo original](https://doi.org/10.1016/j.csbj.2022.04.003)

El código original se distribuye bajo licencia MIT. Este repositorio conserva la
licencia y la atribución correspondientes.

## Cambios realizados para el TFM

El archivo [`test.py`](./test.py) fue adaptado para ejecutar la evaluación en un
entorno local con recursos limitados.

Las modificaciones principales fueron:

- sustitución de las funciones obsoletas
  `skimage.measure.compare_psnr` y `compare_ssim` por
  `skimage.metrics.peak_signal_noise_ratio` y
  `skimage.metrics.structural_similarity`;
- uso explícito de `data_range=1` en PSNR y SSIM;
- uso de `channel_axis=-1` para imágenes multicanal;
- compatibilidad adicional con versiones anteriores mediante
  `multichannel=True`;
- ejecución forzada en CPU mediante:

  ```python
  os.environ["CUDA_VISIBLE_DEVICES"] = "-1"
  ```

- desactivación del envoltorio `multi_gpu_model` en los modelos combinados;
- reducción del tamaño de lote de 4 a 1;
- carga de modelos `.h5` mediante el objeto personalizado
  `InstanceNormalization`;
- cálculo de PSNR, SSIM y correlación de Pearson;
- registro de los resultados de la reproducción experimental.

## Archivos relevantes

- [`test.py`](./test.py): script modificado y utilizado para la reproducción
  funcional.
- [`test_backup.py`](./test_backup.py): copia de respaldo del script anterior.
- [`deblur.py`](./deblur.py): script de entrenamiento procedente de la
  implementación original. Requiere adaptar rutas y configuración de GPU antes
  de ejecutarlo.
- [`requirements.txt`](./requirements.txt): dependencias necesarias para el
  entorno utilizado por `test.py`.
- [`resultados_COMI_dataset_original.txt`](./resultados_COMI_dataset_original.txt):
  registro de las métricas obtenidas durante la reproducción. Este archivo no
  formaba parte del repositorio original.

## Entorno de ejecución

La prueba se realizó en un entorno local bajo WSL, con ejecución en CPU.

- Python 3.7
- TensorFlow 1.14.0
- Keras 2.3.1
- keras-contrib
- NumPy
- SciPy
- scikit-image
- scikit-learn
- OpenCV
- Pillow

Las versiones utilizadas se encuentran especificadas en
[`requirements.txt`](./requirements.txt).

Se trata de un entorno heredado debido a las dependencias del código original
de COMI.

## Instalación

Clonar el repositorio:

```bash
git clone https://github.com/Monicafn/COMI.git
cd COMI
```

Crear y activar un entorno virtual con Python 3.7:

```bash
python3.7 -m venv .venv
source .venv/bin/activate
```

Actualizar las herramientas de instalación con versiones compatibles con
Python 3.7:

```bash
python -m pip install --upgrade "pip<24.1" "setuptools<68" wheel
```

Instalar las dependencias:

```bash
python -m pip install -r requirements.txt
```

> La instalación de `keras-contrib` requiere que Git esté disponible en el
> sistema.

## Datos y pesos necesarios

Los datasets, las imágenes privadas del laboratorio y los pesos preentrenados
no se distribuyen en este repositorio.

Para reproducir la prueba, `test.py` espera una estructura equivalente a:

```text
COMI/
├── confocal/
│   └── C0depth/
│       ├── Z005/
│       └── Z007/
│
├── deblursrgan4_C0depth_Z005_weights/
│   ├── generator_l2h.h5
│   └── generator_h2l.h5
│
├── model/
│   └── vgg19_weights_tf_dim_ordering_tf_kernels_notop.h5
│
├── test.py
└── requirements.txt
```

La configuración utilizada en `test.py` fue:

- conjunto: `C0depth`;
- entrada desenfocada: `Z005`;
- referencia en foco: `Z007`;
- tamaño de entrada: `128 × 128 × 3`;
- tamaño procesado por los generadores: `512 × 512 × 3`;
- tamaño de lote: 1;
- dispositivo: CPU.

## Ejecución

Para ejecutar la evaluación:

```bash
python test.py
```

Para ejecutar la evaluación y guardar simultáneamente la salida de la terminal:

```bash
python test.py 2>&1 | tee resultados_COMI_dataset_original.txt
```

El script genera las predicciones en la carpeta:

```text
deblursrgan4_C0depth_Z005_predict_img/
```

## Resultados de la reproducción funcional

| Comparación | PSNR (dB) | SSIM | Pearson |
|---|---:|---:|---:|
| Referencia nítida vs. entrada desenfocada | 19.95 | 0.325 | 0.920 |
| Referencia nítida vs. salida restaurada (`lq2hq`) | 23.82 | 0.125 | 0.916 |
| Referencia desenfocada vs. dirección inversa (`hq2lq`) | 18.40 | 0.621 | 0.863 |
| Referencia nítida vs. reconstrucción cíclica | 24.82 | 0.487 | 0.973 |

El generador principal aumentó el PSNR en aproximadamente **3.87 dB** respecto
a la entrada desenfocada.

Sin embargo, la reducción de SSIM indica que la salida restaurada modifica la
estructura local, aunque mejore la fidelidad media de intensidad.

La reconstrucción cíclica obtuvo los valores más altos de PSNR y correlación de
Pearson, comportamiento coherente con el objetivo de consistencia cíclica de la
arquitectura.

Estos resultados corresponden a una reproducción funcional dentro del dominio
original de COMI. No demuestran una generalización favorable a las imágenes de
laboratorio del TFM.

## Limitaciones

- Los pesos preentrenados y el dataset original no se incluyen.
- La ejecución se realizó en CPU debido a la incompatibilidad entre TensorFlow
  1.14 y la configuración CUDA disponible.
- El código depende de versiones heredadas de TensorFlow, Keras y
  `keras-contrib`.
- El script [`deblur.py`](./deblur.py) conserva elementos de la implementación
  original, como rutas y configuración multi-GPU, que deben adaptarse antes de
  utilizarlo.
- Los resultados corresponden a una reproducción funcional y no a un nuevo
  entrenamiento completo de COMI.

## Autoría de la adaptación

### Mónica Fernández Navarro

Responsable de:

- configuración del entorno local bajo WSL;
- adaptación de `test.py` para ejecución en CPU;
- actualización de las funciones PSNR y SSIM;
- compatibilidad con versiones recientes de scikit-image;
- reducción del tamaño de lote;
- ejecución de las pruebas;
- registro, análisis e interpretación de los resultados del módulo COMI.

La definición general del estudio, la selección de métricas y la interpretación
global del TFM se realizaron conjuntamente con **María Rodríguez Buitrago**.

## Confidencialidad de los datos

Las imágenes privadas del laboratorio no se publican por motivos de
confidencialidad.

Este repositorio contiene únicamente código, documentación y resultados que
pueden compartirse públicamente.

## Licencia

Este repositorio conserva la licencia MIT del proyecto original. Consulte el
archivo [`LICENSE`](./LICENSE) para obtener más información.
