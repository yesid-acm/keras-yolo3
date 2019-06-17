# keras-yolo3 Personalsoft - Recorte de Imágenes- Votre

[![license](https://img.shields.io/github/license/mashape/apistatus.svg)](LICENSE)

## Introduction

Una implementacion desde Keras de Yolo v3 (Tensorflow backend) inspired by [allanzelener/YAD2K](https://github.com/allanzelener/YAD2K).

---

#### Paquetes necesarios
- Tensorflow
- keras 
- Numpy
- PIL
- os
- Augmentor

## Inicio rápido

1. Descargar YOLOv3 weights (pesos) de  la  página [YOLO website](http://pjreddie.com/darknet/yolo/).
2. Convertir el modelo YOLO Darknet a un modelo de Keras.
3. Ejecutar YOLO detection.

```
* Use los comandos de consola para ejecutar Yolo V3; el script de yolo_video.py
  contiene el código para ejecutar Yolo v3 para imágenes.
wget https://pjreddie.com/media/files/yolov3.weights
python convert.py yolov3.cfg yolov3.weights model_data/yolo.h5
python yolo_video.py [OPTIONS...] --image, for image detection mode, OR
python yolo_video.py [video_path] [output_path (optional)]
```

Para Tiny YOLOv3, se hace de manera similar, sólo es específicar la ruta del modelo y el anchor 
con `--model model_file` and `--anchors anchor_file`, respectivamente.

### Uso
Use --help para ver el uso de yolo_video.py:
```
usage: yolo_video.py [-h] [--model MODEL] [--anchors ANCHORS]
                     [--classes CLASSES] [--gpu_num GPU_NUM] [--image]
                     [--input] [--output]

Argumentos posicionales:
  --input        Video input path
  --output       Video output path

Argumentos opcionales:
  -h, --help         muestra esto mensaje de ayuda y salir
  --model MODEL      Ruta de archivo con los pesos (weight) del modelo, default model_data/yolo_weights.h5
  --anchors ANCHORS  Ruta de los anchor definidos, default
                     model_data/yolo_anchors.txt
  --classes CLASSES  ruta de las class definitions, default
                     model_data/coco_classes.txt
  --gpu_num GPU_NUM  Número de GPU a usar, default 1
  --image            Modo Image detection, ignorará todos los argumentos poscisioneles.
```
---

4. MultiGPU uso: use `--gpu_num N` para usar N GPUs. Esto es pasado a [Keras multi_gpu_model()](https://keras.io/utils/#multi_gpu_model).

## Training

1.  Genere su propio archivo de clases como sigue:
    Una fila para una imagen;  
    Formato de la fila: `image_file_path box1 box2 ... boxN`;  
    Formato de la caja: `x_min,y_min,x_max,y_max,class_id` (sin espacios).  
    ##For VOC dataset, try `python voc_annotation.py`  
    Aquí está un ejemplo:
    ```
    path/to/img1.jpg 50,100,150,200,0 30,50,200,120,3
    path/to/img2.jpg 120,300,250,600,2
    ...
    ```

2.  Asegurece que usted ejecutó `python convert.py -w yolov3.cfg yolov3.weights model_data/yolo_weights.h5`  
    En la carpeta model_data se genera el archivo de pesos convertido con nombre "yolo", por lo que se debe 
    cambiar el nombre a "yolo_weights" con formato .h5, quedando en la ruta model_data/yolo_weights.h5. 
    ésto es para cargar los pesos preentrenados.
    
 ## Algunas modificaciones realizadas para adaptar el modelo a nuestro objetivo de recorte de imágenes:
   1. Se modifica los anchors con 9 opciones.
   2. Se modefica las clases de la carpeta model_data.
   

3.  Modifique el script train.py e inicie el entreno
    `python train.py`  
    Use sus pesos entrenados con la opción de comando `--model model_file` cuando use yolo_video.py
     Recuerde modificar la ruta de la clase y el anchor con `--classes class_file` and `--anchors anchor_file`, respectivamente.
  
 #### Modificaciones del script train.py
    * La función model.fit_generator() tiene capas congeladas y se modifica el epoch=1 y el initiial_epoch=0 (lineas 63 y 64).
      Lo demás por defecto.
    * La función model.fit_generator() tiene capas congeladas y se modifica el epoch=2 y el initiial_epoch=1 (lineas 82 y 83), 
      Esta función es para cuando no se tiene buenos resultados, un entrenamiento más largo; lo demás 
      del script queda igual, Default.

Si usted quiere usar los pesos (weights) preentrenados originales de YOLOv3, haga lo siguiente:
    1. `wget https://pjreddie.com/media/files/darknet53.conv.74`  
    2. Renombre el archivo como darknet53.weights  
    3. `python convert.py -w darknet53.cfg darknet53.weights model_data/darknet53_weights.h5`  
    4. use model_data/darknet53_weights.h5 en train.py

---

## A tener en cuenta.

1. El ambiente de prueba (test) es:
    - Python 3.5.2
    - Keras 2.1.5
    - tensorflow 1.6.0

2. Se usan los anchors por defecto. si se cambian los anchors, probablemente se necesiten algunos cambios.

5. Siempre cargue los pesos preentrenados y congele las capas en el primer stage del entrenamiento ó trate  de entrenar Darknet.
 
   Always load pretrained weights and freeze layers in the first stage of training. Or try Darknet training. Esto es por si hay algún      warning.

7. Para acelerar el proceso de entrenamiento con las capas congeladas se puede usar train_bottleneck.py, esto 
calculará las caracteristicas del bottleneck  del primer modelo congelado y luego entrenará sólo las últimas capas.
Esto hace que el entrenamiento en CPU tenga posiblemente un timepo razonable. See [this](https://blog.keras.io/building-powerful-image-classification-models-using-very-little-data.html) for more information on bottleneck features.
4. MultiGPU uso: `--gpu_num N` para usar N GPUs. Esto es pasado a [Keras multi_gpu_model()](https://keras.io/utils/#multi_gpu_model).
